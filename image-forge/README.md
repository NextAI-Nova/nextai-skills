# Image Forge

Image Forge is a NextAI skill for AI agents to generate and edit images through NextAI Code.

It is not just a thin wrapper around an image API. Image Forge is designed for agent-led image creation: the agent helps clarify the goal, shape an image brief, prepare a better prompt, call the image model, and return the generated or edited file.

## Why it is different

### 1. Built for agent workflows

Most image tools ask for a prompt and immediately call a model. Image Forge is built as a skill, so an agent can use it naturally inside a broader task: product ideation, content planning, campaign drafts, social images, or visual iteration.

### 2. Image Brief before generation

Image Forge includes an Image Brief workflow. Before normal generation or editing, the agent is expected to clarify intent, style, composition, key elements, and success criteria. This reduces vague one-shot prompts and makes outputs more controllable.

### 3. Generate and edit

Image Forge supports both text-to-image generation and image editing from an existing source image, so it can start from a new idea or continue iterating on an existing visual.

### 3.1 Provider parameter coverage

Generation and editing expose the full set of image controls the NextAI gateway accepts: `size`, `quality`, `n` (1-10), `response_format`, `output_format` (png/jpeg/webp), `output_compression`, `background`, `moderation`, and `user`. Editing additionally supports a `mask` for inpainting, `input_fidelity`, and repeated source images for multi-image reference editing. Output files are named from the real payload format, so a webp request produces a `.webp` file.

### 4. Fixed NextAI Code endpoint

The skill uses the fixed NextAI Code API base:

```text
https://www.nextairouter.com/v1
```

The setup flow does not ask users to configure arbitrary API URLs, and runtime checks reject non-NextAI endpoints. This avoids provider mismatch and misconfiguration.

### 5. Local setup, no secrets in repo

On first use, the skill starts a local setup page for API key and model configuration. API keys are stored locally and must never be committed, logged, or echoed in responses.

## Install

The skill is distributed from the GitHub repository; there is no npm or registry package.

```bash
npx skills add NextAI-Nova/nextai-skills@image-forge
```

This shallow-clones the repository and installs the `image-forge/` folder into `.agents/skills/image-forge/`. Re-run the same command to update.

For other agents, clone the repository and point at the folder:

```bash
git clone https://github.com/NextAI-Nova/nextai-skills.git
```

## Use cases

- Product concept images
- Marketing and campaign drafts
- Social media visuals
- Event poster directions
- Image-based iteration and editing
- Agent-driven creative workflows

## Links

- GitHub: https://github.com/NextAI-Nova/nextai-skills
- NextAI Code: https://www.nextairouter.com
