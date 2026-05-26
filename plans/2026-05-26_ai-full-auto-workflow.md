# AI 全自动开发工作流 —— 完整方案

> 基于 Innei (innei.in) 的 AI 工作流方法论，适配到 Hermes Agent 生态。
>
> 日期: 2026-05-26
>
> 相关:
> - Skill: `~/.hermes/skills/automation/session-to-skill/SKILL.md`
> - Skill: `~/.hermes/skills/automation/doc-driven-dev/SKILL.md`
> - Skill: `~/.hermes/skills/automation/multi-agent-orchestration/SKILL.md`
> - 全局约束: `/Users/liaojinchuan/AGENTS.md`
> - GitHub: `https://github.com/MitoroMisaka/SKILL`

---

## 一、架构总览

```
                    ┌─────────────────────────────┐
                    │        AGENTS.md            │
                    │   (全局编码规范 + AI 约束)    │
                    └─────────────┬───────────────┘
                                  │ 注入每次会话
                    ┌─────────────┴───────────────┐
                    │       Hermes Agent          │
                    │  (Claude Code / Codex 等价)  │
                    └─────────────┬───────────────┘
                                  │
          ┌───────────────────────┼───────────────────────┐
          │                       │                       │
   ┌──────▼──────┐       ┌───────▼───────┐      ┌───────▼───────┐
   │ doc-driven  │       │  multi-agent  │      │ session-to-   │
   │    -dev     │       │ orchestration │      │    skill      │
   │             │       │               │      │               │
   │ PRD → TECH  │       │ delegate_task │      │ session 产物  │
   │ → TASKS →   │       │  / bg / tmux  │      │ → skill +     │
   │   执行       │       │               │      │   plan 文档    │
   └──────┬──────┘       └───────┬───────┘      └───────┬───────┘
          │                       │                       │
          └───────────────────────┼───────────────────────┘
                                  │
                    ┌─────────────┴───────────────┐
                    │     GitHub SKILL 仓库        │
                    │  (skill 版本控制 + 同步)      │
                    └─────────────────────────────┘
```

---

## 二、核心工作流循环

```
需求/想法
    │
    ▼
┌──────────────────────────────────────┐
│ 1. 头脑风暴（需求模糊时）              │
│    和 AI 讨论，确定方向               │
└──────────────┬───────────────────────┘
               │ 方向清晰
               ▼
┌──────────────────────────────────────┐
│ 2. 文档驱动（PRD → TECH → TASKS）     │
│    全程只写文档，不写代码              │
│    Skill: doc-driven-dev             │
└──────────────┬───────────────────────┘
               │ 文档就绪
               ▼
┌──────────────────────────────────────┐
│ 3. AI 执行                            │
│    ┌─ 简单任务: 当前会话直接做         │
│    ├─ 中等任务: delegate_task 并行     │
│    ├─ 大型重构: terminal bg spawn     │
│    └─ 需要审核: 每阶段暂停            │
│    Skill: multi-agent-orchestration   │
└──────────────┬───────────────────────┘
               │ 完成后
               ▼
┌──────────────────────────────────────┐
│ 4. Session 沉淀                       │
│    提取关键约束 → skill               │
│    写叙事 → plan 文档                  │
│    Skill: session-to-skill            │
│    Iron rule: skill 先，文档后        │
└──────────────┬───────────────────────┘
               │ 沉淀完成
               ▼
┌──────────────────────────────────────┐
│ 5. 知识回流                           │
│    skill 自动加载到未来会话            │
│    下次遇到同类问题不再重走弯路         │
└──────────────────────────────────────┘
```

---

## 三、各阶段详细说明

### 阶段 1: 头脑风暴

```
触发条件: 需求不清晰
做法: 直接和 Hermes 对话，讨论方向、产品形态、核心功能
产出: 一段话描述方向
注意: 这个阶段不写文档，只是聊天
```

### 阶段 2: 文档驱动

```
触发条件: 需求清晰，准备动手
做法: 加载 doc-driven-dev skill，逐文档推进
产出:
  docs/PRD-<feature>.md    → 产品需求（目标/范围/约束/标准）
  docs/TECH-<feature>.md    → 技术方案（架构/选型/ADR/文件清单）
  docs/TASKS-<feature>.md   → 执行任务列表（可逐项勾选）
```

### 阶段 3: AI 执行

按任务复杂度选择执行模式：

| 规模 | 模式 | 工具 |
|------|------|------|
| 修一行 bug | 当前会话 | 直接 terminal/file |
| 3-10 个文件 | 当前会话分阶段 | terminal/file + git commit |
| 10-50 个文件 | delegate_task | delegate_task(tasks=...) |
| 50+ 文件，多模块 | terminal background | terminal(bg=true, notify=true) |
| 需要人工审查 | 分阶段暂停 | 每阶段暂停等确认 |

### 阶段 4: Session 沉淀

```
触发条件: 任何非琐碎的 AI 协作 session 结束
做法: 加载 session-to-skill skill
产出:
  ~/.hermes/skills/<category>/<name>/SKILL.md  → 可执行操作约束
  ~/.hermes/plans/YYYY-MM-DD_<topic>.md         → 叙事性文档
```

### 阶段 5: 知识回流

```
自动生效: skill 在 ~/.hermes/skills/ 下，Hermes 每次会话自动扫描
手动推送: cd ~/SKILL && git push（可选，用于跨设备同步）
```

---

## 四、快速上手指南

### 新项目启动

```
1. "我要做一个 XXX，帮我讨论一下方向"           → 头脑风暴
2. "加载 doc-driven-dev skill，帮我写 PRD"      → 文档驱动
3. "根据 PRD 写技术方案"
4. "拆解为执行任务"
5. "按 TASKS 实现，每阶段 commit"
```

### 大重构

```
1. 人做 RFC（调研 + 可行性验证）
2. "加载 doc-driven-dev skill，把这些调研结果写进 TECH 文档"
3. "拆解为 5 个 Plan，每个 Plan 再拆子任务"
4. "用 delegate_task 或 terminal bg 执行，跳过 lint/format"
5. 每阶段结束 git log --oneline 审查进度
6. "加载 session-to-skill skill，沉淀这次重构的教训"
```

### 日常 Bug 修

```
直接修。不启动完整工作流。
但如果涉及非显而易见的根因分析 → 沉淀为 skill。
```

---

## 五、文件与目录约定

```
/Users/liaojinchuan/
├── AGENTS.md                          ← 全局编码规范 + AI 协作约束
│
├── Projects/
│   └── <project>/
│       ├── AGENTS.md                  ← 项目级编码规范（可选）
│       ├── docs/                      ← 项目文档
│       │   ├── PRD-<feature>.md
│       │   ├── TECH-<feature>.md
│       │   └── TASKS-<feature>.md
│       └── .hermes/
│           ├── plans/                 ← 项目级 plan 文档
│           └── tasks/                 ← agent 完成标记文件
│
├── SKILL/                             ← GitHub 仓库 (MitoroMisaka/SKILL)
│   └── skills/automation/
│       ├── session-to-skill/SKILL.md
│       ├── doc-driven-dev/SKILL.md
│       └── multi-agent-orchestration/SKILL.md
│
└── .hermes/
    ├── skills/                         ← Hermes 自动加载
    │   └── automation/                 ← 与 SKILL 仓库同步
    ├── plans/                          ← 通用 plan 文档
    │   └── YYYY-MM-DD_<topic>.md
    └── config.yaml
```

---

## 六、与 Innei 原版的关键对应

| 概念 | Innei 的做法 | 我们的做法 |
|------|------------|-----------|
| AI Agent | Claude Code / Codex / GPT-5.5 | Hermes Agent |
| 全局约束 | `~/.claude/CLAUDE.md` | `/Users/liaojinchuan/AGENTS.md` |
| 项目约束 | 项目级 `CLAUDE.md` | 项目级 `AGENTS.md` |
| Skill 存储 | GitHub `Innei/SKILL` | `/Users/liaojinchuan/SKILL` |
| Skill 格式 | 自定义 SKILL.md | Hermes 原生 SKILL.md |
| Blog 输出 | mxs CLI → innei.in | Plan 文档 → `~/.hermes/plans/` |
| 富文本 | LiteXML | 纯 Markdown |
| 多 agent | 5-6 个窗口手动切换 | `delegate_task` + `terminal bg` + `tmux` |
| 知识图谱 | Claude 项目分析 | Hermes `session_search` + `memory` |
| 定时任务 | 无明确提及 | Hermes `cronjob` |
| UI 头脑风暴 | Superpower brainstorming | Hermes 对话 + `vision_analyze` |

---

## 七、关键原则（Iron Rules）

1. **Skill 先，文档后。** 操作约束是可执行的，叙事是辅助的。
2. **文档驱动，代码跟随。** 先写 PRD → TECH → TASKS，再让 AI 按文档实现。
3. **大型重构跳过 lint/format。** 速度优先，后续单独修。
4. **每阶段 commit。** 方便回滚和追溯。
5. **双锚点索引。** Skill 和 Plan 文档互相引用。
6. **Dogfood 自验证。** Skill 写好之后让 agent 重跑一次。
7. **上下文自包含。** 子 agent 不知道你做什么，传足上下文。
8. **简单事情直接做。** 不要为了工作流而工作流。

---

## 八、下一步

1. **立刻试用**: 随便找个小项目，按阶段 0→1→2→3 跑一遍
2. **沉淀第一个 session**: 跑完之后用 session-to-skill 沉淀
3. **完善 AGENTS.md**: 随着使用积累，把项目特有的规范加进去
4. **创建第一个 cron job**: 如果有什么需要定期检查的，试试 cronjob

---

*本方案的目标不是让你变成 AI 的操作工，而是让你从"代码编写者"转型为"架构决策者 + AI 指挥者 + 知识管理者"。*

*Skill: `~/.hermes/skills/automation/session-to-skill/SKILL.md`*
*Skill: `~/.hermes/skills/automation/doc-driven-dev/SKILL.md`*
*Skill: `~/.hermes/skills/automation/multi-agent-orchestration/SKILL.md`*
*GitHub: `https://github.com/MitoroMisaka/SKILL`*
