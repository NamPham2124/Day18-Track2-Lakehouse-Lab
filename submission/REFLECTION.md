# Reflection — Top 5 Lakehouse Anti-Patterns

## Which anti-pattern is our team most at risk of, and why?

**Anti-Pattern: The Small-File Problem (Too Many Tiny Files)**

Our team is most at risk of the small-file problem. In our current LLM observability pipeline, we ingest data through micro-batch streaming — each API call generates a small record that gets written independently to the Bronze layer. Without a disciplined `OPTIMIZE + COMPACT` schedule, this quickly explodes into thousands of tiny Parquet files per partition.

As demonstrated in NB2, 200 small appends created 200 files, and point queries had to scan all of them. After `OPTIMIZE + Z-ORDER`, the speedup was **12×** and files-pruned ratio was **55×** — proving that small files destroy read performance.

The root cause for our team is that we prioritize low-latency ingestion (write-optimized) without scheduling periodic compaction jobs. In production, this means dashboard queries on the Gold layer become progressively slower as the day goes on.

**Mitigation:** Schedule `OPTIMIZE` every 1–2 hours on the Silver layer, apply Z-ORDER on high-cardinality filter columns (`user_id`, `model`), and set `target_size` to match our typical query scan pattern (~128–256 MB per file post-compaction).
