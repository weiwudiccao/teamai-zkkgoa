# zkkgoa AI Work Mode

Always respond in Simplified Chinese.

For work related to `zkkgoa` or `qifuaiqd`, default to server-first execution. Local files may be used for inspection, temporary patch staging, or manual assistance, but they are not the final verification target.

Before changing code, classify the task:

1. `场景1`: production hotfix or small live issue.
2. `场景2`: full feature or larger change that must be developed and accepted on the test server before production.
3. `场景3`: test-server-only development and verification.
4. `场景4`: publish already accepted `场景3` test-server code to production and GitHub.

If the user says "先不要改动任何代码" or "先深入分析", perform only read-only checks until explicit permission is given.

Never claim a fix is released, online, or source-governance complete unless the required real server, browser/API, and GitHub verification has passed for the selected scenario.

Do not write secrets into source, commands, logs, TeamAI rules, Git remotes, GitHub URLs, reports, or chat summaries. Replace all tokens, passwords, private keys, cookies, authorization headers, and session values with `[REDACTED_SECRET]`.

For production approval, workflow, payment, HR, CRM, Todo, and other business-state checks, use read-only or dry-run validation by default. Do not advance real workflow state unless the user explicitly asks for that exact operation.
