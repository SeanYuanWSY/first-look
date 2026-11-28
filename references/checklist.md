# First Look 只读检查清单（执行细则）

> **平台说明**：下方示例命令面向 **POSIX Shell（macOS / Linux）**。在 Windows（PowerShell）下使用对应等效命令（如 `Select-String`、`Get-FileHash`、`tar -tf`）；若某项工具缺失无法执行，如实记为 `未核实`，切勿静默跳过。
> **安全底线**：所有 Git 命令必须禁用可选锁、fsmonitor、仓库 hooks 与凭据弹窗；扫描输出**必须在离开终端前脱敏**，且**禁止用 `|| true` 掩盖工具报错**。

---

## 0. 首屏与外部第一印象

检查外人打开主页 10 秒内看到的内容：
- 根目录是否存在 `README*`、`LICENSE*` / `COPYING*` / `NOTICE*`；若无安装说明，深档直接按“零文档硬停”处理。
- README 首屏是否说清：项目定位、支持平台/环境、安装前置条件、快速开始命令与结果形态。
- 如面向托管平台（GitHub 等）且可只读查看，核对仓库简介（About）、Topics 与默认分支可见性；无权查看标 `未核实`。

## 1. 锁定交付对象与三层可见面

### A. Git 仓库（安全只读封装）

```sh
# 统一只读安全封装：不写索引、不触发仓库 fsmonitor/hooks、不弹凭据框。
# 用函数而非字符串前缀：确保 sh/bash/zsh 行为一致（zsh 默认不对 $VAR 做分词）。
git_safe() {
  env GIT_OPTIONAL_LOCKS=0 GIT_TERMINAL_PROMPT=0 \
    git -c core.fsmonitor=false -c core.hooksPath=/dev/null "$@"
}

git_safe rev-parse --show-toplevel --short HEAD
git_safe status --short
git_safe branch -a
git_safe tag -l
```

- **限定实际发布范围**：确定将推送的 ref（如 `main`、`v1.0.0`），后续历史扫描只扫这些**可达 refs**，不要拿 `--all` 混入不会推送的本地实验分支。
- **匿名远端探测（含超时，防止私仓凭据误判为公开）**：仅对已确认的公开 HTTPS URL 做匿名探测；`timeout` 并非 macOS 自带命令，用 `command -v` 探测后以 `perl` 兜底，两者均保持非交互（`GIT_TERMINAL_PROMPT=0` + 空凭据助手）。若离线、无远端或探测失败，继续本地体检并将“线上已可见”标 `未核实`。

```sh
# 匿名探测远端 URL（优先 timeout；无 timeout 时用 perl alarm 兜底）
probe_remote() {
  url="$1"
  if command -v timeout >/dev/null 2>&1; then
    env GIT_TERMINAL_PROMPT=0 timeout 10 git -c credential.helper= ls-remote "$url"
  else
    perl -e 'alarm shift; exec @ARGV' 10 \
      env GIT_TERMINAL_PROMPT=0 git -c credential.helper= ls-remote "$url"
  fi
}
probe_remote https://...
```
- **平台已公开残留（如适用）**：远端 `ls-remote` 为空不代表平台零痕迹；若仓库曾公开或已有 PR/CI，需提示核查旧 Release 附件、Actions 日志与产物、Pages 与公开 fork（未查标 `未核实`）。

### B. 非 Git 目录 / 源码包 / 发布归档

若受审对象不是 Git 仓库，或额外交付 `.tar.gz` / `.zip` / `.whl` / 容器镜像：
- 记录工件文件名与 SHA-256（`shasum -a 256` / `sha256sum`），版本号仅从打包元数据读取；Git 历史标 `未核实`。
- **逐个遍历归档，严禁多文件通配符直传 `tar` 或用 `head` 静默截断**：

```sh
# 逐个归档检查完整清单并保留退出码；同时检查是否误夹带 .git/ 或越界路径。
# 注意：用 mktemp 而非固定 /tmp 路径（并发安全、尊重 TMPDIR），且用 grep 退出码区分 干净/命中/报错。
(
  manifest="$(mktemp "${TMPDIR:-/tmp}/fl_manifest.XXXXXX")" || exit 2
  trap 'rm -f "$manifest"' EXIT
  # zsh 对未命中的 glob 会报 "no matches found" 并中止循环；subshell 内开启 null_glob 使其按空列表跳过（sh/bash 默认即跳过，无需设置）
  [ -n "${ZSH_VERSION-}" ] && setopt null_glob
  for f in dist/*.tar.gz dist/*.tgz; do
    [ -e "$f" ] || continue
    tar -tzf "$f" > "$manifest" || { echo "ARCHIVE_READ_ERROR:$f" >&2; exit 2; }
    wc -l < "$manifest"
    grep -E '(^|/)\.git(/|$)|(^|/)\.\.(/|$)|^/' "$manifest"; rc=$?
    if [ "$rc" -eq 0 ]; then echo "ARCHIVE_RISK:$f";
    elif [ "$rc" -gt 1 ]; then echo "MANIFEST_SCAN_ERROR:$f:rc=$rc" >&2; fi
  done
)
```

- **Wheel / sdist / OCI 容器镜像**：除源码树外，还须核对实际打包产物清单；OCI 镜像需检查历史层与配置（避免上层删除的密钥残留在下层），无法检查时标 `未核实`。

## 2. 源头脱敏的暴露面扫描

### A. 完整可达提交历史（Author + Committer 邮箱，源头脱敏）

不限 `-n 20`；同时检查 Author（`%an`/`%ae`）与 Committer（`%cn`/`%ce`），并**在终端输出前彻底脱敏（不输出用户名与域名）**，只输出类别与提交计数：

```sh
# 替换 <push-ref> 为实际拟发布分支或标签（如 HEAD 或 main）
git_safe log "<push-ref>" --format='%h%x09%ae%x09%ce' | awk -F'\t' '
function kind(e) {
  if (e ~ /users\.noreply\.github\.com$/) return "noreply"
  return (index(e, "@") > 1 ? "plaintext_email" : "non_email")
}
{ a=kind($2); c=kind($3); if (a!="noreply" || c!="noreply") seen["author=" a " committer=" c]++ }
END { for (k in seen) print k " (commits=" seen[k] ")" }'
```

### B. 凭据、AI/Agent 痕迹、内部计划与本机路径（不打印原始敏感行）

- **凭据扫描**：若本地已装 `gitleaks` 或 `trufflehog`，优先以只读+脱敏模式扫描拟发布范围与完整可达历史（例如 `gitleaks git --redact --no-banner --log-opts="<push-ref>"`）。
- **轻量内置扫描（只输出文件与计数，绝不回显原行，保留退出码）**：
  - 区分退出码：`0` = 有命中、`1` = 干净无命中、`>=2` = 命令报错（不得用 `|| true` 吞掉 `>=2`）。
  - 覆盖实际交付文件（Git 追踪文件、暂存区、以及实际会被打进发布包的未追踪文件；非 Git 目录用 `rg -l` / `grep -rl`）。
  - 注意单引号转义，确保正确命中单反斜杠 Windows 路径（如 `C:\Users\...`）：

```sh
# 1) 疑似凭据/私钥/带认证 URL（只列文件名，绝不打印原行）
PATTERN_SECRET='BEGIN [A-Z ]*PRIVATE KEY|sk-[A-Za-z0-9]{20,}|ghp_[A-Za-z0-9]{20,}|github_pat_|AKIA[0-9A-Z]{16}|://[^/@[:space:]]+:[^/@[:space:]]+@'
git_safe grep -l -E "$PATTERN_SECRET" -- . ; rc=$?
[ "$rc" -le 1 ] || echo "SCAN_ERROR:secret_grep_rc=$rc" >&2

# 2) AI/Agent 协作痕迹、本机绝对路径与明确内部计划线索（只输出 文件:命中数，绝不回显含用户名/路径的原行）
PATTERN_TRACE='\.claude/|CLAUDE\.md|AGENTS\.md|GEMINI\.md|\.cursor/|\.cursorrules|\.windsurfrules|copilot-instructions\.md|\.aider|\.specstory/|\.codex/|\.kiro/|\.ai/|/Users/|/home/|C:\\Users\\|next phase|roadmap|user rule'
git_safe grep -c -i -E "$PATTERN_TRACE" -- . ; rc=$?
[ "$rc" -le 1 ] || echo "SCAN_ERROR:trace_grep_rc=$rc" >&2
```

- **历史已删文件与附注标签（同样源头脱敏）**：私仓转公仓前，须扫完整可达历史中曾提交后删除的文件名（`git_safe log "<push-ref>" --diff-filter=D --summary`），并对附注标签正文做脱敏计数（不直接打印原始标签正文）：

```sh
# 仅输出 标签名 + 命中类别计数，绝不回显原始标签正文或邮箱
git_safe for-each-ref refs/tags --format='%(refname:short)%09%(objecttype)' | while IFS="$(printf '\t')" read -r tag otype; do
  [ "$otype" = "tag" ] || continue
  body="$(git_safe for-each-ref "refs/tags/$tag" --format='%(contents)')"
  # grep -c 退出码：0=有命中、1=干净、>=2=报错（不得用 || true 掩盖）
  sec_hits="$(printf '%s\n' "$body" | grep -c -E "$PATTERN_SECRET")"; sec_rc=$?
  if [ "$sec_rc" -ge 2 ]; then echo "SCAN_ERROR:tag=$tag:rc=$sec_rc" >&2; break; fi
  tr_hits="$(printf '%s\n' "$body" | grep -c -i -E "$PATTERN_TRACE")"; tr_rc=$?
  if [ "$tr_rc" -ge 2 ]; then echo "SCAN_ERROR:tag=$tag:rc=$tr_rc" >&2; break; fi
  [ "${sec_hits:-0}" -gt 0 ] || [ "${tr_hits:-0}" -gt 0 ] && printf 'tag=%s secret_hits=%s trace_hits=%s\n' "$tag" "$sec_hits" "$tr_hits"
done
```

  若因体量未扫完整历史或标签，表后一行必须写明 `完整历史/标签内容：未核实`。
- **降噪与中性原则**：
  - 单独的 `TODO` / `FIXME` / `WIP` 不等同于内部路线图泄露，仅当内容暴露未公开计划/内部代号时列入发现。
  - AI 痕迹**只报可定位文件与行号**，绝不靠“文风像 AI”列发现；主动公开的 `AGENTS.md`/`CLAUDE.md` 如实报为可见事实，不默认定性为错误。
  - 若建议从公开 `.gitignore` 删去私人目录规则，须提醒改写入仅本机生效的 `.git/info/exclude` 或全局 `core.excludesFile`，保留防误提交保护。

## 3. 隐藏元数据、子模块、LFS 与二进制资产

- **锁文件源**：查 `uv.lock`、`poetry.lock`、`package-lock.json`、`pnpm-lock.yaml` 等是否含私服/内网 registry 或带凭据 URL。
- **子模块与 LFS**：不仅查 `.gitmodules` 地址，还核对固定提交与 `.gitattributes` / LFS 指针对应的对象能否匿名拉取；若无法匿名验证标 `未核实`；检查归档打包器是否会解引用越根符号链接。
- **Notebook 与媒体元数据**：查 `.ipynb` 是否残留输出单元格、堆栈路径或样例数据；查 `.svg` 的 `<metadata>` / `inkscape:` / 编辑器作者；大体积二进制/媒体列清单，无法解析元数据时标 `未核实`。

## 4A. 科研 / 模型视角：真实交付边界（5 层核对）

对主流程关键阶段（monorepo 先限定受审入口/包范围，其余标 `未核实`），分开核对 5 层并在表格中写明缺失属于哪一层：

| 层级 | 核对要点 |
| --- | --- |
| **代码** | 源码、CLI 入口与脚本是否随件完整，有无硬编码仓外路径 |
| **解释器与环境** | 默认安装命令建出的环境能否跑通所有子入口（警惕轻量 venv 调重型子进程缺包） |
| **权重与数据** | 权重、数据库、索引、模板是随件自带还是另需下载；下载包内是否真包含所需文件 |
| **许可与平台权限** | 外部工具是否需个人学术/商业申请、账号登录、特定 OS 或 GPU |
| **网络与凭据** | 是否依赖在线 API、外网镜像、环境变量密钥或外部服务 |

## 4B. 公开工具 / Skill 视角：首次上手全流程核对（3 块简洁输出）

站在第一次打开仓库并照着 README / `SKILL.md` 一步步用到产出的外部用户视角，核对 4 步真实旅程：

| 流程步骤 | 核对要点 |
| --- | --- |
| **1. 安装入口** | 有没有一条命令装好（如 `npx skills add`、`pip/npm`）还是得手动克隆软链；克隆地址建仓时需匹配（未建仓前仅作提醒，不标硬停） |
| **2. 前 1 分钟最小体验** | 装好运行环境（Node/Python 等）后，不配账号/密钥能否跑通离线示例或预览图；双语 README 首屏配图与语言是否一致 |
| **3. 接入外部工具与启动口令** | 对外链依赖（CLI / MCP / 浏览器 / API）是否给出复制即跑的接入指令，还是只甩链接；是否提供 1–3 句现成的 Agent 启动示范口令 |
| **4. 跑出一轮真实产出** | 从输入到最终产出的闭环是否完整，遇到页面改版/限流/缺字段时有无平滑降级方案 |

- **汇报去噪铁律**：后台只读核验、脱敏扫描与文档实装对齐照常执行，但**工具视角用户报告中绝不输出“依据”列、内部文件/单测清单或“低风险一切正常”的凑数行**；隐私与泄露干净时一句话带过即可，仅保留用户真需要拍板或补齐的内容。

## 5. 多许可证与随包第三方资产

- 盘点根目录及所有嵌套目录的 `LICENSE*`、`COPYING*`、`NOTICE*` 与包元数据中的 `SPDX` 声明；遇到双许可、嵌套子包不同许可或声明冲突，如实标出，不假设根 `LICENSE` 自动覆盖全部文件。
- 对随件分发的第三方代码（`third_party/`、`vendor/`）、权重、数据与图件，逐项分开判断三件事：**可下载 / 可使用 / 可随包再分发**；缺少上游声明或条款不明时标 `未知`。

## 6. 深档隔离与实跑执行规范（仅深档）

1. **获取实际交付件**：按目标渠道获取（如 `git archive <ref>`、实际发布的 tarball/wheel 或 `git clone --no-local` 到临时目录），保留会随件发出的文件。
2. **一次性沙箱隔离**：在一次性容器/VM 内以空 `HOME`、剥离宿主环境变量/凭据/SSH/容器套接字、受限出站与资源运行；解包归档前先验路径安全。**无法满足隔离条件时直接返回 `证据不足`，不在宿主直接执行安装脚本。**
3. **证据绑定**：每条实跑记录必须绑定「渠道 + 摘要/提交 + 具体任务 + 公开前置条件」；截图与终端日志附证前完成脱敏；结束后清理临时环境并核验无残留。
