---
name: seo-onpage
description: "Use when auditing on-page relevance — titles, H1s, content-to-query match, duplication, internal linking and keyword cannibalization."
---

# SEO On-Page

Judge whether each page can rank **for something specific**. The failure mode of most sites is not a missing tag — it is pages that target nothing.

## 1. Title and H1

For each URL in scope:

```bash
curl -sS "https://<domain>/<path>" -o page.html
grep -o '<title>[^<]*</title>' page.html | sed 's/<[^>]*>//g'
grep -o '<h1[^>]*>[^<]*</h1>' page.html | sed 's/<[^>]*>//g'
```

| Defect | Threshold |
|---|---|
| Duplicate titles | 2+ URLs sharing a title, excluding paginated series |
| Missing H1, or multiple H1s | Any |
| Title ≠ H1 with no relation | Flag for intent review |
| Title length | <30 chars wastes space; >60 truncates in SERPs |
| Title is the brand only | "Acme — Home" ranks for nothing |

**The real check:** does the title name a thing a person would search for? A title of "Bienvenue" is not a title. Report it as a defect, because it is.

## 2. Duplication

Duplicate content usually hides in a template, not in the copy:

```bash
# Compare titles and meta descriptions across a sample
for u in $(head -40 sitemap_urls.txt); do
  curl -sS -A "Mozilla/5.0" "$u" | grep -o '<meta name="description" content="[^"]*"' | sed "s|^|$u |"
done | sort -k2
```

Look for:

- One meta description reused verbatim across a family of pages (category intro, product template)
- Thin pages: near-identical boilerplate with one variable swapped
- Faceted/filter URLs indexed as distinct pages (`?color=`, `?page=`) competing with the real page

Report the **count of affected URLs and the template that causes it** — that is the fixable unit, not each URL.

## 3. Cannibalization

Two or more of your own pages targeting the same query. Detect it from titles and H1s:

- Normalize titles (lowercase, strip brand)
- Group near-identical normalized titles
- Any group with 2+ URLs on the same topic is a cannibalization candidate

Confirm against the SERP if a keyword API is configured. Report as:

```
CANNIBALIZATION: "<query>" — /a (title), /b (title)
EVIDENCE: both pages lead with the same term
FIX: pick the stronger URL, canonical the other, or differentiate the intent
```

## 4. Internal linking

Internal links are how you tell a search engine what matters.

- **Depth:** count clicks from the homepage to each sitemap URL. Anything past 3 is weakly linked.
- **Anchor text:** `grep -o '>[^<]*</a>' ` on key pages. Anchors like "click here" / "en savoir plus" / bare URLs carry no signal.
- **Orphan ratio:** `orphans / total sitemap URLs`. Above ~10% is a structural problem.
- **Navigation reach:** are revenue pages reachable from the global nav, or only from a buried category page?

Report the anchor-text distribution: what share of internal anchors are non-descriptive.

## 5. Content-to-query match

For the top pages, compare page content against search intent:

```bash
# Extract visible headings as a quick intent proxy
curl -sS "https://<domain>/<path>" | grep -o '<h[12][^>]*>[^<]*</h[12]>' | sed 's/<[^>]*>//g'
```

| Signal | Reading |
|---|---|
| Page leads with brand messaging, query is informational | Intent mismatch — will not rank |
| Query is transactional, page is a blog post | Needs a product/section page, not more words |
| Query implies a list/comparison, page is a single item | Wrong page shape |
| Keyword appears in H1 but nowhere in the body | Stuffed heading, thin page |

**Do not recommend a word count.** There is no correct length. Recommend the *content the query requires* — the spec table, the comparison, the price, the answer.

## Scoring

For each page in scope, produce:

```
URL:
  Title:      <value>            [OK | DEFECT: reason]
  H1:         <value>            [OK | DEFECT: reason]
  Duplication:<none | shares template with N URLs]
  Links in:   <count from nav/section>   [OK | weakly linked | orphan]
  Intent:     <matched | mismatched: reason>
```

Then rank the pages by how much traffic they could gain. A defect on `/` outranks the same defect on `/blog/2019/notes`.

## Output

Feed the findings into the audit report's **Relevance** section, and promote to Top 5 any defect that affects a page already receiving impressions — those are the cheapest wins.