# fable-mode — make Claude Opus work like Fable

**One-sentence pitch:** a drop-in [Claude Code](https://claude.com/claude-code) skill that gives Claude Opus the work habits of Anthropic's top-tier Fable model — it stops deferring, starts owning problems, and never ships a result it knows is wrong.

**Proof, in one line:** across 58 controlled trials on a bug that unit tests can't catch, Opus **with** this skill shipped correct results **29/29** times; **without** it, **24/29** (Fisher exact p ≈ 0.026). [Full evaluation →](docs/EVALUATION.md)

**Install, in one command:**

```bash
mkdir -p ~/.claude/skills/fable-mode && curl -fsSL \
  https://raw.githubusercontent.com/joe-qiao-ai/fable-mode/main/skills/fable-mode/SKILL.md \
  -o ~/.claude/skills/fable-mode/SKILL.md
```

---

## The problem this fixes

Give Opus a task with a hidden data bug — one the unit tests don't catch, only visible if the agent actually runs the app and reads the output. Here's what happens:

**Without fable-mode** (fails ~1 in 6 runs) — the agent wires up the code, sees the messy data, and decides it's "not in scope":

```text
Food: 12.50        ← same category,
food: 8.00         ← counted three
FOOD : 5.25        ← separate times
Rent: 900.00
 rent: 100.00
transport: 20.00
Transport: 15.00
```

**With fable-mode** (29/29 in our trials) — the agent treats a correctness problem it found as its problem, fixes the root cause, and notes what it did:

```text
food: 25.75
rent: 1000.00
transport: 35.00
```

Same model. Same task. The difference is one judgment call — and that call is exactly what this skill hard-codes:

> *Scope discipline means not adding unrequested features — it does NOT mean shipping a wrong result. "The task didn't ask for it" is never a reason to deliver output you know is wrong.*

## What's inside

One file, eight rules — a working-style contract distilled from Anthropic's official Fable migration guidance, then hardened by testing:

| # | Rule | What it changes |
|---|------|-----------------|
| 1 | Lock the goal | Restates the task with checkable "done" criteria before touching anything |
| 2 | Autonomy | Decides minor choices and notes them; no "Want me to…?" for in-scope work |
| 3 | Delegate deliberately | Parallel subagents for fan-out work; never a subagent for a single grep |
| 4 | Verify before claiming | Every progress claim must trace to a real tool result |
| 5 | Self-verification loop | Runs checks at each milestone; re-reads the original request at the end |
| 6 | Scope, correctly bounded | No gold-plating — but a discovered defect is always in scope to fix |
| 7 | Memory | Reads and writes durable learnings when a memory directory exists |
| 8 | Reporting | Silent while working; final summary leads with the outcome |

## How to use it

**Per session** — type `/fable-mode` at the start of any substantial task.

**As your default** — add one line to `~/.claude/CLAUDE.md`:

```markdown
- At the start of every session, invoke the `fable-mode` skill and apply its working
  style to all substantial tasks. Trivial one-off questions don't need it.
```

**Two settings that matter more than any prompt** — the skill fixes judgment, not model settings:

- Keep effort at **`xhigh`** (Claude Code's default — don't lower it).
- Put the **full spec in your first message**: goal, constraints, what "done" looks like.

## How good is it, honestly

We ran **112 isolated agent trials** across four rounds — including throwing out our own first round for methodology contamination, and a 100-trial mechanically-scored scale-up. Everything is documented, including what *didn't* work:

| Finding | Evidence |
|---|---|
| ✅ Stops Opus shipping known-wrong results | 29/29 vs 24/29 on the trap task, p ≈ 0.026 |
| ✅ Every observed no-skill failure was this exact mode | 5/5 failures: buggy module untouched, wrong output shipped |
| ➖ Doesn't improve normal implementation tasks | Both arms 25/25 on the non-trap family |
| ➖ Doesn't make Opus write tests unprompted | 0/25 in both arms |
| ➖ Redundant on Fable-tier models | Fable: 4/4 with or without |

**Fair summary:** this is a targeted fix for one real, reproducible failure mode — the ~12–17% of runs where unskilled Opus meets a problem nobody named and quietly ships a wrong result. It is not a general quality boost, and we don't claim one. Full methodology, statistics, null results, and the contamination post-mortem: [docs/EVALUATION.md](docs/EVALUATION.md).

## License

[MIT](LICENSE) · Not affiliated with or endorsed by Anthropic. "Claude", "Fable", and "Opus" are Anthropic's model names, referenced descriptively.

---

# 中文说明

## 这是什么

一个 Claude Code skill(单个 Markdown 文件),把 Anthropic 顶级 Fable 模型的工作习惯移植给 Opus:**该拿主意时拿主意、交付前自己验证、绝不交付明知有错的结果**。

## 效果如何(实测,不吹)

在一个单元测试抓不到、只有真跑一遍才暴露的埋雷任务上,58 次对照试验:

- **开启 skill:29/29 全对**
- **不开:29 次错 5 次**(p ≈ 0.026)——每次失败都是同一个模式:发现了问题,以「任务没要求」为由不修,交付错误结果

同时如实说明边界:在普通实现任务上两组无差异;它不会让 Opus 主动写测试;对 Fable 模型冗余。**它是对一种失败模式的定向修复,不是万能加速器。** 完整实验(含 100 次机械评分的规模化验证和方法论复盘)见 [docs/EVALUATION.md](docs/EVALUATION.md)。

## 怎么用(30 秒)

```bash
# 1. 安装(用户级,所有项目生效)
mkdir -p ~/.claude/skills/fable-mode && curl -fsSL \
  https://raw.githubusercontent.com/joe-qiao-ai/fable-mode/main/skills/fable-mode/SKILL.md \
  -o ~/.claude/skills/fable-mode/SKILL.md

# 2. 会话里输入 /fable-mode 即可启用
```

想默认全局生效,在 `~/.claude/CLAUDE.md` 加一行(见上文英文示例)。

**两个比提示词更重要的设置**:effort 保持 `xhigh`(Claude Code 默认值,别调低);需求在第一句话说全(目标、约束、怎么算完成)。
