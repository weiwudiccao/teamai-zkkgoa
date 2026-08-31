# zkkgoa Release Scenarios

These rules apply to coding, testing, build, release, sync, debugging, source governance, and three-way verification for `zkkgoa` and `qifuaiqd`.

## 场景1

Use for production hotfixes and small live issues.

Required flow:

1. Confirm the real production issue with read-only checks.
2. Back up the relevant production source before writing.
3. Apply the smallest production-server fix.
4. Verify on the real production page, API, or business flow.
5. Sync the same patch back to the test server.
6. Update GitHub from the verified server source.
7. Run production, test, and GitHub three-way verification.

Do not fully overwrite production from the test server in `场景1`.

## 场景2

Use for new modules, larger changes, cross-file work, frontend/backend linked work, or cross-business-flow work.

Required flow:

1. Develop, test, build, and accept on the test server first.
2. Back up production before release.
3. Update production only from the verified test-server source.
4. Build or verify on production itself.
5. Verify real production pages, APIs, and flows.
6. Update GitHub from the corresponding verified server source.
7. Run production, test, and GitHub three-way verification.

Never complete production source directly from GitHub, a release branch, or a local directory in `场景2`.

Do not sync test business data, knowledge bases, Agent Profile, Message Channel, face-attendance configuration, or production business configuration by default. Sync only source code and required release artifacts.

## 场景3

Use for test-server-only development and verification.

Allowed:

1. Modify, build, restart, and verify only on the relevant test server.
2. Report the test-server validation result.
3. Stop and wait for explicit user confirmation before `场景4`.

Forbidden:

1. Do not update production.
2. Do not sync test-server code to production.
3. Do not commit, push, or update GitHub.
4. Do not claim the work is released, online, or three-way verified.

## 场景4

Use only after the user confirms that `场景3` has passed acceptance on the test server.

Required flow:

1. Confirm the accepted `场景3` test-server state.
2. Back up current production source.
3. Update production from the accepted test-server source.
4. Build or verify on production.
5. Verify real production pages, APIs, and flows.
6. Update GitHub from the corresponding test-server source.
7. Run production, test, and GitHub three-way verification.

Never start `场景4` without confirmed `场景3` acceptance.
