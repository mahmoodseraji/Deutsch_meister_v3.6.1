# Deutsch Meister 3.6.5 — test report

- Node.js suite: **72 passed, 0 failed** (`node --test tests/*.test.mjs`).
- `node --check` passes for every JavaScript file and `sw.js`.
- New regression tests cover commit-safe DB writes, settings whitelisting, atomic restore, `rateWord` failure handling, swipe-class reset and service-worker changes.

## Limits
- These are static/unit tests. No browser, IndexedDB or physical iPhone session was run, so the DB, animation and restore changes still need a manual pass using TEST-CHECKLIST.md.
