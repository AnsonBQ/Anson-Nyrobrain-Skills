# DeepAPI Docs — API Reference

> Agent-readable DeepAPI docs. Index: https://deepapi.co/llms.txt · OpenAPI: https://deepapi.co/openapi.json · Skill: https://deepapi.co/deepapi-skill/deepapi/SKILL.md
> Markdown pages: https://deepapi.co/docs/setup.md · https://deepapi.co/docs/reference.md · https://deepapi.co/docs/pricing.md

## Basics

- Base URL: `https://deepapi.co`
- Auth: `Authorization: Bearer <key>` on every request.
- Idempotency: every POST sends an `Idempotency-Key` — any unique name you make up. Retrying with the same name returns the stored result instead of paying twice.
- Cost cap: every scrape sets `maxCostUsd`, the most that call is allowed to cost.
- Email setup: none. Your first call to `/v1/email/send` creates a managed email address for your workspace — nothing to connect. The response returns it as `output.emailIdentity.emailAddress`; replies to it show up in `/v1/email/messages`.
- Email drafts: `/v1/email/send` creates a draft by default and returns its `output.draftId`. Review pending drafts with `GET /v1/email/drafts`, then send one exactly as stored with `POST /v1/email/drafts/{draftId}/send`. Sending re-checks policy and requires direct-send approval for your workspace.

## Try Deep Scrape: research a company

Deep Scrape collects public data across sources into one JSON dossier. Use it for a person, company, or topic; use Deep Research for analysis or comparisons, and regular scraping for one known source.

Paste this task into your agent:

```text
Use DeepAPI Deep Scrape (POST /v1/scrape/deep) to research Stripe, the payments company. Include https://stripe.com in urls and set maxCostUsd to "0.50". Follow the returned polling instructions until the result is ready. Give me a short company brief covering its products, customers, and recent announcements, with source links. Flag missing information, conflicts, and uncertain claims.
```

### 1. Start the request

Send the request with your API key. Keep the same Idempotency-Key when retrying this request. The $0.50 cap is a spending limit, not a fixed price.

```bash
export DEEPAPI_API_BASE_URL="https://deepapi.co"
# Set DEEPAPI_API_KEY to your key before running.
# Create this once; reuse it when retrying the same request.
DEEP_SCRAPE_IDEMPOTENCY_KEY="deep-scrape-$(uuidgen)"

curl -X POST "$DEEPAPI_API_BASE_URL/v1/scrape/deep" \
  -H "Authorization: Bearer $DEEPAPI_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: $DEEP_SCRAPE_IDEMPOTENCY_KEY" \
  -d '{
    "query": "Stripe, the payments company",
    "urls": ["https://stripe.com"],
    "maxCostUsd": "0.50"
  }'
```

### 2. Wait for the result

The POST returns 202. Wait next.afterSecs seconds, then GET next.path with the same Authorization header. Repeat while a polling next is present, even if status already says succeeded. Retrieve a saved result with GET /v1/requests/{requestId} instead of starting another paid call.

### 3. Read the company brief

Read output.subject, profiles, posts, people, websites, and sources. Preserve source links and check partial, confidence, conflicts, and errors before using the brief. Useful partial results are charged for actual usage within the cap. No usable matching data means no charge.

[Agent skill](https://deepapi.co/deepapi-skill/deepapi/SKILL.md) · [Scraping guide](https://deepapi.co/deepapi-skill/deepapi/references/scraping.md)

## Endpoints

### Scrape

- `POST /v1/scrape/website` — Scrape Website: Scrape website pages and return clean page content. With no maxDepth, maxPages, includeUrls, or excludeUrls, only the supplied URLs are scraped. To follow links, pass maxDepth above 0, or omit maxDepth while passing maxPages, includeUrls, or excludeUrls. output is an array of page objects, each carrying the requested content (markdown and/or text — pick one with contentFormat, omit for both) plus a url when the origin reports one. Content-free pages are dropped, not billed. Seeds-only website scrapes also return urlOutcomes: one {url, status} per requested URL (returned or not_returned). Optional per-page metadata: title, description, and language when the page provides it; truncated and totalChars when maxChars caps a page. For deep crawls, maxDepth and includeUrls/excludeUrls glob patterns steer which links are followed. If the origin answers but returns nothing usable, the call is free (debitMicrousd 0): output is an empty array and list.listState reports no_results, or source_blocked when blocking dominated (a login wall, captcha, rate limit, access-denied notice, or HTTP 401/403/407/429/451/503). A billed run whose content is still settling returns status succeeded with output null and a polling next — follow next until it is absent, regardless of status; a backend failure with no working fallback ends status failed, never a free empty success.
- `POST /v1/scrape/linkedin/profile` — Scrape LinkedIn Profile: Scrape public LinkedIn profile details.
- `POST /v1/scrape/github/profile` — Scrape GitHub Profile: Scrape public GitHub user or organization profile details.
- `POST /v1/scrape/github/repo` — Read GitHub Repository: Read public repository metadata, README, languages, license, topics, and statistics.
- `POST /v1/scrape/github/issues` — Read GitHub Issues: List and filter public repository issues, excluding pull requests.
- `POST /v1/scrape/github/pulls` — Read GitHub Pull Requests: List public repository pull requests with merge state, authors, and diff statistics.
- `POST /v1/scrape/github/search` — Search GitHub: Search public repositories, issues, pull requests, or code.
- `POST /v1/scrape/github/contents` — Read GitHub Contents: Read a public repository file or directory listing at a branch, tag, or commit.
- `POST /v1/scrape/github/commits` — Read GitHub Commits: List public repository commit history with author, path, and date filters.
- `POST /v1/scrape/github` — alias — Scrape GitHub: Backward-compatible alias for GitHub profile scraping.
- `POST /v1/scrape/twitter/search` — Search X/Twitter: Scrape X/Twitter posts from a search query or account handles.
- `POST /v1/scrape/linkedin/jobs` — Scrape LinkedIn Jobs: Scrape public LinkedIn job listings for a search query. Requests must allow at least 10 postings: maxItems below 10 and cost caps below $0.05 are rejected.
- `POST /v1/scrape/linkedin/company` — Scrape LinkedIn Company: Scrape public LinkedIn company pages for firmographic details.
- `POST /v1/scrape/linkedin/people` — Search LinkedIn People: Search public LinkedIn profiles by role, location, company, or school. maxCostUsd defaults to 1.25 (2.50 with includeDetails) and cannot go lower.
- `POST /v1/scrape/linkedin/posts` — Scrape LinkedIn Posts: Scrape recent public posts from LinkedIn profiles or company pages.
- `POST /v1/scrape/twitter/user` — Scrape X/Twitter User: Scrape public X/Twitter account profiles, with optional follower and following lists.
- `POST /v1/scrape/twitter/replies` — Scrape X/Twitter Replies: Scrape the public reply thread of an X/Twitter post. maxCostUsd defaults to 1.25; values below 0.50 are rejected.
- `POST /v1/scrape/youtube/transcript` — Scrape YouTube Transcript: Scrape a YouTube transcript as plain text. Set includeSegments false for compact output; omission preserves timed segments for backward compatibility. Long transcripts: bound output with maxChars; truncated: true marks a capped result. Videos without captions return an empty result.
- `POST /v1/scrape/youtube/channel` — Scrape YouTube Channel: Scrape YouTube channel stats and recent videos. maxItems applies per channel; only real videos are returned and billed.
- `POST /v1/scrape/youtube/search` — Search YouTube: Search YouTube videos by keyword and return video metadata. Returns 5 results by default; callers can explicitly request more.
- `POST /v1/scrape/youtube/shorts` — Scrape YouTube Shorts: Scrape YouTube channel Shorts feeds. maxItems applies per channel; only real Shorts are returned and billed. Long-form videos and streams are excluded. Use the transcript endpoint with a Shorts URL to read spoken content.
- `POST /v1/scrape/instagram/profile` — Scrape Instagram Profile: Scrape public Instagram profile details such as bio, follower counts, links, published business location, and related (similar) accounts.
- `POST /v1/scrape/instagram/posts` — Scrape Instagram Posts: Scrape recent public posts and reels from Instagram profiles, with captions and engagement counts.
- `POST /v1/scrape/instagram/comments` — Scrape Instagram Comments: Scrape public comments from an Instagram post or reel.
- `POST /v1/scrape/instagram/hashtag` — Search Instagram Hashtags: Find public Instagram posts or reels by hashtag. Results are bounded across the whole request.
- `POST /v1/scrape/facebook/ads` — Scrape Meta Ads Library: Scrape ads from the Meta Ads Library — every ad running across Facebook, Instagram, Messenger, and Audience Network — by keyword or advertiser page. Returns ad creatives, copy, landing URLs, run dates, platforms, and EU transparency data. Ads cost ~$0.00375 each: go deep with maxItems 100+ and combine advertiser pages with keyword queries. Requests must allow at least 10 ads: maxItems below 10 and cost caps below $0.0375 are rejected.
- `POST /v1/scrape/facebook/groups` — Scrape Facebook Groups: Scrape the newest posts from public Facebook groups. Returns normalized post text, permalinks, timestamps, engagement, authors, group details, and attachments. Private groups and login-only content are not supported.
- `POST /v1/scrape/reddit/search` — Search Reddit: Search public Reddit posts by keyword across all of Reddit, or scoped to one or more subreddits, with sort (relevance, hot, top, new, comments) and time filters (hour to year).
- `POST /v1/scrape/reddit/posts` — Scrape Reddit Posts: Scrape recent public posts from one or more subreddits, with hot/new/top ordering and a since timeframe (24h to year) for top.
- `POST /v1/scrape/reddit/comments` — Scrape Reddit Comments: Scrape the public comment thread of a Reddit post, nested replies included (depth 0 = top-level). Fewer comments than maxItems can come back: a post's advertised count includes deleted and collapsed branches.
- `POST /v1/scrape/reddit/user` — Scrape Reddit User: Scrape public Reddit user profiles: karma breakdown, account age, verification, and follower count.
- `POST /v1/scrape/google/places` — Search Google Maps Places: Search public Google Maps business listings by term and optional location. Returns name, category, address, phone, website, rating, review count, and coordinates per place. Narrow results with categoryFilterWords, or fetch specific places by placeIds or urls. maxCostUsd is a spending cap, not a charge — the final debit covers only the places actually returned.
- `POST /v1/scrape/tiktok/search` — Search TikTok: Search public TikTok videos by keyword or hashtag, with sort (relevance, liked, latest) and time filters (24h to 6 months). Returns one video object per result with engagement counts and author info.
- `POST /v1/scrape/tiktok/profile` — Scrape TikTok Profile: Scrape public TikTok profiles: followers, likes, video count, bio, verification, and account age.
- `POST /v1/scrape/tiktok/posts` — Scrape TikTok Posts: Scrape the public video feed of one or more TikTok creators, with latest/popular/oldest ordering. Returns one video object per post with engagement counts.
- `POST /v1/scrape/tiktok/comments` — Scrape TikTok Comments: Scrape the public comments of a TikTok video: text, author, likes, and reply counts. Comment replies are not included.
- `POST /v1/scrape/tiktok/transcript` — Scrape TikTok Transcript: Scrape a TikTok video's spoken content as plain text. Set includeSegments true for timed segments with speaker labels. Long transcripts: bound output with maxChars; truncated: true marks a capped result. Videos without speech or captions can return short noisy text or an empty result.
- `POST /v1/scrape/amazon/reviews` — Scrape Amazon Reviews: Scrape public reviews of one Amazon product across supported marketplaces. Returns rating, title, text, date, reviewer country, verified-purchase status, helpful votes, and variant. Sort by newest or most helpful; optionally select one star rating.
- `POST /v1/scrape/amazon/product` — Scrape Amazon Product: Scrape one Amazon product across supported marketplaces: title, price, rating, availability, brand, description, features, gallery images, and A+ content when available. $0.025 per product.
- `POST /v1/scrape/amazon/search` — Scrape Amazon Search: Search Amazon products by query and page. Returns listing cards with title, price, rating, and ASIN. US, UK, and DE marketplaces. Defaults to 10 results.
- `POST /v1/scrape/threads/posts` — Scrape Threads Posts: Scrape exact public Threads posts by URL. Returns normalized text, author, date, engagement, and media. Missing posts are listed in missingUrls after both scrapers have been tried.
- `POST /v1/scrape/linkedin` — alias — Scrape LinkedIn: Backward-compatible alias for LinkedIn profile scraping.
- `POST /v1/scrape/twitter` — alias — Scrape Twitter: Backward-compatible alias for X/Twitter search scraping.
- `POST /v1/scrape/deep` — Deep Scrape: Build a multi-source JSON dossier on a person, company, or topic. Returns 202; poll the request.
- `POST /v1/scrape/pdf` — Scrape PDF: Extract the text of a public PDF URL: full text plus title, author, and page count, returned synchronously at a fixed price per document.
- `POST /v1/scrape/youtube/thumbnail` — YouTube Thumbnail: Download the best available standard JPEG thumbnail from one YouTube video URL. Returns image data, source URL, and dimensions.
- `POST /v1/scrape/extract` — Extract Structured Data: Extract one structured JSON object per public page URL using a JSON Schema, a prompt, or both. No crawling. Flat price per page that returns data.

### Email

- `POST /v1/email/send` — Send Email: Create an email draft from a workspace email identity; set send=true to send it.
- `GET /v1/email/messages` — Receive Email: Read messages for a workspace email identity.
- `GET /v1/email/drafts` — List Drafts: List pending email drafts for a workspace email identity.
- `GET /v1/email/identities` — Email Identities: List the workspace email identities and the emailIdentityId values other email routes accept.
- `POST /v1/email/drafts/{draftId}/send` — Send Draft: Approve and send an existing draft by draftId after review.
- `POST /v1/email/domains` — Add Sending Domain: Add a customer-owned domain to send email from, and get the DNS records to publish.
- `GET /v1/email/domains` — Sending Domains: List the workspace's customer-owned sending domains with status and pending DNS records.
- `POST /v1/email/domains/{domainId}/verify` — Verify Sending Domain: Re-check the domain's DNS records and refresh its verification status.
- `DELETE /v1/email/domains/{domainId}` — Remove Sending Domain: Remove a custom sending domain and suspend the identities on it.
- `POST /v1/email/identities` — Create Email Identity: Create a sender identity (optionally on a verified custom domain) and make it the workspace default.
- `PATCH /v1/email/identities/{emailIdentityId}` — Update Email Identity: Update an email identity: change its sender display name, or enable the recurring renewal that keeps a trial inbox.
- `POST /v1/email/find` — Find Email: Find one person's professional email address from their name plus a company domain or company name. Returns the most likely address with a confidence score, catch-all status, verification result, and the public sources it was seen on.

### Enrichment

- `POST /v1/email/verify` — Verify Email: Check one email address and return a standardized deliverability verdict, score, and compact evidence flags.
- `POST /v1/email/enrich` — Enrich Email: Enrich one known professional email with a compact person and employment profile. Personal phone, home-location, avatar, biography, and social-feed data are omitted.
- `POST /v1/company/enrich` — Enrich Company: Enrich one company domain with a compact company profile: identity, classification, location, size, and freshness.

### Research

- `POST /v1/research/deep` — Deep Research: Answer a research question with current web evidence.

### Generate

- `POST /v1/generate/image` — Generate Image: Generate an image from a text prompt.

### Search

- `POST /v1/search/web` — Web Search: Search the web and return ranked results with title, url, snippet, and dateText when the source reports a date, plus a direct answer when one exists.

### SEO

- `POST /v1/seo/keyword` — SEO Keyword Metrics: Look up search volume, CPC, keyword difficulty, search intent, and 12-month trend for up to 100 keywords.
- `POST /v1/seo/rank` — SEO Rank Check: Check where a domain ranks in Google organic results for a keyword, with the live top 10.
- `POST /v1/seo/competitors` — SEO Competitors: Find the domains competing with a site in Google organic search, with overlap and traffic estimates.
- `POST /v1/seo/audit` — SEO Audit: One keyword in, a ranking plan out: live metrics, the current top 10, an analysis of what those pages cover, and a concrete outline to beat them.
- `POST /v1/seo/optimize` — SEO Optimize: Score a draft (or live page) against a target keyword for SEO and AI-answer-engine visibility, with prioritized edits and an optional rewrite.

### Audio

- `POST /v1/transcribe` — Transcribe Audio: Convert uploaded audio into plain text.
- `POST /v1/transcribe/uploads` — Create Audio Upload: Create a temporary signed upload for one audio file.

### Memory

- `GET /v1/memory` — List Memory: List the markdown files in this workspace's hosted memory, with sizes, versions, and usage against the limits.
- `POST /v1/memory/{path}` — Write Memory: Create or update one memory file. Writes replace the whole file and bump its version.
- `GET /v1/memory/{path}` — Read Memory: Read one memory file: full markdown content plus its current version for safe writes.
- `DELETE /v1/memory/{path}` — Delete Memory: Delete one memory file permanently.

### Browser

- `POST /v1/browser/act` — Browser Task: Give a real cloud browser a plain-English goal and get the result back. Built for pages an agent has to operate, not just read: filters and sortable tables, JavaScript pagination, dropdowns, date pickers and sliders, iframes and shadow DOM, infinite scroll, site search, store locators, and comparing several pages in one run. Public web only — no logins, purchases, or CAPTCHA solving.

### Virtual Machines

- `GET /v1/vm/files/{requestId}/{fileIndex}` — Download VM File: Download a saved VM output file after execution, including after the VM has been deleted. Use the downloadPath returned in output.files.
- `POST /v1/vm/run` — Run in a Virtual Machine: Run a task in a fresh, isolated virtual machine with internet access, a writable filesystem, shell tools, administrator access (sudo), and Docker. Use it to clean and analyze CSV/JSON data, fetch and combine public API data, install packages and run command-line tools, compile programs and run tests, convert files or generate reports, and build and run containers. Send one Bash, Python, Node.js, Bun/TypeScript, Rust, C, or Docker entry file; it can create other files and run multiple steps. Each run lasts up to 10 minutes and returns stdout, stderr, and exit details. Supply input files and list outputFiles to download the results after the VM is discarded. There is no persistent session or hosted service.

### Account

- `GET /v1/balance` — Balance: Read the workspace credit balance without spending anything.
- `GET /v1/me` — Account Info: Read what this API key can do: workspace, scopes, spend limits, remaining key budget, rate limits, and balance.
- `GET /v1/capabilities` — Capabilities: List every DeepAPI capability with its live status, or pass capability=<slug> to read one capability's full live contract.
- `GET /v1/usage` — Usage Summary: Read workspace spend totals, a gap-filled per-day series, and a per-capability breakdown over the last sinceDays calendar days, counting today as day one.

### Requests

- `GET /v1/requests` — List Requests: List recent requests created by this API key, newest first. Recovers a recently lost requestId so its result can be re-fetched.
- `GET /v1/requests/{requestId}` — Request Status: Fetch the stored result of a recent request by requestId — free, instead of re-running paid work — or poll it until its GET polling next action is absent (output can settle after status turns succeeded).

### Feedback

- `POST /v1/feedback` — Send Feedback: Send a bug report, idea, or praise to the DeepAPI team. Free, any active key.

## Example requests

Scrape a website — `maxCostUsd` caps what the call may spend.

```bash
curl -X POST "$DEEPAPI_API_BASE_URL/v1/scrape/website" \
  -H "Authorization: Bearer $DEEPAPI_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: scrape-$(uuidgen)" \
  -d '{
    "maxCostUsd": "0.10",
    "waitForFinishSecs": 60,
    "urls": ["https://example.com"],
    "maxPages": 1
  }'
```

Draft an email — stays a draft unless `send` is true. No inbox IDs; the key resolves the identity. The response returns `output.draftId`.

```bash
curl -X POST "$DEEPAPI_API_BASE_URL/v1/email/send" \
  -H "Authorization: Bearer $DEEPAPI_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: email-$(uuidgen)" \
  -d '{
    "to": "person@example.com",
    "subject": "Quick hello",
    "text": "Hi, this is a draft from my agent.",
    "send": false
  }'
```

Review, then approve & send — list pending drafts, pick one, and send it exactly as stored. The body stays empty; the draft already holds recipients and content.

```bash
curl "$DEEPAPI_API_BASE_URL/v1/email/drafts" \
  -H "Authorization: Bearer $DEEPAPI_API_KEY"

curl -X POST "$DEEPAPI_API_BASE_URL/v1/email/drafts/draft_123/send" \
  -H "Authorization: Bearer $DEEPAPI_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: approve-$(uuidgen)" \
  -d '{}'
```

## Response envelope

```json
{
  "requestId": "...",
  "route": "/v1/scrape/website",
  "capability": "scrape.website",
  "status": "succeeded",
  "replayed": false,
  "costFinal": true,
  "debitMicrousd": 2500,
  "output": [],
  "next": null,
  "error": null,
  "balance": {
    "postedCreditsMicrousd": 1000000,
    "postedDebitsMicrousd": 2500,
    "activeReservationsMicrousd": 0,
    "availableMicrousd": 997500
  },
  "skillVersion": "1320a0ca895c"
}
```

`debitMicrousd` is USD micro-dollars — 2500 is $0.0025. If the response includes a polling `next` (a GET of `/v1/requests/{id}`), wait `next.afterSecs` seconds and call it; repeat while that polling `next` is present — even when `status` is already `succeeded` (output can still be settling).

## Custom sending domains

Send from your own domain instead of the standard DeepAPI address: `POST /v1/email/domains` registers it (one-time fee) and returns the DNS records to publish; `POST /v1/email/domains/{domainId}/verify` is free and repeatable; then `POST /v1/email/identities` with `domain` creates your sender address on it. The Email page walks through the same flow with copy-paste DNS records.

Domain already handles email (Google Workspace, Outlook, etc.)? Add a subdomain like `agent.yourcompany.com` instead of the root. Sending needs only the SPF and DKIM records, so existing email keeps working untouched — the MX record is only required if the agent should also RECEIVE mail on that domain.

A verified domain rides its own sending reputation and gets higher automatic send limits than the shared domain.

## Error codes

Every failed response carries `error.code`, `error.retryable`, `error.retryAfterSecs`, and `error.hint`.
`error.retryable: true` means the hint may be followed automatically. When it is false, do not auto-retry; the hint may require fixing input, changing state, or reviewing side effects first. Wait `error.retryAfterSecs` when it is not null.

| Code | HTTP | Retryable | Meaning | What to do |
| --- | --- | --- | --- | --- |
| `missing_api_key` | 401 | false | No bearer API key on the request. | Send `Authorization: Bearer $DEEPAPI_API_KEY`. |
| `invalid_api_key` | 401 | false | The API key is unknown, revoked, or expired. | Ask the user for a valid key. Do not retry with the same key. |
| `missing_idempotency_key` | 400 | false | POST request without an `Idempotency-Key` header. | Send a unique `Idempotency-Key` and retry. |
| `missing_scope` | 403 | false | The API key lacks the scope in `error.requiredScope`. | Ask the user for a key with that scope. Do not retry unchanged. |
| `card_setup_required` | 403 | false | The workspace has not completed required card setup. | Ask the user to complete setup at https://deepapi.co/card-setup, then retry. |
| `invalid_request` | 400 | false | A request field is invalid; `error.field` names it. | Fix the field per `error.message`, then retry with a new `Idempotency-Key`. |
| `insufficient_credits` | 402 | false | The workspace balance cannot cover the requested spend cap. | Pause and point the user to https://deepapi.co/credits — a one-time top-up or Auto Top-Up both unblock it. Ask whether to open the page. If they agree, use `open` (macOS), `Start-Process` (Windows), or `xdg-open` (Linux); if no desktop browser is available, print the link. Then retry with the same `Idempotency-Key`. |
| `api_key_limit_exceeded` | 402 | false | A per-request or total spend limit on this API key blocks the request. | Lower `maxCostUsd`, or ask the user to raise the key limit. |
| `rate_limit_exceeded` | 429 | true | Too many requests, or too many failed auth attempts, this minute. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. |
| `upstream_rate_limited` | 429 | true | The upstream provider rate-limited the request. | Wait `error.retryAfterSecs`, then retry with a new `Idempotency-Key`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `idempotency_operation_mismatch` | 409 | false | This `Idempotency-Key` already belongs to a different operation. | Use a new idempotency key for this operation. Retry the original operation with its original key to recover its outcome. |
| `idempotency_conflict` | 409 | true | The same `Idempotency-Key` belongs to a request that is still in progress. | Wait `error.retryAfterSecs`, then retry with the same key to receive the finished outcome (success or failure is replayed). Use a new key to attempt the operation again after a failure. |
| `unknown_capability` | 404 | false | No such endpoint, scrape target, or kind. | Use a documented endpoint path — `GET /v1` lists every endpoint. Do not retry unchanged. |
| `method_not_allowed` | 405 | false | The endpoint exists but not with this HTTP method; the `Allow` header lists the supported methods. | Retry using a method from the `Allow` header. `GET /v1` lists every endpoint with its method. |
| `resource_not_found` | 404 | false | The requested resource is missing or inaccessible. | Check the resource identifier and access. Do not retry unchanged. |
| `capability_not_configured` | 501 | false | The route exists but has no backend configured. | Do not retry. Report this to the user. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `request_not_found` | 404 | false | No request with this id exists for this API key. | Check `requestId`. Poll only requests created with the same key. |
| `email_identity_not_found` | 404 | false | `emailIdentityId` does not belong to this workspace. | Omit `emailIdentityId` to use the workspace default identity. |
| `email_draft_not_found` | 404 | false | No such draft for this email identity. | List drafts via `GET /v1/email/drafts` and use a returned `draftId`. |
| `email_policy_rejected` | 403 | false | Send policy blocked the request: recipient rules, content rules, a paused workspace, or the daily/monthly send cap. Caps grow automatically with clean sending history. | Follow `error.message`. If a cap was reached, retry after the window resets or create a draft instead. |
| `email_not_configured` | 503 | false | The workspace has no active email inbox. | Create the inbox via `POST /v1/email/identities`: it returns the price, then retry with `confirmInboxCharge: true` after user approval. Or create it at https://deepapi.co/email. Adding credits alone does not create an inbox. |
| `email_identity_confirmation_required` | 409 | false | This email operation needs a new paid inbox. Nothing was created or charged. | Ask the user to choose the sender name and approve `error.setupPriceMicrousd` for the first 30 days and `error.renewalPriceMicrousd` every 30 days. This is one recurring inbox price, not a separate setup fee. Retry with `confirmInboxCharge: true` and optional `username`/`displayName`. |
| `email_domain_not_found` | 404 | false | No custom sending domain with this `domainId` in this workspace. | List domains via `GET /v1/email/domains` and use a returned domain id. |
| `email_domain_not_verified` | 403 | false | The custom domain exists but its DNS records are not verified yet. | Publish the dnsRecords from `GET /v1/email/domains`, then `POST /v1/email/domains/{domainId}/verify` until `verified` is true. Checks are free. |
| `email_domain_limit_exceeded` | 403 | false | The workspace reached its custom sending domain limit. | Remove an unused domain via `DELETE /v1/email/domains/{domainId}`, then retry. |
| `email_domain_conflict` | 409 | false | This domain is already registered with DeepAPI email by another workspace. | Stop and tell the user. If they own the domain, they should contact support. |
| `pdf_too_large` | 403 | false | The PDF file exceeds the size limit (about 50 MB). Nothing was charged. | Use a smaller PDF or a URL that serves the document in parts. Do not retry unchanged. |
| `pdf_not_readable` | 422 | false | The URL did not yield readable PDF text: not a PDF, password-protected, corrupted, or a scanned image with no text layer. Nothing was charged. | Check the URL serves an unencrypted, text-based PDF. Scanned PDFs need OCR, which this route does not do. Do not retry unchanged. |
| `audio_too_large` | 413 | false | The audio file exceeds 25 MB. Nothing was charged. | Compress or split the audio, create a new upload, then retry. |
| `audio_upload_not_found` | 404 | false | The temporary audio upload is missing, expired, or already consumed. | Create and upload a new audio file, then retry with a new `Idempotency-Key`. |
| `audio_upload_failed` | 502 | false | DeepAPI could not read or delete the temporary audio upload. Nothing was charged. | Create and upload a new audio file, then retry with a new `Idempotency-Key`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `audio_not_readable` | 422 | false | The uploaded file did not contain readable supported audio. Nothing was charged. | Check the file format and audio content, then retry with a new upload and `Idempotency-Key`. |
| `transcription_failed` | 502 | true | The transcription backend failed after the request started. Nothing was charged. | Retry the same `uploadId` with a new `Idempotency-Key` before the upload expires. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `memory_file_not_found` | 404 | false | No memory file exists at this path for this workspace. | List files via `GET /v1/memory` to see what exists. To create the file, POST it with `content`. |
| `memory_limit_exceeded` | 403 | false | The workspace memory quota blocks this write: too many files, a file over the per-file size limit, or the workspace total is full. | Delete or shrink memory files via `GET /v1/memory` and `DELETE /v1/memory/{path}`, then retry. |
| `memory_version_conflict` | 409 | false | The file changed since the version you sent as `ifVersion` — another agent wrote it first. | GET the file again, merge your changes into the latest content, and retry with the new version. |
| `browser_task_rejected` | 403 | false | Browser task policy blocked the request. Nothing was charged. | Rework the task to public-web actions only, then retry with a new `Idempotency-Key`. |
| `browser_task_failed` | 502 | false | The browser task failed or stopped after it started. Nothing was charged, but it may have completed some actions. | Review the task state, then use a new `Idempotency-Key` only if another attempt is safe. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `browser_task_state_unknown` | 502 | false | The browser backend did not confirm whether task creation succeeded. Nothing was charged, but the task may have started. | Do not retry automatically. Review external effects before using a new `Idempotency-Key`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `request_failed` | 502 | false | The provider run for a started request failed. Failed calls are free: the credit hold is released, nothing is charged, and `debitMicrousd` is null. | Do not retry automatically. The same `Idempotency-Key` only replays this failure. Start a new request with a new key only after confirming another attempt is safe. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `request_timeout` | 502 | true | The provider run for a started request hit its time limit before finishing. Failed calls are free: the credit hold is released, nothing is charged, and `debitMicrousd` is null. | Wait `error.retryAfterSecs` when provided, then retry ONCE with a new `Idempotency-Key` — the same key only replays this recorded failure. If it times out again, stop retrying and simplify the request. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `request_budget_exceeded` | 400 | false | The scrape hit the request spend cap before finishing. Failed calls are free: the credit hold is released, nothing is charged, and `debitMicrousd` is null. | Raise `maxCostUsd` or narrow the request (fewer URLs, or `maxPages` on website scrapes), then retry with a new `Idempotency-Key`. |
| `scrape_request_failed` | 502 | true | Unexpected server error while handling a scrape request. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `search_request_failed` | 502 | true | Unexpected server error while handling a web search request. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `research_request_failed` | 502 | true | Unexpected server error while handling a deep research request. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `generate_image_request_failed` | 502 | true | Unexpected server error while handling an image generation request. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `transcription_upload_request_failed` | 502 | true | Unexpected server error while handling an audio upload request. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `transcription_request_failed` | 502 | true | Unexpected server error while handling an audio transcription request. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `seo_request_failed` | 502 | true | Unexpected server error while handling an SEO request. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `seo_budget_exceeded` | 400 | false | The SEO analysis stopped because the spend cap could not cover the next step. Nothing was charged. | Raise `maxCostUsd` (or drop expensive include blocks), then retry with a new `Idempotency-Key`. |
| `memory_request_failed` | 502 | true | Unexpected server error while handling a memory request. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `browser_request_failed` | 502 | true | Unexpected server error while handling a browser task request. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `vm_run_request_failed` | 502 | true | Unexpected server error while handling a VM run request. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `email_draft_failed` | 502 | true | Unexpected server error while handling an email draft request. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `email_send_failed` | 502 | true | Unexpected server error while handling an email send request. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `email_retrieval_failed` | 502 | true | Unexpected server error while handling an email read request. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `email_draft_send_failed` | 502 | true | Unexpected server error while handling a draft send request. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `email_domain_request_failed` | 502 | true | Unexpected server error while handling an email domain request. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `email_find_request_failed` | 502 | true | Unexpected server error while handling an email find request. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `email_find_target_unavailable` | 400 | false | The supplied company or domain could not be processed. Nothing was charged. | Do not retry this target unchanged. Continue with the rest of the list. |
| `email_find_opted_out` | 451 | false | This person asked to be excluded from contact discovery, so no address can be returned. Nothing was charged. | Never look this person up again and do not contact them. Continue with the rest of your list. |
| `enrichment_request_failed` | 502 | true | Unexpected server error while handling an enrichment request. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `email_verify_restricted` | 451 | false | The requested email record cannot be returned for legal or privacy reasons. Nothing was charged. | Do not retry this address. Continue with the rest of the task. |
| `email_enrich_restricted` | 451 | false | The requested person record cannot be returned for legal or privacy reasons. Nothing was charged. | Do not retry this address. Continue with the rest of the task. |
| `company_enrich_restricted` | 451 | false | The requested company record cannot be returned for legal or privacy reasons. Nothing was charged. | Do not retry this domain. Continue with the rest of the task. |
| `email_identity_create_failed` | 502 | true | Unexpected server error while handling an email identity create request. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `email_identity_update_failed` | 502 | true | Unexpected server error while handling an email identity update. Nothing was charged. | Wait `error.retryAfterSecs`, then retry the same request once. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `request_lookup_failed` | 502 | true | Unexpected server error while handling a request status lookup. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `request_list_failed` | 502 | true | Unexpected server error while handling a request list read. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `balance_lookup_failed` | 502 | true | Unexpected server error while handling a balance read. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `account_lookup_failed` | 502 | true | Unexpected server error while handling an account info read. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `usage_lookup_failed` | 502 | true | Unexpected server error while handling a usage summary read. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `capability_list_failed` | 502 | true | Unexpected server error while handling a capability list read. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. If this DeepAPI failure remains unresolved, send exactly one free `POST /v1/feedback` with `category: "bug"`, a short description, and this response's `requestId` as `context` when present. Never include secrets or private data. Do not wait for feedback or let it block the user's task. |
| `feedback_submit_failed` | 502 | true | Unexpected server error while handling a feedback submission. Nothing was charged. | Wait `error.retryAfterSecs`, then retry with the same `Idempotency-Key`. If it keeps failing, check `GET /v1/health`. |

---

Rendered live from the DeepAPI capability catalog. skillVersion: 1320a0ca895c
