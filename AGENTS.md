# AGENTS.md

This repository tracks and visualizes the stake distribution (GINI coefficient) of Avalanche validators over time.

## Commands

### Full Pipeline (fetch + plot + git commit + push)
```bash
./stakes.sh                                    # Default api.avax.network
./stakes.sh -a https://custom-api.example.com  # Custom API endpoint
```
**Warning**: `stakes.sh` auto-commits and pushes to `origin`. Never run it casually.

### Setup
```bash
PY=3 ./setup.sh              # Creates virtualenv with --system-site-packages
source bin/activate          # Non-standard path (not .venv/bin/activate)
pip install matplotlib numpy
```

### Individual Operations
```bash
# Fetch (creates json/YYYY-MM-DD/)
./json/validators-fetch.sh
./json/validators-fetch.sh -g  # With GeoIP lookup via ipwho.is

# Plot (requires active virtualenv)
./stakes.py                    # Plot latest data from json/
./stakes.py -g                 # Group validators by reward address
./stakes.py -s                 # Show plot interactively (no -s = headless Agg)
./stakes.py json/2021-05-03    # Plot specific date
```

## Architecture

### Data Flow
1. `stakes.sh` orchestrates: fetch -> plot -> git commit + push
2. `json/validators-fetch.sh` fetches from Avalanche P-Chain API (`platform.getCurrentValidators`) and peer info (`info.peers`), joins by node ID via `json/join-by-id.jq`
3. `stakes.py` loads JSON, computes GINI coefficients, generates SVG plots

### Data Storage
- `json/YYYY-MM-DD/validators.json` — weight, delegatorWeight, totalWeight
- `json/YYYY-MM-DD/validators-ext.json` — extended with GeoIP info
- `image/YYYY-MM-DD.svg` — individual validator plot
- `image/YYYY-MM-DDG.svg` — grouped by reward address plot

### Key Functions in stakes.py
- `load()` — Parses validator JSON; with `-g` groups by reward address; aggregates all files matching `validators*.json` or `validators-ext*.json` in the date directory (early dates use `.000`–`.009` suffixes from batch fetches)
- `by_address()` — Merges validators sharing reward addresses
- `gini()` — GINI coefficient via mean absolute difference
- `gini_00/33/66()` — Reference distributions (equal, uniform, log-logistic)
- `plot_distribution()` — Cumulative distribution plot with 30%/70% control lines

## Gotchas

- **No test suite, linter, or typecheck** — there are no verification commands in this repo.
- **`source bin/activate`** — the virtualenv lives at the repo root, not `.venv/`.
- **`json/join-by-id.jq`** — `validators-fetch.sh` depends on this file (referenced via relative path from `json/`).
- **Multi-file JSON** — some date dirs contain `validators.{000..009}.json` from batched API calls; `load()` aggregates all matches.
- **Headless matplotlib** — `stakes.py` uses `matplotlib.use('Agg')` unless `-s`/`--show` is passed. For interactive debugging, pass `-s`.
- **GeoIP is live** — `validators-fetch.sh -g` calls `https://ipwho.is/` per peer IP (not a local database despite the `geoip` system package name in deps).
- **stakes.sh pushes** — the `commit()` function runs `git push origin` automatically. Do not run the full pipeline unless you intend to publish.

## Dependencies

System: `python3`, `jq`
Python: `matplotlib`, `numpy`
