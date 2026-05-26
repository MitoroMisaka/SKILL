# SKILL — Personal AI Agent Skills

AI agent 操作约束的可执行版本。每次有非琐碎的 AI 协作 session 后，将关键操作约束固化为 skill。

## 设计原则

- **Skill 先，文档后。** 操作约束是可执行的，叙事是辅助的。
- **双锚点索引。** Skill ↔ Plan/Blog 互相引用，两端互为入口。
- **Dogfood 自验证。** 每个 skill 写好之后，让 agent 按着它跑一次来验证。

## 目录结构

```
SKILL/
├── README.md
├── AGENTS.md
└── skills/
    └── automation/
        ├── session-to-skill/
        │   └── SKILL.md
        ├── doc-driven-dev/
        │   └── SKILL.md
        └── multi-agent-orchestration/
            └── SKILL.md
```

## 使用方式

这些 skill 存放在 `~/.hermes/skills/` 下，Hermes Agent 自动加载。本仓库用于：
- 版本控制与历史追溯
- 跨设备同步
- 作为 plan/blog 文档的引用 URL

## 工具链

- **AI Agent**: [Hermes Agent](https://github.com/NousResearch/hermes-agent)
- **开发环境**: macOS + WezTerm
- **编辑器**: VS Code / Neovim
- **Git**: GitHub CLI (`gh`)
