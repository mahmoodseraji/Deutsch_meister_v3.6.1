# Changelog

## 3.6.1 — Simplification and voice stability work

- Simplified Home, Practice, Library, Progress and Settings; removed extra Library filters, favourite controls and duplicate Home shortcuts.
- Kept the four main navigation destinations and the Settings shortcut.
- Kept exactly two voice choices per language: System/Jarvis for German and System/Amy for English.
- First-launch voice downloads run sequentially in the background; the system voice remains available during downloads.
- Kept voice weights out of the app bundle. Models are streamed into OPFS; runtime assets are cached by the service worker.
- Waited for the service worker to control the page before starting the virtual runtime-asset downloads, avoiding a first-launch cache race.
- Removed repeated runtime-asset draining before each inference; the Worker loads the required cached scripts and WASM assets when it initializes.
- Pinned ONNX Runtime Web 1.18.0's WASM-only bundle and one-thread execution, avoiding the newer default JSEP bundle implicated in Safari/WebKit memory reports.
- Kept inference in a dedicated Worker, reused the phonemizer and active-language session, and released the previous session when switching languages.
- Corrected model-cache handling: legacy Thorsten files are no longer relabelled as Jarvis. The old MLS/Thorsten/Amy cache is cleaned and the two selected models are downloaded from their configured sources.
- Added stronger voice-config and model-file checks, explicit ONNX tensor dimensions, phoneme-ID range validation and a bounded text length for neural inference.
- Added graceful system-voice fallback and release of audio/Worker resources when the app is backgrounded.
- Cached practice availability until the user's vocabulary changes and retained 100-word Library pagination.
- Preserved CSV/TSV import/export, all 17 columns, independent Leitner/SM-2 schedules, user-only vocabulary, individual deletion and CSV-batch deletion.

## Validation

- Automated Node suite: 42 tests pass.
- JavaScript syntax, HTML ID references, service-worker asset list, manifest, CSV template and ZIP integrity are checked before packaging.
- Mocked Worker tests do not prove that the real downloaded ONNX files synthesize correctly.
- No physical iPhone 16 / iOS 26 test has been performed; do not claim iPhone verification.

## 3.6.1
- Added neural voice diagnostics showing model initialisation, phonemisation, ONNX inference, audio decoding and playback stages, with a visible recent-event log and explicit failure message.
- Added bounded audio playback waits and an iOS AudioContext running-state check so a playback failure is reported instead of appearing to do nothing.
- Added controls to delete both downloaded neural voice models from this device and to opt out of automatic downloads. Vocabulary, learning progress and system voices are unaffected.
- Added an explicit action to download the neural voices again later.
- Kept the existing simplified learning interface and voice choices.


## 3.6.2
- Home now stays a clean overview until the user starts a review session.
- Review mode hides the overview and bottom navigation so the flashcard is presented as the primary full-screen task without scrolling past the dashboard. Added an explicit End review control.
- Rebuilt the review card as a 3D flip card: Show answer rotates to the reverse side. Rating an answer animates the card off-screen before the next card appears.
- Added subtle tactile press, hover, and elevation animations to buttons and interactive controls, with reduced-motion accessibility support.
- No changes to vocabulary storage, CSV import/export, scheduling algorithms, or neural voice implementation.

## 3.6.3 — Scheduling correctness

- Corrected Leitner failure handling: **Again returns a card to Box 1**; Hard keeps its current box; Good advances one box; Easy advances two boxes. Five box intervals remain 1, 2, 4, 7 and 14 days.
- Replaced the previous modified SM-2 progression with the classic SM-2 interval sequence: first successful repetition 1 day, second 6 days, then previous interval multiplied by the updated ease factor and rounded.
- Explicit SM-2 quality mapping: Again=1 (reset repetitions), Hard=3, Good=4, Easy=5. Ease-factor updates use the standard SM-2 equation and never fall below 1.3.
- Answer buttons show the actual next interval calculated for the selected method and current word.
- Kept Leitner and SM-2 histories independent; added deterministic local-date handling and expanded automated regression tests.
- No changes to vocabulary, IndexedDB stores, CSV format, interface layout, or voice engine.


## 3.6.4 — Method-aware review controls

- Review controls now adapt to the selected learning method. Leitner presents its classic two outcomes (Forgot / Remembered), with the current box, destination box and interval; SM-2 presents Again, Hard, Good and Easy with its calculated interval.
- The four rating buttons moved outside the flip-card and remain visible before and after revealing the answer. A learner can now rate a familiar word immediately without flipping the card.
- Added a short explanation that the displayed day count is the next review interval.
- Leitner no longer exposes SM-2-only Hard/Easy ratings; its scheduler accepts only Forgot/Remembered outcomes, preventing non-standard box behavior.
- Normalized partially populated method-specific schedules safely on startup. Legacy top-level schedule fields are migrated only when neither independent schedule exists, preventing a missing method from inheriting the other method's legacy mirror.
- Added regression tests for Leitner-specific labels, actual destination boxes, SM-2 labels, partial schedule migration and rating controls outside the flip-card.
- No IndexedDB database name, object-store, CSV schema, vocabulary content or voice engine changes.

## 3.6.5 — Reliability and data-safety audit

Fixed
- **Silent data loss on write failure** (`db.js`): writes resolved on the request's `success` event, before the transaction committed, so a quota/commit error was never reported. Writes now resolve on commit and reject on abort. Added atomic `putMany` / `removeMany`.
- **Review card could freeze** (`rateWord`): any storage error left the card stuck in its swipe-out state and ignored all further taps. It now recovers and shows an error. Ending the session during the 230 ms animation no longer throws on a null word.
- **Stale-data overwrite**: a review is now scheduled from the freshest stored copy of the word, not the object captured when the queue was built.
- **Backup restore could brick the app**: an unknown `algorithm` value in a backup made the scheduler throw on every launch. Settings are now whitelisted on load and on restore. Restore also validates the file, normalises schedules, writes atomically, no longer overwrites the learner's `v3StartedOn` streak cutoff, and reports skipped records.
- **Swipe animation never ran** and stale `swipe-left/right` classes were never cleared (an `#studyCard` rule out-specified it). Fixed and scoped to the card so the rating buttons stay put.
- **CSV import and bulk deletes/reset** are now all-or-nothing instead of thousands of separate writes that could leave a half-imported batch.
- Manually adding a word that already exists is now blocked (CSV import already did this).
- Date-dependent scheduler tests failed on any day other than 2026-11-06; `ensureSchedules` now accepts an injectable clock.

Improved
- Review no longer re-reads the entire review-log store after every card.
- Requests persistent storage so Safari/iOS is less likely to evict progress.
- Home refreshes when the app returns to the foreground on a new day.
- Service worker: navigations with a query string use the cached shell; cache bumped to v3.6.5.
- Backup file `version` field now reflects the app version.
