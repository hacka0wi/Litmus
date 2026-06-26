---
name: litmus
description: >
  Litmus — live-verify a defect/fix end-to-end in the REAL app before you call it done. The mandatory
  discipline: reproduce the ticket's exact test steps in the running UI as the reporting user, instrument
  the live app (network hook / front-end scope / DB), prove the defect AND prove the fix, then update the
  tracker. Use whenever about to verify or close a ticket, or when asked to "verify", "live test",
  "reproduce", "prove it". Backend curl / unit-test / DB-only / "it deployed so it's fixed" do NOT count.
---

# Litmus — the live test that a fix is real

**Hard rule:** never mark a ticket *done / ready-to-test* on deploy, unit-test, or DB-only evidence.
Reproduce the ticket's **steps** in the real UI as the **specified user**, observe the defect, then
observe it gone after the fix. If you can't reproduce (wrong user / no access / missing data), say so
and DO NOT close it.

> Environment specifics — app host, tracker API + ids, DB hosts, credentials, relay pods — belong in a
> **private/local config** (e.g. agent memory or a `.env`), **never in this public skill**. The steps
> below reference them as placeholders.

## 0. Read the ticket
Pull the **user**, **test steps**, **actual**, **expected**, and the **screen URL**. If the tracker
redacts/escapes the raw description, read the rendered issue page instead.

## 1. Open the app as the ticket's user
- Drive the browser via an MCP browser tool; create/restore a tab.
- Switch account through the app's own "sign in as another user" — **fill the email, then have the human
  type the password and submit. Never type/submit a password yourself.**
- Select the correct sub-system/tenant if the app has several (wrong one shows wrong menus).
- Confirm identity from the app's `whoami`/`userinfo` endpoint.
- **Access check:** if that user lacks the menu/screen, it's an access gap — note it on the ticket and
  do NOT guess-fix. (Don't modify permissions yourself.)

## 2. Reproduce — follow the steps exactly
Navigate the exact screen; click through the steps; screenshot the failing state; confirm it matches the
ticket's *actual*. If the app embeds modules in an **iframe**, reach them via the iframe's
`contentWindow`/`contentDocument`.

## 3. Instrument the running app (pick what fits)
- **Network hook — hook BOTH `fetch` AND `XHR`.** Many SPA calls use `fetch`; an XHR-only hook misses
  them. Capture request body + response; unwrap nested payloads to read the real fields/action.
- **Front-end scope:** for AngularJS, walk `angular.element(el).scope()` to read the actual data array —
  check counts/dups/values from state, don't trust the rendered (paginated) page.
- **Exercise the real function with dialogs stubbed:** when the UI flow is blocked, override the confirm
  dialog to auto-accept, call the page's real handler, and read the captured request/result. Verify the
  deployed code actually contains the fix: `fn.toString().includes('...')`.
- **Unique variable names per eval** — a shared eval scope throws "already declared" if you reuse names.

## 4. Verify in the database (source of truth)
- Reach the DB through a relay (e.g. a `socat` pod + `kubectl port-forward`) and query with a real driver
  (e.g. python `oracledb`, `disable_oob=True`); lightweight CLIs often fail where the driver works.
- **Trust the live DB definition of procs/functions as canonical, not the repo `.sql`** if your team
  edits the DB directly — read the live source and diff environments.
- **Always tear the relay/port-forward down when done.**

## 5. Prove the fix, then update the tracker
- Show concrete before/after: payload now carries the right value / list now de-duped / the exact DB row
  is now correct. Capture both the SUCCESS response AND the persisted state.
- If you created or changed test data to verify, **revert it**; before any destructive DB change, **back
  up first** (`CREATE TABLE <bk> AS SELECT ...`) and confirm the rows are safe (e.g. childless) to touch.
- Comment on the ticket with the evidence (captured payload, before/after counts, DB values) and set the
  status. Get a human's confirmation before destructive shared-data changes (blast radius).

## Field gotchas
- Browser tab groups can reset and silently drop the session — re-create and re-login.
- "Browser not connected" is usually transient — wait ~10s and retry.
- Search boxes that don't filter on type → verify counts from app state, not the rendered list.
- Shared tables (one row used by many modules): identify the keeper, back up, and confirm before deleting.
