# IREN Markets Desk (GitHub Pages edition)

Single-file markets dashboard for IREN and peers (Nebius, CoreWeave, Oracle, NVIDIA), built and published by GitHub Actions with no dependency on Claude.

- Live page: https://aussie123456.github.io/market-desk/ (repo https://github.com/aussie123456/market-desk)
- Manual refresh: Actions tab > **Refresh dashboard** > **Run workflow**. The page's own **Refresh data** button opens that screen. Allow 3 to 5 minutes.
- Scheduled refresh: daily at 22:00 UTC (9:00am Sydney during daylight saving). See the cron note in `.github/workflows/refresh.yml` for the April DST change.

## Files

| File | Purpose |
|---|---|
| `template.html` | The page: layout, CSS, rendering JS and empty data blocks. Edit this to change the page. Same file as the Claude artifact version. |
| `curated.json` | Hand-maintained data (cap stack, raises, pipeline, quarterly history, broker tables, peer metrics, sector tape). Every figure carries a source URL. |
| `build.py` | Builder: injects `curated.json`, repoints the Refresh button at the workflow, re-fetches every data block with a `data-url`, back-fills failed blocks from the previously published page, stamps `d-meta`, writes `dist/index.html`. Stdlib Python 3.8+. |
| `.github/workflows/refresh.yml` | Builds and deploys to GitHub Pages on the schedule, on **Run workflow**, and on any push that changes the files above. |

## How data flows

Each `<script type="application/json" id="d-...">` block in the template carries a `data-url`. The builder re-fetches each URL (Yahoo Finance chart and quoteSummary with the cookie-and-crumb session, FRED CSV, Google News RSS, SEC EDGAR atom feed) and rewrites the block. The page's JavaScript reads the blocks and renders everything client-side, so the published page needs no network access and no secrets.

If a feed fails, the builder keeps that block's payload from the page currently published on Pages (fetched at the start of the run), and `d-meta` records the error. The build fails, and nothing is published, only if no block has data at all.

## One-time setup

1. Create an empty repository on GitHub (private if your plan supports Pages on private repos; otherwise the page is public).
2. Push this folder to it (`git remote add origin ...; git push -u origin main`).
3. Repo **Settings > Pages > Build and deployment > Source**: choose **GitHub Actions**.
4. Actions tab > **Refresh dashboard** > **Run workflow**. The first run populates the page; the URL appears on the deploy job and on the Pages settings tab.

## Updating curated data

Edit `curated.json`, commit and push. The push triggers a rebuild automatically. Field conventions are unchanged from the artifact version: `iren.quarters`, `iren.capstack`, `iren.raises_ytd`, `iren.pipeline`, `iren.debt_entities`, `iren.projects`, `raises_peers_ytd`, `raises_24h`, `pipeline_peers`, `sector_tape`, `peer_quarters`. Instrument types must be one of: Convertible notes, Straight bonds, Private credit / term loans, Equity, ATM equity, Other.

## Known gaps

- FRED (`d-fred-DFF`, `d-fred-HYOAS`) failed from the Claude cloud runner with "Remote end closed connection". Whether it works from GitHub's runners is confirmed by the first run's step summary.
- Yahoo Finance occasionally rate-limits data-centre IPs. The builder retries the crumb four times and each block three times, and back-fills from the previous page on failure.
