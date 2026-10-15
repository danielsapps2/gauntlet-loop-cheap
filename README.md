# gauntlet-loop

A [Claude Code](https://claude.com/claude-code) skill that turns any goal into one short, paste-ready **gauntlet loop** prompt.

A gauntlet loop makes an agent:

1. set a concrete quality bar — a real, named thing it can fetch and compare against,
2. split the work into small pieces that can be judged on their own,
3. run a builder and a separate, harsh critic (fresh context) on each piece,
4. have the critic compare ours against the bar **blind**, labels stripped, and name the single biggest gap,
5. loop until the critic picks ours.

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
3. It writes one paste-ready prompt (~120–180 words).
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

Break this into the smallest pieces that can be improved and judged on their own - hero, motion, type, colour, imagery, interaction, mobile. For each piece, fan out a builder and a separate critic with fresh context. The critic opens the real page in a browser, puts our screenshot next to Nike's blind with the labels stripped, says which is better, and names the single biggest remaining gap. Then it goes back to the builder.

The critic should be a harsh critic. Praise is not useful. If ours does not win, it keeps going.

/loop on each piece until the critic picks ours blind. Do not stop before that.

Keep a live progress page updating as the work evolves so I can watch it.

Fan out subagents. You orchestrate on Opus 5.5. Planners, builders and critics run on Opus 5.5 at medium effort. Quick commands and terminal work run on Haiku 5.5 at high effort.
```

## Model routing (the "cheap" part)

Every generated prompt ends with the same routing line, so the loop stays affordable while it grinds:

| Role | Model |
|---|---|
| Lead agent / orchestration | Opus 5.5 |
| Planners, builders, critics | Opus 5.5 at **medium** effort |
| Commands and terminal work (builds, tests, screenshots, fetches) | Haiku 5.5 at **high** effort |

The loop fans out with plain subagents rather than Workflow orchestration. No budget cap is added unless you name one.

## Using it outside Claude Code

`/loop` is a Claude Code feature. For other agents, replace the `/loop` line with:

> Keep looping until the critic picks ours. Run the builders and critics as parallel subagents.

Keep the routing line if the agent can choose models per subagent, drop it if not. Everything else carries over.

## What breaks a gauntlet loop

- **A vague bar** — the critic invents a comparison and approves everything.
- **The builder judging its own work** — the critic must be a separate agent with fresh context.
- **A soft critic** — scores out of 10 drift upward; a binary "which is better, A or B" doesn't.
- **An exit after N rounds** — the exit is winning the comparison, or you stopping it.
- **Over-specifying** — every extra instruction is one fewer decision the agent makes with its own judgement.

See [`SKILL.md`](SKILL.md) for the full skill.
