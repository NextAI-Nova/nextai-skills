# Publishing NextAI Skills

Goal: users install by skill/registry name, not by GitHub repository URL.

## Desired install commands

```bash
npx skills add image-forge
```

```bash
openclaw skills install @nextai/image-forge
```

These are two different ecosystems and require two publication steps.

## 1. npx skills / skills.sh

`npx skills add <source>` supports GitHub sources such as:

```bash
npx skills add NextAI-Nova/nextai-skills@image-forge
```

To support the name-only command:

```bash
npx skills add image-forge
```

`image-forge` must be registered with the skills.sh / Vercel `skills` resolver so it maps to this GitHub source and skill:

```text
source: NextAI-Nova/nextai-skills
skill: image-forge
install name: image-forge
```

Evidence from local CLI inspection:

- `npx skills find` queries `https://skills.sh/api/search`.
- Search results print install hints as `npx skills add <owner/repo@skill>`.
- `npx skills add image-forge` currently treats `image-forge` as a Git source and fails unless the resolver/index supports that name.

### npx publish checklist

1. Make `https://github.com/NextAI-Nova/nextai-skills` public.
2. Keep `image-forge/SKILL.md` frontmatter valid:
   - `name: image-forge`
   - non-empty `description`
3. Submit/register the skill with skills.sh so the resolver can map `image-forge` to `NextAI-Nova/nextai-skills@image-forge`.
4. Verify after indexing:

```bash
npx skills find image-forge
npx skills add image-forge --skill image-forge -y
```

If skills.sh requires GitHub-source syntax even after indexing, use the canonical command:

```bash
npx skills add NextAI-Nova/nextai-skills@image-forge
```

## 2. OpenClaw / ClawHub

OpenClaw installs name-only registry skills from ClawHub using owner-scoped refs:

```bash
openclaw skills install @nextai/image-forge
```

Publish the skill folder to ClawHub:

```bash
npm i -g clawhub
clawhub login
cd ~/work/code/nextai-skills
clawhub skill publish ./image-forge --version 1.0.0
```

ClawHub validates publisher ownership. The publishing account must have access to the `@nextai` owner. If `@nextai` is unavailable, use the approved org owner, for example:

```bash
openclaw skills install @nextai-nova/image-forge
```

and publish under that owner instead.

### ClawHub verification checklist

```bash
openclaw skills search image-forge
openclaw skills verify @nextai/image-forge
openclaw skills install @nextai/image-forge --force
openclaw skills info image-forge
```

## Pre-release local checks

```bash
cd ~/work/code/nextai-skills/image-forge
python3 scripts/test_image_forge.py
python3 -m py_compile scripts/image_forge.py scripts/test_image_forge.py
```

Check Git contents before publishing:

```bash
cd ~/work/code/nextai-skills
git status --short
git diff --check
git ls-files
```

## External-write boundary

Do not push, publish to ClawHub, or submit to skills.sh without explicit confirmation from the repo owner.
