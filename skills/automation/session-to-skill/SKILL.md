---
name: session-to-skill
description: "将 AI 协作 session 的产物固化为 skill 和 plan 文档。铁律：skill 先，文档后。"
version: 1.0.0
---

# Session to Skill —— AI 协作 session 沉淀工作流

## When to Use

当一次 AI 协作 session 满足以下任一条件时触发：
- 踩了非显然的坑，来回纠偏多次
- 形成了非琐碎的工程结论
- 发现了新的工作流或操作约束
- 解决了以后可能再次遇到的问题
- session 超过 5 个工具调用且有非标准操作

## Iron Rule

**Skill 先，文档后。**

不是审美。文档（plan/blog）顶部和底部要嵌 skill 路径或 URL，路径必须先在文件系统上 resolve 才能写进去。SKILL.md 的强制结构（frontmatter、scope、workflow、pitfalls table、verification checklist）会把这次 session 的操作约束提前固化成可执行形态，narrative 写在后面，不会扭曲实际的工程教训。

## Scope

- **适用项目**: 所有在 `/Users/liaojinchuan/` 下的项目
- **Skill 存储路径**: `~/.hermes/skills/<category>/<skill-name>/SKILL.md`
- **Plan 存储路径**: `~/.hermes/plans/YYYY-MM-DD_<topic>.md` 或项目下的 `.hermes/plans/`
- **GitHub 仓库**: `github.com/MitoroMisaka/SKILL`（skill 版本管理）

## Workflow（五步骨架）

### Step 1: 提取 session 精华

复盘这次 session，提取：
- 遇到了哪些坑？怎么解决的？
- 哪些操作不是标准流程？
- 形成了什么工程结论？
- 哪些步骤如果重来一次可以跳过？

输出：一份简洁的要點清单。

### Step 2: 固化 SKILL.md

按以下强制结构写 SKILL.md：

```yaml
---
name: skill-name
description: "一句话描述"
version: 1.0.0
---

# Skill Title

## When to Use
触发条件（什么时候加载这个 skill）

## Scope
适用范围（什么项目、什么场景）

## Workflow
分步骤的具体操作流程

## Pitfalls
| 错 | 正 |
|----|-----|
| 常见错误 | 正确做法 |
```

**SKILL.md 写入路径**: `~/.hermes/skills/<category>/<skill-name>/SKILL.md`

### Step 3: 推送 skill（可选但推荐）

如果 GitHub SKILL 仓库已配置：
```bash
cd ~/SKILL && git add -A && git commit -m "skill: <name>" && git push
```

Hermes 环境下 skill 自动生效，无需推送即可使用。但推送到 GitHub 可以：
- 跨设备同步
- 作为文件引用 URL
- 防止丢失

### Step 4: 写 plan 或项目文档

在 plan 文档中：
- 顶部嵌入 skill 的绝对路径：`Skill: ~/.hermes/skills/<category>/<skill-name>/SKILL.md`
- 写叙事性说明——这次做了什么，为什么这样做
- 底部再次引用 skill 路径

### Step 5: 验证

- [ ] SKILL.md 的 5 个强制结构字段齐全（name/description/When to Use/Workflow/Pitfalls）
- [ ] skill 可以通过 `skill_view` 加载
- [ ] plan 文档顶部和底部都嵌入了 skill 引用
- [ ] skill 中的操作约束可以通过重放验证

## Persona 规则

| 场景 | voice | "我"是谁 |
|------|-------|---------|
| 过程记录（agent 干活） | agent first-person | "我"是执行体 AI agent |
| 系统/工具设计（物是主角） | user/site-owner | "我"是开发者 liaojinchuan |
| 混合场景（设计 + dogfood） | site-owner + 第三人称提 agent | "一次 agent 跑通时撞到 X" |

不要中途切换 persona。选一种贯穿始终。

## Pitfalls Table

| 错 | 正 |
|----|-----|
| 文档早于 skill | skill 必先写入，文档才有路径可嵌 |
| 跳过 pitfalls table | SKILL.md 必备字段，也是最易被检索的段 |
| 凭直觉挑 persona | 过程→agent，物/系统→site-owner |
| 中途切换 persona | 选一种贯穿。混合场景用 site-owner + 第三人称 |
| skill 引用只嵌一处 | 双锚点：top + bottom |
| 把 session 过程当 skill | skill 是操作约束，不是叙事。叙事放 plan |
| skill 写完不用 dogfood 验证 | 让 agent 按着 skill 重跑一次，看能不能通 |

## Verification Checklist

- [ ] Step 1 完成——session 精华已提取
- [ ] Step 2 完成——SKILL.md 写入，5 结构字段齐全
- [ ] Step 3 完成——skill 可被 `skill_view` 加载
- [ ] Step 4 完成——plan/文档写了，顶部底部嵌了 skill 引用
- [ ] 没有跳步——没有"先写文档再补 skill"的情况
- [ ] Persona 一致——没有中途切换

## 与 Innei 原版的差异

| Innei | Hermes 适配 |
|-------|-----------|
| blog 作为 narrative 输出 | plan 文档或项目 AGENTS.md 作为 narrative 输出 |
| mxs CLI 发布到博客系统 | 直接写入 `~/.hermes/plans/` 或项目文档 |
| LiteXML 格式 | 纯 Markdown（Hermes 原生支持） |
| GitHub SKILL 仓库 | `~/.hermes/skills/` 本地生效，GitHub 仓库可选 |
