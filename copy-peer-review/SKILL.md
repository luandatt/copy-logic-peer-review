---
name: copy-peer-review
description: Simulates the AWAI-style "Peer Review" meeting for a sales-letter or email lead — a structured group session where a Peer Group rates the headline and lead numerically (1.0-4.0), proposes concrete rewrite suggestions (never compliments, criticisms, or comments), and rates each other's suggestions (Strongly Worse to Strongly Better) — using parallel Agent tool calls as stand-ins for the live human panel. Use whenever the user wants to "testar a headline/lead com um grupo de pares," run a "peer review," get "notas de 1 a 4" on copy, simulate a review panel, or asks "será que essa headline convence," and especially when they name this process explicitly or paste/reference its rules. Works on sales letters, ads, and — with the headline/lead mapped to Asunto+Preview and the opening hook — newsletters and emails. Trigger even if the user just pastes a draft and asks "roda o peer review nisso" or "testa isso com o grupo."
---

# Copy Peer Review (simulated with parallel agents)

## What this is

This reproduces the peer-review meeting format used in direct-response copywriting: a Leader runs a Peer Group of readers through two rounds — rate, then improve, then re-rate — first on the headline, then on the rest of the lead. The two things that make the original format work are structural, and this simulation preserves both:

1. **Suggestions only, never compliments/criticisms/comments.** A suggestion is a concrete rewrite ("instead of X, try Y"). Anything else wastes the round — it doesn't tell the copywriter what to change.
2. **Every suggestion gets rated, not just discussed.** A suggestion that the group unanimously likes gets forced into the draft; one they unanimously dislike gets blocked. This is what stops the loudest voice in the room from steering the edit.

Since there's no live panel here, N separate `Agent` calls stand in for the Peer Group. Be upfront with the user that these are simulated readers, not real ones — useful as a structured pass, not a substitute for testing on an actual audience (same caveat the `cub` skill gives for its peer-review pass).

## Roles in this simulation

- **Leader** — you (the main thread). Runs the meeting, keeps it on the rails, collects ratings, does the math, applies the editing rules, decides when to move to the next round.
- **Peer Group** — 5 parallel `Agent` calls (`subagent_type: general-purpose` is fine; they're reasoning tasks, not code tasks), each with its own fixed `model` per the persona bank below. Each one is a fresh agent with no memory of this conversation, so every prompt you send it must be fully self-contained: the persona, the text under review, and the exact task.
- **Copywriter** — the user. In the original format the Copywriter can't defend or explain the work, only answer factual questions and give suggestions/ratings like anyone else. That constraint matters less here since you're not going to argue with your own agents — but don't let your own judgment substitute for a Peer Group step. Run the round even when you're confident you already know the answer.

## Building the Peer Group personas

The one thing that breaks this simulation is if all 5 agents converge on the same opinion because they're the same underlying model with no reason to disagree. Give each one a genuinely different lens, not just a label — a persona description with no teeth produces 5 identical ratings.

**Every persona must be a working copywriter.** Vary personality, philosophy, and specialty — never vary out basic craft competence. A persona framed as "just a real reader, no marketing background" will happily propose a three-line paragraph as a preview-text rewrite, because nothing in the prompt told it that preview text has a hard visible-character budget, or that a subject line has deliverability constraints, or any of the dozen other mechanical rules a professional would apply automatically. The original human format could get away with mixing in non-marketer readers because a live human still picks up "emails are short" from just existing in the world — an LLM playing a persona only knows what the persona description tells it to know, so an under-specified persona produces under-informed suggestions, not authentic naivety. Diversity of opinion has to come from differences in taste and philosophy between professionals, not from knocking out craft knowledge.

Default bank of 5 (adjust to fit the audience, but keep the spread). Each also has a fixed model assignment — pass `model` explicitly on every `Agent` call for that peer, don't leave it to inherit:

1. **DR veteran, proof-driven** (`model: opus`) — direct-response copywriting background, distrusts unproven claims and vague superlatives, rewards specificity and evidence. Gets the strongest model because "does this claim actually hold up" is the persona's whole job, and that's a judgment call, not a rule lookup.
2. **DR veteran, storytelling lens** (`model: sonnet`) — direct-response background, but judges by emotional pull and curiosity gap rather than proof density.
3. **Niche specialist** (`model: sonnet`) — a copywriter who works specifically with this audience/niche day to day, weighs copy by how well it matches how this exact reader actually talks and what they're actually frustrated by.
4. **Format/channel specialist** (`model: haiku`) — a copywriter who specializes in the specific medium under review (email/newsletter, ad, etc.) and is obsessive about its mechanical constraints: preview-text character budgets, subject line length, platform-specific truncation, deliverability quirks. This persona exists specifically to catch the kind of practical constraint a purely content-focused reviewer would miss — checking a character budget is a rule-application task, not a nuanced-judgment one, so the lighter model is enough here.
5. **Contrarian skeptic** (`model: haiku`) — a copywriter whose default posture is distrust of anything that reads like a marketing trick or quick win; plays devil's advocate against the group's consensus pick. The contrarian stance is a fixed posture more than a deep judgment call, so a lighter model still produces a useful dissenting vote.

All five are copywriters; what differs is what each one optimizes for. This produces real disagreement (proof vs. curiosity, craft rules vs. audience voice, consensus vs. contrarian) without ever bottoming out in "this persona didn't know a basic rule of the medium." The model split isn't about giving personas 1-3 "better opinions" — it's matching model strength to how much of that persona's task is actual judgment (opus/sonnet) versus rule-checking or a fixed stance (haiku).

## The flow

Mirror the original meeting's structure exactly — headline first, fully resolved, before touching the rest of the lead.

### 0. Introduce the copy

Before running any rounds, state (to yourself, and briefly to the user) the offer, price, and intended audience in 1-2 sentences. This is context every persona prompt below needs, so nail it down first.

### 0.5 Mark the boundary

Explicitly state where the headline ends and where the lead ends, before evaluating anything — ambiguity here wastes a round. For a sales letter this is usually visually obvious. For email/newsletter format, map it like this:

- **Headline** = Subject line + preview text together (the two things a reader sees before opening).
- **Lead** = the opening hook, from the first line of the body up to where it pivots into the core teaching/offer content.

State this mapping to the user so they can correct it before you proceed.

### 0.75 Read the whole piece and verify the headline keeps its promise

Read the entire piece end to end before running any round — not just the headline and lead. A headline's job is to make a promise the body then pays off; you can't judge (or have the Peer Group judge) whether it's honest without knowing what the body actually delivers.

Specifically check any concrete claim the headline makes — a number ("3 tips"), a named technique, a specific outcome — against the body. If the count or claim doesn't match reality, fix it immediately, before running a single peer round: there's no reason to spend a round debating a promise you already know is false, and a false specific is worse than a vague one (an audience that counts "2" after being promised "3" loses trust in everything after it, per the same logic as CUB's Unbelievable flag).

This context isn't optional filler for the peer prompts either. From here on, every prompt sent to a Peer Group agent — rating rounds and suggestion rounds alike — must include either the full body text or an accurate plain-language summary of what it delivers, not just the 1-2 sentence offer/audience blurb from step 0. A peer that only ever sees the headline in isolation can rate how appealing it sounds, but has no way to catch an overpromise — that's a blind spot in the review, not an acceptable limitation of it.

### 1. Evaluate the headline (numeric rating round)

Dispatch all 5 Peer Group agents **in parallel, in one message, with `run_in_background: false`** (you need every rating back before you can average them — see the Agent tool's guidance on foreground vs. background). Each prompt must include: the persona, the headline text, the offer/audience context, the body content or summary from step 0.75, and this exact task —

> Rate this headline from 1.0 to 4.0, in increments of 0.1: 1.0-1.9 "I would definitely not read further," 2.0-2.9 "I would probably not read further," 3.0-3.9 "I would probably read further, but with skepticism," 4.0 "I would enthusiastically read further." Respond with exactly one line: `RATING: X.X — [one-sentence reason for the Leader's own log, not shown to anyone as feedback]`.

Add your own rating (you, as Leader, read it cold too) and average all 6. Show the user the spread, not just the average — a 4.0/4.0/1.2/1.5/3.8/3.9 tells a different story than six 2.8s even though the average is similar.

### 2. Improve the headline (suggestion round)

Dispatch the same 5 agents in parallel again, same self-contained format, task —

> Propose exactly one concrete rewrite of this headline you think would work better. Not a compliment, not a criticism of the current one, not a general comment about what makes headlines work — just the replacement text itself. Any specific claim in your rewrite (a number, a named technique) must be something the body actually delivers — you've been given the body content, so check it. Respond with exactly one line: `SUGGESTION: [replacement headline]`. If you have no improvement to propose, respond with `SUGGESTION: (none)`.

Then dispatch a second parallel round where each agent rates every *other* agent's suggestion (never its own) on: Strongly Worse / Worse / Neutral / Better / Strongly Better. Don't hand raters the isolated snippet in a vacuum — for each suggestion, construct the **complete before** (the full current headline/lead as it stands) and the **complete after** (that same full text with just this one suggestion's edit applied), and give raters both full versions to compare. A snippet judged out of context hides how it actually reads against the surrounding text — a rewritten opening line can clash with the paragraph right after it, or duplicate something the piece already says a few lines down, and none of that is visible unless the rater sees the whole thing. This matters even for a headline (small enough that subject+preview already read as one unit) and is essential for a lead, where suggestions are more likely to touch different, non-adjacent spans. Give raters one line per suggestion ID they didn't author.

If multiple suggestions from different peers touch the same span of text, treat them as competing alternatives for that slot rather than independent edits to stack — construct a separate full before/after for each one, rate them against each other in addition to against the original, and don't apply more than one to the same span.

Apply the editing rules exactly:
- **All ratings positive** (Better/Strongly Better) → this suggestion **must** go into the draft.
- **All ratings negative** (Worse/Strongly Worse) → this suggestion **must not** go in.
- **Neutral + positive only, no negatives** → apply it (encouraged default), but say so.
- **Mix of positive and negative** → flag it to the user and let them make the call — don't decide this one yourself.

### 3. Re-evaluate the headline

Rerun step 1's exact rating round against the new headline (fresh agents, no memory of the old score).

- **≥ 3.0** → move on to the rest of the lead.
- **2.8-2.9** → your call: either repeat steps 2-3 now, or note it and revisit after the full lead is done.
- **< 2.8** → repeat steps 2-3 on this same headline before moving on.

### 4. Evaluate the rest of the lead

Same numeric-rating round as step 1, but on the lead text (excluding the headline, now that it's locked in).

- **Average < 2.5** → stop. Per the original format, a lead this weak isn't a candidate for line-edits — say so plainly to the user and recommend a rewrite from scratch rather than running the improve step on it.
- **Average ≥ 2.5** → continue to step 5.

### 5. Improve the rest of the lead

Same mechanics as step 2, applied to the lead, with one addition that matters specifically because a lead is multiple paragraphs instead of one short line: **the suggestion task itself must require the agent to analyze and hand back the complete lead, not just the isolated excerpt it's changing.**

Here's why this isn't optional. A rewrite of a single sentence can look sharp when you only look at that sentence — but the lead has other paragraphs the agent already read, and a local edit can duplicate an anecdote that shows up two paragraphs later, spoil a reveal, or repeat a beat without the agent ever noticing, because nothing forced them to look at the whole thing again after drafting the change. The fix isn't asking them to "be careful" — it's asking for a different deliverable. Task:

> Read the whole lead. Pick the ONE or TWO spots you'd change most (move fast — this is longer text than a headline, don't edit every sentence). Then write out the **complete lead with your edit(s) applied**, start to finish — not just the isolated excerpt — and check it yourself against everything around it before handing it back: does it repeat something the piece already says elsewhere, does it give away something meant to land later, does the tone shift oddly at the seam. Not a compliment, not a criticism, not a general comment — the full revised text itself. Any specific claim you introduce must still be consistent with the full body you were given.

Respond with the complete lead text (not a diff, not just the changed line), plus a one-line note of which excerpt(s) changed for the Leader's own bookkeeping. If the agent has no improvement to propose, `SUGGESTION: (none)` is still fine.

This also makes the rating round in step 2 easier to run for a lead: you already have each agent's full "after" version handed to you, instead of having to reconstruct it yourself from an isolated excerpt.

### 6. Re-evaluate the lead

Rerun the rating round on the revised lead.

- **≥ 3.0** → this is now the official lead.
- **< 3.0** → repeat steps 5-6.

## Presenting each round

Show the user a compact table after every round — peer persona, rating or suggestion, nothing padded out with prose. They're the one who has to act on this, so make it scannable: which suggestions got forced in, which got blocked, which are theirs to decide.

## When this isn't the right tool

If the ask is a single quick gut-check pass rather than a structured multi-round panel, `cub` is faster and cheaper (one pass, three flags, no agent dispatch). If the ask is deep structural work on voice, proof, or offer strength rather than reader reaction to a specific headline/lead, that's `copywriting` or `copy-editing` territory. This skill is specifically for the rate → improve → re-rate loop on a headline and lead.
