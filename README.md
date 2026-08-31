# zkkgoa TeamAI Rules

This repository stores shared AI rules and non-secret operation notes for `zkkgoa` and `qifuaiqd`.

## Member Setup

Run inside the local project directory:

```bash
npm install -g teamai-cli
teamai init 'https://github.com:443/weiwudiccao/teamai-zkkgoa.git' --scope project --agent codex
teamai pull
teamai status
```

For other tools, replace `--agent codex` with `--agent claude`, `--agent cursor`, or another supported agent.

## Included Rules

- `rules/00-zkkgoa-work-mode.md`: language, read-only-first, secret handling, and business-state safety.
- `rules/10-release-scenarios.md`: `场景1` / `场景2` / `场景3` / `场景4` release routing.
- `rules/20-server-management.md`: host map, fixed paths, backend service effectiveness gate, and GitHub governance.
- `docs/server-management.md`: server management boundaries and release evidence checklist.

## Secret Policy

Server management can be documented here only as policy and non-secret inventory.

Do not commit passwords, private keys, GitHub tokens, cookies, session IDs, authorization headers, real `.env` values, or tokenized clone URLs.
