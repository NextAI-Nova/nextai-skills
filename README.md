# NextAI Skills

Official NextAI skills for AI agents.

## Skills

### ImageForge

Generate and edit images through the fixed NextAI Code image API.

- Skill folder: [`image-forge`](./image-forge)
- API base: `https://www.nextairouter.com/v1`
- Get API Key: https://www.nextairouter.com

## Install

Skills are distributed from this GitHub repository; there is no npm or registry package.

```bash
npx skills add NextAI-Nova/nextai-skills@image-forge
```

`npx skills` shallow-clones the repository and copies the `image-forge/` folder into `.agents/skills/image-forge/`, recording the source commit in `skills-lock.json`. Run the same command to update.

Other agents can clone the repository and point at the folder directly:

```bash
git clone https://github.com/NextAI-Nova/nextai-skills.git
```

## Development

Run tests:

```bash
cd image-forge
python3 scripts/test_image_forge.py
```

## Publishing

See [`PUBLISHING.md`](./PUBLISHING.md).
