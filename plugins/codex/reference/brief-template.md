# Brief template for a Codex consult

Keep it tight. The brief is half the rate-limit budget and the whole signal. Give Codex what it needs to judge, nothing more. No conversation history it does not need.

## Round 1 (full critique)

```
GOAL: <one line: what this plan/decision/diff/draft is trying to achieve>

CONTEXT: <2 to 4 lines of only the constraints that change the answer: stack,
deadline, who it is for, what must not change, prior decisions already locked>

THE WORK UNDER REVIEW:
<the actual plan as numbered steps, or the decision + the options you weighed,
or the diff (for code mode, also pass --cd <repo>), or the draft text>

WHAT I THINK: <one or two lines of your own current view, so Codex can attack
it rather than restate it>

QUESTIONS FOR YOU (1 to 3, sharp):
1. <the specific thing you are unsure about>
2. <...>
```

## Round 2 / 3 (focused follow-up, only on a material unresolved point)

```
CONTESTED POINT: <the single disagreement we are resolving>

CODEX SAID: <Codex's claim, one line>
I PUSH BACK BECAUSE: <your concrete counter-argument>

QUESTION: Given that, does your objection still hold? If yes, give the strongest
version of it; if no, say so plainly.
```

## Rules of thumb

- Numbered steps beat prose; Codex critiques specifics.
- State the constraints that actually bound the solution (deadline, "no new deps", "must stay backward compatible"). Omit the rest.
- Always include WHAT I THINK so Codex red-teams your position instead of writing a generic essay.
- For `--mode code`, paste the diff (or the key function) into the brief AND pass `--cd <repo>` so Codex can read the surrounding files.
- 1 to 3 questions, not 10. Force prioritisation.
