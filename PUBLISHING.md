# Publishing NextAI Skills

Skills are distributed **only** from this GitHub repository. This project does
not publish to npm and does not rely on a skills.sh index registration.

The user-facing install command is:

```bash
npx skills add NextAI-Nova/nextai-skills@image-forge
```

## How distribution actually works

`npx skills add` does not download an npm tarball. It shallow-clones the
repository and copies the skill folder into the consumer's workspace:

1. `git clone --depth 1` of `NextAI-Nova/nextai-skills` into a temp directory.
2. `fetchRepoTree` + `discoverSkills` find every directory containing a valid
   `SKILL.md` (name and description in frontmatter).
3. The `@image-forge` selector picks the `image-forge/` folder, which is copied
   to `.agents/skills/image-forge/`.
4. `skills-lock.json` records `source`, `commit`, `skillPath`, and
   `computedHash`.

Two consequences follow from that, and both matter for releasing:

- **Everything committed is distributed**, not just the npm `files` allowlist.
  `scripts/test_image_forge.py` and `scripts/install_command.sh` ship to users
  because they live in the folder. Keep them clean.
- **Updating users means pushing a commit.** `npx skills update` re-resolves the
  locked commit; there is no registry version to bump.

## Release checklist

```bash
cd ~/work/code/nextai-skills/image-forge

# 1. Tests and syntax
python3 scripts/test_image_forge.py
python3 -m py_compile scripts/image_forge.py scripts/test_image_forge.py

# 2. Version must agree in both places (there is a test for this)
grep -n '^SKILL_VERSION' scripts/image_forge.py
grep -n '"version"' package.json
```

```bash
cd ~/work/code/nextai-skills

# 3. Review what will ship
git status --short
git diff --check
git ls-files

# 4. Commit and push
git add -A
git commit -m "feat(image-forge): ..."
git push origin main
```

## Verify a release from the consumer side

Install into a throwaway directory and confirm the shipped version:

```bash
cd "$(mktemp -d)"
npx skills add NextAI-Nova/nextai-skills@image-forge -y
grep -n '^SKILL_VERSION' .agents/skills/image-forge/scripts/image_forge.py
cat skills-lock.json
```

The lockfile `commit` must match the pushed commit. If it does not, the clone
picked up a stale cache or the push did not land.

## Versioning

`package.json` `version` and `SKILL_VERSION` in `scripts/image_forge.py` are
kept identical by a unit test. Bump both together. `package.json` is metadata
only; it exists so tools can read a version, not as an npm publish target.

## External-write boundary

Do not push, and do not introduce a new distribution channel, without explicit
confirmation from the repo owner.
