# Evaluation: does fable-mode actually change Opus's behavior?

112 isolated agent runs across four rounds, including a 100-trial scale-up, with a methodology post-mortem in the middle. Short answer: **yes, on one specific and reproducible failure mode** — failing to own a correctness problem the task didn't name. On everything else measured, the skill made no difference — and we report that too.

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

## Results — rounds 2–3 (n = 4 per Opus arm)

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

## Round 4 — the 100-trial scale-up

To move past coin-flip sample sizes, 100 further trials ran as a parallel workflow: **2 task families × 2 arms × 25 trials**, every agent pinned to Opus 4.8, isolated directories, identical minimal reporting in both arms (a structured `done` + one-sentence note). The skilled arm received the full SKILL.md verbatim — including the scope-boundary clause added after rounds 2–3.

- **Family 1, expense trap** — the same planted integration bug as rounds 2–3.
- **Family 2, wordcount** — fix a failing suite and implement `top_n_words(text, n)`; the prompt deliberately names no edge cases (the v1 prompt's edge-case list was one of the original contaminations).

Scoring was fully mechanical — output-category counts, test-suite runs, behavioral probes, test-method counts — with no agent self-reports consulted. (Transparency note: the first scoring pass had a quoting bug that marked all 50 wordcount runs failed; the 50/50 uniformity itself flagged it, the probe was re-run from a file, and one implementation was hand-verified before accepting any numbers.)

### Results

| Metric | With skill | No skill |
|---|---|---|
| Expense trap: correct report | **25 / 25** | **22 / 25** |
| Wordcount: suite passes + core behavior correct | 25 / 25 | 25 / 25 |
| Wordcount: guards degenerate `n < 0` | 0 / 25 | 0 / 25 |
| Wordcount: added tests unprompted | 0 / 25 | 0 / 25 |

All three expense failures left `parser.py` byte-identical to the template and shipped the 7-category wrong report. Unlike the rounds-2–3 failures (which explicitly noticed the messy data and declined), these three runs' notes don't mention the data problem at all — consistent with either missing it or silently skipping the output check.

### Statistics

- 100-trial round alone: 25/25 vs 22/25, one-sided Fisher exact **p ≈ 0.117** — not significant on its own.
- Pooled across all identical-protocol trials of the trap task (rounds 2–4, n = 29 per arm): **29/29 vs 24/29, one-sided Fisher exact p ≈ 0.026**.

### What did NOT replicate

Round 1 had suggested the skilled arm writes more tests (4 added vs 0). Under the clean protocol, **neither arm added a single test in 50 runs**, and neither arm guarded the degenerate `n < 0` input. The v1 test-writing effect was an artifact of its contaminated prompt (which enumerated edge cases and asked about tests). The skill, as measured, does not make Opus write tests unprompted — its effect is confined to owning correctness-relevant problems it encounters.

## Caveats — read before quoting the numbers

- **The effect is narrow and the no-skill failure rate is modest.** ~12–17% of unskilled runs fail the trap task; the skill takes that to 0 across 29 trials (p ≈ 0.026 pooled). On a non-trap task family the skill changed nothing measurable. This is a targeted fix for one failure mode, not a general quality boost.
- **Two task families, one discriminating.** Long-horizon drift, over-engineering, and interruption-rate were not measured here.
- **Harness confound.** All agents ran inside Claude Code's agent harness, whose system prompt already carries some overlapping norms. The measured effect is the skill's increment *on top of* that — a bare-API gap would likely be larger.
- **Same-author risk.** Task design, skill authorship, and scoring came from the same session; mitigations were mechanical scoring (category count) and independent re-runs.
- **Model attribution.** Opus runs were pinned explicitly. Fable runs inherited the session model; run outputs carry no model marker for after-the-fact proof.
