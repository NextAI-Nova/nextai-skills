# NextAI Skills

Official NextAI skills for AI agents.

## Skills

### ImageForge

Generate and edit images through the fixed NextAI Code image API.

- Skill folder: [`image-forge`](./image-forge)
- API base: `https://www.nextai-code.com/v1`
- Get API Key: https://www.nextai-code.com

## Install

### npx skills

Target user-facing install name:

```bash
npx skills add image-forge
```

Current `npx skills` releases do not resolve arbitrary skill names directly in `add`. To make the command above work, `image-forge` must be accepted by the skills.sh / `skills` CLI resolver. Until then, the GitHub-source command is:

```bash
npx skills add NextAI-Nova/nextai-skills@image-forge
```

### OpenClaw

Target user-facing install name:

```bash
openclaw skills install @nextai/nextai-image-forge
```

This is the published ClawHub skill reference.

## Development

Run tests:

```bash
cd image-forge
python3 scripts/test_image_forge.py
```

## Publishing

See [`PUBLISHING.md`](./PUBLISHING.md).
