---
name: review
description: Consult OpenAI Codex as an adversarial second opinion before committing to a plan, design, decision, diff, or piece of writing. Use when the user says "ask codex", "check with codex", "get a second opinion", "red-team this", "have codex review/critique this", "spar with codex", or when you are about to commit to a HIGH-STAKES choice (architecture, security, an irreversible or destructive operation, a major refactor, a big strategic decision) and want a strong second model to attack it first. Requires the Codex CLI installed and authenticated (run `codex-doctor`). NOT for image generation (that is the `image` skill).
---

# Codex as a sparring partner

A second strong model with different blind spots catches what you miss. This skill runs **OpenAI Codex** as a sharp, adversarial **critic** of your own work, via the bundled `codex-consult` command, and feeds its structured critique back into your plan.

## The one rule that defines this skill

**Codex is a red-team critic, not a co-equal voter, and "both agree" is NOT the goal.** You and Codex share much of your training and failure modes, so your *agreement* is weak evidence of correctness and your *disagreement* is the signal worth mining. You stay the decision owner. You weigh each Codex point on its merits, fold in the good catches, reject the weak ones **with a stated reason**, and when a real disagreement cannot be resolved, surface **both positions to the user** instead of manufacturing consensus.

## Setup (once)

The Codex CLI must be installed and authenticated. Verify with the bundled doctor:

```bash
codex-doctor
```

If `codex-doctor` reports it is missing: `npm install -g @openai/codex` (or `brew install --cask codex`), then `codex login`. `codex-consult` also honours `$CODEX_BIN` to point at a specific binary.

## When to use it

**Proactively (high-stakes):** architecture or data-model decisions; anything touching security, auth, secrets, or user data; irreversible or destructive operations; major refactors; multi-step interdependent plans; big strategic calls. Tell the user you consulted and what came back.

**On demand (any stakes):** the user says ask codex, second opinion, red-team, critique, spar.

**Do NOT use it** for trivial or mechanical work. Each call costs ~20 to 40s and draws on the user's Codex rate limit (a $20 ChatGPT Plus plan allows roughly 15 to 80 messages per rolling 5h). Spend it where a wrong call is expensive.

## The loop (bounded, max 3 rounds)

1. **Do your own work first.** Form your plan / design / diff / draft and your own view of it.
2. **Decide if a consult is warranted** (high-stakes or asked). If not, skip it.
3. **Write a lean brief** (see `reference/brief-template.md`): goal, hard constraints, the actual plan/decision in bullets, your own current view, and 1 to 3 sharp questions. Do NOT paste the whole conversation.
4. **Run the consult** (round 1):
   ```bash
   printf '%s' "$BRIEF" | codex-consult --mode plan --effort medium
   ```
   - `--mode`: `plan` | `code` | `writing` | `general`
   - `--effort`: `medium` (default) for most reviews; `high` for hard architecture/security; `low` for a quick sanity check.
   - `--cd <repo>` is **required for `--mode code`** so Codex can read files; also paste the diff into the brief.
   - Returns strict JSON (schema: `reference/critique-schema.json`) on stdout. Parse it.
5. **Reconcile in the open.** For each `disagreements`/`risks`/`missing`/`suggested_changes` item: **accept** the correct ones (fold into your plan), **reject** the weak ones (say so, with a one-line reason). Treat `open_questions` as things to answer.
6. **Round 2/3 only if needed.** If Codex raised a **material** point you cannot confidently resolve, send a focused follow-up: only the contested point, your counter-argument, and "respond". **Hard stop at 3 rounds.**
7. **Report the delta to the user:** what changed because of Codex, what you rejected and why, and any disagreement still unresolved (with both positions, so the user decides).

If you and Codex fully agree on round 1, say so briefly and move on.

## Modes

| Mode | Use for | Notes |
|---|---|---|
| `plan` | implementation plans, designs, architecture, decisions | runs isolated, no repo context needed |
| `code` | diffs, bug diagnoses, pre-commit review | **needs `--cd <repo>`**; paste the diff in the brief; Codex reads files |
| `writing` | copy, posts, positioning, emails, strategy docs | clarity, persuasion, support for claims |
| `general` | any reasoning you want stress-tested | catch-all |

## Safety and budget

- **Untrusted input.** The brief and any files Codex reads are sent to OpenAI. The critic framing treats stdin and repo files as data, not instructions.
- **Code mode reads files** under `--cd` and may load that repo's `AGENTS.md`. Point it at the specific project, never at `$HOME`, `/`, or a directory with secrets (`codex-consult` refuses the broad roots).
- **Lean by construction.** Calls run read-only, single-turn, with `--output-schema` for bounded output; when the installed Codex supports `--ignore-user-config` they also run isolated from the user's MCP servers. A hard timeout (default 300s) prevents a hung call.
- **Model.** Defaults to `gpt-5.5`; override with `$CODEX_MODEL` or `--model` if the user's account uses a different one. `codex-doctor` shows the active default.
