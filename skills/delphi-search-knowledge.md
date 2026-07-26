---
name: Search a Delphi digital mind for RAG
description: Discover knowledge-base content sources and retrieve relevant passages to ground an answer.
api: openapi/delphi-openapi.yml
operations: [searchContent, searchQuery]
method: generated
generated: '2026-07-18'
source: openapi/delphi-openapi.yml + conventions/delphi-conventions.yml
---

# Search a Delphi digital mind for RAG

Power a custom search or retrieval-augmented-generation pipeline over a clone's
knowledge base.

## Auth
`x-api-key` header, clone-scoped. See `authentication/delphi-authentication.yml`.

## Steps
1. **(Optional) Discover content sources** — `searchContent`
   (`POST /v3/search/content`) with `{query: ["topic", …]}` to list matching
   sources (podcasts, articles, PDFs, videos) and their `contentId`s.
2. **Retrieve passages** — `searchQuery` (`POST /v3/search/query`) with:
   - `query`: semantic (meaning-based) strings.
   - `keywords`: exact-phrase strings routed through hybrid BM25 boosting.
   - Scope with `content` (descriptions) or `contentIds` (from step 1).
   - `limit` (1–50, default 10) and an access-tier `tag` (e.g. PUBLIC, PREMIUM).
   The response's `chunks[]` are the passages; the `content[]` array carries
   full source metadata for attribution without extra calls.
3. **Ground your answer** — cite `chunks[].sources[].title` / `contentId` when
   presenting results.

## Rules
- Rate limit: 120 requests / 60s per key → back off on `429`.
- Prefer `contentIds` over free-text `content` scoping when you already know the
  source, to avoid an extra resolution step.
- `tag` defaults to the broadest access; set it to respect tiered content.
