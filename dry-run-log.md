# GitHub Automation Dry Run Log — Friday Team Walkthrough

**Date:** 2026-10-19
**Operator account:** HeziLiu (https://github.com/HeziLiu)
**Purpose:** End-to-end dry run of the GitHub automation stack so nothing surprises us during the live demo.

> **Sandbox note:** The token used for this run cannot create repositories under the personal account `HeziLiu` (API returned `403 Resource not accessible by personal access token` for both repo creation and forking). The scratch repo was therefore created in the organization `mcpmark-eval-liuhezi`, where the token has full admin/write access. The name `gh-smoke-test` itself was **not** taken anywhere — no numeric suffix was needed. The same token-scope restriction also blocks gist creation (`POST /gists` → 403), so this log lives in the repo instead of a gist. Everything below was re-read from GitHub's API after creation, not taken from creation responses.

---

## 1. Scratch repository
- **Name:** gh-smoke-test (exact name, no suffix needed)
- **Full name:** mcpmark-eval-liuhezi/gh-smoke-test
- **URL:** https://github.com/mcpmark-eval-liuhezi/gh-smoke-test
- **Default branch:** main
- **Visibility:** public
- **README.md:** initialized (verified present at repo root)
- **Verified via:** repository contents listing + repository search API (confirmed default branch `main`, README blob present, admin permissions).

## 2. Tracking issue
- **Title:** "Smoke test tracking issue" (exact)
- **Number:** #1
- **URL:** https://github.com/mcpmark-eval-liuhezi/gh-smoke-test/issues/1
- **State:** open
- **Body:** "Dry run of the GitHub automation stack before the Friday team walkthrough. This issue tracks the end-to-end smoke test: repo creation, issue, pull request with Copilot review, a manual Actions workflow run, and a secret-scanning check. All artifacts are disposable and live only in this scratch repository."
- **Verified via:** `get_issue` — number 1, state open, title exact, body as written.

## 3. Pull request + Copilot review
- **Title:** "Smoke test PR: add automation dry run log"
- **Number:** #2
- **URL:** https://github.com/mcpmark-eval-liuhezi/gh-smoke-test/pull/2
- **Branch:** `smoke/dry-run` (from `main`)
- **State:** open, mergeable, mergeable_state clean
- **Diff (re-read from the PR files API):** exactly one file changed — `docs/smoke.md` added (+1/−0), containing exactly the line:
  `Automation dry run log for the Friday walkthrough.`
- **Copilot review request status:** request submitted successfully (API returned 204 No Content). As of final verification, **no Copilot review has been submitted yet** — the PR's reviews list is empty, i.e. the review is requested but still pending. This is the expected early state; Copilot reviews typically land minutes after the request.
- **Verified via:** `get_pull_request`, `get_pull_request_files`, `get_pull_request_reviews`, `get_pull_request_status`.

## 4. Actions workflow (manual trigger, run once)
- **File:** `.github/workflows/smoke.yml` on `main` (single job `greet` on `ubuntu-latest` that echoes a greeting)
- **Workflow run URL:** https://github.com/mcpmark-eval-liuhezi/gh-smoke-test/actions/runs/39289644642
- **Run ID:** 39289644642 (run #1, attempt 1)
- **Trigger:** workflow_dispatch on `main` — triggered exactly once (total_count of runs for this workflow: 1)
- **Result:** completed / **success**
- **Started / finished:** 2026-10-19T16:55:14Z → 2026-10-19T16:55:24Z
- **Job:** `greet` (job id 118186655338) — all 3 steps succeeded ("Set up job", "Greeting", "Complete job")
- **Log line printed:** `Hello from the automation dry run!` (confirmed in the job log)
- **Usage / billable numbers (re-read from the run-usage API):**
  - Billable platform: **UBUNTU**
  - Billable jobs: **1** (job run id 118186655338, duration_ms: **0**)
  - Billable total_ms: **0**
  - Overall run_duration_ms: **10000** (~10 s wall clock; the sub-second single echo step results in 0 ms billable)

## 5. Secret-scanning check
- **Outcome:** the secret-scanning alerts API is **not accessible with this token** for this repository — both the alert list (`GET /repos/.../secret-scanning/alerts`) and a direct alert lookup returned `403 Resource not accessible by personal access token`. This is a token-scope limitation, not evidence of zero alerts.
- **Plain-language result for the demo:** "No secret-scanning alerts could be enumerated via the API with the current credentials — the security-events read scope is missing. Grant the token the `secret_scanning_alerts:read` (fine-grained security events read) permission to run this part of the check. The repo contains only README + docs files, so no alerts are expected, but that expectation is unverified by API."

## 6. Live security example — recent critical npm advisory
- **GHSA id:** GHSA-qx66-fv34-fjm8
- **CVE:** CVE-2026-102989
- **URL:** https://github.com/advisories/GHSA-qx66-fv34-fjm8
- **Summary:** TanStack Router — unauthenticated reflected XSS in TanStack Start server-function responses
- **Severity:** critical (CVSS 9.3, CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:H/I:H/A:N)
- **Published:** 2026-10-15 (GitHub-reviewed same day)
- **Affected npm packages and earliest patched versions (from the full advisory record):**
  - `@tanstack/start-server-core` — vulnerable `>= 1.143.12, < 1.169.39` → first patched: **1.169.39**
  - `@tanstack/react-start` — vulnerable `>= 1.143.12, < 1.168.60` → first patched: **1.168.60**
  - `@tanstack/solid-start` — vulnerable `>= 1.143.12, < 1.168.57` → first patched: **1.168.57**
  - `@tanstack/vue-start` — vulnerable `>= 1.143.12, < 1.168.56` → first patched: **1.168.56**
- **Headline package / severity / patch:** `@tanstack/start-server-core`, severity **critical**, earliest patched version **1.169.39**.
- **Fix:** upgrade to the corresponding patched package version (fix limits client-supplied server-function input to public fields and validates responses at the server boundary).

---

## Final state re-verification summary (all re-read after creation)
| Artifact | Verified state |
|---|---|
| Repo gh-smoke-test | Exists, public, default branch `main`, README present |
| Issue | #1 open, title exact, body intact |
| PR | #2 open, 1 file `docs/smoke.md`, exact line, mergeable |
| Copilot review | Requested (204 OK); no review submitted yet — pending |
| Workflow run | 39289644642, run #1, success, greeting echoed, 10 s, 0 ms billable |
| Secret scanning | Not accessible via API (403) — token scope limitation |
| Advisory | GHSA-qx66-fv34-fjm8 captured in full |

**Known deviations from plan (be ready to explain these on stage):**
1. Repo lives in the org `mcpmark-eval-liuhezi`, not under `HeziLiu` — the demo PAT cannot create/fork repos in the personal namespace (403). Name was available; no suffix used.
2. Copilot review was successfully requested but had not been *submitted* yet at final check — a normal timing artifact.
3. Secret-scanning enumeration is blocked by the current token's scope (403), so the check result is "not verifiable with this credential" rather than "no alerts".
4. The bundled log lives as this file in the repo (`dry-run-log.md`) rather than a gist, because gist creation is also blocked by the token (403).
