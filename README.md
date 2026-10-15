# gauntlet-loop

A [Claude Code](https://claude.com/claude-code) skill that turns any goal into one short, paste-ready **gauntlet loop** prompt.

A gauntlet loop makes an agent:

1. set a concrete quality bar — a real, named thing it can fetch and compare against,
2. split the work into small pieces that can be judged on their own,
3. run a builder, a separate harsh critic (fresh context) and a separate blind judge on each piece,
4. have the critic capture ours next to the bar as a **blind** A/B pair with labels stripped and measure the gaps,
5. have the judge, who sees only that pair, pick a winner and name the single biggest gap,
6. loop until the judge picks ours, keeping only rounds that beat the best so far.

It works for builds, websites, writing, code, research, decks and design.

## Why

Most "make it great" prompts fail the same way: the agent grades its own work against a vague idea of quality and declares victory on round one. A gauntlet loop fixes that by forcing a real reference, a separate critic, and a binary A/B judgement, with no exit until ours wins.

The bar is the whole trick. A good bar is:

- **Named** — "Stripe's pricing page", not "award-winning SaaS sites".
- **Fetchable** — the critic can actually screenshot, read, run or open it.
- **Comparable** — the two can sit side by side and a judge can pick one.

## Install

Clone into your Claude Code skills folder:

```sh
git clone https://github.com/danielsapps2/gauntlet-loop-cheap.git ~/.claude/skills/gauntlet-loop
```

Or for a single project, clone into `.claude/skills/gauntlet-loop` inside that repo.

Restart Claude Code (or start a new session) and the skill is available.

## Usage

Trigger it with any of:

- `/gauntlet-loop <goal>`
- "gauntlet this: …"
- "make a gauntlet prompt for …"
- "loop until it beats X"

**Flow:**

1. You give a goal.
2. If you didn't name a reference, it offers 2–3 candidate bars and waits for your pick.
3. It writes one paste-ready prompt (~230–290 words).
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

## Model routing (the "cheap" part)

Every generated prompt ends with the same routing line, so the loop stays affordable while it grinds:

| Role | Model |
|---|---|
| Lead agent / orchestration (merges, blind key, budget guard) | Opus 5.5 |
| Planners and judges (blind A/B verdict) | Opus 5.5 at **medium** effort |
| Builders (one small task each, no planning) | Sonnet 5.5 at **medium** effort |
| Critics (capture, blind A/B folders, measurements) | Sonnet 5.5 at **medium** effort |
| Commands and terminal work (builds, tests, screenshots, fetches) | Haiku 5.5 at **high** effort |

The loop fans out with plain subagents rather than Workflow orchestration.

On a real multi-day run, moving builders and critics to Sonnet roughly halved usage (about 21% of the 5-hour window per hour with four parallel builders, against 35–40% with Opus builders and critics) without losing quality. Sonnet builders averaged ~190k tokens and 17 minutes per round against ~356k and 27 minutes on Opus. The lead and judges stay on Opus, because their mistakes are the ones nothing downstream catches.

## Budget

Every prompt also carries a default budget sized so a Claude Max 5x plan lasts 7 days:

- No more than 4 subagents alive at once, and only 1 of them on Opus.
- Check usage before every round, and stop launching new agents above 70% of the 5-hour window (hitting 100% kills running agents).
- No more than 12% of the weekly limit spent per day.
- When the day's share is used up, the loop pauses until the next day.

If you name a different plan, run length or cap, those values replace the defaults.

## Lessons from real runs

`SKILL.md` has a **lead playbook** the agent follows when it runs the loop itself. The short version:

- **Split critic and judge.** The critic builds the blind folders, so it knows which side is ours; a fresh judge sees only the folder.
- **Check every blind folder before judging.** Same crops and treatment on both sides, nothing that gives the source away. Send the critic back if it's unfair.
- **Measured targets, not directions.** "Bigger" or "lighter" makes builders overshoot round after round; "score digits 2.0% of screen height" doesn't.
- **Trust the critic's numbers.** Builders over-report their own measurements.
- **Ratchet.** Merge a round only if it beats the best so far; restart from the best after a regression.
- **Protect running agents.** Builders commit work-in-progress often. Never clear the lead's context while subagents run.

## Using it outside Claude Code

`/loop` is a Claude Code feature. For other agents, replace the `/loop` line with:

> Keep looping until the judge picks ours. Run the builders, critics and judges as parallel subagents.

Keep the routing line if the agent can choose models per subagent, drop it if not. Everything else carries over.

## What breaks a gauntlet loop

- **A vague bar** — the critic invents a comparison and approves everything.
- **The builder judging its own work** — the critic must be a separate agent with fresh context.
- **A judge who knows which side is ours** — use a separate judge that sees only the blind pair.
- **A soft judge** — scores out of 10 drift upward; a binary "which is better, A or B" doesn't.
- **Directional briefs** — "bigger" and "lighter" make builders overshoot; give measured targets.
- **Merging every round** — keep only rounds that beat the best so far.
- **An exit after N rounds** — the exit is winning the comparison, or you stopping it.
- **Over-specifying** — every extra instruction is one fewer decision the agent makes with its own judgement.

See [`SKILL.md`](SKILL.md) for the full skill.
