# Copy Logic Peer Review

A Claude Code skill that simulates the AWAI-style "Peer Review" meeting for a sales-letter or email lead — a structured group session where a Peer Group rates the headline and lead numerically (1.0–4.0), proposes concrete rewrite suggestions, and rates each other's suggestions — using parallel `Agent` tool calls as stand-ins for a live human panel.

## What it does

- Runs the headline and lead through separate rate → improve → re-rate rounds.
- Dispatches a Peer Group of 5 simulated copywriters (fixed personas + model assignments) in parallel via the `Agent` tool.
- Verifies the headline's promise is actually kept by the body content before evaluating anything.
- Applies strict editing rules: unanimous positive ratings force a suggestion in, unanimous negative blocks it, mixed ratings get flagged back to the user.

## Install

Copy `copy-peer-review/SKILL.md` into your Claude Code skills directory (e.g. `~/.claude/skills/copy-peer-review/`).

## Usage

Trigger it by asking to "rodar um peer review" / "testar a headline com um grupo de pares" on a sales letter, ad, or newsletter/email lead.
