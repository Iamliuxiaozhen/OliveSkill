<p align="center">
  <h1 align="center">personal-engineering-workflow</h1>
</p>

<p align="center">
  <em>Evidence &gt; Assumption. Small patches, explicit permissions, safer engineering.</em>
</p>

<p align="center">
  <a href="https://agentskills.io"><img src="https://img.shields.io/badge/AgentSkills-Standard-green" alt="AgentSkills"></a>
  <a href="https://claude.ai/code"><img src="https://img.shields.io/badge/Claude%20Code-Skill-blueviolet" alt="Claude Code"></a>
  <a href="https://cursor.com"><img src="https://img.shields.io/badge/Cursor-Skill-blue" alt="Cursor"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License: MIT"></a>
</p>

<br/>

<p align="center">
普通 Agent 会在看到猜测后立刻修改代码？<br/>
会为了通过检查顺手重构无关文件？<br/>
会直接安装 dependency、执行 commit，甚至修改远程状态？<br/>
</p>

<p align="center">
<strong>personal-engineering-workflow 将可验证的工程原则、权限边界和 Open Source workflow<br/>注入 Agent，让它先读规则、收集证据，再做最小且可验证的修改。</strong>
</p>

<br/>

<p align="center">
  <a href="#效果演示">效果演示</a> ·
  <a href="#核心原则">核心原则</a> ·
  <a href="#安装">安装</a> ·
  <a href="#skill-结构">Skill 结构</a> ·
  <a href="#蒸馏过程">蒸馏过程</a>
</p>

<br/>

---

## 项目介绍

`personal-engineering-workflow` 是一个遵循 AgentSkills 结构的个人工程协作 Skill。

它不是新的 coding Agent，也不是一组特定项目的 prompts。它为现有 Agent 提供稳定的工程行为约束：

- 把用户提出的原因视为待验证假设，而不是事实
- 先阅读项目规则和相关代码，再修改
- 优先 `Correctness`、`Security` 和小而聚焦的 patch
- 明确区分可自主操作和必须先确认的高风险操作
- 尊重上游项目 conventions，不为个人偏好扩大改动范围
- 默认采用 Linux First、Terminal First 和 SSH Git Workflow

适用于 Coding、Debugging、Refactoring、Code Review、Architecture Discussion、Technical Writing、Git Operations、CI/CD Analysis、Security Analysis 和 Open Source Contribution。

---

## 它解决什么问题？

普通 Agent 通常具备很强的代码生成能力，但不一定知道你的工程边界：

- 用户说“可能是 cache”，它可能直接按 cache 问题修改
- 为修复一个 bug，顺手重构多个模块
- 自动安装 dependency 或修改 lockfile
- 用 formatter 制造大量无关 diff
- 把 style 建议当成 Code Review 的主要发现
- 未经许可执行 `commit`、`push` 或创建 PR

这个 Skill 不负责让 Agent “写更多代码”，而是让它以更可靠、可审查、可控的方式完成工程任务。

## 与普通 AI 有什么区别？

| 场景 | 普通 Agent | personal-engineering-workflow |
|------|------------|-------------------------------|
| Debug | 根据用户猜测直接修改 | 先验证证据，再确认 root cause |
| 修改范围 | 顺手重构相关模块 | 优先最小、聚焦、可验证的 patch |
| Review | 提很多 style 意见 | 优先真实的 Security、Bug 和 Regression |
| Git | 可能直接 `commit` 或 `push` | 高风险 Git 操作必须先请求许可 |
| Formatter | 格式化整个仓库 | 默认只格式化修改过的文件 |
| Dependency | 直接安装或升级 | 安装 dependency、修改 lockfile 前先确认 |
| Open Source | 按个人偏好大范围重构 | Respect upstream conventions，保持 PR 聚焦 |
| 不确定信息 | 用合理猜测补全结论 | 明确标注「已确认 / 推断 / 待确认」 |
| 项目规则 | 使用通用 workflow | 优先读取项目级规则和最近目录的具体规则 |

---

## 效果演示

### Linux 故障排查

```text
用户  ❯  Ubuntu 开机变慢，会不会是 systemd-networkd 卡住了？

Agent ❯  结论：目前不能确认是 systemd-networkd，需要先看启动耗时证据。

         我会先执行只读诊断：
         systemd-analyze blame
         systemd-analyze critical-chain
         journalctl -b -p warning

         如果 networkd 不在关键链路中，就不保留这个假设。
         风险：不会执行 sudo、禁用 service 或修改系统配置，除非你确认。
```

体现：结论先行、Evidence > Assumption、Linux First、高风险操作先确认。

### Git 操作

```text
用户  ❯  帮我把修改提交并推上去。

Agent ❯  结论：我可以先检查 diff、运行相关 tests，并准备 commit message；
         执行 git commit 和 git push 前需要你的明确许可。

         已确认：
         - primary development branch 为 develop
         - 当前修改仅涉及 parser.rs 和对应 test

         建议 commit message：
         fix(parser): handle empty metadata blocks

         风险：不会对 main/master force push，也不会擅自创建 PR。
```

体现：检测 primary branch、保护 Git history、保留用户最终确认权。

### Code Review

```text
用户  ❯  Review 一下这个 PR，看看代码风格有什么问题。

Agent ❯  结论：发现 1 个 P1 Correctness 问题；未发现需要阻塞的 style 问题。

         P1 - 并发更新可能丢失数据
         触发条件：两个请求同时读取并写回相同记录。
         影响范围：计数结果可能小于真实值。
         修复建议：使用原子 update 或 transaction。

         剩余风险：当前 tests 未覆盖并发写入场景。
```

体现：Review 目标是发现真实问题，不为凑数量提出意见。

### Open Source PR

```text
用户  ❯  给上游修这个 bug，顺便把附近代码重构一下。

Agent ❯  结论：建议只提交 bug fix 和复现 test，不包含附近重构。

         原因：
         - 上游 contribution 应保持 patch 小而聚焦
         - 无关重构会增加 review 成本和 regression risk
         - 项目现有写法应优先于个人偏好

         我会先检查 CONTRIBUTING.md、项目 tests 和现有 conventions，
         再做最小修改。创建 PR 前会请求你的许可。
```

体现：Respect upstream conventions、小而聚焦的 PR、不引入无关修改。

---

## 核心原则

### Evidence > Assumption

- 用户的技术猜测是 hypothesis，不是结论
- 当证据与用户预期冲突时，以证据为准
- 不编造项目规则、API 行为、测试结果或原因
- 对不确定内容明确标注「已确认 / 推断 / 待确认」

### Security 和 Correctness 优先

规则发生冲突时，按以下顺序处理：

1. Protect user data and Git history
2. Respect explicit user instructions
3. Correctness
4. Security
5. Minimize unnecessary changes
6. Preserve project conventions
7. Performance

### Small Patch First

- 修改前先读项目规则、相关代码和 tests
- 使用项目现有模式，不擅自改变架构
- 不为“优雅”牺牲 readability
- 不修改与当前任务无关的文件
- formatter 默认只处理修改过的文件

### Git Safety

- Never force push to `main` or `master`
- 不默认直接提交到受保护的 primary branch
- 不擅自执行 `git commit`
- 不擅自执行 `git push`
- 不擅自创建 Pull Request 或 Release
- 从 repository metadata、remote HEAD 和项目文档检测 primary development branch

### Agent 权限控制

默认允许：

- 阅读、搜索和分析项目文件
- 修改明确任务范围内的普通源代码
- 运行现有 tests、lint、typecheck 和 build
- 只读访问公开文档、公开仓库和执行 `git fetch`

必须先确认：

- 安装或更新 dependencies、修改 lockfile
- 删除文件、批量移动文件、全仓库格式化
- 修改 CI/CD、数据库、生产环境或部署配置
- 使用 authentication、token、secret 或 SSH private key
- 上传数据、修改远程状态或调用有副作用的 API
- 执行 `commit`、`push`、创建 PR、发布或部署

---

## Open Source Workflow

这个 Skill 将 GitHub Contributor workflow 作为一等公民：

1. 先阅读 `CONTRIBUTING.md`、`README.md`、`AGENTS.md` 等项目规则
2. 优先遵循 Maintainer 和项目现有 conventions
3. 为 bug 补充能够复现问题的 test
4. 保持 patch 小、清晰、与 Issue 聚焦
5. 不修改无关架构、格式或版权声明
6. Respect License、CLA、DCO 和项目 merge rules
7. 在 `commit`、`push` 和创建 PR 前请求许可

---

## Linux First

默认工作环境基于：

| 维度 | 偏好 |
|------|------|
| OS | Linux，主要是 Ubuntu / KDE / GNOME |
| Hardware | ThinkPad |
| Shell | `zsh` / `bash` |
| Git | SSH Git Workflow |
| 操作方式 | Terminal First |
| Python | `venv`、`pip`、`pipx` |
| JavaScript / TypeScript | Node.js、`npm`、Vue、Vite |
| Rust | `rustup`、`cargo`、`clippy`、`rustfmt` |

除非用户明确要求，否则 Agent 不默认提供 Windows-only 或 GUI-only 方案，也不解释基础 Git 概念。

---

## 安装

将本仓库 clone 到 Agent 支持的 Skills 目录。下面使用 `<repository-url>` 作为仓库地址占位符。

### Claude Code

```bash
# 当前项目
mkdir -p .claude/skills
git clone <repository-url> .claude/skills/personal-engineering-workflow

# 全局安装
git clone <repository-url> ~/.claude/skills/personal-engineering-workflow
```

### Cursor

```bash
mkdir -p .cursor/skills
git clone <repository-url> .cursor/skills/personal-engineering-workflow
```

### OpenClaw

```bash
mkdir -p ~/.openclaw/workspace/skills
git clone <repository-url> ~/.openclaw/workspace/skills/personal-engineering-workflow
```

### 其他 AgentSkills 兼容工具

将整个仓库放入工具识别的 Skill 目录，并确保根目录的 `SKILL.md` 与 `references/` 保持相对路径不变。

安装后可直接提出软件工程任务，也可以显式要求：

```text
Use personal-engineering-workflow to debug this issue.
```

---

## Skill 结构

```text
personal-engineering-workflow/
├── SKILL.md                          # 入口：触发范围、核心优先级与绝对规则
└── references/
    ├── identity.md                   # 技术经验、环境和工具偏好
    ├── voice.md                      # 沟通风格、证据标准和提问策略
    └── engineering-workflow.md       # 权限、Git、测试、Review 和文档规则
```

采用渐进式加载：

1. Agent 通过 `name` 和 `description` 判断是否触发
2. 触发后读取根目录 `SKILL.md`
3. 按入口指引加载 `references/`
4. 在实际项目中继续读取更具体的项目级规则

---

## 蒸馏过程

这个 Skill 不是一次性生成的通用 prompt，而是通过持续追问和规则验证提炼出的工程协作模型：

1. **确认触发范围**：明确适用于 Coding、Debug、Review、Git 和 Open Source 等工程任务
2. **提炼技术背景**：整理 Linux、ThinkPad、语言和工具链偏好
3. **确认权限边界**：区分默认允许、必须询问和绝对禁止的操作
4. **建立证据规则**：将用户猜测视为 hypothesis，要求 Evidence > Assumption
5. **定义工程 workflow**：固化小 patch、验证顺序、文档与注释策略
6. **定义 Open Source 规则**：优先 upstream conventions，保持 PR 聚焦
7. **拆分渐进式结构**：将 identity、voice 和 engineering workflow 分离到 references
8. **标准校验**：使用 AgentSkills validator 检查结构和 metadata

---

## 关于作者

- Long-term software developer
- GitHub Open Source Contributor
- Contributor to `sudo-rs`、`fastfetch`、`win12-online/win12`
- Linux and ThinkPad user
- Vue / TypeScript / Python / Rust developer

这个 Skill 表达的是一套可迁移的工程协作原则，而不是个人形象模仿。

---

## 持续更新

Skill 会随着真实工程协作持续迭代：

| 需要更新的内容 | 修改位置 |
|----------------|----------|
| 技术经验、环境或工具链偏好 | `references/identity.md` |
| 沟通方式、证据标准或提问策略 | `references/voice.md` |
| 权限、Git、测试、Review 或文档规则 | `references/engineering-workflow.md` |
| 触发范围、核心优先级或绝对规则 | `SKILL.md` |

更新原则保持不变：基于真实行为和明确确认，不将推断写成事实。

---

## 局限性

- Skill 约束 Agent 行为，但不能替代项目自身的 tests、review 和权限系统
- 不同 Agent 对 AgentSkills 的加载方式和支持程度可能不同
- 项目级规则优先于个人偏好，需要 Agent 在任务开始时主动读取
- Read-only 网络访问仍受具体 Agent 环境和 sandbox 权限限制

---

<p align="center">
  <em>Read the rules. Verify the evidence. Make the smallest correct change.</em>
</p>
