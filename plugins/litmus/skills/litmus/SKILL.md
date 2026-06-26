---
name: litmus
description: >
  Live-verify a NEB defect/fix end-to-end before marking it "Ready to test" — the mandatory
  workflow (Rawi's rule): reproduce the ticket's test steps in the REAL UI via Chrome MCP as the
  specified user, instrument the running app (network hook / Angular scope / DB relay), prove the
  defect AND prove the fix, then update Plane. Use whenever about to verify/close a NEB-E2E ticket,
  or when the user says "verify", "live test", "reproduce", "พิสูจน์", "เทสจริง", "ตรวจสอบ defect".
  Backend curl / DB-only / "deployed so it's fixed" do NOT count as verification.
---

# Live-verify a NEB defect (the verify-before-Ready discipline)

**Hard rule (Rawi):** never move a ticket to *Ready to test* on deploy/structural/DB evidence alone.
Reproduce the ticket's **Test step** in the real UI as the **specified user**, see the defect, then
see it gone after the fix. If you cannot reproduce (wrong user/access/data), say so and DO NOT mark Ready.

## 0. Read the ticket first
- Get the ticket's **User**, **Test step**, **Actual**, **Expected**, and the **Screen URL** (often embedded).
- Plane API (same-origin fetch on `projects.oneweb.tech`): board `neb-e2e`, projectId
  `68f36f0e-bffa-4cf7-a540-ca83e3060557`. Fetch issue by `?sequence_id=N`; description via the rendered
  issue page (`/neb-e2e/browse/NEBE2-N/` + get_page_text) — the raw `description_html` is often redacted
  by the MCP as `[BLOCKED: ...]` when it contains a URL/query string.

## 1. Chrome MCP + login as the ticket's user
- `tabs_context_mcp {createIfEmpty:true}` → navigate `https://uat-neb.bb.go.th/app`.
- Switch user: avatar (top-right) → **เข้าด้วยบัญชีอื่น** → on the login page **fill the email only**,
  then **clear the password and ask the user to type `P@ssw0rd` + click เข้าสู่ระบบ**. NEVER type/submit
  the password yourself (prohibited). Wait for "เข้าให้แล้ว".
- After login, **pick the NEB system** if prompted (avatar shows NEB/SUPPORT/etc.) — wrong system shows the
  wrong menus. Confirm identity via `/app/api/userinfo` → `userInfo.{AGENCY_ID,MINISTRY_ID,USER_NAME}`.
- **Access check:** if the ticket's user lacks the menu (e.g. search the sidebar finds nothing, or only
  unrelated modules show), that's an access gap — comment on the ticket that the test user is wrong /
  needs the menu granted, and DO NOT guess-fix. (I cannot modify permissions myself.)

## 2. Reproduce — follow the test steps exactly
- Navigate the exact screen URL from the ticket (OneWeb apps are at `/neb-<app>/<SCREEN>?...`; or via the
  portal menu, which embeds OneWeb in an **iframe** — access it via the iframe's `contentWindow`/`contentDocument`).
- Click through the steps; screenshot the failing state. Confirm Actual == ticket.

## 3. Instrument the running app (pick what fits)
- **Network hook — MUST hook BOTH fetch AND XHR.** `req_microflow` uses `fetch`, not XHR; an XHR-only hook
  misses it. Filter on `/microflow\/service/` or `/neb-...-service/api/`. Capture request body + response.
  Parse OneWeb payloads: `o.object → Object.values()[0].request` (a JSON string) → the real fields/ACTION.
- **Angular scope** (OneWeb is AngularJS): find the list/data by walking `angular.element(el).scope()` for
  the array (e.g. `dataList`); read raw rows to check dups/values without trusting the rendered page.
- **Run the real deployed function with dialogs stubbed:** to exercise an exact code path when the UI flow
  is blocked, override `window.alertModal` to auto-confirm (`if(type==='confirm') cb({isConfirmed:true})`),
  call the page's real function (e.g. `publishWSS01_01_SC01_T01()`), and read the captured payload/result.
  Verify the deployed JS actually contains the fix: `fn.toString().includes('...')`.
- **Unique var names every call:** the javascript_tool shares a scope across calls — reusing `out`/`os`/`all`
  throws "already declared". Suffix vars per call (`r57b`, `oSend60`, …).

## 4. DB verify (the source of truth) — UAT / SIT / PROD
- Relays exist as pods (`kubectl --kubeconfig ~/.kube/<cfg> -n budget-bb port-forward pod/<relay> 152NN:1521`):
  - UAT: `~/.kube/uat-dr-config`, pod `ora-relay` → 15212 (nebdb, APP / `P@ssw0rd`).
  - PROD: `~/.kube/prod-config --insecure-skip-tls-verify`, pod `ora-relay-prod` → 15213 (SEPARATE DB).
  - SIT: create a socat relay if none (canonical for procs).
- Query with **python3.11 oracledb** (`connect(user,password,dsn="127.0.0.1:152NN/NEBDB",disable_oob=True)`;
  `fetch_lobs=False` style). sqlplus 19 often hits ORA-12514 here; python works.
- **Procs/functions: trust SIT `ALL_SOURCE` as canonical, not the repo `.sql`** (repo lags — Rawi edits the DB
  first). Read the live source, compare UAT vs SIT.
- **Always tear down**: `pkill -9 -f "kubectl.*port-forward"` when done.

## 5. Prove the fix end-to-end, then mark Ready
- Show: payload now carries the right value / list now de-duped / DB column now correct (e.g. re-query the
  exact row). Capture the SUCCESS response AND the persisted DB state.
- If you created/changed test data to verify, **revert it** (and back up before any destructive DB change:
  `CREATE TABLE APP.BK<ticket>_... AS SELECT ...` so it's reversible; confirm rows are childless first).
- Update Plane: POST `/api/workspaces/neb-e2e/projects/<proj>/issues/<id>/comments/` `{comment_html}` +
  PATCH state. X-CSRFToken from `/auth/get-csrf-token/`. Ready-to-test state = `12bbf318-2526-4e1c-ad98-903e2123053b`.
  Comment with the concrete evidence (captured payload, before/after counts, DB values).

## Gotchas seen in the field
- **Tab group resets** silently drop the Chrome session (re-login needed). Re-create with `createIfEmpty:true`.
- **Chrome "not connected"** is usually transient — wait ~12s and retry the same call.
- OneWeb search boxes sometimes don't filter on type — verify counts from the **scope**, not the rendered page.
- Destructive shared-data changes (e.g. `M_WORKPLAN` is shared across BPN03/BPN10/…): VERIFY which is the
  keeper before deleting, back up, and confirm with the user (re-confirm on blast radius).

Related memory: `verify-defects-via-chrome-mcp-steps`, `speed-mode-no-skip-verify`, `db-procs-trust-sit-not-repo`,
`db-function-fix-workflow`, `uat-oracle-db-access`, `portal-must-pick-neb-system-first`, `oneweb-callmicroflow-hang-pattern`.
