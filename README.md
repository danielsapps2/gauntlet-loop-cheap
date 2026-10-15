# gauntlet-loop-cheap

A [Claude Code](https://claude.com/claude-code) skill that turns any goal into one short, paste-ready **gauntlet loop** prompt, tuned to get high-quality output for a low usage cost.

The standard gauntlet loop gets quality by brute force: every planner, builder and critic runs on the top model, with no spending limit. This version keeps the same quality bar but spends the expensive model only where judgement matters. The volume work goes to cheaper models, and a built-in budget makes a long run fit inside a fixed plan.

On a real multi-day run, this setup used about **half the usage** of the all-Opus version while matching or beating its quality per round.

## What a gauntlet loop does

A gauntlet loop makes an agent:

1. set a concrete quality bar, a real named thing it can fetch and compare against,
2. split the work into small pieces that can be judged on their own,
3. run a builder, a separate harsh critic and a separate blind judge on each piece,
4. have the critic set ours next to the bar as a **blind** A/B pair, labels stripped, and measure the gaps,
5. have the judge, who sees only that pair, pick a winner and name the single biggest gap,
6. loop until the judge picks ours.

The bar is the whole trick. A good bar is **named** ("Stripe's pricing page", not "award-winning SaaS sites"), **fetchable** (the critic can actually screenshot, read, run or open it) and **comparable** (the two can sit side by side and a judge can pick one).

## How this differs from the standard gauntlet loop

| | Standard gauntlet loop | gauntlet-loop-cheap |
|---|---|---|
| Builders | Opus | **Sonnet**, one small pre-planned task each |
| Critics | Opus; the critic also judges | **Sonnet**; captures, builds the blind A/B folders, measures gaps |
| Judges | (none; the critic judges) | **Opus**, separate, sees only the blind pair |
| Planners | Opus | Opus |
| Lead / orchestrator | Opus | Opus |
| Commands, builds, screenshots | Haiku | Haiku |
| Budget | none unless you ask | built in: concurrency cap, daily cap, 5-hour-window guard |
| Builder briefs | open-ended | measured targets taken from the bar, plus "make measured changes" |
| Keeping rounds | every round | **ratchet**: keep a round only if it beats the best so far |
| Lead's job | orchestrate | orchestrate, plus check every blind folder before judging |

### Why each change saves money without losing quality

- **Sonnet builders.** Building is most of the token volume. On a real run, Sonnet builders averaged about 190k tokens and 17 minutes per round, against about 356k and 27 minutes for Opus builders.
- **Sonnet critics, Opus judges.** The critic's work (build, capture, crop, mask, measure) is mechanical and checkable, so it runs on Sonnet. The verdict needs taste, so a separate Opus judge makes it. The judge only reads a small blind folder: about 100k tokens and under a minute. Splitting the roles also makes judging fairer, because the critic knows which side is ours and the judge doesn't.
- **Opus only where mistakes go unchecked.** The lead (merges, the blind key, the budget) and the judges stay on Opus. A cheap builder's mistake gets caught by the judge; a lead's or judge's mistake doesn't.
- **Measured targets.** Cheaper builders tend to overshoot the last critique: too big, then too small, then too big again. Giving them numbers from the bar ("score digits 2.0% of screen height") instead of directions ("make it bigger") stopped the swinging, so fewer rounds are wasted.
- **Trust the critic's numbers.** Builders over-report their own measurements. Deciding on the critic's numbers stops bad rounds from looking good.
- **Ratchet.** Merging only rounds that beat the best so far means a cheap regression never costs you the progress you already paid for.
- **Folder checks.** The lead looks at each blind folder before the judge does, so a sloppy crop from a cheap critic doesn't waste an Opus verdict.

### Measured on a real run

| | All-Opus setup | gauntlet-loop-cheap |
|---|---|---|
| Builder, per round | ~356k tokens, ~27 min | ~190k tokens, ~17 min |
| Critic, per round | ~175k tokens, 5–14 min | ~150k tokens, ~4 min |
| 4 parallel builders, 5-hour window | ~35–40% per hour | ~21% per hour |
| Verdict margins | baseline | matched or improved |

## Install

Clone into your Claude Code skills folder:

```sh
git clone https://github.com/danielsapps2/gauntlet-loop-cheap.git ~/.claude/skills/gauntlet-loop-cheap
```

Or for a single project, clone into `.claude/skills/gauntlet-loop-cheap` inside that repo.

Restart Claude Code (or start a new session) and the skill is available.

## Usage

Trigger it with any of:

- `/gauntlet-loop-cheap <goal>`
- "gauntlet this: …"
- "make a gauntlet prompt for …"
- "loop until it beats X"

**Flow:**

1. You give a goal.
2. If you didn't name a reference, it offers 2–3 candidate bars and waits for your pick.
3. It writes one paste-ready prompt (about 230–290 words).
4. Paste it into a fresh session, or say "run it" and Claude becomes the lead agent and runs it right there.

### Example

> **You:** gauntlet this: a landing page for my running brand, athletic, green and dark, has to feel alive
>
> **Claude:** A) Nike's current running campaign page B) On Running's homepage C) Gymshark's product landing page
>
> **You:** A

```
Build a landing page for a running brand. Athletic, peak performance, green and dark, energetic, aimed at a young healthy audience. It needs to be interactive and visually unmistakable.

The bar is Nike's current running campaign page. Screenshot it at desktop and mobile and compare against those directly, not against a description of them.

Break this into the smallest pieces that can be improved and judged on their own - hero, motion, type, colour, imagery, interaction, mobile. For each piece, a planner cuts it into small tasks and each builder implements one quickly, without planning. A separate critic with fresh context screenshots ours and Nike's at the same viewport, strips the labels, and measures the gaps. A separate judge sees only that blind pair, picks the better one and names the biggest remaining gap. Then it goes back to the builder.

Critic and judge are harsh; praise is useless. Give builders measured targets from the bar, not "bigger" or "lighter". Trust the critic's numbers over the builder's, and keep a round only if it beats the best so far.

/loop on each piece until the judge picks ours blind. Do not stop before that.

Keep a live progress page updating as the work evolves so I can watch it.

This run has to last 7 days on a Claude Max 5x plan. Keep at most 4 subagents alive at once, with no more than 1 of them on Opus. Check usage before every round and stop launching above 70% of the 5-hour window. Spend no more than 12% of the weekly limit in a day. When today's share is gone, schedule a wakeup for tomorrow and pause instead of pushing through.

Fan out subagents. You orchestrate on Opus 5.5. Planners and judges run on Opus 5.5 at medium effort. Builders and critics run on Sonnet 5.5 at medium effort. Quick commands and terminal work run on Haiku 5.5 at high effort.
```

## The budget

Every prompt carries a default budget sized so a Claude Max 5x plan lasts 7 days with room left for your own work:

- No more than 4 subagents alive at once, and only 1 of them on Opus.
- Check usage before every round, and stop launching new agents above 70% of the 5-hour window. Hitting 100% kills running agents mid-work.
- No more than 12% of the weekly limit spent per day (84% over the week, about 15% left for you).
- When the day's share is used up, the loop pauses until the next day instead of draining the week.

If you name a different plan, run length or cap, those values replace the defaults. You can also set a round cap per piece (for example 6 rounds); when it's hit, the loop logs the last gap and moves on.

## Running it yourself

When Claude runs the loop for you, it follows a **lead playbook** in `SKILL.md`, built from a multi-day run with dozens of rounds. Besides the points above, it covers:

- keeping the A/B key secret and alternating which letter is ours,
- running confirmation judges only when ours wins,
- asking you when the bar conflicts with your own design intent (for example a stylised character judged against realistic ones),
- telling builders to commit work-in-progress often, and never clearing the lead's context while subagents run, since both a usage cap and a context clear kill running agents,
- short final replies from subagents, so the lead's context lasts a long run,
- checking the actual work, not just each agent's report.

## Using it outside Claude Code

`/loop` is a Claude Code feature. For other agents, replace the `/loop` line with:

> Keep looping until the judge picks ours. Run the builders, critics and judges as parallel subagents.

Keep the routing line if the agent can choose models per subagent, drop it if not. Everything else carries over.

## What breaks a gauntlet loop

- **A vague bar.** The critic invents a comparison and approves everything.
- **The builder judging its own work.** The critic must be a separate agent with fresh context.
- **A judge who knows which side is ours.** Use a separate judge that sees only the blind pair.
- **A soft judge.** Scores out of 10 drift upward; a binary "which is better, A or B" doesn't.
- **Directional briefs.** "Bigger" and "lighter" make builders overshoot; give measured targets.
- **Believing the builder's numbers.** Decide on the critic's measurements.
- **Merging every round.** Keep only rounds that beat the best so far.
- **Unbounded fan-out.** Ten Opus agents in parallel can spend a week's allowance in an afternoon.
- **Over-specifying.** Every extra instruction is one fewer decision the agent makes with its own judgement.

See [`SKILL.md`](SKILL.md) for the full skill.
