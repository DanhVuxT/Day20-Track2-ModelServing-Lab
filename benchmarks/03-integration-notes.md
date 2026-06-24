# 03 - Milestone Integration Notes

`03-milestone-integration/pipeline.py` ran end-to-end against the native OpenAI-compatible llama-server on `localhost:8080`.

Raw output: `benchmarks/03-integration-output.txt`.

| Query | Retrieved contexts | retrieve (ms) | llama-server (ms) | total (ms) |
|---|---|---:|---:|---:|
| Why is goodput more useful than throughput? | `n20-paged`, `n20-radix`, `n20-disagg` | 0.0 | 5639.7 | 5639.8 |
| What problem does PagedAttention actually solve? | `n20-paged`, `n20-radix`, `n20-disagg` | 0.0 | 4933.2 | 4933.3 |
| When should I think about disaggregated serving? | `n20-disagg`, `n20-paged`, `n20-radix` | 0.0 | 5786.8 | 5786.8 |

## N16-N19 Mapping

| Day | Piece used | Status |
|---|---|---|
| N16 Cloud/IaC | localhost llama-server endpoint | stub |
| N17 Data pipeline | curated in-memory serving notes | stub |
| N18 Lakehouse | `TOY_DOCS` rows as processed records | stub |
| N19 Vector + Feature Store | keyword-overlap retriever over `TOY_DOCS` | stub |

Observation: retrieval is effectively free in this toy pipeline. The llama-server call dominates by several seconds, so on this CPU-only serving path the LLM decode/prefill stage is the clear bottleneck.
