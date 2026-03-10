---
id: PLACEHOLDER_ID_07
title: "Page Speed & Core Web Vitals — PLACEHOLDER_CLIENT"
status: todo
priority: medium
assignee: PLACEHOLDER_ASSIGNEE
client: PLACEHOLDER_CLIENT
project: PLACEHOLDER_PROJECT
created_by: "@claude"
created_date: PLACEHOLDER_DATE
updated_date: PLACEHOLDER_DATE
due_date: PLACEHOLDER_DUE_07
estimated_hours: 2
actual_hours: 0
labels: [seo, technical, core-web-vitals]
dependencies: []
repeat: none
conflict_flag: false
---

## Context

Core Web Vitals (LCP, INP, CLS) are confirmed Google ranking signals. This
task runs in parallel with other audit tasks since it requires no upstream data.

**Client:** PLACEHOLDER_CLIENT
**Website:** PLACEHOLDER_DOMAIN

Most page speed issues require developer involvement to fix. Identify and
clearly classify what the SEO team vs the dev team owns.

## Checklist

**Baseline Measurement**
- [ ] Run Google PageSpeed Insights on the homepage (mobile + desktop)
- [ ] Run PageSpeed Insights on 3 key landing pages (highest organic traffic)
- [ ] Check Core Web Vitals in Google Search Console (CWV report — field data)
- [ ] Note: LCP (< 2.5s good), INP (< 200ms good), CLS (< 0.1 good)

**Identify Issues**
- [ ] Flag render-blocking resources (JS/CSS loaded in <head> without defer/async)
- [ ] Check image optimisation: formats (WebP/AVIF), sizing, lazy loading
- [ ] Check server response time (TTFB — < 800ms is good)
- [ ] Note hosting / CDN situation (Cloudflare? AWS? Shared hosting?)
- [ ] Check for unused CSS/JS being loaded

**Classify by Owner**
- [ ] List fixes the SEO / content team can implement (image optimisation, etc.)
- [ ] List fixes requiring a developer (server config, JS defer, CDN setup)

## Acceptance Criteria

- CWV scores for homepage + 3 key pages documented (mobile + desktop)
- All failing metrics have a root cause identified
- Fixes clearly split: content-team-actionable vs developer-required
- Estimated performance gain (score improvement) noted for each fix

## Time Log

| Date | Hours | By | Notes |
|---|---|---|---|

## AI Notes

> Written and maintained by AI only. Humans do not edit this section.

_@claude — 2026-03-10_
