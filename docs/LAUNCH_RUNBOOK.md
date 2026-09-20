# HISABIQ Launch Runbook V2

## Phase 0 — Build
- Implement the approved product scope.
- Keep Astra as the sole reasoning engine.
- Use tools as execution/data infrastructure.

## Phase 1 — Dry run
- Run deterministic calculator tests.
- Validate analytics event schema.
- Validate fulfillment artifacts.
- Confirm no live payment path is active.
- Record failures before deployment.

## Phase 2 — Organic launch
- Publish approved public build.
- Run organic content experiments from the approved creative/SEO pack.
- Measure calculator usage and conversion events.
- Do not fabricate results or testimonials.

## Phase 3 — Payment readiness
- Verify applicable business/e-commerce eligibility.
- Verify payment-provider activation requirements.
- Verify refund/support/customer-contact requirements.
- Keep live checkout disabled until the owner-approved gate is satisfied.

## Phase 4 — Controlled paid experiment
- Define one hypothesis.
- Define exact budget and stop condition.
- Obtain owner approval before spending.
- Run one controlled experiment.
- Do not auto-scale spend.

## Phase 5 — Factory automation
Target routine execution:
`Build → Test → Package → Deploy → Verify → Monitor → Iterate`

Astra should perform routine deployment operations through an approved, least-privilege deployment adapter once that capability is implemented.

### Deployment authority
Owner approval is required for:
- new credentials or external account connections
- paid infrastructure or subscriptions
- domain purchase
- advertising spend
- live checkout/payment activation
- pricing changes
- legal registration/submission
- external commercial commitments
- new product scope

Routine deployment of an already-approved release may be automated only after the deployment target, credential scope, and release policy have been pre-authorized.

## Current HISABIQ deployment
- Cloudflare Pages project: `hisabiq`
- Production branch: `main`
- Automatic deployments: enabled
- Public domain: `hisabiq.pages.dev`
- Deployment shown successful by Cloudflare
- Spend: SAR 0
- Independent production HTTP verification: pending

## Release rule
A successful Git commit or provider deployment status is not sufficient to declare the product LIVE. Production verification must pass.

## Rollback rule
If verification fails:
1. Mark release NOT VERIFIED.
2. Preserve the failing release record.
3. Diagnose.
4. Roll back only if the rollback policy has already been approved.
5. Otherwise escalate to owner review.
