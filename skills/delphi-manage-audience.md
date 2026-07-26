---
name: Manage a Delphi audience and its memory
description: Look up an audience user, store contextual memory about them, and organize them with tags.
api: openapi/delphi-openapi.yml
operations: [lookupUser, listUsers, createUserInfo, getUserInfo, createTag, tagUser, getUserUsage]
method: generated
generated: '2026-07-18'
source: openapi/delphi-openapi.yml + conventions/delphi-conventions.yml
---

# Manage a Delphi audience and its memory

Curate the people who talk to a clone and the context the clone remembers.

## Auth
`x-api-key` header, clone-scoped. See `authentication/delphi-authentication.yml`.

## Steps
1. **Find or list users** — `lookupUser` (`POST /v3/users/lookup`) by exactly one
   of `email` or `phone_number`; or page the whole audience with `listUsers`
   (`GET /v3/users?limit=…&cursor=…`). Pagination is opaque cursor: pass
   `next_cursor` as `cursor` until `has_more` is false.
2. **Store memory about a user** — `createUserInfo`
   (`POST /v3/users/{user_id}/info`) with `{info, info_type}` where `info_type`
   is one of the enum (GOAL, PREFERENCES, INTERESTS, EXPERTISE, …). This data is
   embedded into the clone's memory to personalize responses.
3. **Read stored memory** — `getUserInfo` (`GET /v3/users/{user_id}/info`),
   newest first.
4. **Organize with tags** — `createTag` (`POST /v3/tags`, 409 if the name
   already exists), then `tagUser`
   (`POST /v3/users/{user_id}/tags/{tag_name}`). Tagging/untagging is
   idempotent — safe to repeat.
5. **Check usage** — `getUserUsage` (`GET /v3/users/{user_id}/usage`) for
   per-user message/voice/video quotas and remaining allowance this period.

## Rules
- Rate limit: 120 requests / 60s per key → back off on `429`.
- `createUserInfo` is NOT idempotent; guard against duplicate memory on retry.
- Handle `409` on `createTag` by reusing the existing tag (`listTags`).
