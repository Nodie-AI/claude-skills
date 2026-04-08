# Claude Code Skill 编写指南

> How to write high-quality SKILL.md files — lessons from official docs, the anthropics/skills repo, and our own mistakes.

## 来源

- [Skill Authoring Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) — 官方最完整参考
- [Extend Claude with Skills](https://code.claude.com/docs/en/skills) — Claude Code 特有功能
- [anthropics/skills repo](https://github.com/anthropics/skills) — 官方示例
- [Equipping Agents for the Real World](https://claude.com/blog/equipping-agents-for-the-real-world-with-agent-skills) — 设计哲学
- 我们自己的 `/verify` skill 踩坑经验 (2026-04-08)

---

## 1. Description 是一切

Description 是 Claude 决定是否加载 skill 的唯一依据。从 100+ skills 中被选中，靠的就是这一行。

**规则：**
- 第三人称（"Processes PDF files"，不是"I can help you"）
- 前置核心用途，250 字符后会被截断
- 包含用户会自然说出的关键词
- 写得稍微"pushy"——Claude 倾向于 under-trigger（该用时不用）
- 同时说明 **做什么** 和 **什么时候用**

```
# BAD
"Helps with verification"

# GOOD  
"Verify agent task completion — check acceptance criteria against code, 
run regression tests, report pass/fail status. Use after completing a 
feature or before committing."
```

## 2. 三层渐进式加载

这是核心架构原则：

| 层级 | 内容 | Token 消耗 | 何时加载 |
|------|------|-----------|---------|
| Metadata | name + description | ~100 tokens，始终在 context | 始终 |
| SKILL.md body | 主要指令 | 触发时加载，**控制在 500 行以内** | 用户调用或 Claude 自动触发 |
| references/ | 详细文档、模板、例子 | 按需加载，无上限 | SKILL.md 中引用时 |

关键含义：
- SKILL.md 是**目录**，不是百科全书
- 详细内容放 `references/` 下，一层深度，不要嵌套
- 脚本放 `scripts/`，执行而非加载（零 token 直到输出返回）

```
my-skill/
├── SKILL.md              # 短，<500 行
├── references/
│   ├── report-format.md   # 详细报告模板
│   └── regression.md      # 回归检查细节
└── scripts/
    └── check.sh           # 可执行脚本
```

## 3. 解释 Why，不要喊 MUST

Claude 有很好的理解力。与其用全大写命令，不如解释原因。

```
# BAD — 黄旗信号
You MUST NEVER write to source code files. ALWAYS run tests first. 
CRITICAL: do NOT skip regression checks.

# GOOD — 解释原因
Only write to test files and .ai-verify/ directory. Writing source code 
would create untested changes and hide the gap between "agent says done" 
and "actually done" — defeating the purpose of verification.
```

**何时用强指令：** 只在后果严重且 Claude 很可能忽略时（比如删除数据库）。日常行为用解释引导。

## 4. 不要解释 Claude 已经知道的东西

Claude 知道怎么跑 `npm test`、怎么读代码、什么是 pass/fail。不需要教它。

```
# BAD — 浪费 token
To check if tests pass, run `npm test` in the terminal. 
This command executes the test suite. If the exit code is 0, 
tests passed. If non-zero, some tests failed.

# GOOD — 只说 Claude 不知道的
Run the project's test suite. If no test command is obvious, 
check package.json scripts or look for pytest.ini/go.mod.
```

问自己："Claude 真的需要这个解释吗？" 如果答案是否，删掉。

## 5. 用 frontmatter 做硬约束

Prose 说"不要写文件"是软约束，容易被忽略。Frontmatter 是硬约束。

```yaml
---
name: verify
description: ...
allowed-tools: Read, Grep, Glob, Bash, Agent  # 没有 Write 和 Edit
---
```

| Frontmatter | 作用 |
|-------------|------|
| `allowed-tools` | 限制可用工具（最重要的硬约束） |
| `disable-model-invocation: true` | 只能用户手动调用 |
| `context: fork` | 在隔离子 agent 中运行，不污染主对话 |
| `model` | 指定模型（轻量任务用 haiku 省成本） |

## 6. 给例子，不给规则

Examples > rules。Claude 跟例子走得最准。

```
# BAD — 抽象规则
Generate acceptance criteria for the current task. 
Each criterion should be concrete and verifiable.

# GOOD — 具体例子
Example output:

Verification: Add user registration
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  ✓ POST /register returns 201 with user object
    └ tests/auth.test.ts:15 — test passes

  ✗ Passwords hashed with bcrypt
    └ src/models/user.ts:12 — plaintext storage found

  ○ Verification email sent
    └ needs manual check — no email service in test env

Progress: ●●○ 1/3
```

## 7. 匹配自由度与脆弱度

| 脆弱度 | 自由度 | 指令方式 | 例子 |
|--------|--------|---------|------|
| 高（错了后果严重） | 低 | 精确脚本 | 数据库迁移 |
| 中 | 中 | 伪代码/模板 | 报告格式 |
| 低（多种做法都行） | 高 | 文字方向 | 如何检查代码质量 |

比喻：悬崖边的窄桥 = 精确指令；开阔草地 = 给个方向就行。

## 8. 测试方法

### Two-Claude 模式

1. Claude A 写 skill
2. **新开一个 session**，Claude B（从未见过 skill 源码）加载并使用
3. 观察 Claude B 是否按预期行为
4. 回到 Claude A 修改

### 评估驱动开发

1. 先**不用** skill 跑几个代表性任务，记录失败
2. 针对失败写 3+ 测试场景
3. 写最小 skill 内容解决这些场景
4. 迭代

### 跨模型测试

Opus 上 work 的 skill 在 Haiku 上可能需要更多细节。确保在你计划使用的所有模型上测试。

## 9. 常见错误清单

| 错误 | 后果 | 修复 |
|------|------|------|
| 模糊 description | Claude 不会触发 | 写具体、包含关键词 |
| 给太多选项 | Claude 选择困难 | 给一个默认，附逃生口 |
| 深层嵌套引用 | Claude 只读 head -100 | 保持一层深度 |
| 过度解释 | 浪费 token，降低信噪比 | 删除 Claude 已知的内容 |
| 术语不一致 | Claude 困惑 | 全文统一术语 |
| 假设工具已安装 | 运行时失败 | 显式声明依赖 |
| 用 prose 做约束 | 容易被忽略 | 用 frontmatter allowed-tools |
| 没有测试 | 首次使用即生产 | Two-Claude 模式 |

## 10. 我们的踩坑经验

### /verify skill v1 问题

1. **Verifier 变成了 implementer** — skill 没有限制写文件权限，结果 verify agent 直接改了 `index.html` 的源代码，还改坏了。应该用 `allowed-tools` 去掉 Write/Edit。

2. **没有 regression 检查** — 改了文件后旧功能（tab 切换）坏了，verify 完全没发现。因为 criteria 只针对新任务，没有自动扫描已有功能。

3. **假 pass** — 用 "code inspection" 判断 UI 审美达标，实际上从没在浏览器里打开过。"I read the code and it looks right" 不是有效证据。

4. **200+ 行全塞在 SKILL.md** — 没有 references/，token 全部前置加载。应该只留核心流程，细节放子文件。

5. **全大写 CRITICAL/NEVER 满天飞** — 不仅没起到约束作用，反而降低了整体信噪比。

### 教训

> Skill 的质量不取决于写了多少规则，而取决于 Claude 在实际运行时是否做对了事。规则写得再多，如果没有硬约束（allowed-tools）和真实测试（Two-Claude），就是纸上谈兵。
