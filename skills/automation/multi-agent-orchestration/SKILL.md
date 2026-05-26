---
name: multi-agent-orchestration
description: "多 agent 并行编排：如何同时驱动多个 AI agent 做不同的事，最大化吞吐量。"
version: 1.0.0
---

# 多 Agent 并行编排

## When to Use

- 有多个独立任务需要同时推进（一个项目的前端 + 后端，多个项目并行）
- 大型重构需要拆解为并行子任务
- 需要在等待一个任务的同时推进另一个
- 需要不同模型处理不同类型任务（推理型 vs 执行型）

## 核心理念

> 一个人同时指挥多个 agent。在 A 项目发指令，切换到 B 项目发指令，来回穿梭。AI 不会累，但人会——所以尽量减少上下文切换的成本。

## 三种并行模式

### 模式 1：delegate_task（推荐，最常用）

Hermes 原生同步子 agent。父进程等待子 agent 完成后继续。

**适用**：独立子任务，几分钟到十几分钟，需要结果汇总。

```python
# 并行执行三个独立任务
delegate_task(tasks=[
    {"goal": "实现后端 API 的 /users 接口", "context": "...", "toolsets": ["terminal", "file"]},
    {"goal": "实现前端用户列表页面", "context": "...", "toolsets": ["terminal", "file"]},
    {"goal": "写单元测试", "context": "...", "toolsets": ["terminal", "file"]},
])
```

**注意事项**：
- 最多 3 个并发（限制在 config.yaml 的 `delegation.max_concurrent_children`）
- 子 agent 有独立终端和文件系统，不能共享状态
- 子 agent 的 summary 是自述，关键外部操作（写入文件、API 调用）自己验证

### 模式 2：Fire-and-Forget（长期任务）

用 `terminal(background=True, notify_on_complete=True)` spawn 独立 hermes 进程。

**适用**：需要几十分钟到几小时的任务，不要阻塞当前会话。

```bash
# 启动独立 agent 做长期任务
terminal(
  command="hermes chat -q '从零搭建 React + Vite 前端项目，包含路由、状态管理、API 层、基础组件库。完成后写入 /tmp/done-frontend.txt'",
  background=true,
  notify_on_complete=true,
  timeout=3600
)

# 同时启动另一个做后端
terminal(
  command="hermes chat -q '从零搭建 FastAPI 后端，包含 auth、CRUD、数据库迁移。完成后写入 /tmp/done-backend.txt'",
  background=true,
  notify_on_complete=true,
  timeout=3600
)

# 继续在当前会话做其他事
```

### 模式 3：Tmux 交互式（需要人工介入的长期任务）

**适用**：需要中途查看/调整的任务，几小时级别的长期工作。

```bash
# 启动独立 hermes 在 tmux 中
terminal(command="tmux new-session -d -s agent-frontend -x 120 -y 40 'hermes'", timeout=10)
terminal(command="sleep 5 && tmux send-keys -t agent-frontend '重构整个前端项目，先读 AGENTS.md 了解规范' Enter", timeout=15)

# 启动第二个
terminal(command="tmux new-session -d -s agent-backend -x 120 -y 40 'hermes'", timeout=10)
terminal(command="sleep 5 && tmux send-keys -t agent-backend '给后端加 API 限流和缓存层' Enter", timeout=15)

# 查看进度
terminal(command="tmux capture-pane -t agent-frontend -p | tail -40", timeout=5)

# 发追加指令
terminal(command="tmux send-keys -t agent-frontend '把路由改成 lazy load' Enter", timeout=5)

# 关闭
terminal(command="tmux send-keys -t agent-frontend '/exit' Enter && sleep 2 && tmux kill-session -t agent-frontend", timeout=10)
```

## 模式选择决策树

```
任务需要拆分吗？
├── 是 → 每个子任务独立吗？
│   ├── 是，<30 分钟 → delegate_task（模式 1）
│   ├── 是，>30 分钟 → terminal background（模式 2）
│   └── 需要中途查看 → tmux（模式 3）
└── 否 → 在当前会话直接做
```

## 并行工作的最佳实践

### 1. 每个 agent 做独立的事

**好**：agent A 做前端页面，agent B 做后端 API。
**坏**：两个 agent 改同一个文件（会冲突）。

### 2. 上下文要自包含

子 agent 不知道你在做什么。传递足够上下文：
```python
delegate_task(goal="实现用户管理模块",
  context="""
  项目路径: /Users/liaojinchuan/Projects/myapp
  技术栈: React 19 + TypeScript + TailwindCSS + Vite
  API 已存在: GET/POST/PUT/DELETE /api/users
  类型定义: src/types/user.ts
  编码规范: 见 /Users/liaojinchuan/AGENTS.md
  语言: 用中文回复
  """)
```

### 3. 输出约定

让每个 agent 在完成时写一个标记文件，方便确认：
```
完成后写入 <项目>/.hermes/tasks/<task-name>.done
内容: {status: "done", summary: "...", files_changed: [...]}
```

### 4. 避免上下文切换疲劳

Innei 的经验：
- 同一时间只跟进 2-3 个 agent，不要超过 5 个
- 每个 agent 给它足够的上下文和明确的目标，减少反复纠偏
- 用 cronjob 做定时检查，不要手动轮询
- 给 agent 设"阶段完成信号"（如"每完成一个阶段就 commit"）

### 5. 阶段化 + commit 驱动

每个 agent 的指令中都加上：
```
每个阶段结束后执行: git add -A && git commit -m "phase N: <描述>"
大型重构时跳过 lint/format，直接 commit。
```

这样你可以通过 `git log --oneline` 快速了解所有并行 agent 的进度。

## 与 Innei 原版的对应

| Innei | Hermes |
|-------|--------|
| 同时开 5-6 个 Claude Code / Codex 窗口 | `delegate_task` + `terminal bg` + `tmux` |
| 在不同窗口间切换发指令 | `process(action='submit')` 或 `tmux send-keys` |
| Agent 自己写 blog 记录过程 | `session-to-skill` skill 自动沉淀 |
| 代码审查 | 父 agent 读取子 agent 的 summary 后验证 |

## Pitfalls

| 错 | 正 |
|----|-----|
| 给子 agent 的上下文太少 | 传文件路径、技术栈、编码规范、语言要求 |
| 让两个 agent 改同一个文件 | 确保子任务互斥，有交集时串行 |
| 不验证子 agent 的输出 | 关键操作（文件写入、API 调用）自己检查 |
| 同时 spawn 太多 agent | 一次 2-3 个，多了跟不过来 |
| 用 delegate_task 做几小时的任务 | delegate_task 有中断风险，长期用 bg |
| 子 agent 之间的依赖不明确 | 明确先后顺序，上游完成后传结果给下游 |

## Verification Checklist

- [ ] 任务被合理拆分，子任务互相独立
- [ ] 每个子 agent 拿到了足够上下文
- [ ] 选对了并行模式（delegate vs bg vs tmux）
- [ ] 输出约定明确（标记文件 / commit）
- [ ] 子 agent 的结果已验证
