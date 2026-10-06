---
name: seo-crawl
description: "Use when auditing a site's indexability — fetch robots.txt, sitemaps, and pages; find crawl and indexing defects with evidence."
---

# SEO Crawl

Fetch what a search engine actually sees, and report defects with URLs as evidence.
Run this **first** in any audit. A site that cannot be indexed cannot rank, no matter how good its content is.

## 1. robots.txt

```bash
curl -sS -A "Mozilla/5.0 (compatible; SEOAudit/1.0)" https://<domain>/robots.txt
```

Check, in order:

| Defect | Evidence to capture |
|---|---|
| `Disallow: /` present | The literal line, plus who it applies to |
| Key paths disallowed | Each blocked path (`/wp-admin` is fine; `/`, `/blog`, `/produits` is not) |
| Sitemap not declared | Absence of `Sitemap:` line |
| robots.txt returns 200 but empty/missing | HTTP status + body |
| Blocks a major crawler by name | The `User-agent:` + `Disallow:` pair |

A `Disallow` for a path that is also in the sitemap is a contradiction — report it.

## 2. Sitemap truth vs. reality

```bash
curl -sS https://<domain>/sitemap.xml | grep -o '<loc>[^<]*</loc>' | sed 's/<[^>]*>//g' > sitemap_urls.txt
wc -l sitemap_urls.txt
```

Then test a sample (or all, if under ~200 URLs) for the two killer conditions:

```bash
# Spot-check status codes across the sitemap
head -50 sitemap_urls.txt | while read u; do
  code=$(curl -s -o /dev/null -w '%{http_code}' -A "Mozilla/5.0" "$u")
  [ "$code" != "200" ] && echo "$code  $u"
done
```

| Defect | What it means |
|---|---|
| Sitemap URL returns non-200 | Sitemap is lying to crawlers |
| `noindex` pages in the sitemap | Contradictory signal — wasted crawl budget |
| Real pages missing from sitemap | Discovery depends on internal links only |
| Sitemap > 50k URLs, no index | Needs a sitemap index |

Cross-check: pages linked from the homepage but **absent** from the sitemap are candidates for orphan pages.

## 3. Indexability of a single page

```bash
curl -sS -A "Mozilla/5.0" -D headers.txt "https://<domain>/<path>" -o page.html
```

Read `headers.txt` for:

- `X-Robots-Tag: noindex` — a header-level block that ignores your `<meta>` tags
- `HTTP/2 301` / `302` chains — count the hops; 2+ is a defect
- `Set-Cookie` + `Vary: User-Agent` — possible cloaking, flag for review

Then in `page.html`:

| Tag | Defect |
|---|---|
| `<meta name="robots" content="noindex">` | Page is excluded from the index |
| `<link rel="canonical" href="...">` pointing elsewhere | Page will not rank as itself |
| canonical → a `noindex` page or a 404 | Broken chain, page may drop entirely |
| No `<title>` or an empty one | Search engines invent one |
| `<title>` identical across many URLs | Cannibalization |

**Canonical chains are the most common silent killer** — check that the canonical target's canonical points at itself. If A → B and B → C, A is orphaned.

## 4. JS-rendered sites

If `curl` returns a near-empty `<body>` but the site looks fine in a browser, the content is client-rendered:

```bash
wc -c page.html
grep -c '<script' page.html
```

Few hundred bytes of body plus a script bundle = crawlers see nothing. Confirm with the browser tool, then report it as a **critical indexability finding** — not a performance note.

## 5. Orphans and depth

From `sitemap_urls.txt`, collect every internal link on the homepage:

```bash
curl -sS https://<domain>/ | grep -o 'href="[^"]*"' | sed 's/href="//;s/"//' | grep -v '^http' | sort -u
```

URLs in the sitemap reachable from **neither** the homepage nor a section page within ~3 clicks are orphans. Report the count and the first 10.

## Output

Return a findings list. Each finding:

```
DEFECT: <one-line description>
EVIDENCE: <URL> — <the literal thing you saw>
AFFECTED: <count, or list>
SEVERITY: critical | high | medium
```

Critical = cannot be indexed. High = can be indexed but is mistargeted. Medium = friction.
Do not report a severity you did not derive from evidence.