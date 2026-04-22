# AI Integration

---

## What It Is

How your product uses AI — text generation, content analysis, image understanding, and image generation. This covers the architecture for calling AI APIs, managing prompts, handling responses, tracking costs, and providing a good user experience around AI features.

## Providers

**Text & Vision — Claude API (Anthropic):**
- Text generation (summaries, content creation, analysis, suggestions, chat)
- Vision/image understanding (analyze uploaded images, extract text from screenshots, describe visual content)
- Structured output (JSON schema responses for data extraction and classification)
- Tool use (AI decides when to call functions you define)

**Image Generation — Recraft:**
- Generate images from text prompts (raster and vector)
- Native SVG output — the only model that produces real vector files with clean geometry
- Icon, illustration, and design asset generation
- Background removal, inpainting, outpainting

## Model Selection

Not every AI feature needs the most powerful model. Match the model to the task complexity:

| Task | Model |
|---|---|
| Simple classification, tagging, short summaries | Claude Haiku |
| Content generation, analysis, longer reasoning | Claude Sonnet |
| Complex multi-step reasoning, nuanced analysis | Claude Opus |
| Design assets, icons, illustrations | Recraft V4 (standard for drafts, Pro for production) |

Default to Sonnet. Drop to Haiku for high-volume simple tasks. Escalate to Opus only when you've tested and confirmed Sonnet can't handle the specific task.

## Architecture

AI calls are backend operations. Never call AI APIs from the client.

```
Client
  │ (user triggers AI feature)
  ▼
Server Action / API Route
  │ 1. Validate auth + input
  │ 2. Check usage limits (AI calls are metered)
  │ 3. Build prompt (from template + user context)
  │ 4. Call AI API
  │ 5. Parse/validate response
  │ 6. Store result
  │ 7. Return to client (or stream)
  ▼
Client
  │ (renders result)
```

**Design decisions:**
- AI API keys live in server environment variables — never exposed to the client
- All AI calls go through the service layer, same as database queries
- AI calls are metered against the workspace's usage limits (covered in 09-credit-usage-system.md)
- For long-running AI operations (batch processing, large document analysis), offload to a background job (covered in 10-background-jobs.md)

## Prompt Management

Prompts are not hardcoded strings scattered across your codebase. They're managed assets.

**Design decisions:**
- Store prompt templates in `services/ai/prompts/`
- One file per prompt template, named by purpose (e.g., `summarize.ts`, `classify.ts`, `generateDescription.ts`)
- Each prompt template is a function that accepts context variables and returns the formatted prompt — this keeps the template and the variable injection together
- System prompts (the instructions that define AI behaviour) are separate from user prompts (the specific request) — system prompts are reused across many requests, user prompts change every time
- Version your prompts — when you change a prompt, you change AI behaviour. Treat prompt changes with the same care as schema migrations.

**Prompt caching:**
- Claude API supports prompt caching for repeated system prompts — cached tokens cost ~10% of normal input tokens
- If your feature uses the same system prompt for every request (which it should), caching significantly reduces cost
- This happens automatically on Anthropic's side for repeated prefixes — no configuration needed beyond consistent system prompts

## Streaming vs Non-Streaming

**Use streaming when:**
- The user is watching the AI generate text in real-time (chat, content generation, writing assistant)
- The response is long enough that waiting for the full response would feel slow (>2 seconds)

**Use non-streaming when:**
- The AI response is structured data (JSON classification, extracted fields) — you need the complete response before you can use it
- The response is short and fast (<2 seconds)
- The AI call is in a background job — nobody is watching

**Streaming design decisions:**
- Stream via a Server Action or API route that returns a ReadableStream
- The client renders tokens as they arrive — show a typing indicator before the first token, then stream text progressively
- Parse the complete response after streaming finishes for any validation or storage
- Handle stream interruption gracefully — if the connection drops, show what was received so far with a "generation interrupted" message and a retry button

## Structured Output

When you need AI to return specific data (not free-form text), use structured output with JSON schemas.

**Design decisions:**
- Define the expected output shape as a JSON schema in the API request
- Claude validates its response against the schema before returning — you get guaranteed valid JSON matching your structure
- Use structured output for: classification, data extraction, feature flagging, content analysis, any task where you need to programmatically use the AI's response
- Validate the response with Zod after receiving it — belt and suspenders. The schema enforcement is Claude's guarantee, Zod is your verification.

## Cost Management

AI API calls have real costs. Track and control them.

**Design decisions:**
- Every AI call logs: model used, input tokens, output tokens, total cost, workspace ID, feature that triggered it
- AI calls count against the workspace's usage limits — define an AI usage quota per plan
- Show AI usage in the billing/settings usage breakdown alongside other metered resources
- Use the cheapest model that produces acceptable results — default to Haiku for simple tasks, Sonnet for complex ones
- Cache AI responses when the same input produces the same output (e.g., classifying the same text twice should return the cached result, not make a second API call)
- Set a maximum token limit per request to prevent runaway costs from unexpectedly long inputs or outputs

## Error Handling

AI APIs are external services — they can be slow, rate-limited, or down.

**Design decisions:**
- Retry on transient errors (429 rate limit, 500 server error, network timeout) with exponential backoff, up to 3 retries
- Do not retry on 400 errors (bad request, invalid prompt) — these are bugs in your prompt, not transient failures
- Set a request timeout (30 seconds for streaming, 60 seconds for non-streaming) — if the API hasn't responded, fail gracefully
- When AI is unavailable, the feature should degrade gracefully — show "AI is temporarily unavailable, please try again" rather than crashing the page
- Never block core product functionality on AI availability — if AI enhances a feature but the feature works without it, the feature should still work when AI is down

## Image Generation

For generating images (not understanding them — that's Claude Vision).

**Design decisions:**
- Use Recraft for all image generation — it handles both raster (PNG, WebP) and vector (SVG) output
- Image generation runs as a background job — it takes 5-30 seconds, too slow for a synchronous request
- The user triggers generation, sees a loading state, and the result appears when ready (notification or polling)
- Generated images go through the same file upload pipeline (06-file-uploads.md) — stored in object storage, tracked in database, served via CDN
- AI-generated images should be flagged as such in the database (for content moderation, attribution, or user transparency)
- Cost per image: ~$0.04 for standard raster, ~$0.08 for vector (Recraft V4), ~$0.25-0.30 for Pro tier. Track and meter this.

## Security Considerations

- AI API keys are stored in server environment variables — never exposed to the client
- AI calls go through the service layer — the client never calls AI APIs directly
- AI responses displayed to other users must be filtered for harmful content
- AI prompt injection: user input included in prompts should be clearly delineated from system instructions
- Cost controls: per-request cost ceiling and per-workspace usage limits prevent abuse

## What to Test

- AI API calls go through the service layer (never from client)
- AI usage is metered against workspace plan limits
- AI response is validated with Zod before use
- Structured output matches the expected JSON schema
- Retry on 429/500 errors (3 retries with backoff)
- No retry on 400 errors (bad prompt)
- Graceful degradation when AI provider is unavailable (feature works without AI)
- Cost logging records model, tokens, and workspace for every call
- Cached responses return immediately for identical idempotent queries
- Streaming connection drop shows partial result with retry button

## Edge Cases

- **AI returns invalid or unexpected response** — even with structured output, validate with Zod. If validation fails, retry once with the same prompt. If it fails again, log the raw response for debugging and return a user-friendly error.
- **AI response contains harmful or inappropriate content** — for text that gets displayed to other users (shared content, public-facing), filter the output before storing. For private/personal output visible only to the requesting user, don't filter.
- **User sends extremely long input to AI** — cap input length before sending to the API. Define a maximum per feature (e.g., max 10,000 characters for summarization). Reject with a clear message if exceeded.
- **AI cost spike from a single workspace** — a workspace with a lot of activity could burn through AI quota quickly. The usage metering system enforces limits, but also set a per-request cost ceiling (reject requests that would cost more than a threshold, e.g., $0.50 per single call).
- **Streaming connection drops mid-response** — show what was received so far, offer a retry button. Don't discard partial results — they may still be useful to the user.
- **Same request made twice** — cache AI responses by input hash for idempotent queries (classification, extraction). Don't cache for generative queries where the user expects different output each time (creative writing, brainstorming).
- **AI feature added to a plan that didn't have it** — when upgrading a plan to include AI features, the usage counter starts at 0 for the current period. Don't retroactively charge for the part of the period before the upgrade.
- **Rate limited by the AI provider** — Anthropic rate limits by tokens per minute. If you hit the limit, queue requests and process them as capacity becomes available rather than failing immediately.

## Implementation Checklist

### Infrastructure
- [ ] Claude API key stored in server environment variables
- [ ] Recraft API key stored in server environment variables
- [ ] AI calls routed through service layer (never from client)
- [ ] AI usage metered against workspace plan limits
- [ ] Cost logging per call (model, tokens, cost, workspace)

### Prompt Management
- [ ] Prompt templates in services/ai/prompts/
- [ ] One file per prompt template, named by purpose
- [ ] System prompts separated from user prompts
- [ ] Prompt caching active (consistent system prompts across requests)

### Response Handling
- [ ] Streaming implemented for user-facing text generation
- [ ] Structured output with JSON schema for data extraction/classification
- [ ] Zod validation on all AI responses
- [ ] Retry on transient errors (3 retries with backoff)
- [ ] Request timeout configured (30s streaming, 60s non-streaming)
- [ ] Graceful degradation when AI is unavailable

### Image Generation
- [ ] Recraft configured for image generation
- [ ] Generation runs as background job
- [ ] Generated images stored through file upload pipeline
- [ ] AI-generated images flagged in database
- [ ] Image generation cost tracked and metered

### Cost Control
- [ ] AI usage quota defined per plan
- [ ] AI usage shown in billing/settings usage breakdown
- [ ] Response caching for idempotent queries (by input hash)
- [ ] Maximum token limit per request configured
- [ ] Per-request cost ceiling enforced

## Common Mistakes

- Exposing AI API keys to the client (anyone can use your key and run up your bill)
- Hardcoding prompts as inline strings in components (impossible to manage, version, or test)
- Defaulting to the most expensive model for every task (Haiku handles 80% of structured tasks at a fraction of the cost)
- Not metering AI usage (a single power user burns through your API budget)
- Blocking core features on AI availability (AI is an enhancement, not a dependency — features should work without it)
- Not caching repeated identical requests (same classification query runs 100 times, costs 100x)
- No timeout on AI calls (a hung API call blocks the user indefinitely)
- Streaming without handling disconnection (user refreshes mid-stream, partial response is lost)
- Not validating structured output with Zod (trusting the AI's JSON without verification)
- Treating prompt changes casually (a prompt change is a behaviour change — test it before deploying)
