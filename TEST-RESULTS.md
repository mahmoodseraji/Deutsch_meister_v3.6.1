# Deutsch Meister 3.6.6 — test report

- Automated Node.js test suite: **93 passed, 0 failed** (`node --test tests/*.test.mjs`).
- JavaScript syntax checks passed for the application and new study-session and flashcard modules.
- Added tests for due-first session ordering, separate daily new-word limits, avoiding same-day new-word repeats, switching algorithms without reclassifying learned words, SM-2 learning stages and Leitner/SM-2 distributions.
- Added static regression checks for flashcard structure/flipping, populated conjugation rendering, generated Verbformen links, Progress metrics, unchanged IndexedDB schema and service-worker cache version.
- Existing scheduler tests continue to pass for both Leitner and SM-2, including independent schedules and due dates.

## Manual validation still required

No real browser session with an existing user IndexedDB profile or physical iPhone session was run in this environment. Existing data preservation is supported by retaining IndexedDB database name, schema version 2 and all existing object stores, but must still be verified against a real v3.6.5 profile before claiming device-level migration validation. Card visuals, touch interaction, dictionary navigation and offline/PWA behaviour should also receive a manual browser pass.
