# Reflection — Lab 20 (Personal Report)

**Họ Tên:** Thanh Danh  
**Cohort:** not provided  
**Ngày submit:** 2026-06-24

---

## 1. Hardware spec (từ `00-setup/detect-hardware.py`)

- **OS:** Windows 10 (AMD64)
- **CPU:** 12th Gen Intel(R) Core(TM) i7-12700H
- **Cores:** 14 physical / 20 logical
- **CPU extensions:** AVX / AVX_VNNI / AVX2 / F16C / FMA / BMI2 shown by native llama.cpp log
- **RAM:** 15.7 GB
- **Accelerator:** NVIDIA GeForce RTX 3060 Laptop GPU, 6144 MiB
- **llama.cpp backend đã chọn:** CPU native for the measured runs (`-DGGML_NATIVE=ON`, `-ngl 0`) because CUDA Toolkit / `nvcc` is not installed
- **Recommended model tier:** Qwen2.5-1.5B-Instruct (Q4_K_M)

**Setup story:** Existing `.venv` was broken because Python 3.11 was blocked in the sandbox. I recreated `.venv` with the installed Python 3.11.9 outside the sandbox, installed dependencies, downloaded the real Qwen2.5 1.5B GGUF files, then built native llama.cpp CPU with `-DGGML_NATIVE=ON`.

---

## 2. Track 01 — Quickstart numbers (từ `benchmarks/01-quickstart-results.md`)

Settings: `n_threads=14`, `n_ctx=2048`, `n_batch=512`, `n_gpu_layers=0`.

| Model | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode rate (tok/s) |
|---|--:|--:|--:|--:|--:|
| qwen2.5-1.5b-instruct-q4_k_m.gguf | 878 | 314 / 366 | 83.1 / 86.8 | 5558 / 5818 / 5882 | 12.0 |
| qwen2.5-1.5b-instruct-q2_k.gguf | 271 | 307 / 318 | 76.0 / 81.6 | 5075 / 5141 / 5152 | 13.2 |

**Một quan sát:** Q2_K was about 10% faster in decode rate (13.2 vs 12.0 tok/s) and loaded much faster. I would still prefer Q4_K_M unless RAM is tight, because the speed gap is modest.

---

## 3. Track 02 — llama-server load test

Python server (`python -m llama_cpp.server`, CPU-only):

| Concurrency | Total RPS | TTFB P50 (ms) | E2E P95 (ms) | E2E P99 (ms) | Failures |
|--:|--:|--:|--:|--:|--:|
| 10 | 0.131 | not separately reported by locust | 47000 | 47000 | 0 |
| 50 | 0.126 | not separately reported by locust | 54000 | 54000 | 0 |

Native server metrics run (`llama-server.exe --metrics`, `-t 2`, `--parallel 4`, `-ngl 0`):

| Metric | Value |
|---|---:|
| `requests_processing` peak | 4 |
| `requests_deferred` peak | 6 |
| `n_busy_slots_per_decode` peak | 3.98 |
| `tokens_predicted_total` final | 1091 |

**Batching observation:** peak `llamacpp:n_busy_slots_per_decode` / `requests_processing` at concurrency 10 was `3.98 / 4`. The native server kept almost all four parallel slots busy and deferred overflow requests. Continuous batching worked; CPU-only decode was still the bottleneck.

---

## 4. Track 03 — Milestone integration

- **N16 (Cloud/IaC):** stub: localhost-only llama-server endpoint
- **N17 (Data pipeline):** stub: in-memory curated serving notes
- **N18 (Lakehouse):** stub: `TOY_DOCS` rows as processed records
- **N19 (Vector + Feature Store):** stub: keyword-overlap retrieval over `TOY_DOCS`

**Nơi tốn nhiều ms nhất** trong pipeline:

| Query | retrieve (ms) | llama-server (ms) | total (ms) |
|---|---:|---:|---:|
| Why is goodput more useful than throughput? | 0.0 | 5639.7 | 5639.8 |
| What problem does PagedAttention actually solve? | 0.0 | 4933.2 | 4933.3 |
| When should I think about disaggregated serving? | 0.0 | 5786.8 | 5786.8 |

**Reflection:** The bottleneck is the llama-server call, not retrieval. That matches expectation because this demo uses a toy in-memory retriever; under CPU-only serving, generation dominates the pipeline.

---

## 5. Bonus — The single change that mattered most

**Change:** cap native llama.cpp CPU decode threads based on measured thread sweep, not on physical/logical core count.

**Before vs after** (from `benchmarks/bonus-thread-sweep.md`):

```text
before: -t 14 => 11.23 tok/s
after:  -t 2  => 19.29 tok/s
speedup: ~1.72×
```

**Tại sao nó work:** This result was different from my first expectation. I expected the physical-core count to win, but the measured curve peaked at only 2 threads and got worse at 7, 14, 20, and 40. For this small CPU-only decode benchmark, adding threads increased coordination and memory/cache contention faster than it added useful compute.

The lesson is that local LLM decode can be memory-bandwidth/cache-bound rather than pure compute-bound. On this laptop, "use all cores" is a bad default. The best serving knob was to measure the thread curve and cap threads aggressively.

Bonus C8 semantic cache was also run offline: 3/8 hits, 38% hit rate, 3 LLM calls saved in the synthetic prompt stream.

---

## 6. (Optional) Điều ngạc nhiên nhất

The most surprising result was that `-t 2` beat `-t 14` by about 1.72×. More CPU threads made this workload slower, not faster.

---

## 7. Self-graded checklist

- [x] `hardware.json` đã commit
- [x] `models/active.json` đã commit
- [x] `benchmarks/01-quickstart-results.md` đã commit
- [x] `benchmarks/02-server-results.md` and `benchmarks/02-server-metrics.csv` đã commit
- [x] `benchmarks/bonus-*.md` đã commit
- [x] Ít nhất 6 screenshots trong `submission/screenshots/`
- [x] `make verify` / `python scripts/verify.py` exit 0
- [ ] Repo trên GitHub ở chế độ **public**
- [ ] Đã paste public repo URL vào VinUni LMS
