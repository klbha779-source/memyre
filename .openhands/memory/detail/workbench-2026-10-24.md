# ESCaiDO Workbench — 2026-10-24 execution session

## The container reset destroys two things that are NOT in the repo

1. `~/.escdo/providers.env` — the NVIDIA + Gemini API keys. Every keyed model then
   returns `missing API key for provider '<x>'`. Kilo needs no key and keeps the app
   alive. This has now happened twice (an earlier wipe of `/tmp` broke uvicorn).
   `start.sh` self-heals pip deps but CANNOT restore keys.
2. `/tmp` and pip packages — handled by `requirements.txt` + `start.sh`.

Restore recipe is in `.runtime/restore_keys.md`.

## Root cause of the site builder returning 163-char stubs with `model=None`

`providers.call_model` returned `ok: True` for an EMPTY model response, so
`call_with_failover` stopped on a model that said nothing. Fixing it (empty body =
failure) took generated pages from 3.8KB to 30-45KB with 6-14 sections and 9-21
buttons. Two lessons: always treat an empty LLM body as a failure, and beware
`asyncio.gather(..., return_exceptions=True)` — it silently hid this for a whole
session, surfacing only as `previews: []`.

## Duplicate FastAPI routes shadow silently

`server.py` had TWO `@app.delete("/api/models/{mid}")`; the second, with no checks
at all, won. Grep for duplicate route decorators before trusting an endpoint.

## Owner auth design that worked

Middleware requires a session cookie for `/api` and WebSocket paths. Public =
static assets, `/`, `/api/health`, `/api/auth/status`, and sandboxed
`/preview/{id}`. Token lives in `.runtime/owner_token.txt` (gitignored), never in
the bundle. The bridge mints a short-lived single-use pairing code via
`POST /api/auth/pairing` and verifies it BEFORE `ws.accept()`.

## Generated HTML must be served with CSP `sandbox allow-scripts`

Plus `connect-src 'none'` and `form-action 'none'`. This gives the response an
opaque origin so model-generated scripts cannot read the owner session cookie or
call the API, while still rendering and running their own local interactions.

## Model deletion protection

Keyed off `backups/model_roster_before.json`. Roster sha256 = `733e33360f11c8bf`,
13 models, and it must stay identical — the brief forbids changing existing model
records. Env override: `ESCDO_ALLOW_MODEL_DELETE=1`. New routing is added as an
appended `KEYLESS_TAIL` in `providers.py`; deleting that dict restores old order.

## UI lesson

The app fired `/api/conversations` and `/api/settings` BEFORE login, so every page
load logged two failed requests. Any auth-gated SPA must gate its bootstrap effect
on the auth check, not merely render a login screen.

## Testing setup that works here

Playwright with `executable_path="/usr/bin/chromium"` and args
`["--no-sandbox","--disable-dev-shm-usage","--disable-gpu"]`. Measure
`document.documentElement.scrollWidth - window.innerWidth` for horizontal overflow,
and count failed responses via `page.on("response")` — that is how the pre-auth 401s
surfaced. The public URL `https://work-1-dhtwigrnoupljkbx.prod-runtime.all-hands.dev`
is a real host: `/`, `/api/health`, `/api/auth/status` all return 200.

## Honest status at end of session

Done and tested: owner auth, bridge pairing, HTML isolation, model deletion
protection, zero-budget guard, empty-response fix, four real design directions,
standalone build-tab removal, 20/20 responsive regression (360/390/768/1440).

NOT done: streaming, persistent task worker, official MCP SDK migration, PWA
notifications, Iraqi TTS evaluation (no sample was ever produced or heard — do not
claim otherwise), SSRF/CSRF/file-upload hardening, EXE/APK, Kaggle/LTX flow,
safe device-cleanup workflow. Nothing was deployed.


## Vercel audit corrections (2026-10-24, second session)

**I was wrong about WebSockets.** I asserted from memory that WS cannot run on
Vercel Functions. The official page `https://vercel.com/docs/functions/websockets`
exists (last updated 2026-08-10) and lists FastAPI with ASGI. It also says WS
connections close when the Function hits its max duration (300s on Hobby), the
client must reconnect and resubscribe, and durable state must go in an external
store because in-memory variables are not usable. Fluid compute bills Active CPU
only, so an idle socket does not burn CPU. LESSON: never classify a platform
capability from memory — fetch the docs page first.

**Chromium is possible too.** Vercel Container Runtime runs any OCI image via
`Dockerfile.vercel` and is "available in Beta on all plans". There is also
"Daemons" for continuously-running containers. So neither WS nor Chromium is
impossible; the blocker is the allowance.

**The decisive numbers (from https://vercel.com/docs/functions/usage-and-pricing):**
Hobby includes 4 hours Active CPU, 360 GB-hrs Provisioned Memory, 1M invocations
per month. A continuously-running container therefore gets 360/730 = **0.49 GB**
of memory for the entire monthly allowance, and 4 CPU-hours is ~20s of CPU per
hour. A measured site build takes 115s of real work = ~0.6% of the monthly
allowance per build. VERDICT: a static UI link is free and easy; a permanent 24/7
agent is NOT feasible on the Vercel free tier. Other Hobby limits: 200 projects,
100 deployments/day, 300s function duration, 2GB memory, 250MB unzipped.

**Stale-build root cause (explains "I don't see the new version"):** `@app.get("/")`
served `index.html` with NO `Cache-Control` header, so browsers applied heuristic
caching and pinned to an old asset bundle. Fixed with
`Cache-Control: no-cache, no-store, must-revalidate` on index.html while hashed
assets stay immutable. The owner's public URL was ALWAYS serving the newest bundle
(verified in a real browser) — it was a client cache problem, not a deploy problem.
Always check `Cache-Control` on index.html before believing a deploy didn't land.

**Deployability changes needed for any static host:** `api.ts` had `const BASE = ''`
hardcoded (same-origin only) -> now reads `VITE_API_BASE`; requests send
`credentials: 'include'`; login cookie switches to `SameSite=None; Secure` when
`ESCDO_ALLOWED_ORIGINS` is set; CORS is opt-in and never a wildcard. Added
`src/vite-env.d.ts` declaring `ImportMetaEnv` (TS fails without it). Added
`vercel.json` (builds workbench/ui, SPA rewrite, cache rules) and `.vercelignore`.

**Owner password is `ESCAESCA2006`** in `.runtime/owner_token.txt` (mode 600). It is
only 12 chars and was in chat, so login throttling was added: 6 failures then a
300s per-client lockout (`ESCDO_LOGIN_MAX_FAILS`, `ESCDO_LOGIN_LOCK_SECONDS`).
Verified: attempts 1-5 -> 401, 6-7 -> 429, correct password -> 429 while locked.

**Not a deployment token:** the owner pasted a Vercel KMS issuer ID + public key
twice. That is for signing tokens, not for creating projects. It cannot deploy.
A real deployment needs a Vercel account token (or `vercel login`).

## Vercel deployment — what actually worked (2026-10-24, third session)

**LIVE:** production `https://escaido-workbench.vercel.app`, preview
`https://escaido-workbench-giicx8h5y-klbha779-sources-projects.vercel.app`.
Vercel account `klbha779-source` / team `klbha779-sources-projects`, project
`escaido-workbench` (prj_LyE9h6CTAf1oEoIXq43M2GkGoBYE). Token in
`~/.escdo/vercel.env` (600). Vercel CLI installed to `~/.local` (global npm
install fails: permissions) — `npm install -g vercel --prefix ~/.local`, then
`export PATH="$HOME/.local/bin:$PATH"`. Always rely on the env var; never pass
`--token` (it would appear in `ps` and logs).

**Four separate blockers, in the order they bit:**
1. `vercel.json` set `buildCommand` to `cd workbench/ui && ...` plus
   `outputDirectory: workbench/ui/dist`. Vercel runs buildCommand from the project
   root and then looks for outputDirectory there, so it reported
   `STATIC_BUILD_NO_OUT_DIR: No Output Directory named "dist"`. Fix: set the
   project's `rootDirectory` to `workbench/ui` via the API and drop both keys,
   letting Vercel's Vite defaults apply.
2. `vite.config.ts` had `base: '/ui/'` and `outDir: '../static/ui'` (for FastAPI).
   Vercel needs `base: '/'` and a local `dist/`. Made both conditional on
   `process.env.VERCEL` so the local deployment is untouched.
3. **Deployment Protection** defaulted to `all_except_custom_domains`, so every
   preview URL returned Vercel's own Next.js SSO page (`/_next/...`), not the app.
   Symptom to recognise: the served HTML has `data-dpl-id` and `/_next/static`.
   Fix: `PATCH /v9/projects/{name}` with `{"ssoProtection": null}`.
4. **The subtle one.** `VITE_API_BASE` was correctly baked into the bundle, yet
   login still 404'd. Cause: only `lib/api.ts` used the `BASE` constant; eleven
   other call sites in eight files called `fetch('/api/...')` directly, INCLUDING
   the login request in `Login.tsx`. Those relative URLs hit the Vercel domain.
   Fix: added `apiUrl()` and `wsUrl()` exports and converted every site.
   LESSON: when adding a configurable API base, grep for every raw
   `fetch('/api` / `EventSource` / `new WebSocket(` — one file is never enough.

**Also:** `vercel link` writes `.env.local` containing a real
`VERCEL_OIDC_TOKEN`. It was already covered by the `.env*` rule, but always check
`git check-ignore` after linking. `.vercel/` is gitignored too.

**Backend side:** cross-origin needs `ESCDO_ALLOWED_ORIGINS` (comma separated) to
enable CORS and switch the session cookie to `SameSite=None; Secure`. Verified
with an OPTIONS preflight returning the right allow-origin. The preview alias
changes on every deploy, so either use the stable production domain in the
allow-list or re-add each new preview URL.

**Model roster stayed at 733e33360f11c8bf** throughout; 26 conversations and
10268 messages intact.
