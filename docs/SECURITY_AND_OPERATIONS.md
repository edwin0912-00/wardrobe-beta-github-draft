# Security and operations

## Secrets

Never commit:

- provider API keys or CLI auth stores;
- demo PINs and session secrets;
- God View keys;
- browser cookies or profiles;
- runtime databases, uploads, private originals or paid-provider receipts containing private metadata;
- `.env` files with real values.

Use environment variables or a deployment secret store. Documentation should name variables but never include values.

## Access layers

### Demo access

`ZEELY_DEMO_PIN` enables the application PIN gate. The session cookie is HTTP-only, strict same-site and secure by default.

### Browser profile ownership

Anonymous profiles are bound to the browser session. They are not a substitute for user accounts, but they prevent ordinary cross-browser access to saved looks.

### God View

God View intentionally crosses session boundaries for testing. It must be read-only and separately gated. `ZEELY_GOD_VIEW_OPEN_TESTERS=true` is acceptable only on a controlled engineering beta.

### Real-time Look

The camera route requires explicit privacy and cost consent. Tokens are short-lived and server-issued. Camera teardown must be verified for close, Escape, page hide and error paths.

## Asset privacy and caching

- original and master assets: `private, no-store`;
- explicit preview derivatives may use bounded private caching;
- public API errors must not return absolute paths;
- prompts use logical reference labels rather than local filenames;
- logs and SSE projections redact tokens, paths and provider metadata.

## Deployment rules

1. Build from an exact commit SHA.
2. Run focused tests and the complete release gate.
3. Record a release receipt.
4. Deploy through the repository deployment tool, not by copying files manually.
5. Verify `/api/health` reports the intended SHA.
6. Perform browser smoke against the public hostname.
7. Do not equate a healthy tunnel with a healthy application host.
8. Keep beta and future main release identities independent.

## Current security work

As of the audited commit, `npm audit` reports two high-severity findings in `@fastify/static` and transitive `brace-expansion`. Resolve them with compatible upgrades and regression tests; do not use a forced major update without reviewing behavior.

