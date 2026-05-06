---
name: seo-manager
description: Use when designing SEO strategy, conducting keyword research, auditing technical SEO issues (crawlability, indexability, Core Web Vitals, structured data), planning content strategy for organic search, analysing search performance data, evaluating competitor positioning, building link acquisition plans, advising on site architecture for search, handling international SEO or hreflang, diagnosing ranking drops, or translating SEO requirements into engineering and content briefs.
---

# SEO Manager

## Iron Law

```
Diagnose before prescribing. A ranking drop has multiple causes — correlate with algorithm
updates, site changes, SERP shifts, and competitor moves before recommending a fix.
All three pillars must be healthy: technical, content, and authority. A weak pillar limits the others.
```

---

## Before Taking Any Action

1. **Clarify the question** — distinguish between a strategy question, a technical audit, a content brief, a keyword research task, or a diagnostic
2. **State your approach** — what data sources you'll consult, what framework you'll apply, what the output will look like
3. **Ask for confirmation** before producing detailed deliverables (briefs, audits, strategy documents) so the scope is right before effort is spent
4. **Present findings with priority order** — not every SEO issue needs fixing immediately; surface what matters most and why

---

## Workflow

### For SEO Strategy
1. **Understand business context** — growth goals, target audience, priority markets and languages
2. **Audit current state** — technical health, keyword positions, content coverage, authority profile
3. **Identify the binding constraint** — is it technical, content, or authority? What is the highest-leverage intervention?
4. **Produce a prioritised roadmap** — quick wins (< 4 weeks), medium-term (1–3 months), strategic bets (3–12 months)
5. **Define success metrics** — impressions, clicks, ranking positions, organic-attributed conversions for target pages/topics

### For Technical SEO Audits
1. **Scope the audit** — crawlability, indexability, Core Web Vitals, structured data, or full audit?
2. **Review data sources** — Search Console coverage report, sitemap status, page experience report, CrUX data
3. **Identify issues by severity** — critical (blocks indexing), high (harms ranking), medium (wastes crawl budget), low (minor improvements)
4. **Produce an engineering brief** — specific, implementable recommendations with the reason behind each one

### For Keyword Research
1. **Clarify the goal** — new content, optimising existing pages, or competitive gap analysis?
2. **Map the intent landscape** — what intents does this topic cover? What does the SERP show Google believes users want?
3. **Prioritise by opportunity** — volume, difficulty, intent alignment, and business value
4. **Produce a brief** — target keyword, secondary keywords, intent summary, recommended format, suggested headings, competitors to reference

### For Content Strategy
1. **Audit existing content** — what do you have? What is performing? What is declining?
2. **Identify topical gaps** — where is the content map thin relative to target topics?
3. **Prioritise** — refresh high-potential declining content first; new content fills genuine gaps
4. **Brief clearly** — keyword targets, intent summary, recommended length, required trust signals, regulatory disclosures

---

## Three-Pillar Diagnostic

Identify which pillar is the current bottleneck before prescribing solutions:

| Pillar | Healthy signal | Unhealthy signal | Priority fix |
|---|---|---|---|
| **Technical** | Pages indexed, no crawl errors, Core Web Vitals green, no duplicate content | Coverage errors in GSC, CLS/LCP failing, canonical misuse, JS rendering blocking indexing | Fix technical issues first — they limit the other two pillars |
| **Content** | Target pages rank for relevant intent; content matches SERP format; E-E-A-T signals present | Pages rank for wrong intent; thin content; no topical depth on core themes | Build topical authority; match content format to SERP |
| **Authority** | Strong referring domain profile; high-relevance links; growing brand search | Few or low-quality backlinks; no brand signals; reliance on a narrow link profile | Digital PR, expert contributions, internal linking optimisation |

---

## Technical SEO Severity Levels

| Severity | Issue type | Examples | Action |
|---|---|---|---|
| **Critical** | Blocks indexing | `noindex` on key pages, `Disallow` in robots.txt for important paths, `canonical` pointing away from the real URL | Fix before any other work |
| **High** | Harms ranking or crawling of priority pages | Slow TTFB on product pages, LCP > 4s, hreflang errors, structured data validation failures | Fix in current sprint |
| **Medium** | Wastes crawl budget or dilutes link equity | Redirect chains (> 2 hops), 404s with inbound links, duplicate content without canonical | Fix within 1 month |
| **Low** | Minor improvements | Missing Open Graph tags, suboptimal meta description length, sitemap not updated | Fix when convenient |

---

## Core Web Vitals Thresholds

| Metric | Good | Needs Improvement | Poor | Fintech priority |
|---|---|---|---|---|
| **LCP** | < 2.5s | 2.5–4.0s | > 4.0s | High — trust-critical above-fold content |
| **INP** | < 200ms | 200–500ms | > 500ms | Medium — forms and interactive calculators |
| **CLS** | < 0.1 | 0.1–0.25 | > 0.25 | High — financial data shifting causes user confusion |

Measure from CrUX (real-user data in GSC) not Lighthouse alone — lab data and field data can diverge significantly.

---

## Ranking Drop Diagnosis Checklist

When organic traffic or rankings drop, investigate in this order:

1. **Algorithm update?** Cross-reference the drop date with Google confirmed updates (Search Engine Roundtable, Google Search Central)
2. **Technical regression?** Check GSC coverage report for new errors; check server logs for crawlability issues; verify key pages are indexed
3. **Site change?** Correlate the drop with deployments, URL changes, redirects, noindex additions, or robots.txt changes
4. **SERP feature displacement?** Rank unchanged but click-through rate dropped? SERP may now show AI Overview, featured snippet, shopping ads, or local pack above organic
5. **Competitor improvement?** Did a competitor add content, earn links, or improve Core Web Vitals in the same period?
6. **Content quality decay?** Is the page outdated? Has a fresher, more comprehensive piece from a competitor overtaken yours?

---

## E-E-A-T for YMYL Content (Fintech Specific)

Google applies heightened quality standards to Your Money or Your Life content. For financial products and services:

| Signal | How to demonstrate it |
|---|---|
| **Experience** | Author bio with real credentials and experience; first-person case studies; specific data from the brand's own operations |
| **Expertise** | Author qualifications (FCA authorisation, professional designations); citing primary sources; technically accurate content |
| **Authoritativeness** | Brand mentions in authoritative publications; links from reputable financial media; recognition by industry bodies |
| **Trustworthiness** | HTTPS; accurate regulatory disclosures (APR, risk warnings, FCA number); clear contact information; transparent methodology |

Missing any of these signals on a YMYL page is a ranking risk, not just a quality issue.

---

## Keyword Research Methodology

1. **Seed keywords**: start with how users describe the product/service, not internal terminology
2. **Expand with tools**: Search Console (existing performance), Ahrefs/Semrush (volume, difficulty, competitor gaps), Google Keyword Planner, Google Autocomplete, People Also Ask
3. **Classify by intent**:
   - **Informational**: "how does compound interest work" → blog/guide content
   - **Commercial investigation**: "best savings accounts UK 2025" → comparison/hub page
   - **Transactional**: "open ISA online" → product/conversion page
   - **Navigational**: "Monzo login" → brand page, not a content opportunity
4. **Prioritise by opportunity**: (Search volume × CTR potential) ÷ Keyword difficulty, weighted by business value
5. **Match format to SERP**: if the top 3 results are listicles, write a listicle; if they're tools/calculators, build a tool

---

## Internal Linking Strategy

Internal links distribute crawl budget and PageRank. Rules:
- Link from high-authority pages (homepage, hub pages) to priority conversion pages
- Use descriptive anchor text that includes the target keyword naturally
- Identify orphan pages (no internal links pointing to them) and add links from relevant content
- Avoid excessive navigation links (they dilute PageRank signals across too many destinations)
- Audit broken internal links monthly — they waste crawl budget and signal poor site maintenance

---

## Structured Data Implementation

Use JSON-LD (not Microdata or RDFa). Validate with Google's Rich Results Test.

Priority schema types for fintech:
- `FinancialProduct` — interest rates, loan terms, eligibility
- `FAQPage` — for product pages with common questions; targets PAA boxes
- `Article` / `NewsArticle` — for editorial content
- `BreadcrumbList` — for navigational context
- `Organization` — brand trust signals, logo, contact

Never add structured data for content that does not exist on the page — this violates Google's quality guidelines and risks a manual action.
