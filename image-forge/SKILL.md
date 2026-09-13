---
name: image-forge
description: "Use when an agent invokes ImageForge to generate or edit raster images with NextAI Code, configure API key/model, diagnose setup, or check whether the image-forge skill has a newer version."
---

# ImageForge

Use this skill to generate or edit raster images through the fixed NextAI Code image API.

## First Execution Gate

On every ImageForge skill use, run ensure-ready as the first command:

```bash
WORKSPACE_ROOT="$(git rev-parse --show-toplevel 2>/dev/null || pwd)"
IMAGE_FORGE_SCRIPT=""
for candidate in \
  "$WORKSPACE_ROOT/.agents/skills/image-forge/scripts/image_forge.py" \
  "$PWD/.agents/skills/image-forge/scripts/image_forge.py" \
  "$HOME/.agents/skills/image-forge/scripts/image_forge.py"; do
  if [ -f "$candidate" ]; then IMAGE_FORGE_SCRIPT="$candidate"; break; fi
done
if [ -z "$IMAGE_FORGE_SCRIPT" ]; then
  echo "ImageForge helper not found. Reinstall or update the image-forge skill."
  exit 2
fi
python3 "$IMAGE_FORGE_SCRIPT" ensure-ready
```

ensure-ready runs preflight. If configuration is missing, it stops the current flow, starts `setup-server`, waits for the user to save API key/model, then runs preflight again. Do not do any other work until preflight passes.

## Image Brief Gate

After `ensure-ready` passes and before any image generation or editing command, Read `references/image-brief.md`.

Before calling `generate` or `edit`, present a compact Approved Image Brief and get user confirmation. Keep the interaction short:

- If the request already contains enough information (subject, use, style/composition intent), present the brief in your first response and ask for one confirmation, then run.
- If key information is missing, ask all clarifying questions in one batch (do not ask one question at a time), then present the brief for one confirmation.
- Only iterate further when the user requests changes or information is genuinely ambiguous.

The helper enforces this gate: normal generation/editing must pass a structured `--brief '<approved brief>'` with explicit user approval. Hollow or one-line briefs are rejected before any provider call.

Direct mode: if the user explicitly says to generate/edit directly, not ask questions, or use the prompt exactly as written, skip the brief entirely and proceed with `--direct`. Do not use `--direct` unless the user explicitly asked to skip clarification.

## Hard Rules

- Never store API keys in this skill folder, Git, logs, or replies.
- Prefer `scripts/image_forge.py` over hand-written HTTP commands.
- Use only the fixed NextAI Code API base: `https://www.nextai-code.com/v1`. Never use or suggest another API URL.
- Default model is `gpt-image-2` unless the user configures another NextAI-supported model.
- Default output directory is the project root; write only `.png` image files and do not create output sidecar `.json` files.
- If `preflight` reports missing API key or model, start the local `setup-server` yourself; do not ask the user to run setup commands.
- Never continue with local layout, local drawing, screenshots, or any non-ImageForge fallback when ImageForge is unconfigured.
- Never turn a vague request into an API call without the Image Brief Gate unless Direct mode applies; the command must include either `--brief '<approved brief>'` or `--direct`.
- Editing must support one source image plus a text instruction; multi-image edit attempts must degrade clearly if the backend rejects them.
- Do not auto-upgrade the skill during image generation or editing.

## Command Resolution

Use the `IMAGE_FORGE_SCRIPT` resolver above instead of hard-coding a single install directory. `npx skills add` may install to a workspace `.agents/skills/image-forge` directory, while other agents may install to `$HOME/.agents/skills/image-forge`.

Do not make normal users configure `PATH`. `image-forge` is only a convenience alias when already available; otherwise run `python3 "$IMAGE_FORGE_SCRIPT" <command>`.

## Runtime Configuration

Configuration priority:

1. Fixed API URL: `https://www.nextai-code.com/v1`
2. `IMAGE_FORGE_API_KEY`, `IMAGE_FORGE_MODEL` environment variables
3. Project-level override at `.image-forge/config.json` (`defaultModel`, `outputDir`) — optional, overrides the user-level config for this directory only
4. User-level global config at `~/.config/image-forge/config.json` (`defaultModel`, `outputDir`) — shared across all working directories
5. User secret at `~/.config/image-forge/secrets.json` (`apiKey`) — global

Configuration is stored at the user level by default, so it works in any working directory. A project-level `.image-forge/config.json` is only needed to override the model or output directory for one project. On first run after upgrading, any existing project-level `defaultModel`/`outputDir` is migrated to the user-level config automatically.

`IMAGE_FORGE_API_URL` and `.image-forge/config.json` `apiUrl` are accepted only when they equal `https://www.nextai-code.com` or `https://www.nextai-code.com/v1`; any other URL is rejected before provider calls.

Configuration gate:

1. Resolve `IMAGE_FORGE_SCRIPT`.
2. Run:

```bash
python3 "$IMAGE_FORGE_SCRIPT" ensure-ready
```

3. If the browser does not open automatically, share the printed local URL with the user.
4. Wait for the user to submit API key and default model in the local page.
5. Continue only after `ensure-ready` returns ready.

The setup page listens on `127.0.0.1`, uses a one-time token URL, never echoes the API key, shows the locked NextAI Code URL, links to `https://www.nextai-code.com` for registration/API key retrieval, persists configuration, and shuts down after saving.

Required readiness check:

```bash
python3 "$IMAGE_FORGE_SCRIPT" ensure-ready
```

If this command fails, do not continue the user's image task until configuration is complete.

## Workflows

- First-use setup: agent runs `python3 "$IMAGE_FORGE_SCRIPT" ensure-ready`.
- CLI fallback setup: run `python3 "$IMAGE_FORGE_SCRIPT" setup` only if a browser form is unavailable; it prints the fixed NextAI Code URL and asks only for model/key.
- Setup server only: run `python3 "$IMAGE_FORGE_SCRIPT" setup-server` only for manual diagnostics.
- Preflight: run `python3 "$IMAGE_FORGE_SCRIPT" preflight`.
- Generate: after the Image Brief Gate, run `python3 "$IMAGE_FORGE_SCRIPT" generate --brief '<approved brief>' --prompt '<approved prompt>'`.
- Edit: after the Image Brief Gate, run `python3 "$IMAGE_FORGE_SCRIPT" edit --brief '<approved brief>' --image '<path>' --prompt '<approved instruction>'`.
- Diagnose: run `python3 "$IMAGE_FORGE_SCRIPT" doctor`.
- Version check: run `python3 "$IMAGE_FORGE_SCRIPT" check-version`.

## References

- API details: `references/openai-compatible-images.md` (NextAI Code fixed-base edition)
- Image brief workflow: `references/image-brief.md`
- Installation: `references/installation.md`
- Troubleshooting: `references/troubleshooting.md`

## Completion Report

Report the generated or edited image path, model, output size, and any degraded version-check or provider-compatibility status. Default outputs are written directly to the project root. Never report secrets.
