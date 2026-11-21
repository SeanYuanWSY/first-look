---
name: first-look
description: >-
  Pre-release repository audit and cold-start verification from a stranger's first-visit
  perspective (仓库发布前外部视角体检与冷启动验收). Default Light mode (read-only, no installs
  or large downloads): one-sentence verdict, staged delivery boundaries (bundled vs
  user-supplied vs failure mode), external exposure audit across working tree, publishable
  Git history, archives, metadata, and third-party licenses, plus safe public-claims
  guardrails. Explicit Deep mode (only when user asks to run/install): isolated cold-start
  execution on the actual release artifact. Not for unit tests, code review, pentesting, or
  legal sign-off.
license: MIT
metadata:
  version: "1.0.0"
---

# First Look：外部视角发布前体检与冷启动验收

以**第一次点进仓库的外部用户**视角回答三件事：**外人看它是什么、公开交付带了什么与缺了什么、哪些未计划公开的内容已经暴露**。
默认**只查不改、不自动清理、不自动发布**；不替代单元测试、代码审查、安全渗透或法律合规评审。
**语言规则**：使用用户的提问语言回复；英文环境下使用文末中英对照标签，不混用语言。

## 1. 两档模式（默认轻档）

- **轻档（默认：交付边界 + 暴露面体检）**：只读检查拟发布工件、首屏文档、关键入口与将公开的 Git 历史/元数据；**不安装依赖、不下载大包、不展开内部函数实现或运行日志**。开工前**必须读取** [references/checklist.md](references/checklist.md) 执行只读排查；首次出报告前按需对照 [references/examples.md](references/examples.md)。
- **深档（冷启动实跑验收）**：**仅当用户明确要求实跑/安装验证时启用**。必须在一次性隔离环境中对**实际拟交付渠道工件**（保留会随件发出的文件）执行安装与 1–2 个文档宣称的核心任务；若无法隔离或仓库缺少安装文档，直接按第 5 节停步，绝不在宿主环境硬跑或凭记忆补救。

## 2. 观察者纪律、范围锁与证据等级

1. **一行锁定对象**：首句用固定枚举标注 `对象@版本/提交〔渠道｜候选或线上｜静态或实跑〕`（渠道如 `Git 树` / `源码包` / `Wheel` / `Release 附件` / `容器镜像`）；大型 monorepo 须写明受审子路径，未查路径与未生成产物标 `未核实`。
2. **证据隔离**：作者本人也能跑轻档。先只凭拟发布工件描述陌生人看到的事实，再按用户明示的发布意图判偏离。**将公开的 Git 历史可作曝光证据**；旧会话、私人交接、兄弟仓库不得用来补全外部用户认知或运行条件，也不写删掉零损失的旧上下文否定句（“它不是 X”）。
3. **证据等级严格分开**：只用 `文档声明` / `静态核实` / `实跑核实` / `未核实`。轻档绝不宣称“实际跑通”，只写“公开交付提供到哪一步、首个前置条件是什么”。
4. **源头脱敏**：扫描命令在进入终端或工具日志前就必须脱敏，**严禁把疑似密钥、令牌或明文私人邮箱原值打进日志或报告**，只报位置/提交与信息类型。

## 3. 轻档检查要点（执行细则见 `references/checklist.md`）

- **首屏与定位**：核对 README 首屏、快速开始可见性、许可证文件与仓库元信息。
- **真实交付边界（逐阶段拆 5 层）**：分开核对 `代码` / `解释器与环境` / `权重与数据` / `许可与平台权限` / `网络与凭据`；缺失后果标 `硬停 / 降级 / 替代 / 未知`，标清证据等级与出处。
- **外部暴露面（三层可见性分开标）**：严格区分 `仅本地` / `拟随发布可见` / `线上已可见`。覆盖：
  - **可定位的 AI/Agent 协作痕迹**（如 `.claude/`、`CLAUDE.md`、`AGENTS.md`、`GEMINI.md`、`.cursor/`、`.cursorrules`、`.github/copilot-instructions.md`、`.aider*`、`.codex/`、`.kiro/`、`.ai/` 或会话注释）：**只报可定位文件/行号，绝不凭“猜文风”列发现**；如实报为可见事实，**不把主动公开的 Agent 配置默认当缺陷**，仅对照作者发布意图判偏离。
  - **内部计划、私有与隐藏信息**：未发布路线图/内部编号、拟推送历史中的 Author/Committer 邮箱与已删文件、本机路径、疑似密钥、锁文件/子模块/LFS/归档/容器层/notebook/SVG 元数据、私人 fork 链接。
  - **多许可证与随包第三方资产**：排查根及嵌套 `LICENSE`/`COPYING`/`NOTICE`、SPDX 字段及随包第三方代码/权重/数据/图件，分开判断 `可下载 / 可使用 / 可随包再分发`；冲突或不清标 `未知`。

## 4. 轻档固定 4 块输出（短句直给，高危前置）

1. **一句判断**：`对象@版本〔渠道｜候选/线上｜静态〕` 外人看它是 ___，公开交付提供到 ___，首个前置条件是 ___（如有发布阻断级硬停或高危泄露，此句末尾直接点明）。
2. **实际交付表**：`阶段 | 随件已带 | 另需自备（标层级） | 缺失后果 | 依据`（只列主路径与影响结论的关键分支）。
3. **外部暴露面表（按风险从高到低排）**：`风险 | 发现 | 外人会知道什么（直接事实；推断另标） | 位置/渠道 | 可见状态 | 处置建议`。表后一行写明已查范围、扫描限制与未覆盖项。
4. **对外口径红线**：各列 1–3 条极短句的 `对外可写` 与 `证据不足，暂勿写`。

> 除上述 4 块外不另开长篇章节；允许在表 3 的 `处置建议` 列给出一句话建议，但不自动修改文件。

## 5. 深档补充规则（仅用户明确要求实跑时执行）

- **强制安全隔离**：在一次性容器/VM（空 `HOME`、无宿主凭据/SSH/容器套接字、受限出站与资源）中执行；解包归档前先校验路径穿越与越界符号链接。**宿主无法提供有效隔离时，直接停止并返回 `证据不足`，严禁在宿主硬跑不受信任安装钩子。**
- **零文档停步**：仓库缺少可读 README 或安装说明时，将“缺少安装文档”记为首个硬停并跳过实跑，不凭经验猜测补齐。
- **证据绑定与脱敏**：每条 `实跑核实` 必须绑定到实际安装渠道、不可变摘要/提交、执行任务与公开前置条件，不得把源码检出实跑泛化到 Wheel/容器等其他渠道；截图与日志附证前须脱敏；结束后清理临时环境并报告清理结果。
- **深档输出**：在轻档 4 块后仅追加 `实跑结果与真实阻塞` 一节，给出冷启动结论：`通过 / 部分通过 / 不通过 / 证据不足`。

## 6. 中英标签对照（按用户语言择一）

`候选 / 线上` ↔ `Candidate / Live` · `静态 / 实跑` ↔ `Static / Executed` · `硬停 / 降级 / 替代 / 未知` ↔ `Hard stop / Fallback / Alternative / Unknown` · `文档声明 / 静态核实 / 实跑核实 / 未核实` ↔ `Doc claim / Static verified / Executed / Unverified` · `仅本地 / 拟随发布可见 / 线上已可见` ↔ `Local only / Planned public / Already public` · `高 / 中 / 低` ↔ `High / Medium / Low` · `通过 / 部分通过 / 不通过 / 证据不足` ↔ `Pass / Partial / Fail / Insufficient evidence`
