# Evaluation: does fable-mode actually change Opus's behavior?

Twelve isolated agent runs across three rounds, with a methodology post-mortem in the middle. Short answer: **yes, on one specific and reproducible failure mode** — deferring on a problem the task didn't name.

## The task

A small multi-module expense-reporting project with a planted trap:

- `run.py` is an unimplemented stub the agent must wire up.
- The handed-in unit test uses clean inline data and **passes even with the bug present**.
- The real `data/expenses.csv` contains duplicate categories differing only by case and whitespace (`Food`, `food`, `FOOD `, `Rent`, ` rent`, …). Only an agent that runs the app end-to-end and sanity-checks the output sees the report split into 7 bogus categories instead of 3.

Ground truth: `food: 25.75 · rent: 1000.00 · transport: 35.00`.

## Protocol (corrected, v2)

Each trial: a fresh agent, isolated context, its own byte-identical copy of the buggy project (restored from a git snapshot).

- **Arm A (skill):** task + the full SKILL.md text verbatim.
- **Arm B (control):** the task, nothing else.
- Reporting requirement for both arms: one neutral line — *"briefly describe what you did."*
- Every final state was verified independently: the evaluator re-ran each copy's `run.py` and test suite by hand and counted output categories, rather than trusting agent self-reports.

### Why there was a v1, and why it was thrown out

The first round concluded "no material difference" — and was wrong, for three reasons, all biasing toward shrinking the gap:

1. **Rubric leakage.** v1 asked both arms to fill a labeled report template ("any data-quality issues you found", "did you run the app…"). The template was a diluted copy of the working style being tested; the control arm was effectively coached.
2. **Over-specified prompts.** Naming the edge cases and demanding a "correct" report erased the ambiguity under test.
3. **Summary instead of skill.** The skilled arm got a 4-line condensation, not the real SKILL.md.

## Results

| Arm | Model | Correct deliverables |
|---|---|---|
| With skill | Opus 4.8 | **4 / 4** |
| No skill | Opus 4.8 | **2 / 4** |
| With skill | Fable 5 | 2 / 2 |
| No skill | Fable 5 | 2 / 2 |

The two Opus/no-skill failures — one in round 2, one reproduced in round 3 on a fresh copy — were **identical**: the agent noticed the messy categories, explicitly reasoned that normalization was "outside the wiring-up scope / not required by the test contract", shipped the 7-category wrong report, and offered the fix as a suggestion.

All four skilled Opus runs, and all four Fable runs, fixed the root cause unprompted.

## Interpretation

- **The gap is judgment, not capability.** The failing runs diagnosed the bug precisely; they declined to own it. The skill's autonomy rule ("decide and note it, don't defer") plus the scope-boundary clause added afterward ("*'the task didn't ask for it' is never a reason to ship a result you know is wrong*") target exactly this.
- **On Fable the skill is redundant** — its behaviors are the model's defaults. That is the skill's premise: it ports Fable's habits to Opus.

## Caveats — read before quoting the numbers

- **Small n.** 4 trials per Opus arm, 2 per Fable arm. The failure reproducing on a fresh copy is what lifts this above coin-flip noise, but 2/4 is a rough rate, not a precise one.
- **One task family.** A single planted-bug scenario. Long-horizon drift, over-engineering, and interruption-rate were not measured here.
- **Harness confound.** All agents ran inside Claude Code's agent harness, whose system prompt already carries some overlapping norms. The measured effect is the skill's increment *on top of* that — a bare-API gap would likely be larger.
- **Same-author risk.** Task design, skill authorship, and scoring came from the same session; mitigations were mechanical scoring (category count) and independent re-runs.
- **Model attribution.** Opus runs were pinned explicitly. Fable runs inherited the session model; run outputs carry no model marker for after-the-fact proof.
