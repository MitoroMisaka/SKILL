---
name: doc-driven-dev
description: "文档驱动开发：先写 PRD 和技术方案，再让 AI 按文档执行。从脑暴到交付的完整流程。"
version: 1.0.0
---

# 文档驱动开发（Document-Driven Development）

## When to Use

- 新项目从零搭建（MVP 阶段）
- 中等以上复杂度的功能开发（>3 个文件变更）
- 需要先理清思路再动手的任何场景
- 用户说"帮我做一个..."但需求模糊

**不适用**：修一行 bug、改配置、加一个简单字段——这些直接做就行。

## 核心理念

> 全程只跟 AI 聊文档，不聊代码。文档写清楚后，丢给 agent 说"按文档实现"。

## Scope

- 所有开发项目
- 前端/后端/全栈均适用
- Hermes Agent 原生支持（`delegate_task` + `terminal` + `skill`）

## Workflow

### 阶段 0：头脑风暴（需求模糊时）

如果需求不清晰，先和 AI 讨论方向：

```
用 brainstorming 模式讨论：
"我想做一个 XXX，但还没想好具体方向。
 跟我讨论一下可能的产品形态、核心功能和目标用户。"
```

产出：一个清晰的方向描述（一段话，不是文档）。

### 阶段 1：PRD（产品需求文档）

在项目根目录或 `~/.hermes/plans/` 下创建 `PRD-<feature>.md`：

```markdown
# Product Requirements Document: <功能名>

## 目标
- [明确、可衡量的功能目标]

## 用户故事
- 作为 <角色>，我想要 <功能>，以便 <价值>

## 功能范围
### In Scope
- 核心功能 A
- 核心功能 B

### Out of Scope（这个版本不做）
- 非核心功能 X

## 技术约束
- [不可谈判的技术决策]
- [必须遵循的架构原则]

## 质量标准
- 性能基准
- 可维护性要求
- 测试覆盖标准

## UI/UX 参考
- 参考项目 A（链接）
- 参考项目 B（链接）
- 风格倾向：<简洁/现代/Apple-like/...>
```

### 阶段 2：技术方案

在 PRD 基础上写 `TECH-<feature>.md`：

```markdown
# 技术方案: <功能名>

## 架构概览
[目录结构/组件树/数据流]

## 技术选型
| 层 | 选择 | 理由 |
|----|------|------|
| 框架 | React + Vite | 现有栈 |
| 样式 | TailwindCSS | 项目约定 |
| 状态 | Zustand | 轻量，够用 |

## API 设计（如有后端）
| 方法 | 路径 | 用途 |
|------|------|------|
| GET | /api/xxx | 获取列表 |
| POST | /api/xxx | 创建 |

## 关键决策记录（ADR）
1. 为什么选 A 不选 B
2. 为什么...

## 文件变更清单（预估）
| 文件 | 操作 | 说明 |
|------|------|------|
| src/pages/xxx.tsx | 新增 | 主页面 |
| src/components/xxx.tsx | 新增 | 核心组件 |
| src/api/xxx.ts | 新增 | API 层 |
```

### 阶段 3：拆解为执行计划

把技术方案拆成可执行的任务列表 `TASKS-<feature>.md`：

```markdown
# 执行任务: <功能名>

## Task 1: 项目基础设施
- [ ] 创建目录结构
- [ ] 安装依赖
- [ ] 配置路由

## Task 2: 数据层
- [ ] 定义类型/接口
- [ ] 实现 API 层

## Task 3: UI 组件
- [ ] 核心组件 A
- [ ] 核心组件 B

## Task 4: 页面组装
- [ ] 主页面布局
- [ ] 交互逻辑

## Task 5: 测试 & 收尾
- [ ] 基础功能测试
- [ ] 边缘情况
- [ ] 构建验证
```

### 阶段 4：AI 执行

把所有文档放到一个目录下，对 AI agent 说：

```
阅读 <路径>/PRD-xxx.md、TECH-xxx.md、TASKS-xxx.md，
按 TASKS 分阶段执行。每个阶段结束后 git commit。
大型重构时跳过 lint/format，直接 commit。
```

执行模式：
- **MVP 快速版**：用 `delegate_task` 交给子 agent 执行
- **大型重构版**：用 `terminal` spawn 独立 `hermes` 进程
- **需要审核版**：亲自动手执行，每阶段暂停审阅

### 阶段 5：后续跟进

如果有改动，更新文档优先于改代码：
- 技术选型变了 → 更新 TECH 文档
- 新增功能 → 更新 PRD 文档
- 实现细节 → 更新 TASKS 文档

**改完文档再让 AI 按文档改代码。**

## Pitfalls

| 错 | 正 |
|----|-----|
| 跳过文档直接写代码 | 文档是 AI 执行时的唯一依据，没文档 = 凭运气 |
| 需求模糊时不讨论就开始 | 先头脑风暴，方向清楚了再写文档 |
| 文档写完就不管了 | 文档是活的，后续改动先改文档 |
| 在代码中混入架构决策 | 架构决策属于 TECH 文档，代码只是实现 |
| 让 AI 边做边想 | AI 执行时必须只依赖文档，不要让它"自己判断" |
| 文档太长写不完 | 先写核心部分，执行中发现缺什么再补 |
| 文档放错位置 | 项目文档放项目根目录 `docs/`；通用文档放 `~/.hermes/plans/` |

## Templates

以上模板（PRD/TECH/TASKS）可直接复制使用。也可以让 AI 帮你填——把想法说给 AI，让它按模板输出。

## 与 Innei 原版的对应

| Innei | Hermes 适配 |
|-------|-----------|
| Claude Project + Analysis | Hermes 会话 + `~/.hermes/plans/` |
| Superpower brainstorming skill | 直接用 Hermes chat 讨论 |
| Claude Code `/generate-prps` + `/execute-prp` | `delegate_task` 或 `terminal` spawn hermes |
| 文档目录丢给 agent | Skill `doc-driven-dev` 加载后 agent 自动遵循 |

## Verification Checklist

- [ ] 需求清晰（一段话能说清楚做什么）
- [ ] PRD 文档存在且包含：目标/范围/技术约束/质量标准
- [ ] TECH 文档存在且包含：架构/选型/ADR/文件清单
- [ ] TASKS 文档存在且可逐项执行
- [ ] 所有文档互相引用，没有孤岛
- [ ] 执行时 agent 只依赖文档，不自行发挥
