# Benchmarks

## Docker images

| Image | Size on disk | Compressed | Build time (cold) | Rebuild (no change) | User |
|---|---|---|---|---|---|
| naive (`python:3.12`) | 3.12GB | 792MB | 2m54s | 3s | root |
| dev (multi-stage, `python:3.12-slim`) | 847MB | 180MB | 1m20s | 2s (code-only change) | appuser |

## Container (compose, `--workers 2`, 512MB limit)

Load test: `python scripts/loadtest.py` (2000 requests, 25 concurrent clients)

| p50 | p99 | max | Throughput | Non-2xx |
|---|---|---|---|---|
| 49.7 ms | 96.4 ms | 118.2 ms | 486 req/s | 0 |

| Memory (fraud-api) | Graceful stop |
|---|---|
| 306.5MiB / 512MiB (60%) | 1.1s |
