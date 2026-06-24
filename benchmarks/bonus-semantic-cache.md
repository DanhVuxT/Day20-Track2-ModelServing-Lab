# Bonus - Semantic Cache Demo (C8)

Command:

```powershell
.\.venv\Scripts\python.exe BONUS-llama-cpp-optimization\semantic-cache-demo.py --offline --sweep
```

Raw output: `benchmarks/bonus-semantic-cache-output.txt`.

## Threshold Sweep

| Threshold | Hits / 8 |
|---:|---:|
| 0.70 | 3 |
| 0.80 | 3 |
| 0.85 | 3 |
| 0.90 | 3 |
| 0.95 | 3 |

## Replay Result

| Requests | Hits | Hit rate | LLM calls saved | Simulated decode skipped |
|---:|---:|---:|---:|---:|
| 8 | 3 | 38% | 3 | ~750 ms |

Observation: the offline bag-of-words embedder only catches lexical paraphrases, but the serving lesson is still clear. A semantic-cache hit sits above prefix/KV cache and skips the full model call, so a hit saves both prefill and decode. The risk is returning the wrong answer if the threshold is too low, plus tenant leakage if a shared cache is not isolated.
