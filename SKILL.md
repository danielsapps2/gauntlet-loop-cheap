---
name: gauntlet-loop-cheap
description: Turns any goal into one short, paste-ready "gauntlet loop" prompt - a prompt that makes an agent set a concrete quality bar, split the work into small judgeable pieces, run a builder, a separate harsh critic and a blind judge on each, compare blind against the bar, and loop until it wins. Works for builds, writing, code, research, or design. Triggers on "/gauntlet-loop", "gauntlet loop", "gauntlet this", "make a gauntlet prompt", "loop until it beats X".
---

# Gauntlet Loop

The user gives a goal. You give back ONE short prompt they can paste into a fresh agent session.

You are not doing the work. You are writing the prompt that makes another agent grind on the work until it beats a real reference.

## Flow

1. **Read the goal.** One line restatement in your head, not on screen.
2. **Set the bar.** If the user supplied a reference, use it. If not, offer **2 or 3 candidate bars**, one line each, and stop. Wait for their pick. Do not write the prompt yet.
3. **Write the prompt.** One block, paste-ready, no preamble, no headings inside it, no narration after it.
4. **Offer to run it.** One flat line under the prompt: "I can run this here." Not a question.

If they say run it, you become the lead agent and follow the prompt you just wrote, using the **lead playbook** below. Spawn subagents with the Agent tool, setting `model` and `effort` per the routing below (`opus` + `medium` for planners and judges; `sonnet` + `medium` for builders and critics; `haiku` + `high` for command and terminal runs). Give each builder exactly one small, already-planned task, with the files and the definition of done spelled out, so it can implement straight away without planning. Obey the budget below. Do not use the Workflow tool or ultracode.

## The bar is the whole trick

Everything else in a gauntlet loop is scaffolding. The loop only produces quality if the thing it compares against is real.

A bar has to pass three tests:

- **Named.** A specific thing, not a category. "Stripe's pricing page" works. "Award-winning SaaS sites" does not.
- **Fetchable.** The critic can actually get it - screenshot the live page, read the published piece, run the binary, open the repo, watch the footage. If the agent cannot obtain it, it will hallucinate the comparison.
- **Comparable.** Both can sit side by side and a judge can pick one. If you cannot imagine the A/B, it is not a bar.

Bars by goal type:

| Goal | Bar that works |
|---|---|
| Website, app, UI | The live site of a specific best-in-class product, screenshotted at the same viewport |
| Game, 3D, visual | Real footage or screenshots from a named shipped title |
| Writing | A specific published piece by a named author or publication, same length and format |
| Code, tooling | A named repo's implementation, plus its benchmark or test suite as the measurable half |
| Research, analysis | A named analyst report or a paper's methods section, judged on rigour and coverage |
| Deck, doc, deliverable | A real artifact from a firm known for it, same page count |

When you propose bars, prefer the hardest one the agent can genuinely reach. A bar that is too easy makes the loop exit on round one.

If the goal has a measurable half (load time, token cost, benchmark score, word count, pass rate), name it alongside the reference. Taste plus a number beats taste alone.

## Prompt template

Adapt the wording every time. Fill the brackets, keep it short, keep the last two lines.

```
Build [GOAL].

The bar is [BAR]. Get the real thing first and compare against it directly, not against a description of it.

Break this into the smallest pieces that can be improved and judged on their own. For each piece, a planner cuts it into small tasks and each builder implements one quickly, without planning. A separate critic with fresh context captures the real output, pairs it blind with the bar, labels stripped, and measures the gaps. A separate judge sees only that pair, picks the better one and names the biggest remaining gap. Then it goes back to the builder.

Critic and judge are harsh; praise is useless. Give builders measured targets from the bar, not "bigger" or "lighter". Trust the critic's numbers over the builder's, and keep a round only if it beats the best so far.

/loop on each piece until the judge picks ours blind. Do not stop before that.

Keep a live progress page updating as the work evolves so I can watch it.

This run has to last 7 days on a Claude Max 5x plan. Keep at most 4 subagents alive at once, with no more than 1 of them on Opus. Check usage before every round and stop launching above 70% of the 5-hour window. Spend no more than 12% of the weekly limit in a day. When today's share is gone, schedule a wakeup for tomorrow and pause instead of pushing through.

Fan out subagents. You orchestrate on Opus 5.5. Planners and judges run on Opus 5.5 at medium effort. Builders and critics run on Sonnet 5.5 at medium effort. Quick commands and terminal work run on Haiku 5.5 at high effort.
```

Rules for what you fill in:

- Bake the bar in as a concrete, fetchable thing. URL, product name, repo, title.
- Always include the budget line, word for word, as the second-to-last line. If the user names a different plan, run length or cap, swap those values into it.
- Always keep the model routing line as the last line, word for word.
- Add tool names only if the goal needs them (image or video generation, a browser, a deploy target).
- Everything else stays out. No architecture, no file layout, no decomposition, no round count, no stack choice unless the user demanded it. The agent decides those, and it decides better than a spec written before the work started.

## Length and voice

Short. Around 230 to 290 words; the fixed budget and routing lines are about 100 of those. If the prompt needs a heading to stay readable, it is too long.

Plain sentences. No bullet lists inside the prompt. It should read like someone telling an agent what perfect looks like and refusing to accept less.

## Portability

`/loop` is a Claude Code feature. It reruns the prompt on an interval or lets the model pace itself. Never write the word "ultracode" anywhere in the prompt, not even to negate it - the keyword alone opts the session into Workflow orchestration; the lead agent fans out with plain subagents instead of Workflow orchestration.

For any other agent, swap the `/loop` line for: "Keep looping until the judge picks ours. Run the builders, critics and judges as parallel subagents." Keep the model routing line if that agent can pick models per subagent; drop it if not. The structure carries over unchanged.

## Budget

The default budget assumes a Claude Max 5x plan that has to last 7 days. Anthropic does not publish exact weekly caps. Third-party estimates for Max 5x put Opus at about 22 to 52 hours a week, and that allowance is shared with everything else the user does in Claude. So plan from the low end, around 25 Opus hours a week, or about 3.5 a day.

Every Opus agent running in parallel burns that allowance at once, and the lead counts as one. That is why the volume work goes to Sonnet: builders write the code and critics do the capturing, cropping and measuring. Opus only does the judgement: planning tasks, the blind verdict, and the lead's merge decisions. Judges are cheap (one short read of a blind folder, about 100k tokens and under a minute), so holding Opus to 1 slot rarely slows the loop.

Measured on a real run: Sonnet builders averaged about 190k tokens and 17 minutes per round, against about 356k and 27 minutes for Opus builders. Sonnet critics averaged about 150k tokens and 4 minutes, against about 175k and 5 to 14 minutes on Opus. Four parallel builders with Sonnet critics and Opus judges used about 21% of the 5-hour window per hour, against about 35 to 40% with Opus builders and critics. Margins held or improved once the briefs carried the measured-change line below.

The concurrency cap only controls how fast the budget is spent. The daily cap is what makes it last the week. 12% a day comes to 84% over 7 days, which leaves about 15% for the user's own work. The lead checks remaining quota before each round, using `/usage` or the session usage tool. It stops launching new agents above about 70% of the 5-hour window, because hitting 100% kills running agents mid-work. When the day's share is spent, it pauses with ScheduleWakeup instead of draining the week early.

## Model routing

Every prompt ends with the same routing line. Do not change it unless the user asks.

- **Lead agent / orchestration:** Opus 5.5 (`claude-opus-5-5`). Owns merges, the blind key and the budget guard. Keep it on Opus: it is a small share of total spend, and its mistakes (merging a regression, leaking the key) are the ones nothing downstream catches.
- **Planners and judges:** Opus 5.5 at medium effort. Planners split each piece into small tasks a builder can finish in one short run. Judges see only the blind folder and give a binary verdict plus the single biggest gap.
- **Builders:** Sonnet 5.5 (`claude-sonnet-5-5`) at medium effort. One small task per builder. Builders don't plan or redesign; they implement the task they were given, fast, and hand it back.
- **Critics:** Sonnet 5.5 at medium effort. They build and capture the real output, prepare the blind A/B folders, and measure the gaps. Their work is mechanical and checkable, so the lead inspects their folders instead of paying Opus for it.
- **Quick command execution and terminal work** (running builds, tests, screenshots, fetches, file shuffling): Haiku 5.5 (`claude-haiku-5-5`) at high effort.

## Lead playbook

What the lead does when it runs the prompt. These come from a multi-day run with dozens of rounds; each one fixed a real failure.

**Split critic and judge.** The critic knows which side is ours, because it built the folders. The judge must not. Give the judge only the blind folder path, the criteria, and a reply format: `WINNER: A|B / MARGIN: narrow|clear|overwhelming / GAP: ... / WHY: ...`. Tell it not to open any other path.

**Keep the key secret.** Store which letter is ours outside any path the judge can read, and alternate the letter between rounds. Make two folders per round with the letters swapped, so a confirmation judge can rerun on the mirror. Only spend confirmation judges when ours wins.

**Check every blind folder yourself before the judge sees it.** Same crops, sizes and framing on both sides; masking or blur applied identically; no labels, timestamps, metadata or key files. Look at one or two images. If it is unfair to either side (a crop that misses our element, over-blurring of non-identifying text), send the critic back with the fix before judging. A Sonnet critic got this wrong about one time in eight.

**Stop the overshoot.** Builders over-correct the last gap: holes made bigger, then too small, then blurry; panels too heavy, then too light, then heavy again. Two lines in every builder brief fixed it: "make measured changes and compare against the bar after each one", and hard numeric targets measured from the bar (for example "score digits 2.0% of screen height", not "make the score bigger").

**Trust the critic's numbers.** Builders' self-reported measurements often fail to reproduce in the critic's capture. Tell builders to measure with the same tool and method the critic uses, and decide on the critic's numbers.

**Ratchet.** Merge a round only if it is at least as good as the best version so far. If a round regresses, keep it on its branch (tag it), and start the next round from the best one, not from the regression.

**Cap rounds per piece if the user wants a budget.** The default exit is winning. If the user sets a cap (for example 6 rounds for visual pieces), log the final gap when it is hit and move on. When the same gap survives three rounds of tuning, say so: it usually needs engine work (a new system), not settings.

**Ask about design intent.** If the bar conflicts with the user's own reference or taste (a stylised character judged against realistic soldiers), ask which direction they want, and tell the judge to rate each side on how well it renders its own design.

**Protect running agents.** Background subagents die if the lead's context is cleared or usage hits 100%. Tell builders to commit work-in-progress after each working improvement. Never clear context while agents run; write a handoff file first, and relaunch from their committed work if they are lost.

**Keep briefs complete and replies short.** Each brief carries the latest judge gap verbatim, the critic's numbers, a concrete direction, the steps (merge, build, capture, compare, iterate, test, commit, save a before/after image), "commit only in your own worktree", and a disk warning for capture-heavy work. Cap final replies (builders under 200 words, critics under 250) so the lead's context lasts.

**Check the work, not the report.** Look at each before/after image before launching the critic, and check the main checkout is still clean after each builder: one wrote into the wrong folder.

## Two filled examples

**Visual goal.** User: "landing page for my running brand, athletic, green and dark, has to feel alive."

Bars offered: A) Nike's current running campaign page B) On Running's homepage C) Gymshark's product landing page. User picks A.

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

**Non-visual goal.** User: "a 2000-word explainer on vector databases for non-engineers."

Bars offered: A) a specific Stripe engineering blog explainer B) a named Julia Evans post C) the Wikipedia article plus a comprehension test. User picks B.

```
Write a 2000-word explainer on vector databases for readers who are smart but not engineers.

The bar is Julia Evans' writing on hard technical topics. Pull three of her actual posts and compare against them directly, not against a description of her style.

Break this into the smallest pieces that can be judged on their own - the opening, each explanation, the diagrams, the analogies, the ending. For each piece, a planner cuts it into small tasks and each writer drafts one quickly, without planning. A separate critic with fresh context sets ours beside a matching passage of hers, bylines stripped, and measures the gaps (sentence length, jargon count). A separate judge sees only that pair, picks the one a non-engineer would understand faster and names the biggest remaining gap. Then it goes back to the writer.

Critic and judge are harsh; praise is useless. Give writers measured targets from the bar, not "simpler" or "punchier". Trust the critic's numbers over the writer's, and keep a round only if it beats the best so far.

/loop on each piece until the judge picks ours blind. Do not stop before that.

Keep a live progress page updating as the work evolves so I can watch it.

This run has to last 7 days on a Claude Max 5x plan. Keep at most 4 subagents alive at once, with no more than 1 of them on Opus. Check usage before every round and stop launching above 70% of the 5-hour window. Spend no more than 12% of the weekly limit in a day. When today's share is gone, schedule a wakeup for tomorrow and pause instead of pushing through.

Fan out subagents. You orchestrate on Opus 5.5. Planners and judges run on Opus 5.5 at medium effort. Builders and critics run on Sonnet 5.5 at medium effort. Quick commands and terminal work run on Haiku 5.5 at high effort.
```

## What breaks a gauntlet loop

- **A vague bar.** The critic invents a comparison and approves everything. Most common failure by far.
- **The builder judging its own work.** The critic must be a separate agent with fresh context. It should not know how hard the builder tried.
- **A judge who knows which side is ours.** The critic built the folders, so it cannot judge them. Use a separate judge that sees only the blind pair.
- **A soft judge.** Say "harsh" in the prompt and give it a binary job: which one is better, A or B. Scores out of 10 drift upward every round.
- **Directional briefs.** "Bigger", "lighter" and "more" make builders swing past the target round after round. Give measured targets from the bar.
- **Believing the builder's numbers.** Builders over-report. Decide on the critic's measurements.
- **Merging every round.** Regressions slip in. Keep only rounds that beat the best so far.
- **Named exit after N rounds.** The exit is winning the comparison, or the user stopping the run. A round cap is a budget choice the user makes, not the goal.
- **Unbounded fan-out.** Ten Opus agents in parallel can spend a week's allowance in one afternoon, and then the loop dies with the work half done. Keep the cap and the daily pacing.
- **Clearing the lead's context mid-run.** It kills the running subagents and loses their uncommitted work.
- **Over-specifying.** Every extra instruction is one fewer decision the agent makes with its own judgment. Minimal wins.
