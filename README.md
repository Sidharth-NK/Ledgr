# Ledgr
![CI](https://github.com/Sidharth-NK/Ledgr/actions/workflows/ci.yml/badge.svg)
## What it does

Take a photo or upload a PDF of a bill or receipt. Ledgr extracts the merchant, date, total, tax and line items into a local database. You can then ask questions about your spending.

Everything runs on the phone. There are no accounts, no cloud, and the app declares **no `INTERNET` permission**, so it cannot send your data anywhere. You can verify this in `app/src/main/AndroidManifest.xml`.

## How it works

```
Photo/PDF → Preprocess → OCR (text + boxes) → LLM (grammar-constrained JSON)
          → Grounding check + arithmetic validator → retry/fallback
          → Room record → Review UI
```

- OCR is ML Kit, on device.
- The LLM runs through llama.cpp (GGUF, 4-bit quantized). A grammar forces valid JSON.
- A deterministic check rejects any number the LLM outputs that isn't in the OCR text, and verifies that items sum to the total. Failures are retried, then flagged for review instead of being silently saved.
- A rules-based parser is the baseline. The LLM has to beat it to justify its cost.

## Target metrics

These are initial targets, not results. They will be tightened after the Phase 2 measurements, and actual numbers will go in `docs/BENCHMARKING.md`.

| Metric | Target | Measured |
|---|---|---|
| Total amount exact match (labeled set) | ≥ 95% | TBD |
| Purchase date exact match | ≥ 90% | TBD |
| Merchant match | ≥ 85% | TBD |
| Receipts needing manual review | ≤ 15% | TBD |
| Scan to saved record, warm, S23 Ultra | ≤ 10 s | TBD |
| Peak RAM during extraction | ≤ 2 GB | TBD |
| APK size (excluding model) | ≤ 50 MB | TBD |
| Network permissions | none | none |

Rules-parser baseline: TBD (Phase 1).

## Docs
- [Decisions log](docs/DECISIONS.md)
  
## Status

See the [plan](#plan) below. Currently: Phase 0 complete, Phase 1 (labeled set and baseline) in progress.

## Plan 
| Phase | Goal | Done when |
|---|---|---|
| 0 | Repo, README with target metrics, DECISIONS.md, CI | Scaffold builds and installs on a real device (done 2026-10-02) |
| 1 | ~50 labeled receipts, rules parser, unit tests | Baseline accuracy number recorded |
| 2 | llama.cpp on device via JNI, hardcoded OCR text | TTFT, tok/s, peak RAM logged on S23 Ultra |
| 3 | OCR → LLM → grounding → Room → review screen | Rules vs 0.5B vs 1.5B compared on the same labeled set |
| 4 | Quantization, threads, prompt trimming, KV cache reuse | Sweep results on all test devices, thermal runs |
| 5 | Tagged release, APK, ~1,500-word writeup, posts, upstream PR | Public, reproducible, installable by someone else |

**Test devices:** Galaxy S23 Ultra (flagship), OnePlus Nord 4 (upper-mid), OnePlus Nord 6 (newest; specs unchecked), Moto G4 (floor; model unknown).Gap: no true ₹10–20k-band phone. Borrow one if possible.

---
