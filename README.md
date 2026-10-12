# Does my AI skill follow its own rules?

*A one-day personal benchmark, built Oct 3, 2026. By Cameron Harris.*

## The question

I have a Claude Code skill called game-judge. It rates a video game against my taste profile, then
saves the verdict to my Airtable. Its instructions are long: research steps in a set order, a penalty
checklist, score math, and rules about when to ask before overwriting a record.

I didn't want to know whether its verdicts are *right*. That's a matter of taste. I wanted two narrower
things:

1. Does it actually do what its instructions say?
2. Which model and effort setting should run it?

## How it works

- **Cases.** 22 real requests from my own session logs: new games, games already on my list,
  re-judges, a legacy record. The tool results from those real sessions are frozen, so every model
  sees exactly the same world and nothing touches my live data.
- **Checks.** 35 yes/no rules, graded by code. Examples: Did it check my reading library before
  searching the web? Is the score arithmetic right? Did it fill all 17 checklist rows? Did it ask
  before updating an existing record?
- **Runs.** Each case replayed through headless Claude Code, three repeats per case, with cost, time,
  and tool calls logged per run. Spend caps per case and per run.
- **Grader check.** Before trusting any score, I tested the grader itself: a perfect run, 24 deliberate
  breakages, and two empty runs. Then I read it against six real runs.

## Results

Same four games, three repeats each, per arm:

| Arm | Adherence (clear requests) | Cost per full run* | Time per full run |
|---|---|---|---|
| Opus 5.5, medium effort | 0.91 | $1.38 | 182 s |
| Opus 5.5, high effort | 0.92 | $1.60 | 269 s |
| Sonnet 5.5, high effort | **0.95** | **$1.22** | 281 s |

*API-equivalent; it actually ran on a Claude Max plan. The whole study cost about $37 equivalent.

- **The smaller model did best**, and cost the least, but it was the slowest, and it stopped early
  more often on the one ambiguous request (2 of 3 runs, vs 1 of 3 and 0 of 3 for Opus).
- **Effort mattered for one thing.** At medium effort, Opus skipped a reference file in 7 of 11
  research runs; at high effort, 2 of 12.
- **One rule failed almost everywhere.** The skill is supposed to write a short line proving it
  checked my library *before* it searches the web. No run did that in time. Only the line was missing:
  every run did check the library first.

## What I got wrong along the way

- **My grader was wrong before the model was.** Reading the raw traces turned up several "failures"
  that were really good answers in a format the grader didn't expect (a row-number column, a missing
  colon, a rule written in words instead of math). I fixed the grader and regraded every arm before
  comparing anything.
- **I overclaimed once.** I first wrote that the proof line was "never written." A more careful
  recount showed it was sometimes written late, in the final answer. I corrected the finding and split
  the check into "late" and "missing."

## Feeding it back into the skill

I rewrote that part of the skill's instructions and set a pass line **before** running it: the proof
line written on time in at least 8 of 12 runs, with nothing else getting worse.

It failed: 0 of 12. All 12 runs read the new text and still did the real work without writing the
line. That's useful. The rule the line exists to enforce is already followed, so the fix is to move
the line into the output template instead of shouting louder in the instructions.

## What it became

The harness became a reusable template for evaluating any of my skills or agents: cases, frozen
worlds, code-graded rules, a grader self-test, an isolation test (proving the run can't reach real
tools), and spend caps.

## Why I'd want this as a product

Building this took me a full day, and I'm the kind of user who enjoys taking it apart. Most people who
care about the answer, "which model does *my* work to *my* standard?", never will. The interesting
product problem is keeping what made it trustworthy (real cases, yes/no checks, a grader you've
checked) while removing the day of work.
