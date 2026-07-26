---
name: Converse with a Delphi Digital Mind
description: Start a conversation with a clone and stream its text response, then read back the history.
api: openapi/delphi-openapi.yml
operations: [createConversation, streamResponse, getConversationHistory, updateConversationTitle]
method: generated
generated: '2026-07-18'
source: openapi/delphi-openapi.yml + conventions/delphi-conventions.yml
---

# Converse with a Delphi Digital Mind

Use the Delphi v3 API to hold a chat with a Digital Mind (an AI clone).

## Auth
Every request carries an `x-api-key` header. The key is scoped to one clone
(Immortal plan). No OAuth. See `authentication/delphi-authentication.yml`.

## Steps
1. **Create a conversation** — `createConversation` (`POST /v3/conversation`).
   Optionally pass `user_email` to link it to an audience user; omit for an
   anonymous chat. Capture the returned `conversation_id` and `initial_message`.
2. **Send a message and stream the reply** — `streamResponse` (`POST /v3/stream`)
   with `{conversation_id, message}`. The response is `text/event-stream`
   (Server-Sent Events); accumulate chunks until the `[DONE]` event.
3. **(Optional) Title the conversation** — `updateConversationTitle`
   (`PUT /v3/conversation/{conversation_id}/title`) with a 1–500 char `title`.
4. **Read history** — `getConversationHistory`
   (`GET /v3/conversation/{conversation_id}/history?include_citations=true`) to
   fetch all messages oldest-first, with source citations when requested.

## Rules
- Rate limit: 120 requests / 60s per key → back off on `429`.
- `streamResponse` has no idempotency key; do not blind-retry a partially
  streamed message.
- Errors are a plain JSON `{"error": "..."}` envelope; branch on status code
  (401 auth, 404 missing conversation, 429 rate limit). See `errors/`.
