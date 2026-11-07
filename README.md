# Deutsch Meister 3.6.5

Deutsch Meister is a small, offline-first German vocabulary PWA. It contains **no bundled vocabulary**: learners add words manually or import their own CSV/TSV files.

## Scheduling algorithms (V3.6.5)

The two methods keep independent schedules for every word. Changing the selected method does not overwrite the other method's history.

- **Leitner (five boxes):** Box intervals are 1, 2, 4, 7 and 14 days. The review screen offers two outcomes: Forgot returns the word to Box 1; Remembered advances it one box. The active box is shown on the review card. Movement is capped at Box 5.
- **Classic SM-2:** Again=quality 1 (reset repetitions and review after 1 day); Hard=3; Good=4; Easy=5. Successful repetitions use intervals of 1 day, then 6 days, then the previous interval multiplied by the updated ease factor. The standard ease-factor equation is applied with a minimum of 1.3.
- Review buttons display the calculated next interval for the selected method and current word. Leitner offers two clear outcomes (Forgot / Remembered); SM-2 offers four quality ratings (Again / Hard / Good / Easy).
- Rating controls stay visible below the flashcard, so a learner who already knows a word can rate it without revealing the answer.
- Due dates use local calendar dates, avoiding UTC date shifts around midnight and daylight-saving transitions.

## What this release changes

- Simplifies all five pages: Home, Practice, Library, Progress and Settings.
- Keeps only four bottom-navigation destinations; Settings stays in the top-right control.
- Removes unnecessary Library category/favourite filters and duplicate Home shortcuts. Search, edit, individual deletion and CSV-batch deletion remain.
- German voice choices: **System voice / Jarvis (Medium)**.
- English voice choices: **System voice / Amy (Medium)**.
- Downloads Jarvis and Amy automatically, sequentially, in the background on first launch. The app remains usable with system voices while this happens.
- Saves voice models in OPFS and runtime assets in a version-independent Cache Storage cache. Completion markers are written only after model/config files have been written and checked.
- Uses a dedicated classic Worker for phonemization and ONNX inference. The runtime is pinned to ONNX Runtime Web 1.18.0's WASM-only bundle, configured for one thread. It does not use the default JSEP/WebGPU runtime bundle.
- Reuses the phonemizer and one ONNX session for repeated speech in the same language. Switching languages terminates the previous Worker so only one model session is active.
- Avoids loading the neural models into memory during download. A model is loaded only when the user chooses that neural voice and requests speech.
- Keeps system SpeechSynthesis as the fallback if neural initialization or playback fails.
- Deletes the obsolete MLS/Thorsten cache from the old `piper` OPFS directory. It deliberately does **not** relabel an old Thorsten model as Jarvis.
- Limits Library rendering to 100 entries at a time and caches practice-availability calculations until vocabulary changes.
- Retains the 17-column user CSV template, optional fields, independent Leitner and SM-2 schedules, and imported-batch deletion.

## Voice sources and licensing

- German: [Jarvis German Medium on PiperHub](https://piperhub.org/model/jarvis-german-de-medium/) — [ufozone model repository](https://huggingface.co/ufozone/piper-de_DE-jarvis-medium). License: **CC BY-NC-SA 4.0**; it is not permitted for commercial use.
- English: [Amy English Medium on PiperHub](https://piperhub.org/model/amy-english-en-medium/) — runtime model source: [rhasspy/piper-voices Amy Medium model](https://huggingface.co/rhasspy/piper-voices/tree/v1.0.0/en/en_US/amy/medium). Its model card points to the MycroftAI Mimic 3 voice dataset; see the dataset's [CC BY-SA 4.0 licence](https://github.com/MycroftAI/mimic3-voices/blob/master/LICENSE).

The ONNX model files are approximately 63 MB each (about 126 MB total), excluding small configuration files. The models are downloaded from their source repositories; they are not bundled in this repository. Attribution and licence notes are also shown in Settings. See [VOICE-SOURCES.md](VOICE-SOURCES.md).

## Run locally

Serve this folder from `localhost` or HTTPS. Do not open `index.html` as a `file://` URL; IndexedDB, service workers and OPFS require a secure context.

```bash
python -m http.server 8000
```

Open `http://localhost:8000`.

## Offline behaviour

- Vocabulary, scheduling data and review history are stored locally in IndexedDB.
- The app shell is cached by the service worker.
- Voice models are stored in OPFS. ONNX Runtime and phonemizer assets are cached by the service worker after first-launch setup.
- Browser storage can be cleared or evicted. Export a JSON backup periodically.
- A cold-start offline neural playback test still requires a real browser/device. System voices remain the fallback.

## Automated checks

Run with Node.js 20+:

```bash
node --experimental-default-type=module --test tests/*.test.mjs
```

The suite covers CSV/TSV schema, user-supplied vocabulary, challenge generation, Leitner box transitions, classic SM-2 calculations, method isolation, due-date boundaries, voice configuration, mocked worker downloads/inference, and simplified navigation. Automated tests are not a substitute for testing the downloaded ONNX files in real Safari.

**Device status:** This release has not been physically tested on an iPhone 16 / iOS 26. It must not be described as iPhone-verified until that test is completed.
