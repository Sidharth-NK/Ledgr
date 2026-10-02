##  How results will be measured (2026-10)

**Decision.**
- Report **median and p95**, with **cold start and warm runs separated**.
- Metrics: time to first token, tokens/sec, peak RAM, end-to-end scan time,
  field-level extraction accuracy, and how often output lands in `needs_review`.
- Tools: in-app benchmark harness writing JSON/CSV, Perfetto, Android Studio
  Profiler, `dumpsys meminfo`. Plots with Python (pandas, matplotlib).
- Thermal runs (sustained load) on every device, not just one burst.
- Document limitations honestly in the writeup.

**Why.** The deliverable is evidence. Numbers that cannot be reproduced or that
hide cold-start and throttling are worth little.

---
