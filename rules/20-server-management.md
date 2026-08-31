# zkkgoa Server Management

This file records server management rules, host aliases, paths, and verification boundaries. It must not contain secrets.

## Host Map

- Test backend: `my-cloud:/home/zkkgoa/frappe-bench`
- Production backend: `root@47.107.116.189:/home/zkkgoa/frappe-bench`
- Test frontend: `root@8.138.219.235:/var/www/html/code`
- Production frontend: `zs-new-zhinaoqiantai` / `root@8.138.244.233:/var/www/html/code`
- Legacy production frontend: `zs-zhinaoqiantai` / `root@47.107.105.79:/var/www/html/code`

Use the legacy production frontend only when the user explicitly requests it.

## Fixed Paths

- Frontend source/build directory: `/var/www/html/code`
- Frontend served root: `/var/www/html`
- Backend bench root: `/home/zkkgoa/frappe-bench`
- Backend bench command: `/home/zkkgoa/frappe-bench/env/bin/bench`
- Backend Python command: `/home/zkkgoa/frappe-bench/env/bin/python`
- Production backup root: `/test/backups`

Do not treat `/tmp/qifuaiqd-push` as the long-term source of truth. It is only a temporary transfer directory.

## Production Backend Gate

For production backend Python changes in `场景1`, the production phase of `场景2`, or `场景4`:

1. Do not use `bench restart` alone as proof that live Web traffic is running the new logic.
2. Restart the serving Web process with `systemctl restart frappe-web.service`.
3. If queue jobs, notifications, schedulers, or async workers are affected, restart the corresponding worker services too.
4. Verify `systemctl show frappe-web.service -p MainPID -p ExecMainStartTimestamp`.
5. Verify the real Gunicorn process with `ps -ef | grep 'gunicorn -b 127.0.0.1:8000'`.
6. `ExecMainStartTimestamp` must be later than the changed code file mtime.
7. Verify the real production domain with HTTP/API or browser checks.

`bench execute` starts a fresh Python process and is not proof that the live Gunicorn Web process loaded the new code.

For destructive or workflow-advancing checks, use dry-run validation and then verify the document and audit tables were not advanced accidentally.

## GitHub Governance

- Frontend GitHub repo: `https://github.com/weiwudiccao/qifuaiqd.git`
- Backend GitHub repo: `https://github.com/weiwudiccao/zkkgoa.git`
- TeamAI rules repo: `https://github.com/weiwudiccao/teamai-zkkgoa.git`

Default GitHub push source is the verified server source, not the local workspace.

For `qifuaiqd`, push from the test frontend server source under `/var/www/html/code` using the configured server-side deploy key or URL rewrite.

For `zkkgoa`, push from the test backend server source under `/home/zkkgoa/frappe-bench` using the configured server-side `github-zkkgoa` rewrite. Preserve the `frappe-bench/...` path prefix when mapping backend files to GitHub.

Never write GitHub tokens into source files, remote URLs, shell history, logs, TeamAI rules, or reports.
