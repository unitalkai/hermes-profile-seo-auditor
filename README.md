# SEO Auditor

A Hermes Agent [profile distribution](https://hermes-agent.nousresearch.com/docs/user-guide/profile-distributions) — a continuous SEO auditor that returns **impact-ranked action plans**, not diagnostics.

## Install

```bash
hermes profile install github.com/unitalkai/hermes-profile-seo-auditor --alias
```

## Configure

```bash
cp ~/.hermes/profiles/seo-auditor/.env.EXAMPLE ~/.hermes/profiles/seo-auditor/.env
```

| Variable | Required | Purpose |
|---|---|---|
| `OPENAI_API_KEY` | yes | Model access |
| `PAGESPEED_API_KEY` | no | Google PageSpeed Insights — free key |
| `SERPAPI_KEY` | no | Keyword/SERP data; falls back to keyless web search |

The agent works without the two optional keys — it loses Core Web Vitals detail and SERP positions, not crawling.

## Use

```bash
seo-auditor chat
```

Then just name a site:

> Audit example.com

Or in one shot:

```bash
hermes -p seo-auditor chat -q "Audit example.com and write the report"
```

Reports land in `reports/<domain>-<date>.md` inside the profile directory.

## What it actually does

1. **`seo-crawl`** — fetches `robots.txt`, sitemaps, and pages as a crawler sees them. Finds blocks, canonical chains, contradictions between the sitemap and reality, orphan pages, and JS-rendered sites whose content is invisible.
2. **`seo-onpage`** — titles, H1s, template duplication, cannibalization, internal linking depth and anchor text, content-to-intent match.
3. **`seo-report`** — ranks every finding by Impact ÷ Effort and writes the report.

Indexability is always checked before performance. A fast page that cannot be indexed is still worthless — the agent will tell you that rather than report a Lighthouse score.

## Output shape

Every finding carries four things: what is wrong, what it costs, its impact/effort score, and a concrete fix. If a finding cannot supply all four, it does not appear.

The **Top 5 actions** table is the report; everything below it is supporting evidence.

```
# SEO Audit — example.com
Audited 2026-10-06 · Pages crawled: 214 · Method: sitemap

## Top 5 actions
| # | Action | Impact | Effort | Pages affected |
|---|--------|--------|--------|----------------|
| 1 | Add distinct canonical to /produits/* | 5 | 1 | 47 |
```

## Scheduled audits

Two cron jobs ship **paused** — review, then enable deliberately:

```bash
hermes -p seo-auditor cron list
hermes -p seo-auditor cron resume weekly-site-audit    # Mondays 08:00
hermes -p seo-auditor cron resume monthly-index-coverage
```

| Job | Schedule | What it does |
|---|---|---|
| `weekly-site-audit` | Mondays 08:00 | Full audit; flags regressions vs. last week's report |
| `monthly-index-coverage` | 1st of month, 08:00 | Sitemap sampling; reports only new defects |

Tell the agent which site to track — put it in the job prompt after enabling, or in `local/` so updates never overwrite it.

## Review before you trust

Cron ships paused. `SOUL.md` and the three skills are active the moment you chat — read them first if you installed from a repo you do not control.

## Update

```bash
hermes profile update seo-auditor
```

Distribution-owned files (`SOUL.md`, `skills/`, `mcp.json`, `cron/jobs.json`) are replaced. Your `config.yaml`, `reports/`, memories, sessions and API keys stay put.

## Version

**1.0.0** — track with `hermes profile info seo-auditor`.

## License

MIT