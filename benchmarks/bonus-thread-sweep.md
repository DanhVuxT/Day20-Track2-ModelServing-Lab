# Bonus - Thread sweep (CPU native)

Model: `qwen2.5-1.5b-instruct-q4_k_m.gguf`. Native llama.cpp build: `-DGGML_NATIVE=ON`. GPU layers: `0` because CUDA Toolkit / `nvcc` is not installed.

| threads | tg64 (tok/s) |
|---:|---:|
| 1 | 12.90 |
| 2 | 19.29 |
| 7 | 15.57 |
| 14 | 11.23 |
| 20 | 8.06 |
| 40 | 4.24 |

**Best:** `-t 2` at 19.29 tok/s.

The curve did **not** peak at physical-core count. On this Windows CPU-only build, `-t 2` was fastest and higher thread counts degraded sharply. My interpretation is that the tiny decode benchmark is memory/cache-bound and the overhead of coordinating many CPU threads outweighs extra compute. This is exactly why the sweep matters: the best serving knob on this machine is not "use all cores"; it is "measure and cap threads."
