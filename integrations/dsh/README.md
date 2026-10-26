# 接入 DeepSeek Harness（dsh）

dsh 读 AGENTS.md 和 skills，两者都支持，而且是自动热加载。这份文档说明怎么把本仓库的内容装进去。

## 文件放哪

| 位置 | 生效范围 |
|---|---|
| `~/.dsh/AGENTS.md` | **全局**，所有会话 |
| `<项目根>/AGENTS.md` 或 `CLAUDE.md` | 整个项目（项目根 = 往上找第一个 `.git`） |
| `<子目录>/AGENTS.md` | 只在该子目录生效 |
| `AGENTS.local.md` | 同目录的本地覆盖层，在基础文件之后加载 |

同目录有多个候选时全部加载，**AGENTS.md 优先于 CLAUDE.md**。渲染上限 64KB，单文件超 1MB 直接忽略，超预算静默截断、不报错。

## 安装

**规则（常驻）**——把 `AI-CONTEXT.md` 的精简版写成全局规则：

```bash
cp AI-CONTEXT.md ~/.dsh/AGENTS.md
```

**模板（按需）**——做成技能，不常驻、不占每轮 token：

```bash
mkdir -p ~/.dsh/skills/prompts
cp prompts.md ~/.dsh/skills/prompts/SKILL.md
```

技能文件必须带 frontmatter，否则会被静默忽略：

```yaml
---
name: prompts
description: 一句话说清这个技能在什么时候用（模型靠它判断要不要加载）
whenToUse: 可选
---
```

## 技能目录优先级

| 目录 | 说明 |
|---|---|
| `<项目>/.dsh/skills/` | 项目专属，最高 |
| `<项目>/.agents/skills/` | 项目共享 |
| `~/.dsh/skills/` | **个人全局** |
| `~/.agents/skills/` | 全局共享 |

格式两种：`<目录>/SKILL.md` 或单文件 `.md`。

## 注入机制（影响用法）

AGENTS.md **不是塞进系统提示词的**，而是作为**用户角色消息的前缀**注入——模型看到的效果是对话开头多了一条用户消息，每段带 `Instructions from:` 标记。首次注入完整内容，之后只在文件变化时增量替换。

由此有两个推论：

1. **它每轮都算 token。** 所以全局 AGENTS.md 要短（本仓库的精简版约 2.5KB），别把模板库塞进去。
2. **它的优先级低于系统提示词。** 如果 dsh 内置提示词和你的规则冲突，内置的赢。规则要写得具体，别跟工具本身的行为抢。

## 更新流程

改完仓库里的文件后：

```bash
cp AI-CONTEXT.md ~/.dsh/AGENTS.md   # 注意：如果本地做过精简，用精简版覆盖
```

技能文件改完自动热更新，不用重启。
