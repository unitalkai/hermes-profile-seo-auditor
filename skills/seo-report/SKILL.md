---
name: seo-report
description: "Use when producing the final SEO audit deliverable — impact/effort-ranked action plan in markdown, saved to reports/."
---

# SEO Report

Turn findings into a ranked action plan. This is the deliverable; the raw findings are evidence.

## The ranking rule

Every action gets **Impact** and **Effort**, each 1–5. Rank by `Impact / Effort` descending. Ties break toward actions affecting more URLs.

| Score | Impact means | Effort means |
|---|---|---|
| 5 | Blocks indexing, or affects the pages that drive revenue | Template/config change, under 1 hour |
| 4 | Affects a page family already getting impressions | Template change, needs a dev |
| 3 | Real but localized loss | Per-page work, dozens of pages |
| 2 | Marginal or long-tail | Per-page work, hundreds of pages |
| 1 | Theoretical | Requires a redesign or a migration |

Be willing to score effort at 1 for a templated fix and 5 for anything touching routing or CMS structure. Impact is about traffic, never about how interesting the finding is.

**Never produce an action with Impact ≤ 2.** It belongs in a "not worth doing yet" note at the bottom, if anywhere.

## Report shape

Write to `reports/<domain>-<YYYY-MM-DD>.md`.

````markdown
# SEO Audit — <domain>

Audited <YYYY-MM-DD> · Pages crawled: N · Method: sitemap | crawl | provided list

<One or two sentences: the single most important thing about this site's search health.>

## Top 5 actions

| # | Action | Impact | Effort | Pages affected |
|---|--------|--------|--------|----------------|
| 1 | <imperative: "Add canonical…"> | 5 | 1 | 47 |
| 2 | … | | | |

### 1. <Action title>

**Problem.** <What is wrong, with the URL and the literal evidence.>

**Cost.** <The mechanism. "These 47 pages share one title, so search engines cannot distinguish them and pick the wrong one.">

**Fix.** <Concrete steps a developer can execute. Name the file, template, or CMS field if you found it.>

**Verify.** <How to confirm the fix landed.>

### 2. …
(2–5 same shape)

## Indexability

<Findings from seo-crawl. Criticals first. Each with URL evidence.>

## Relevance

<Findings from seo-onpage. Table of the worst pages.>

## Structure

Schema markup present/missing per page type, breadcrumbs, URL shape, pagination, hreflang if multilingual.

## Performance

Core Web Vitals. Only include this section when the higher sections are clean — otherwise say so explicitly and move on.

## Not checked / blocked

<Every URL you could not fetch, with the reason. Every check you skipped and why.
This section protects the audit's credibility. An empty section means you checked everything,
which is rare — do not leave it empty to look thorough.>

## Not worth doing yet

<Impact ≤ 2 items, one line each. Honest, not padded.>
````

## Rules for the writing

1. **The Top 5 table is the report.** A reader who stops after it knows what to do next. Write it last, after you have all findings, even though it appears first.

2. **No finding without evidence.** Every claim in a "Problem" paragraph carries a URL. If you did not fetch it, it does not go in the report — it goes in "Not checked / blocked."

3. **Name the leverage explicitly.** Do not make the reader infer why action 1 beats action 4. One sentence of "why this first" in the intro is worth more than a longer list.

4. **Quantify, never approximate.** "47 pages", not "many pages". If you sampled (e.g. checked 50 of 3,000 URLs), say the sample size and state the extrapolation as an estimate, labelled as one.

5. **One report per audit.** Do not split findings across files unless the user asks.

## Severity language

Lead with severity. If the site cannot be indexed, the first line of the report says so:

> **The site is largely de-indexed: 812 of 900 sitemap URLs return `noindex`. Nothing below matters until this is fixed.**

Softening a critical finding to look balanced is the worst thing this report can do.

## After the report

State where it was written and the top action in chat. Do not paste the whole report into the conversation unless asked.

```
Rapport : reports/example.com-2026-10-06.md
Pages : 214 auditées · 47 bloquées en indexation
Top action : canonicals dupliqués sur /produits/* (impact 5, effort 1)
```

Then stop. The user decides whether to fix it themselves or hand the report to a dev.