---
name: image
description: Generate an image with OpenAI Codex's built-in image tool from inside Claude Code. Use when the user asks to generate, render, or create an image, icon, logo, avatar, illustration, hero, or visual, and wants it produced directly (not just a prompt to paste elsewhere). Requires the Codex CLI installed and authenticated with a ChatGPT account (run `codex-doctor`); image generation is NOT available with API-key auth.
---

# Generate images with Codex

This skill renders an image by driving the Codex CLI's built-in image tool through the bundled `codex-image` command, then verifies a real PNG landed on disk.

## Prerequisite (firm)

Image generation works **only with ChatGPT-account auth** (`codex login` → "Sign in with ChatGPT"). It does **not** work when Codex is authenticated with an `OPENAI_API_KEY`. Check first:

```bash
codex-doctor
```

If doctor reports API-key auth, tell the user to run `codex logout` then `codex login` and pick the ChatGPT option. Do not attempt generation until auth is ChatGPT.

## Usage

```bash
codex-image "<full self-contained image prompt>" <output.png> [WIDTHxHEIGHT]
```

- **Output path:** pass where the image should land. Relative paths resolve against the current directory; the parent directory is created automatically. Prefer an explicit path inside the user's project (for example `assets/logo.png`), never a throwaway in `/tmp` if the user wants to keep it.
- **Size:** optional, default `1024x1024`. Use the aspect ratio the use case needs (for example `1536x1024` for a wide hero).
- The command **verifies** the file exists and is a valid PNG (magic bytes) before reporting success, and prints the byte size.

## How to use it well

- **Write one complete, self-contained prompt.** Subject, style, composition, colour, mood, and any text. The model has no other context.
- **For a consistent set** (an icon family, an avatar series), embed the SAME art-direction block in every prompt and only vary the subject line.
- **Inspect the result.** After `codex-image` reports success, `Read` the PNG and check it matches the ask before integrating it. Regenerate with a refined prompt if it misses.
- **Batches:** call `codex-image` once per image; run several in parallel with `xargs -P 4` if you have many. Each render takes roughly 1 to 2 minutes and draws on the user's Codex rate limit.

## Failure modes the command already handles

- Codex CLI not installed → clear install instructions.
- Not logged in / API-key auth → explicit message to switch to ChatGPT auth.
- Model produced no file, or a non-PNG file → reported as a failure, not a false success.
