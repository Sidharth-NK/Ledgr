# Decisions
A running log of engineering decisions for Ledger: why each was made, what was rejected, and when to revisit. Newest entries go at the bottom. Entries are short on purpose.

Format: **Context** (what forced the choice) → **Decision** → **Alternatives** → **Revisit when**.

---

## D-001: Product scope: offline receipt/bill ledger (2026-10)

**Context.** The project started as an offline medical-report explainer. It was dropped because of liability exposure and weak retention (people read a report once). Receipts and bills are used repeatedly, the output is verifiable (numbers either match the paper or they don't), and India-specific formats (GST, GSTIN, CGST/SGST, day-first dates, ₹ misread by OCR) make it a real problem rather than a toy.

**Decision.** Build an Android app where a photo or PDF of a receipt becomes a structured purchase record. Analysis is later just queries over the log. No accounts, no cloud, **no INTERNET permission**.

**Alternatives.** Medical-report explainer (rejected above); a generic chat-with-your-documents app (no verifiable output, hard to benchmark).

**Revisit when.** Never for the privacy stance. Scope of analysis features (NL-to-SQL, categorization) is deferred until the extractor is measured.

---

## D-002: Pipeline shape (2026-10)

**Decision.**

```
Photo/PDF → Preprocess → OCR (text + boxes) → LLM (grammar-constrained JSON)
          → Grounding check + arithmetic validator → retry/fallback
          → Room record → Review UI
```

- The LLM is the core extractor (merchant, date, total, tax, line items).
- A **deterministic rules parser** is the baseline and safety net. It is also
  the number the LLM has to beat.
- **Grounding check** is plain code, not another model: every number the LLM
  outputs must appear in the normalized OCR text; dates must parse and match;
  merchant must fuzzy-match the top lines; items must sum to subtotal and
  subtotal + tax must equal total. Failure means one retry, then `needs_review`.
- Output is JSON because it feeds code and a database. Constrained decoding
  (GBNF grammar) guarantees it parses.

**Why.** Small on-device models hallucinate numbers. Checking the output against the OCR text turns "the model might be wrong" into "wrong answers are caught or flagged", which is the property users actually need.

**Alternatives.** LLM-only with no validation (unsafe for money); rules-only (brittle across receipt formats); vision-language model end to end (deferred as a later challenger, not the first build).

**Revisit when.** Baseline numbers exist (Phase 1) and LLM numbers exist (Phase 3). If the LLM does not beat rules on the labeled set, say so in the writeup and re-scope.

---

## D-003: Money stored as integer paise (2026-10)

**Decision.** All amounts are stored as `Long` paise, never floating point.

**Why.** Float rounding makes "items sum to total" checks unreliable.
Integer arithmetic makes the arithmetic validator exact.

**Revisit when.** Multi-currency is ever needed (not planned).

---

## D-004: Store raw OCR text and the source image (2026-10)

**Decision.** Each record keeps `raw_text`, `image_path`, and `pipeline_meta` (model, prompt version, OCR engine, timings) alongside the extracted fields.

**Why.** Improving the parser or model later should be a re-parse, not a re-scan. It also makes failures debuggable and benchmarks reproducible.

**Cost.** More storage per receipt. Acceptable; images can be downscaled.

---

## D-005: Stack: native Kotlin + C++ via JNI (2026-10)

**Decision.** Kotlin, Jetpack Compose, MVVM with coroutines/Flow, CameraX, Room. llama.cpp is built through the NDK/CMake and called over JNI.
ML Kit on-device OCR sits behind an interface so engines can be swapped.

**Why.** The portfolio goal is on-device inference and optimization evidence (latency, tokens/sec, RAM, thermals). That requires direct control over threads, memory, and the native runtime, which cross-platform wrappers hide.

**Alternatives.** Flutter / React Native with a native plugin (extra layer between me and the measurements); an existing Android LLM wrapper library (hides the thing I want to measure and contribute to).

**Revisit when.** Never for the native core. The UI layer is incidental.

---

## D-006: Inference runtime and models (2026-10)

**Decision.** llama.cpp with GGUF models, Q4 quantization to start, GBNF grammar for JSON. Start with Qwen2.5 0.5B and 1.5B. A fine-tuned small model (LoRA) comes after the baseline comparison.

**Deferred on purpose:** other runtimes (MLC, ExecuTorch, LiteRT, ONNX), NPU delegates, speculative decoding, embedding model, tiny classifier, NL-to-SQL, VLM challenger, PaddleOCR. Each is reasonable but would dilute the first result. They return only if the core pipeline is measured and stable.

**Revisit when.** Phase 4 (optimization) is done and there is time left.

---

## D-007: Minimum SDK 26 (Android 8.0) (2026-10)

**Decision.** `minSdk = 26`.

**Why.** Covers essentially all active Android devices, avoids legacy compatibility workarounds, and supports the APIs this app needs (CameraX,
Room, 64-bit ARM builds).

**Open.** The "floor" test device (Moto G4) model is unknown. If it is the 2016 model, it is a stress test for a lite path (smaller model or rules-only), not a reason to lower the floor. Update this entry once the model is confirmed.

**Revisit when.** Moto G4 model is identified, or a target device cannot run the app.

---

## D-008: Build toolchain and pinned versions (2026-10-02)

**Decision.** Gradle Kotlin DSL with a version catalog. Everything that affects reproducibility is pinned and recorded here.

| Component                            | Version                                            |
|--------------------------------------|----------------------------------------------------|
| Gradle (wrapper)                     | 9.6.0                                              |
| Android Gradle Plugin                | 9.4.1                                              |
| Kotlin (project)                     | 2.2.10                                             |
| Compose BOM                          | 2026.02.01                                         |
| JDK for the daemon                   | 25 (Temurin 25.0.3, auto-provisioned via foojay into ~/.gradle/jdks) |
| JDK launching the wrapper            | 17.0.20                                            |
| minSdk / targetSdk                   | 26 / 37                                            |
| compileSdk                           | 37                                                 |
| NDK                                  | 30.0.16248370                                      |
| CMake                                | 3.22.1                                             |
| platform-tools (adb)                 | 37.0.1                                             |
| llama.cpp                            | `TODO: commit hash, once added`                    |
| Model files                          | `TODO: filename + SHA-256, once added`             |

**Fallback.** If NDK 30 causes trouble building llama.cpp, drop to NDK 27.x or 28.x and record the change here.

**Rule.** Do not bump any pinned item without a new entry.

---

## D-010: Repo hygiene and privacy of data (2026-10)

**Decision.**
- `.gitignore` excludes `*.gguf`, `/private_data/`, `.cxx/`, `.externalNativeBuild/`.
- Real receipt images and labels live only in `/private_data/` and are never
  committed. The public repo contains **synthetic** receipts only.
- Small, focused commits. Conventional prefixes (`chore:`, `feat:`, `docs:`).

**Why.** Receipts contain personal purchase history. A public portfolio repo
must be safe by construction, not by remembering.

---

## Template for new entries

```
## D-0XX: Title (YYYY-MM-DD)

**Context.** What forced the choice.
**Decision.** What was chosen.
**Alternatives.** What was rejected and why.
**Revisit when.** The trigger for reconsidering.
```


```
