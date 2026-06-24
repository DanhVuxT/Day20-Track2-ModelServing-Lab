# 02 - llama-server Results

Server A: `python -m llama_cpp.server` on `127.0.0.1:8080`, CPU-only (`n_gpu_layers=0`, `n_threads=14`). Used for smoke test and core locust 10/50 runs.

Server B: native `llama-server.exe` built from source with `-DGGML_NATIVE=ON`, CPU-only (`-ngl 0`, `-t 2`, `--parallel 4`, `--metrics`). Used for `/metrics` and continuous batching observation.

## Smoke Test

`02-llama-cpp-server/smoke-test.py` passed against the Python server.

Observed latency: `5375 ms` for a short chat completion.

## Locust Load Tests (Python server)

| Concurrency | Total requests | Total RPS | E2E P50 (ms) | E2E P95 (ms) | E2E P99 (ms) | Failures |
|---:|---:|---:|---:|---:|---:|---:|
| 10 | 7 | 0.131 | 22000 | 47000 | 47000 | 0 |
| 50 | 7 | 0.126 | 33000 | 54000 | 54000 | 0 |

Raw CSV:

- `benchmarks/locust-10_stats.csv`
- `benchmarks/locust-50_stats.csv`

## Native Metrics Under Load

Native server load run: `benchmarks/native-locust-10_stats.csv`.

| Metric | Peak / Final |
|---|---:|
| `llamacpp:requests_processing` peak | 4 |
| `llamacpp:requests_deferred` peak | 6 |
| `llamacpp:n_busy_slots_per_decode` peak | 3.98 |
| `llamacpp:tokens_predicted_total` final | 1091 |

Raw metrics CSV: `benchmarks/02-server-metrics.csv`.

Observation: with `--parallel 4`, the native server kept almost all four decode slots busy (`n_busy_slots_per_decode ~= 3.98`) and deferred extra requests under load. That is continuous batching working, but CPU-only decode still leaves high E2E latency.
