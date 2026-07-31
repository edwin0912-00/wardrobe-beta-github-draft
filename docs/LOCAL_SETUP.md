# Installation, configuration and provider connections

This guide covers a minimal UI-only installation, a complete engineering installation, and every provider connection represented in the current beta architecture.

## 1. Supported operating modes

| Mode | What works | External connection |
|---|---|---|
| UI and catalog review | Pages, catalogs, saved-state UI, static assets | None |
| Mock/replay verification | State machine, receipts, fixtures and deterministic tests | None |
| Real image pipeline | Avatar, clothing, background and Fashion Shoot generation | Higgsfield CLI or OpenRouter API |
| Semantic QA | Garment extraction and visual QA | Codex CLI or OpenRouter API |
| Fashion Video | Reference-conditioned video jobs | Higgsfield CLI; OpenRouter Seedance fallback where the request remains compatible |
| Real-time Look | Short live generative camera session | FAL API / Lucy |
| Interactive agent enhancement | Agent-driven tools or experiments | Optional Magnific MCP |

Magnific must not be confused with the beta server provider. The repository contains MCP-era contracts and some style-unit provenance mentioning Magnific, but no accepted `MagnificImageProvider` is wired into `src/web/generation-provider.js`.

## 2. System dependencies

Required:

- Git 2.40+;
- Node.js 22+;
- npm;
- `ffmpeg` and `ffprobe`;
- standard HTTPS certificate store;
- enough free disk for uploaded images, derivatives, generated candidates and videos.

Recommended for development:

- `curl`;
- `jq`;
- a Chromium-based browser;
- build tools required by native Node dependencies if a prebuilt `sharp` binary is unavailable.

### macOS with Homebrew

```bash
xcode-select --install
brew install git node@22 ffmpeg jq
brew link --overwrite node@22
```

Confirm:

```bash
git --version
node --version
npm --version
ffmpeg -version | head -1
ffprobe -version | head -1
```

### Ubuntu or Debian

Install Node.js 22 from the NodeSource repository or another trusted Node distribution, then:

```bash
sudo apt-get update
sudo apt-get install -y git ffmpeg jq build-essential ca-certificates
node --version
npm --version
```

Do not use an old distribution Node package if it resolves below Node 22.

## 3. Clone and install

SSH:

```bash
git clone git@github.com:<owner>/<private-repository>.git
```

HTTPS:

```bash
git clone https://github.com/<owner>/<private-repository>.git
```

Then:

```bash
cd <private-repository>
npm ci
```

`npm ci` is preferred because it installs exactly the dependency tree recorded in `package-lock.json`.

## 4. Minimal local start

```bash
export ZEELY_RUNTIME_ROOT="$PWD/runtime"
export ZEELY_COOKIE_SECURE=false
npm run app
```

Open `http://127.0.0.1:4173`.

`ZEELY_COOKIE_SECURE=false` is only for direct local HTTP. Do not set it on an HTTPS deployment.

Health check:

```bash
curl -fsS http://127.0.0.1:4173/api/health | jq
```

## 5. Local environment file

For development, create a file outside Git such as `.env.local`:

```bash
ZEELY_RUNTIME_ROOT=/absolute/path/to/private/runtime
ZEELY_GENERATION_PROVIDER=higgsfield
ZEELY_VLM_PROVIDER=codex
ZEELY_COOKIE_SECURE=false
PORT=4173
```

Load it only into the current shell:

```bash
set -a
source .env.local
set +a
npm run app
```

Ensure `.env.local` remains ignored. Never put real values in a committed example file.

## 6. Codex CLI

Current use:

- default semantic VLM/QA evaluator;
- optional test-only image-generation worker;
- host for optional MCP connections such as Magnific.

Install the official Codex CLI according to the OpenAI distribution available to the host, then authenticate:

```bash
codex login
codex login status
codex --version
```

Use Codex for semantic QA:

```bash
export ZEELY_VLM_PROVIDER=codex
```

The image-generation path through Codex is explicitly test-only in current code:

```bash
export ZEELY_GENERATION_PROVIDER=codex-imagegen-test
export ZEELY_ENABLE_CODEX_IMAGEGEN_TEST_ONLY=true
export ZEELY_CODEX_IMAGEGEN_TIMEOUT_MS=360000
npm run app
```

Startup verifies ChatGPT login and image-generation capability. Do not present this route as the production provider without changing and reviewing the product contract.

## 7. Higgsfield CLI

Current use:

- primary image transport in the default server configuration;
- primary Fashion Video transport;
- async create → journal → wait/resume behavior;
- GPT Image 2, Nano Banana 2 and Nano Banana Pro through fixed Higgsfield selectors.

After installing the official Higgsfield CLI, authenticate with browser OAuth:

```bash
higgsfield auth login
higgsfield account status --json
higgsfield --version
```

Select it:

```bash
export ZEELY_GENERATION_PROVIDER=higgsfield
export ZEELY_VLM_PROVIDER=codex
npm run app
```

Useful diagnostics:

```bash
higgsfield model list
higgsfield model list --video
higgsfield workflow list
higgsfield account status
```

The application executes the CLI as an argument array without a shell. Credentials remain in the CLI-owned host store, not the repository.

CLI versions can differ in account/workspace behavior. Pin the version proven on the target host and verify `account status` after upgrades instead of installing an unbounded latest version during release.

## 8. OpenRouter API

Current use:

- alternative image-generation provider;
- alternative VLM/semantic evaluator;
- Seedance video fallback where the request does not require a motion-video reference that OpenRouter would drop.

Configure the key in the host environment or deployment secret store:

```bash
export OPENROUTER_API_KEY='<secret-from-secure-store>'
```

Image generation through OpenRouter:

```bash
export ZEELY_GENERATION_PROVIDER=openrouter
export ZEELY_VLM_PROVIDER=openrouter
npm run app
```

Mixed provider example:

```bash
export ZEELY_GENERATION_PROVIDER=higgsfield
export ZEELY_VLM_PROVIDER=openrouter
npm run app
```

Optional variables:

```bash
export OPENROUTER_BASE_URL='https://openrouter.ai/api/v1/chat/completions'
export ZEELY_OPENROUTER_VLM_MODEL='<pinned-vision-model-id>'
export ZEELY_OPENROUTER_SCENE_MODEL='<pinned-vision-model-id>'
export ZEELY_OPENROUTER_REFERER='https://app.adbraze.com'
export ZEELY_OPENROUTER_TITLE='adbraze'
```

The current image route maps to pinned provider IDs in code. Do not send a moving alias such as `auto` as evaluator authority. OpenRouter startup checks that a key is present but deliberately does not make an external paid/network probe.

Video uses the `bytedance/seedance-2.0` OpenRouter route only through the video provider router. A source frame must be exposed through a private, bounded HTTPS capability:

```bash
export ZEELY_PUBLIC_HTTPS_ORIGIN='https://<private-beta-host>'
```

## 9. Magnific MCP and API boundary

Magnific can be connected to an interactive Codex agent through OAuth MCP:

```bash
codex mcp add magnific --url https://mcp.magnific.com
codex mcp login magnific
codex mcp get magnific
codex mcp list
```

OAuth opens a localhost callback on the machine running Codex. For remote operation, the authorization URL must be opened by the operator and the callback returned to that same host/session.

What this connection enables:

- an interactive agent can call Magnific tools while the MCP session exists;
- research, enhancement or one-off controlled generation can be performed by that agent;
- resulting assets may enter the project only with hashes, provenance and QA receipts.

What it does not enable today:

- the long-running Node server cannot inherit an agent's MCP session;
- setting `ZEELY_GENERATION_PROVIDER=magnific` is unsupported;
- there is no committed Magnific image provider adapter;
- Magnific is not an automatic fallback in the current beta.

Magnific also documents a REST service that uses an `x-magnific-api-key`, but an API key alone does not create project support. A production connection would require a new provider adapter implementing the existing generation contract, idempotency, download validation, prompt privacy, receipts and tests. Until that adapter exists, keep any Magnific API key out of the app environment.

## 10. FAL API and Real-time Look

Current use: short-lived Lucy 2.5 real-time camera sessions.

```bash
export FAL_KEY='<secret-from-secure-store>'
npm run app
```

The server exchanges the long-lived key for a short-lived browser token scoped to the approved live model. The browser must not receive `FAL_KEY` itself.

Real-time Look additionally requires:

- an approved saved look;
- explicit privacy consent;
- explicit cost consent;
- camera permission;
- a hard session timeout;
- teardown on close, Escape, page hide and errors.

## 11. Fashion Video reference assets

The video service needs the committed manifest plus a runtime root containing the exact hash-bound reference files:

```bash
export ZEELY_VIDEO_REFERENCE_ROOT='/absolute/path/to/video-reference-assets'
export ZEELY_PUBLIC_HTTPS_ORIGIN='https://<private-beta-host>'
```

The service fails closed when the approved look, style video, playback derivative or motion reference does not match its declared SHA-256.

## 12. Demo PIN and sessions

```bash
export ZEELY_DEMO_PIN='123456'
export ZEELY_SESSION_SECRET="$(openssl rand -hex 32)"
npm run app
```

The PIN must contain 4–12 digits. The session secret must contain at least 32 characters.

## 13. God View

God View is disabled by default.

Separate protected access:

```bash
export ZEELY_GOD_VIEW_KEY="$(openssl rand -hex 32)"
export ZEELY_GOD_VIEW_SESSION_SECRET="$(openssl rand -hex 32)"
```

Controlled beta tester mode:

```bash
export ZEELY_GOD_VIEW_OPEN_TESTERS=true
```

Do not enable open-tester mode on a customer deployment.

## 14. Additional application configuration

```bash
# Runtime and server
export ZEELY_RUNTIME_ROOT='/private/runtime/path'
export PORT=4173
export ZEELY_PUBLIC_HTTPS_ORIGIN='https://beta.example.com'

# Product routes
export ZEELY_LOOK_IMAGE_ROUTE=quality
export ZEELY_FASHION_SHOOT_QA_MODE=review

# Optional output finishing
export ZEELY_FRAME_OVERSAMPLE=off
export ZEELY_FRAME_GRAIN=0

# Monitor
export MONITOR_PORT=4174
export ZEELY_APP_HEALTH_URL='http://127.0.0.1:4173/api/health'
export ZEELY_SUPERVISOR_AGENT=false
```

Do not change route or QA settings just to make a failing result pass. Configuration changes must preserve the same evidence and acceptance contract.

## 15. Verification

Focused checks that require no paid generation:

```bash
npm run test:conditioning
npm run test:runner
node --test --test-concurrency=2 test/web/god-view*.test.js
node --test --test-concurrency=2 test/video/*.test.js
```

Complete project gate:

```bash
npm test
npm run verify:contracts
npm run verify:canon
npm run verify:output
```

Check dependencies:

```bash
npm audit
```

The resource preflight intentionally refuses a large suite when host memory, swap or disk conditions are unsafe. Diagnose the host rather than removing the guard from a release workflow.

## 16. Provider readiness checklist

```bash
codex login status
higgsfield account status --json
codex mcp list
test -n "$OPENROUTER_API_KEY" && echo 'OpenRouter key present'
test -n "$FAL_KEY" && echo 'FAL key present'
curl -fsS http://127.0.0.1:4173/api/health | jq
```

Presence is not the same as a successful paid smoke. Record the exact provider, model, request/job ID, immutable input hashes, result hash and QA receipt for each real generation test.

## 17. Secret-handling rules

- never paste real keys into README files, issues, commits or agent logs;
- never commit CLI credential stores or OAuth callback data;
- pass credentials through environment variables or a deployment secret manager;
- rotate any key accidentally exposed in chat or Git history;
- do not store ChatGPT/Codex browser authentication inside the project;
- do not include absolute local paths in provider prompts or public API responses.

## 18. Runtime recovery warning

`ZEELY_RUNTIME_ROOT` is load-bearing. Changing it points the application at a different profile, run, scene, shoot and video tree. Existing data may still be intact at the previous root even when the UI appears empty. Confirm and back up the effective runtime root before migration.

