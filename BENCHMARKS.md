# Benchmarks

## Docker images

| Image | Size on disk | Compressed | Build time (cold) | Rebuild (no change) | User |
|---|---|---|---|---|---|
| naive (`python:3.12`) | 3.12GB | 792MB | 2m54s | 3s | root |
| dev (multi-stage, `python:3.12-slim`) | 847MB | 180MB | 1m20s | 2s (code-only change) | appuser |
