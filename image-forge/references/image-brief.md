# Image Brief Workflow

Use this before ImageForge generation or edit after configuration is ready. Turn the user's request into a compact Approved Image Brief, confirm once, then run.

Default to the user's language. If the user writes Chinese, guide the whole flow in Chinese.

<GATE>
Do NOT run `generate` or `edit` before the user has approved the brief (or Direct mode applies). Never fabricate `User approval: yes` without an explicit user confirmation.
</GATE>

## Fast path (default)

If the request already pins down subject, purpose, and enough visual direction (style, composition, or text requirements), skip questions:

1. Write the compact brief below in your first response.
2. Ask for one confirmation ("对吗？确认后我直接生成").
3. On approval, run ImageForge with `--brief '<approved brief>'`.

One confirmation total. Do not add extra review rounds the user did not ask for.

## Ask path (only when information is missing)

Only when subject, purpose, or visual direction is genuinely missing or ambiguous:

1. Ask all clarifying questions in ONE batch message (2-4 questions max). Do not ask one question at a time.
2. After the answers, write the compact brief and ask for one confirmation.
3. On approval, run ImageForge with `--brief '<approved brief>'`.

Two interactions total (one Q&A batch + one confirmation). If the user's answers still leave critical gaps, ask one more focused batch — but prefer reasonable defaults over more rounds.

## Question checklist (pick only what is missing)

- Purpose / deliverable: where the image is used, quantity, aspect ratio or size.
- Subject: main subject, product, scene, must-include elements, exact text copy.
- Style: photo / illustration / 3D / flat graphic, mood, palette, references.
- Constraints: what to avoid, no watermark, no extra logos, brand/legal limits.
- Edit-specific: source image path, what must stay unchanged, what changes.

## Approved Image Brief (compact format)

Present this exact structure. Every field must be filled (use "not applicable" / "无" where a field does not apply):

```text
Approved Image Brief
Context: <one line: where/how the image is used>
Requirements: <subject, quantity, size/ratio, exact text or "no text">
Approach: <chosen visual direction: style, composition, lighting, palette>
Constraints: <avoid list, preservation requirements, brand/legal limits>
Edit scope: <what stays unchanged / changes; "not applicable" for generation>
User approval: yes
```

The helper rejects empty fields, placeholders, and missing explicit user approval. Do not fill `User approval: yes` before the user actually confirms.

## Direct mode

Direct mode applies when the user explicitly says things like "直接生成", "别问", "按这个 prompt 做", or "use this exact prompt". In Direct mode:

- Do not ask anything and do not write a brief.
- Briefly restate the execution understanding in one or two sentences.
- Proceed with `--direct`, keeping the prompt faithful to the user's wording.

## Prompt construction

After approval, convert the brief into a provider prompt:

- Put the core subject and purpose first.
- Include style, composition, lighting, color, and exact text requirements.
- Include constraints as direct negative instructions.
- For edits, state preservation requirements before requested changes.
- Keep prompts specific but not bloated; avoid contradictory style words.
