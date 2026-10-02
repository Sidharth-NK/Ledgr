# Ledgr
### Primary Functionality
- Takes photo of bills / uploads pdfs of bills,receipts etc
- updates the database 
- ask financial questions regarding your spend culture or income spent areas etc

### Important Factor
It is 
- Fast
- Local
- Lightweight
- Private
- Accurate
- Free

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
