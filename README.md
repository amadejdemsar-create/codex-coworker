# codex-coworker

A Claude Code plugin that puts **OpenAI Codex** to work *inside* Claude Code, as a coworker rather than a black box.

- **`/codex-coworker:review`** runs Codex (gpt-5.5) as a sharp, adversarial **second opinion**. Claude drafts a plan, diff, or piece of writing; Codex red-teams it; Claude reconciles the critique in the open and reports what changed. Built on the idea that two strong models with different blind spots beat one, and that their *disagreement* is the signal worth mining, not their agreement.
- **`/codex-coworker:image`** renders images through Codex's built-in image tool and verifies the PNG actually landed.

It is deliberately lean: critiques run read-only, single-turn, with a strict JSON output schema, so they stay cheap on a $20 ChatGPT Plus plan.

## Prerequisites

You need the **Codex CLI**, installed and authenticated:

```bash
# install (pick one)
npm install -g @openai/codex
brew install --cask codex
curl -fsSL https://chatgpt.com/codex/install.sh | sh

# authenticate (ChatGPT account recommended; required for image generation)
codex login
```

> Image generation works **only with ChatGPT-account auth**, not with an `OPENAI_API_KEY`. The critic works with either.

## Install the plugin

```bash
# add this marketplace, then install the plugin
claude plugin marketplace add amadejdemsar-create/codex-coworker
claude plugin install codex-coworker@codex-coworker
```

Then verify everything is wired up:

```bash
codex-doctor
```

`codex-doctor` reports your codex binary, version, auth mode, whether image generation is available, flag support, the default model, and timeout tooling.

## Usage

Once installed, just talk to Claude:

- "ask codex to red-team this plan"
- "get a second opinion on this diff"
- "have codex critique this landing-page copy"
- "generate a minimalist fox logo as assets/logo.png"

Or invoke the skills directly: `/codex-coworker:review`, `/codex-coworker:image`.

### Proactive critiques (optional)

A plugin cannot install a global behaviour policy, so if you want Claude to consult Codex **automatically** before high-stakes decisions, paste this into your own `~/.claude/CLAUDE.md`:

```md
## Codex second opinion
Before committing to a HIGH-STAKES choice (architecture, security, an irreversible or
destructive operation, a major refactor, a big strategic decision), use the /codex-coworker:review
skill to have Codex red-team it first, then reconcile its critique in the open and tell me
what changed. Codex is a critic, not a co-equal voter; I stay the decision owner. On demand,
"ask codex" / "second opinion" / "red-team this" triggers the same skill.
```

## The two commands (also usable from any shell)

```bash
# critic: reads a brief on stdin, returns strict JSON
printf '%s' "$BRIEF" | codex-consult --mode plan --effort medium
printf '%s' "$BRIEF" | codex-consult --mode code --cd /path/to/repo --effort high

# image: prompt + output path (+ optional size), verifies the PNG
codex-image "a flat-vector navy fox on cream" ./logo.png 1024x1024

# preflight
codex-doctor
```

`codex-consult` modes: `plan` | `code` | `writing` | `general`. Effort: `none|minimal|low|medium|high|xhigh` (default `medium`). Override the model with `--model` or `$CODEX_MODEL`; override the binary with `$CODEX_BIN`.

## Permissions

Claude Code will prompt before running the plugin's commands the first time. To skip the prompt, add to your `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(codex-consult:*)",
      "Bash(codex-doctor:*)",
      "Bash(codex-image:*)"
    ]
  }
}
```

## Compatibility

| Dimension | Tested | Notes |
|---|---|---|
| macOS (Apple Silicon) | yes | bundled binary + Homebrew |
| Linux | should work, untested | scripts are bash-3.2-safe, POSIX-ish; uses GNU `timeout` if present |
| Windows | should work, untested | via the Codex CLI; bash required |
| ChatGPT auth | yes | critic + image generation |
| API-key auth | partial | critic works; image generation does **not** |
| Codex CLI with `--output-schema` | required | the critic needs it; `codex-doctor` checks |
| Codex CLI with `--ignore-user-config` | optional | newer CLIs run isolated/lean; older ones run with full config and still work |

## Troubleshooting

- **"Codex CLI not found"** — install it (above) and run `codex login`, or set `$CODEX_BIN` to the binary path.
- **Critic fails with a schema/flag error** — your Codex CLI is likely too old. `npm install -g @openai/codex` to update, then `codex-doctor`.
- **Image generation says API-key auth** — run `codex logout` then `codex login` and choose "Sign in with ChatGPT".
- **Rate limited** — Plus allows roughly 15 to 80 gpt-5.5 messages per rolling 5h; the critic defaults to a single round at medium effort to stay within it.

## How the critic works (design notes)

1. Claude does its own thinking first, then writes a tight structured brief (not the whole conversation).
2. `codex-consult` runs `codex exec` read-only with a strict `--output-schema`, capturing only the final JSON so Codex's verbose reasoning never bloats Claude's context.
3. Claude reconciles each finding on its merits, accepts the good catches, rejects the weak ones with reasons, and escalates only material disagreements (hard cap: 3 rounds).
4. Unresolved disagreements are surfaced to you, not papered over.

## License

MIT. See [LICENSE](LICENSE).
