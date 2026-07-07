# fable-mode

A [Claude Code](https://claude.com/claude-code) skill that carries the working habits of Anthropic's Fable-tier model over to Claude Opus — so Opus decides instead of deferring, verifies instead of assuming, and ships instead of asking.

> **Measured result (58 trials on the discriminating task):** on a planted bug that unit tests can't catch, Opus 4.8 **with** this skill shipped a correct result **29/29** times; **without** it, **24/29** (one-sided Fisher exact p ≈ 0.026). Every failure shipped a report the agent could have caught by checking its own output. On a second, non-trap task family both arms scored 25/25 — the effect is a targeted fix for one ownership failure mode, not a general quality boost. Fable 5 needed no skill (4/4 either way). Full methodology, statistics, and null results: [docs/EVALUATION.md](docs/EVALUATION.md).

## What it does

`fable-mode` is a working-style contract in eight rules, distilled from Anthropic's official Fable migration guidance and hardened by A/B testing:

1. **Lock the goal** — restate the task with checkable "done" criteria before acting; ask at most one batched clarifying question.
2. **Autonomy** — decide minor choices and note them; never end a turn on "Want me to…?" for in-scope work.
3. **Delegate deliberately** — parallel subagents for fan-out work; never a subagent for a single grep.
4. **Verify before claiming** — every progress claim must trace to a tool result from this session.
5. **Self-verification loop** — run checks at each milestone, then re-read the original request as if reviewing someone else's work.
6. **Scope discipline, correctly bounded** — no unrequested features, but *"the task didn't ask for it" is never a reason to ship a result you know is wrong*. (This clause exists because both measured failures used exactly that excuse.)
7. **Memory** — read and write durable learnings when a memory directory exists.
8. **Reporting** — near-silence between tool calls; a final summary written for a reader who saw none of the work, outcome first.

## Install

```bash
# user-level: applies to every project
mkdir -p ~/.claude/skills/fable-mode
curl -fsSL https://raw.githubusercontent.com/joe-qiao-ai/fable-mode/main/skills/fable-mode/SKILL.md \
  -o ~/.claude/skills/fable-mode/SKILL.md
```

Or clone and copy:

```bash
git clone https://github.com/joe-qiao-ai/fable-mode.git
cp -r fable-mode/skills/fable-mode ~/.claude/skills/
```

Then either invoke it per-session with `/fable-mode`, or make it the default for all sessions by adding one line to `~/.claude/CLAUDE.md`:

```markdown
- At the start of every session, invoke the `fable-mode` skill and apply its working style to all substantial tasks (coding, research, multi-step agentic work). Trivial one-off questions don't need it.
```

## Pair it with the right settings

The skill changes *judgment*, not model settings. Two levers matter more than any prompt:

- **Keep effort at `xhigh`** (Claude Code's default) — the single biggest quality lever on Opus.
- **Front-load the spec** — Opus performs far better with goal, constraints, and acceptance criteria stated in the first message than with requirements drip-fed across turns.

## What to expect (honestly)

- The skill closes a **judgment gap**, not a capability gap. In our tests the failing runs diagnosed the bug perfectly — they just declined to act. That's what the skill fixes.
- On Fable-tier models it is **redundant but harmless** (4/4 with or without).
- Evidence is from small controlled runs (n = 4 per Opus arm, one task family). Directional, replicated once, not a benchmark. Full methodology, contamination post-mortem, and caveats: [docs/EVALUATION.md](docs/EVALUATION.md).

## License

[MIT](LICENSE)

*Not affiliated with or endorsed by Anthropic. "Claude", "Fable", and "Opus" are Anthropic's model names, referenced here descriptively.*

---

## 中文说明

`fable-mode` 是一个 Claude Code skill,把 Fable 级模型的工作习惯("自己拿主意、先验证再汇报、做完再收工")显式移植给 Opus。

**实测效果**(判别任务共 58 次试验):在一个单元测试抓不到、只有跑真实数据才暴露的埋雷任务上,Opus 4.8 开启此 skill 后 29/29 全对;不开则 29 次里错 5 次(单侧 Fisher 精确检验 p ≈ 0.026)。每次失败都交付了一份自己本可以查出来的错误报表。在第二个非埋雷任务族上两组均 25/25——此 skill 是对「该拿主意时不拿」这一种失败模式的定向修复,不是全面提质。Fable 5 无论开关均 4/4(它默认就这么干活)。完整方法、统计和无效结果见 [docs/EVALUATION.md](docs/EVALUATION.md)。

**安装**:把 `skills/fable-mode/` 复制到 `~/.claude/skills/`,会话里用 `/fable-mode` 调用;想全局默认生效,在 `~/.claude/CLAUDE.md` 加一行(见上文英文示例)。

**搭配建议**:effort 保持 `xhigh`;需求在第一句话说全。这两个杠杆比任何提示词都重要。

完整实验方法与保留意见见 [docs/EVALUATION.md](docs/EVALUATION.md)。
