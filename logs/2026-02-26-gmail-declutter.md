# Gmail Declutter Agent

**Date:** 2026-02-26
**Location:** /Users/henryrussell/Projects/Claude builds/gmail_declutter

## Summary
Built a fully autonomous Gmail filtering agent that runs daily in the cloud with zero ongoing effort. The agent classifies incoming emails using heuristics + Claude Haiku, automatically unsubscribes from mailing lists by firing real HTTP unsubscribe requests, archives junk, and sends a weekly digest email every Sunday. On first run it cleared a backlog of 6,106 unread emails.

## Tech Stack
- Python 3.13 (local) / Python 3.14 (Lambda)
- Google Gmail API + People API (OAuth 2.0)
- Anthropic Claude Haiku (email classification)
- AWS Lambda (serverless execution)
- AWS EventBridge (daily + weekly cron scheduling)
- AWS Secrets Manager (OAuth token storage)
- httpx (HTTP unsubscribe requests)
- boto3 (AWS SDK)

## Files Built
- `src/gmail_client.py` — Gmail API wrapper: auth, read emails, archive, label, send, contacts lookup
- `src/classifier.py` — heuristics-first classification engine with Claude Haiku fallback
- `src/unsubscriber.py` — List-Unsubscribe header parser; tries RFC 8058 POST → mailto → HTTP GET
- `src/weekly_summary.py` — weekly HTML + plain text digest generator
- `src/main.py` — orchestrator with DRY_RUN mode and process/weekly run types
- `src/aws_secrets.py` — Secrets Manager read/write helper for Lambda token persistence
- `lambda_function.py` — AWS Lambda entry point; accepts EventBridge config via event payload
- `scripts/setup_credentials.py` — one-time OAuth browser flow to generate token.json
- `scripts/build_lambda.sh` — packages code + Linux binaries into lambda.zip for upload
- `scripts/mark_all_inbox_as_read.py` — utility to bulk-mark all unread inbox emails as read
- `tests/test_classifier.py` — unit tests with mocked Claude API
- `.env.example`, `.gitignore`, `requirements.txt`

## Key Decisions
- **Heuristics first, Claude second** — check sender patterns, marketing domains, and List-Unsubscribe headers before hitting Claude API. Keeps cost near zero (~$0.001/batch)
- **DRY_RUN=true by default** — safe to run anytime; prints decisions without touching the inbox
- **Metadata-only email fetching** — switched from `format="full"` to `format="metadata"` in Gmail API calls to avoid timeout errors when processing 500 emails sequentially
- **Secrets Manager for OAuth token** — Lambda can't write to disk between invocations, so token.json is stored in Secrets Manager and written back after each refresh
- **`_IN_LAMBDA` flag** — detected via `AWS_LAMBDA_FUNCTION_NAME` env var (set automatically by Lambda), so local dev requires zero changes
- **Linux binaries for Lambda** — used `--platform manylinux2014_x86_64` with pip to get correct compiled binaries for the `cryptography` library
- **batchModify for bulk operations** — used Gmail batchModify API to mark thousands of emails as read in one request rather than one-by-one

## Lessons Learned
- Gmail's `resultSizeEstimate` is wildly inaccurate — reported 201 unread when there were actually 6,106
- Lambda default timeout is 3 seconds — must increase to 5 minutes for any real workload
- AWS Secrets Manager is region-specific — secrets created in us-east-1 are not visible to a Lambda in us-east-2 unless you explicitly set the region in the boto3 client
- Python 3.14 on Lambda + old anthropic library causes Pydantic V1 warnings — upgrade to `anthropic>=0.50.0`
- macOS-compiled Python packages with C extensions (like `cryptography`) produce `invalid ELF header` errors in Lambda — always build Lambda zips with `--platform manylinux2014_x86_64`
- Gmail OAuth refresh tokens don't change often, but access tokens expire hourly — always write the refreshed token back to Secrets Manager so the next invocation doesn't have to re-authenticate
