---
id: PLACEHOLDER_ID_04
title: "Technical SEO Audit — PLACEHOLDER_CLIENT"
status: todo
priority: high
assignee: PLACEHOLDER_ASSIGNEE
client: PLACEHOLDER_CLIENT
project: PLACEHOLDER_PROJECT
created_by: "@claude"
created_date: PLACEHOLDER_DATE
updated_date: PLACEHOLDER_DATE
due_date: PLACEHOLDER_DUE_04
estimated_hours: 4
actual_hours: 0
labels: [seo, technical]
dependencies: []
repeat: none
conflict_flag: false
---

## Context

Technical SEO issues prevent good content from ranking regardless of how
well-optimised the on-page content is. This audit runs in parallel with
keyword research and can start immediately.

**Client:** PLACEHOLDER_CLIENT
**Website:** PLACEHOLDER_DOMAIN

Focus on crawlability, indexability, and site architecture. Surface issues
that require developer involvement and estimate implementation effort.

## Checklist

**Crawl & Index**
- [ ] Run a crawl using Screaming Frog (free up to 500 URLs) or Sitebulb
- [ ] Check robots.txt: no important pages accidentally blocked
- [ ] Check XML sitemap: exists, submitted to GSC, all pages valid, no 404s
- [ ] Check for noindex tags on pages that should be indexed
- [ ] Identify crawl errors in Google Search Console (Coverage report)
- [ ] Check for redirect chains (3+ hops) and fix to single redirects

**Site Architecture**
- [ ] Confirm HTTPS on all pages (no mixed content warnings)
- [ ] Check canonical tags: all pages have self-canonical or point to correct canonical
- [ ] Identify duplicate content (URL parameters, trailing slashes, www vs non-www)
- [ ] Check pagination handling (rel=prev/next or canonical)
- [ ] Verify mobile-friendliness (Google Mobile-Friendly Test)

**Structured Data**
- [ ] Check existing schema markup for errors (Google Rich Results Test)
- [ ] Identify pages that would benefit from schema (FAQs, products, reviews)

**Indexing Health**
- [ ] Check number of pages indexed vs total pages on site (should be close)
- [ ] Identify thin or duplicate pages that should be noindexed or consolidated

## Acceptance Criteria

- Crawl complete with errors documented and categorised
- All critical technical issues flagged with severity and fix instructions
- Developer-required fixes separated from content-team fixes
- Estimated effort per fix included (hours or story points)

## Time Log

| Date | Hours | By | Notes |
|---|---|---|---|

## AI Notes

> Written and maintained by AI only. Humans do not edit this section.

_@claude — 2026-03-10_
