# ImageForge Troubleshooting

ImageForge errors are returned as JSON with `error` and `message` fields. Messages are redacted before display, but agents must still avoid echoing secrets.

## `missing_config`

Cause: API key, model, or an edit image path is missing; the API URL is fixed to NextAI Code.

Fix first-use configuration by having the agent run the readiness gate:

```bash
python3 "$IMAGE_FORGE_SCRIPT" ensure-ready
```

If configuration is missing, `ensure-ready` opens the setup page. The user registers/logs in at `https://www.nextairouter.com`, gets an API Key, then fills API key and default model in the browser page. The API URL is shown as locked. The setup server listens only on `127.0.0.1`, uses a one-time token URL, and shuts down after saving. Do not ask normal users to configure `PATH` or run setup manually.

Do not continue the current ImageForge flow until preflight passes.

Check current local status:

```bash
python3 "$IMAGE_FORGE_SCRIPT" doctor
```

Check whether ImageForge is allowed to proceed:

```bash
python3 "$IMAGE_FORGE_SCRIPT" preflight
```

If `preflight` fails, do not continue the image task through local rendering or any other fallback. Configure ImageForge first.

## `auth_failed`

Cause: the provider returned `401` or `403`.

Fix:

- Confirm the API key was created at `https://www.nextairouter.com`.
- Confirm the key is valid for image generation and editing.
- Prefer updating the key through the secure prompt or `IMAGE_FORGE_API_KEY`.
- Do not paste the key into logs, Git, or replies.

## `provider_rejected` or `protocol_error`

Cause: the provider rejected the request, returned invalid JSON, omitted `data`, or omitted both `data[].b64_json` and `data[].url`.

Fix:

- Confirm NextAI Code is reachable and implements OpenAI-compatible `/v1/images/generations` and `/v1/images/edits`.
- Confirm the configured model is accepted by NextAI Code.
- Confirm the requested `--size`, `--quality`, `--n`, `--output-format`, `--output-compression`, `--background`, and `--moderation` values are supported by that provider.
- Image responses may return either `data[].b64_json` (decoded and written directly) or `data[].url` (downloaded then written). Both are written using the extension detected from the payload bytes. If both fields are present, `b64_json` wins.

## `download_failed`

Cause: the provider returned an image URL but downloading it failed (network error, HTTP error, expired link).

Fix:

- The error message keeps the original URL; the user can fetch the image manually.
- Retry the generation, or ask the provider to return `data[].b64_json` instead of URLs.
- Check proxy/VPN/firewall access to the URL host.

## `multi_image_unsupported`

Cause: the provider rejected an edit request with multiple `--image` values.

Fix:

```bash
python3 "$IMAGE_FORGE_SCRIPT" edit --brief '<approved brief>' --image '<path>' --prompt '<instruction>'
```

Use one image path unless the target provider has been validated for multi-image edits.

## `invalid_output_format`, `invalid_background`, `invalid_moderation`, `invalid_input_fidelity`

Cause: an enum-valued option received a value outside its allowed set. Validation happens locally before any provider call, so no quota is consumed.

Allowed values: `--output-format png|jpeg|webp`, `--background auto|transparent|opaque`, `--moderation auto|low`, `--input-fidelity low|high`.

## `invalid_output_compression`

Cause: `--output-compression` was outside `0`-`100` or was not an integer.

Fix: use an integer in range. It only applies to `jpeg` and `webp`; ImageForge drops it for `png` because png is lossless and the provider ignores it.

## `invalid_n`

Cause: `--n` was outside `1`-`10`. The provider gateway allows a larger range, but this local cap prevents an accidental large spend in one call.

Fix: request up to 10 images per call, or run the command again for more.

## Transparency requested but PNG is opaque

Cause: this NextAI deployment accepts `--background transparent` and still returns an opaque image, echoing `background: auto`.

Fix: no client-side workaround exists. Report the outcome honestly rather than claiming transparency was applied, and treat a real alpha channel as unavailable until the upstream supports it.

## Setup page cannot list models

Cause: the 获取模型 button calls `POST /v1/models` from the local setup server using the API key in the form (or the already stored key). It fails when the key is wrong, the key has no access to any image model, or the network is unavailable. Failures are shown inline and never block saving.

Fix:

- Confirm the key is valid and was created at `https://www.nextairouter.com`.
- Confirm the token has image-model permission; the gateway returns `该令牌无权访问模型` for models outside the token's scope.
- Type the model name manually — the field accepts free text, and the picker is only a convenience.

## `network_error`

Cause: DNS, TLS, routing, proxy, firewall, timeout, or provider availability failure.

Fix:

- Confirm the machine can reach the fixed NextAI Code API URL.
- Check corporate proxy, VPN, and firewall settings.
- Retry after transient provider/network incidents.
- Run `doctor` to verify the configured host without exposing secrets.

## `version_check_unavailable`

Cause: Git or remote origin lookup is unavailable.

Fix:

```bash
python3 "$IMAGE_FORGE_SCRIPT" check-version
```

If it remains unavailable, continue using generate/edit normally. Version checks degrade safely and never block generation or editing.

When an update is available, the version-check output includes:

```bash
npx skills update image-forge
```

## Secret hygiene checklist

- Store API keys only in `~/.config/image-forge/secrets.json` or `IMAGE_FORGE_API_KEY`.
- Keep `.image-forge/config.json` limited to non-secret project configuration.
- Never commit API keys, shell history snippets, logs, or agent replies containing secrets.
- Never store API keys in `image-forge/`, `SKILL.md`, reference docs, agent metadata, or Git.
- Redact provider errors before sharing them.
