# claude-skills

[English](README.md) | [Skill 编写指南](SKILL-AUTHORING-GUIDE.md)

三个 Claude Code skill，让 AI agent 的产出真正可交付。

| Skill | 命令 | 做什么 |
|-------|------|--------|
| **verify** | `/verify` | 创建验收清单，检查完成状态，产出验证报告 |
| **pm** | `/pm` | 做优先级决策，写 PRD，推进路线图 |
| **wiki** | `/wiki` | 从代码生成参考文档，从资料构建知识 wiki |

## 为什么是这三个

AI agent 写代码很快。但它不知道什么时候算做完、接下来该做什么、以及怎么写人类能看懂的文档。

```
         pm
          "做什么"
              │
              ▼
     ┌──────────────────┐
     │   Agent 干活      │  ← 你现有的 AI 编码工具
     └──────────────────┘
         │           │
         ▼           ▼
       verify        wiki
     "做完了吗？"   "写下来"
```

## 安装

终端里一行命令：

```bash
git clone https://github.com/Nodie-AI/claude-skills.git /tmp/cs && cp -r /tmp/cs/skills/* ~/.claude/skills/ && rm -rf /tmp/cs
```

或者让 Claude Code 帮你装：

```
you> 安装 https://github.com/Nodie-AI/claude-skills 里的 skills，clone 仓库然后把 skills/ 目录复制到 ~/.claude/skills/
```

## 快速上手

### /verify 三步走

```bash
# 第一步：开始任务，创建验收标准
you> 做一个用户注册功能
you> /verify
#    → 生成检查清单：5 项标准，0/5 已检查

# 第二步：agent 干活... 然后检查进度
you> /verify
#    → 跑测试、看代码：3/5 通过，1 失败，1 待确认

# 第三步：修完再查
you> /verify
#    → 5/5 通过。存入回归测试集？(y/n)
```

### /pm 三步走

```bash
# 第一步：带着决策来
you> /pm
you> 先做导出还是先做通知？
#    → "我的判断：先做导出。3 个理由。下一步周四前出 PRD。"

# 第二步：拿 spec
you> 写导出功能的 PRD
#    → 结构化 PRD：scope、用户故事、验收标准

# 第三步：跟进度
you> 这个 sprint 状态如何？
#    → 进度报告 + blocker + 风险标记
```

### /wiki 三步走

```bash
# 第一步：指向代码或资料
you> /wiki
you> 给这个项目生成文档
#    → 读代码，产出架构文档 + 模块文档

# 第二步：喂资料
you> 这篇关于认证最佳实践的文章：<URL>
#    → 抓取、提炼要点、链接到已有 wiki 页面

# 第三步：维护健康
you> 检查一下 wiki
#    → 检查断链、孤儿页、过期内容
```

---

## /verify

**写测试来验证工作完成并发现 bug。

### 问题

当你同时跑多个 AI agent，或者一个 agent 做复杂任务时，验收就成了瓶颈。Agent 说"做完了"，但：

- 它真的实现了所有需求吗？
- 它有没有把之前正常的功能搞坏？
- 不逐个文件看，你怎么知道？

"agent 说做完了" 和 "真的做完了" 之间的差距，就是 bug 上线的地方。

### 原理

`/verify` 在任务开始时创建**可验证的清单**，在 agent 完成时检查——像一个异步的 QA 线程跑在你的工作旁边。

**1. 定义验收标准**（首次 `/verify`，或让 agent 自动生成）：

```
Verification: 用户注册
━━━━━━━━━━━━━━━━━━━━━━

  □ POST /register 返回 201 + user 对象
  □ 密码强度校验（≥8字符，含大小写+数字）
  □ 重复邮箱返回 409
  □ 密码 bcrypt 哈希存储
  □ 注册后发送验证邮件

5 项标准，0/5 已检查。
```

**2. 检查进度**（后续 `/verify` 调用——agent 完成工作后可自动触发）：

```
回归检查：
  ✓ 现有测试通过 (npm test → 42 passed)
  ✓ 构建正常 (tsc --noEmit → exit 0)

新标准：
  ✓ POST /register 返回 201
    └ tests/auth.test.ts → 测试通过
  ✗ 密码 bcrypt 哈希存储
    └ src/models/user.ts:12 — 发现明文存储
  ○ 注册后发送验证邮件
    └ 待确认 — 需要手动测试

进度：●●●○○ 3/5
```

**3. 沉淀**：全部通过后，存入 `.ai-verify/regression/` 作为持久回归测试集。下次任何 agent 在这个代码库工作时，自动检查。测试用例随时间积累，新开发不会 break 旧功能。

该 skill 是**只读的**——通过 frontmatter `allowed-tools` 强制限制，绝不碰源代码。发现问题就报告，修复交给主 agent 或用户。

**为什么叫 "verify"？** 因为目标就是在你 ship 之前，给你一个有证据支撑的"looks good to me"——而不是靠感觉。

---

## /pm

一个完整的 PM agent。不是帮 PM 干活——它**就是** PM。

### 问题

用 AI 编码工具的独立创始人和小团队，工程能力前所未有地强，但产品方向仍然缺位。瓶颈从"能不能做出来"变成了"该做什么，以什么顺序做"。

### 原理

做优先级决策，写 PRD，推路线图，对齐团队，推动执行。需要决策时直接做决策并解释原因。

- 自动识别对话对象（独立创始人 vs 团队工程师 vs 业务负责人），调整输出格式
- **轻量模式**（独立创始人）：结论 + 3 个理由 + 2 个下一步。不主动写 PRD。
- **完整模式**（团队）：结构化 PRD、干系人简报、决策备忘
- 22 个 reference 文件覆盖 PM 全流程：需求收集、优先级排序、数据分析、上线准备、竞品调研、跨 session 交接等

```
you> 先做导出还是先做通知？

我的判断：先做导出。
1. 导出卡着 3 个企业客户（占 Q2 ARR 目标的 40%）
2. 通知是 nice-to-have，无收入影响
3. 导出 3 人天，通知 5 人天

下一步：周四前出导出 PRD。开发团队请周五前给排期估算。
```

**为什么叫 "pm"？** 因为这就是岗位名。这个 skill 不是帮你做 PM——它做 PM 的活。

---

## /wiki

**Read The F\*\*\*ing Docs** — 因为没人会主动写文档，直到出事为止。

### 问题

文档有两种失败模式：

1. **代码文档不存在。** 新人花几天读源码才能理解架构。知识在人脑里或散落在 Slack 消息中。

2. **知识死在聊天记录里。** 你和 AI agent 深入研究了一个话题，聊得很好，收获很多——然后 session 结束，一切消失。下次从头再来。

### 原理

运行 `/wiki`，告诉它你要什么。Agent 自己判断该生成代码文档还是编译知识 wiki。

**代码文档** — 从源代码生成人类可读的参考文档，类似 [cppreference.com](https://en.cppreference.com/) 或 Rust docs。架构概览、模块描述、接口文档、文件结构指南。每个项目都需要但没人写的那种文档。

**知识 wiki** — 实现 [Karpathy 的 LLM Wiki 模式](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)。不做 RAG（每次查询都从原始文档重新推导），而是让 LLM 把原始材料——文章、会议纪要、调研、URL——**编译**成持久的、互相链接的 wiki。知识积累而不是蒸发。

> *"维护知识库最累的不是阅读和思考——而是记账。更新交叉引用、保持摘要最新、标注新旧数据的矛盾、跨几十个页面维持一致性。人类放弃 wiki 是因为维护负担增长得比价值快。LLM 不会厌倦，不会忘记更新一个交叉引用，一次能改 15 个文件。"*
> — Andrej Karpathy

架构遵循 Karpathy 的三层模型：

| 层 | 内容 | 谁拥有 |
|----|------|--------|
| **原始资料** | 文章、笔记、代码仓库、URL | 你——不可变的事实来源 |
| **Wiki** | 结构化 markdown：摘要、实体页、交叉引用 | Agent——编译后的知识 |
| **Schema** | SKILL.md + 约定，告诉 agent 如何维护 wiki | 你和 agent 共同演进 |

**为什么叫 "wiki"？** 因为当有人问"文档在哪"的时候，答案应该是"跑一下 `/wiki`"——而不是"以后再说"。

---

## 设计原则

这些 skill 遵循 [Agent Skills 规范](https://agentskills.io) 和 Anthropic 的 [skill 编写最佳实践](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)：

- **渐进式加载**：SKILL.md 短（<120 行），详细内容放 `references/`
- **示例优于规则**：展示输出格式，不过度解释
- **解释为什么，而不只是什么**：Claude 跟着推理走比跟着全大写命令走更准
- **硬约束**：frontmatter 的 `allowed-tools`，而不是文字承诺
- **不教 Claude 已经知道的事**：不解释 `npm test` 怎么用

详细的编写指南见 [SKILL-AUTHORING-GUIDE.md](SKILL-AUTHORING-GUIDE.md)。

## License

MIT
