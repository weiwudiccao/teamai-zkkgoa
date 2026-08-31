# Server Management Notes

This document is for AI-assisted operations on the `zkkgoa` and `qifuaiqd` systems.

## Operating Boundary

Server management can be documented in TeamAI, but only as operational policy and non-secret inventory. Keep credentials outside Git and TeamAI.

Allowed content:

1. Host aliases, public IPs, and project paths.
2. Scenario routing rules.
3. Backup, release, rollback, and verification checklists.
4. Service names and non-secret commands.
5. GitHub repository mapping and path mapping.

Forbidden content:

1. Passwords, private keys, personal access tokens, cookies, session IDs, and authorization headers.
2. Real `.env` values or production secret config values.
3. Raw credential helper output.
4. Tokenized clone URLs.
5. Any report that exposes a secret value.

## Two-Minute Preflight

Before touching a server:

1. Classify the scenario: `场景1`, `场景2`, `场景3`, or `场景4`.
2. Identify target: backend, frontend, or full chain.
3. Confirm server: test, production, or legacy production.
4. Decide whether the operation is read-only, write, restart, build, release, or GitHub sync.
5. If production write is needed, create a backup plan first.

## Release Evidence

Good evidence:

1. File hash or exact source diff on the target server.
2. Focused test, build, or compile result.
3. Service process start time later than changed file mtime when backend Python changes are involved.
4. Real domain HTTP/API or browser-login validation.
5. Production/test/GitHub hash verification when the scenario requires source-governance closure.

Weak evidence:

1. Local-only tests.
2. Seeing files in `/var/www/html/code` without checking `/var/www/html`.
3. `bench execute` alone for live Web logic.
4. Build success without checking the active bundle served by the real domain.
5. A screenshot that does not verify the resulting business state.
