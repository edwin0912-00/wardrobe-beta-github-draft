# Product and release status

Audit date: 2026-07-31  
Audited commit: `1a4a69e701798ef771d3ab01c636946ad67a4334`  
Public beta: `https://beta.madeforthisjob.com/`

## Release identity

The public health endpoint returned HTTP 200 and the same SHA as the audited checkout:

```json
{
  "status": "ready",
  "generation": "available",
  "semantic_qa": "available",
  "runtime_status": "ready",
  "editorial_generation": "available",
  "fashion_shoot_qa_mode": "off",
  "release_sha": "1a4a69e701798ef771d3ab01c636946ad67a4334"
}
```

`ready` means the service and configured transports passed startup health. It does not prove every product path end to end.

## Block-by-block truth

### 1. Inputs, avatar, garments and master look

Implemented:

- person and garment upload;
- immutable source receipts and SHA-256 bindings;
- deterministic normalization, crops, cutouts and reference cards;
- structured extraction with `UNKNOWN` for invisible facts;
- readiness outcomes: `READY`, `REPAIRABLE`, `NEEDS_INPUT`, `INCOMPATIBLE`;
- avatar generation and semantic QA;
- clothing application and outfit QA;
- persisted master look and 30-day browser profile;
- bounded retry of only the failed stage.

Verified in this audit:

- conditioning tests: 31/31 pass;
- deterministic runner tests: 39/39 pass.

Not freshly proven in this audit:

- a new paid-provider browser run from fresh uploads through saved master look on the public beta.

### 2. Profile and pipeline life

Implemented:

- anonymous browser-owned profile;
- saved avatars and looks;
- draft persistence across refresh;
- add-clothing continuation from a saved avatar;
- job progress, events, checkpoints, retry and terminal-state projection;
- private originals separated from cacheable previews.

Engineering extension:

- God View can aggregate all tester sessions read-only when explicitly enabled;
- it is not the normal customer profile and has separate authorization.

### 3. Standard backgrounds

The live endpoint returns 16 published `std.*` presets. Runtime catalog:

1. Glass corridor at sunset
2. Amber cobblestone alley
3. Golden-hour city gloss
4. Neon night on wet asphalt
5. Concrete rooftop at sunset
6. Abandoned palace light shaft
7. Morning gallery gloss
8. Industrial brick loft
9. Sheer-curtain golden light
10. Foggy forest light shaft
11. Ocean blue hour
12. Concrete, grass and golden hour
13. Black spotlight studio
14. Taupe Rembrandt studio
15. Terracotta raking-light studio
16. White studio with window honeycomb

Contract:

- one selected background produces one standard image;
- it is not a Fashion Shoot and does not inherit a Fashion Shoot style pack;
- identity and clothing remain locked while the environment is replaced;
- scene QA covers framing, identity, garment fidelity, environment, light, anatomy and delivery.

Known issue: `config/scene-presets.json` still describes an older ten-preset set, while the release candidate catalog and runtime publish sixteen. The runtime catalog is authoritative for this beta, but the duplicate configuration should be consolidated.

### 4. Creative Universe and Fashion Shoot

Terminology:

- **Creative Universe** builds and validates an internal style unit from reference material.
- **Fashion Shoot** is the user-facing output that applies that unit to an approved master look.
- legacy filenames and routes still use `editorial` for compatibility; this is internal naming debt.

Live catalog:

- 19 visible cards;
- 17 executable modes;
- 15 complete Create Universe `shoot.*` style units;
- two ready legacy modes;
- two legacy preview-only cards blocked by incomplete reference evidence.

Each complete Create Universe style unit carries a versioned manifest, source observations, palette and a set of role-specific sheets. A Fashion Shoot returns five customer frames. The internal hero candidate is a QA checkpoint, not a sixth deliverable.

Evidence already in the repository:

- 15/15 technical hero smoke results are recorded in `docs/qa/FASHION_SHOOT_HERO_SMOKE_MATRIX_2026-07-30.md`;
- three style-atlas contact sheets show the current fifteen Create Universe units.

This does **not** prove all 75 final customer frames. In addition, the public health response currently reports `fashion_shoot_qa_mode: off`; blocking style QA therefore is not active on the live beta and must be restored before a production claim.

### 5. Fashion Video

Implemented:

- approved master-look input contract;
- three verified reference-video styles with hash-bound preview derivatives;
- verified motion reference;
- four motion modes;
- Higgsfield primary transport;
- OpenRouter Seedance fallback only after exactly three retryable Higgsfield create failures;
- no fallback when it would silently drop the required video reference;
- resumable provider jobs, download, `ffprobe`, extracted-frame QA and explicit retry;
- audio disabled by contract.

Verified in this audit: `test/video/*.test.js` passes 151/151.

Not freshly proven: a paid provider run from the current public beta to a delivered MP4. Focused tests prove the control plane, not an external provider result.

### 6. Real-time Look

Implemented:

- saved-look capability check;
- explicit privacy and cost consent;
- server-issued FAL/Lucy token route;
- short bounded camera session;
- no hidden recording contract;
- UI and local fallback structure.

Current code/config caps a live session at 15 seconds and records the provider price as $0.04 per second. Older documentation that mentions a 60-second session is stale.

Not proven:

- full granted-camera E2E on the public beta;
- one teardown test still fails around close/Escape/pagehide camera cleanup.

### 7. Engineering surfaces

God View:

- read-only aggregation across profiles and assets;
- separately authenticated or explicitly opened for beta testers;
- focused tests pass 4/4.

Live monitor:

- strict public event projection;
- REST and SSE views;
- checkpoints and stalled-stage diagnostics;
- optional supervisor process;
- redaction of filesystem paths and provider secrets.

## Verification status

Focused checks run on the audited commit:

- conditioning: 31/31;
- runner: 39/39;
- God View: 4/4;
- video: 151/151.

Complete suite result:

- official `npm test` stopped at resource preflight because host swap was 4.82 GiB against a 1.50 GiB limit;
- raw underlying suite: 988 tests, 906 pass, 82 fail.

Failure families include:

- schema canvas and fixture dimension drift;
- framing and scene-repair expectation drift;
- Fashion Shoot service tests bound to earlier hero approval/concurrency behavior;
- run-service regressions;
- Real-time Look teardown;
- release-context assumptions in a detached audit worktree.

The project should not be described as production-ready until these are classified and the real failures are fixed without weakening gates.

## Dependency status

`npm audit` currently reports two high-severity dependency findings:

- direct: `@fastify/static`;
- transitive: `brace-expansion`.

Upgrade and regression testing are required before a production release.

