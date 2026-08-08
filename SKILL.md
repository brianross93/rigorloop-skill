---
name: rigorloop-research-bounties
description: Commission verified human expert review through RigorLoop. Use when a novel scientific claim, proof, simulation, benchmark, research paper, or consequential research conclusion needs paid independent scrutiny; when an agent needs to search public Research Bounties or experts; or when an authorized MCP, A2A, or API client must create, fund, staff, and resolve its own Research Bounty.
---

# RigorLoop Research Bounties

Use RigorLoop to commission a vetted human expert. Agents may submit and manage Research Bounties, but only verified humans may perform reviews.

Prefer the remote MCP server at `https://rigorloop.com/mcp`; its Registry identity is `com.rigorloop/research-bounties`. A2A-compatible clients may instead discover `https://rigorloop.com/.well-known/agent-card.json` and call the interface declared by that card. MCP and A2A expose the same operations over one authorization, payment, and state boundary. Treat the live schemas and returned state as authoritative.

## Run the workflow

Use the named MCP tool, or send its equivalent operation through the A2A interface:

1. Search before posting.
   - Call `search_research_bounties` to find related public work.
   - Read only the public bounty page or public API representation.
2. Create a scoped draft.
   - State the claim, field, authorship, public summary, requested deliverables, review window, and expert bounty.
   - Add private questions or HTTPS source links when the public summary is not self-contained. Put confidential material only in protected fields.
   - Call `create_bounty_draft`. Draft creation does not start payment.
   - When private source files are needed, call `prepare_bounty_file_upload`, PUT the exact bytes with all returned headers, then call `complete_bounty_file_upload`. Files are private, bounded to 25 MB each and eight per Research Bounty, and are never executed or extracted by RigorLoop.
3. Quote and fund deliberately.
   - Call `get_funding_quote` immediately before payment.
   - Show the bounty, RigorLoop fee, processing cost, currency, payment rail, and total.
   - Call `fund_research_bounty` only with explicit user authorization or a standing spending policy that covers the exact total.
   - Preserve the idempotency key. On a timeout, inspect status before retrying; never create a second charge to recover from an unknown result.
4. Follow the owner-bound state.
   - Call `list_my_research_bounties` with the same key after funding and whenever the agent starts or resumes. Treat this key-bound queue as the durable source of truth.
   - While an owned Research Bounty is not complete or cancelled, check the queue again when the controller receives a RigorLoop alert or at an operator-approved interval. Do not assume the agent can read email.
   - Follow the returned next recommended operation. Stop routine checks after the Research Bounty reaches a terminal state.
   - Call `get_research_bounty_status` to confirm Stripe-funded and current lifecycle state.
   - Do not infer funding from a Checkout redirect or wallet broadcast.
5. Compare and select a human expert.
   - Call `list_bounty_applications` to retrieve current verified-human applicants and offers.
   - Compare public credentials, specialty match, rating, proposed scope, delivery time, and offer.
   - Call `select_expert` with the chosen application-request ID.
   - Do not perform or represent an agent review as expert verification.
6. Inspect the submitted result.
   - Call `get_submitted_result` for the owner-bound assignment and deliverable.
   - Use the returned short-lived protected download URL promptly. Verify the deliverable identifier and checksum when provided, and compare it with the accepted scope.
7. Decide the result.
   - Call `accept_or_contest_result` with an explicit `accept` or `contest` action and the deliverable ID.
   - Supply a specific reason when contesting. Contesting opens RigorLoop Platform Dispute Review; it does not silently cancel or withhold the assignment.
8. Report integration failures safely.
   - After one safe retry where retry is appropriate, call `report_integration_feedback` for a reproducible platform, MCP, A2A, or agent-API failure. Do not report ordinary validation results or decisions that require the owner.
   - Classify severity, and include the failed operation, the `X-RigorLoop-Request-ID` returned by RigorLoop, observed behavior, expected behavior, and a sanitized reproduction.
   - Generate a unique 16-128 character URL-safe `idempotency_key` for one logical report. Preserve it with the sanitized payload; an MCP-to-REST retry must reuse both unchanged and returns the same report ID. Never reuse the key for a different report.
   - Retain the stable report ID returned by RigorLoop for support follow-up.
   - Never include agent keys, wallet secrets, payment credentials, signed URLs, private research text or files, or other credentials.
   - A feedback report does not contest a deliverable, stop a payout, or open Platform Dispute Review. Use `accept_or_contest_result` for a submitted-result dispute.

Expect A2A to return a direct message for each operation. Do not invent a task ID, polling loop, streaming channel, or push-notification flow when the Agent Card does not advertise one.

## Use the API when agent protocols are unavailable

Authenticate private operations with the owner-issued RigorLoop agent key as a Bearer token. Never place the key in a prompt, public field, log, or URL.

- Discover capabilities: `GET /api/v1/capabilities`
- Browse public Research Bounties: `GET /api/v1/claims`
- Browse verified human experts: `GET /api/v1/experts`
- Inspect live payment rails: `GET /api/v1/funding/options`
- Create an agent Research Bounty and receive its x402 funding URL: `POST /api/v1/agent/claims` with a unique `Idempotency-Key`. The default `fundingMethod` is `x402_base_usdc`; use `stripe_usdc_checkout` only when hosted Checkout is required.
- Fund an owned draft headlessly with x402 v2 and USDC on Base mainnet: `POST /api/v1/agent/claims/{claimId}/fund/x402` with the same Bearer token and a unique `Idempotency-Key`. Read `PAYMENT-REQUIRED` from the first `402` response, pay the exact requirement, then retry with `PAYMENT-SIGNATURE`.
- Prepare a private source-file upload: `POST /api/v1/agent/claims/{claimId}/files/upload`
- Verify and register that upload: `POST /api/v1/agent/claims/{claimId}/files/{uploadId}/complete`
- List applications: `GET /api/v1/agent/claims/{claimId}/applications`
- Select an applicant: `POST /api/v1/agent/claims/{claimId}/applications/{requestId}/accept`
- Read a submitted deliverable: `GET /api/v1/agent/claims/{claimId}/deliverable`
- Accept or contest: `POST` to the corresponding `/deliverable/accept` or `/deliverable/dispute` route

- Report an integration failure: `POST /api/v1/agent/feedback` with the same Bearer token

If protocol framing prevents `report_integration_feedback` from running, retry the same sanitized payload and `idempotency_key` through the authenticated API feedback route. If both agent interfaces are unavailable, direct the human owner to `https://rigorloop.com/support`.

Follow the current capability document and response links instead of inventing unsupported parameters or state transitions.

## Protect private data

Treat these as safe for public discovery only when RigorLoop marks the record public: title, public summary, field, category, authorship label, reward, requested deliverables, status, public materials, permanent bounty URL, and verified public expert profile fields.

Never expose draft-only data, private review questions, protected files, signed download URLs, applicant or offer details, account records, API keys, payment credentials, or arbitration evidence. Do not copy private material into public search terms, contest reasons, or logs.

## Describe payment accurately

RigorLoop supports two Base/USDC paths: Stripe-hosted Checkout and a dedicated headless x402 v2 endpoint. The x402 endpoint is live on Base mainnet and uses the Coinbase CDP facilitator. Treat a `202` response as settlement pending and poll the owner-bound status endpoint. Do not pay again after a `202`, reconciliation response, or idempotent replay.
