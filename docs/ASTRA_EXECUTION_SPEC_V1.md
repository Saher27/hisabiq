# Astra Execution Specification V1.0 — Deployment & External Execution Extension

## 1. Purpose

This document extends the HISABIQ / Astra execution architecture from product construction into repeatable product deployment.

Target operating loop:

`Build → Test → Package → Deploy → Verify → Monitor → Iterate`

Astra remains the single reasoning engine. External services such as GitHub, Cloudflare, analytics, storage, payment providers, and automation systems are execution/data infrastructure rather than additional reasoning brains.

## 2. Authority Model

### Astra may execute autonomously
- Inspect and modify the approved product codebase.
- Run deterministic local tests.
- Generate deployment manifests/configuration.
- Create commits in the approved repository.
- Prepare deployment requests.
- Read deployment status and logs.
- Run post-deployment deterministic smoke tests.
- Record results, failures, and rollback recommendations.
- Prepare the next engineering/content/marketing task.

### Human approval is required
- Entering or changing credentials/secrets.
- Connecting a new external account.
- Paying for infrastructure, domains, APIs, advertising, or subscriptions.
- Activating live payment collection.
- Making legal/regulatory submissions.
- Sending external commercial communications when not already explicitly authorized.
- Changing product pricing.
- Expanding product scope beyond the approved product brief.
- Destructive production actions without a previously approved rollback policy.

### Current implementation boundary

The current Astra browser controller is intentionally restricted. It does not log into external services, handle API keys, make purchases, submit forms, or make external commitments.

Therefore the deployment capability defined below is the **target execution specification**, not a claim that the current Astra worker can already perform authenticated Cloudflare deployment by itself.

## 3. Execution States

### State machine

`DRAFT → BUILDING → TESTING → PACKAGED → DEPLOY_PENDING → DEPLOYING → VERIFYING → LIVE`

Failure paths:

`BUILDING → FAILED`
`TESTING → FAILED`
`DEPLOYING → FAILED`
`VERIFYING → FAILED`

Recovery:

`FAILED → DIAGNOSE → RETRY | ROLLBACK | HUMAN_REVIEW`

### State definitions

| State | Meaning | Exit condition |
|---|---|---|
| DRAFT | Approved work item exists | Build started |
| BUILDING | Code/artifacts being generated | Build completes |
| TESTING | Deterministic QA running | Required tests pass |
| PACKAGED | Deployment candidate is immutable/versioned | Deployment target prepared |
| DEPLOY_PENDING | External deployment action is ready | Permission gate satisfied |
| DEPLOYING | Deployment has been submitted | Provider reports result |
| VERIFYING | Production smoke tests running | Acceptance tests pass |
| LIVE | Deployment verified | Monitoring active |
| FAILED | A stage failed | Diagnostic record created |
| HUMAN_REVIEW | Automated execution cannot safely continue | Owner decision |

## 4. Deployment Contract

### Input schema

```json
{
  "task_id": "string",
  "product": "hisabiq",
  "repository": "Saher27/hisabiq",
  "branch": "main",
  "deployment_target": "cloudflare_pages",
  "project_name": "hisabiq",
  "build_command": "exit 0",
  "output_directory": "/",
  "requires_live_credentials": true,
  "owner_approval_ref": "string|null"
}
```

### Result schema

```json
{
  "task_id": "string",
  "state": "LIVE|FAILED|HUMAN_REVIEW",
  "repository": "Saher27/hisabiq",
  "commit": "string",
  "deployment_url": "string|null",
  "verified": false,
  "tests": [],
  "errors": [],
  "rollback_available": false,
  "timestamp": "ISO-8601"
}
```

## 5. Required Tools / Adapters

### Repository adapter
Capabilities:
- read file
- create/update file
- commit
- inspect branch
- compare changes
- retrieve commit metadata

Current implementation: GitHub connector.

### Build/test adapter
Capabilities:
- execute deterministic tests
- validate HTML/JS structure
- run calculator test matrix
- produce machine-readable results

Current implementation: local/Python execution plus repository artifacts.

### Deployment adapter — target capability
Capabilities:
- authenticate through an approved credential mechanism
- create/update deployment configuration
- trigger deployment
- retrieve deployment status
- retrieve build/deployment logs
- identify active production deployment

Target implementation options:
1. Cloudflare API token stored outside prompts and exposed only to the deployment adapter.
2. A controlled CI/CD identity with least-privilege permissions.
3. Git-based automatic deployment where Astra only pushes an approved commit and then verifies provider status.

Preferred design: **Git-based deployment + least-privilege provider integration**, because it minimizes direct credential handling.

### Verification adapter
Capabilities:
- HTTP GET
- inspect status code
- inspect page title
- verify required DOM markers
- verify calculator JavaScript execution
- run known input/output cases
- confirm prohibited live-payment behavior remains disabled

## 6. Permission Rules

### Allowed without owner approval
- Read repository.
- Modify approved source files.
- Run tests.
- Create commits in the approved repository.
- Trigger a deployment only when the deployment target and credentials have already been approved and are within the pre-authorized scope.
- Read deployment logs/status.
- Run production smoke tests.
- Recommend rollback.

### Blocked
- Discovering or extracting secrets from arbitrary locations.
- Printing secrets into logs.
- Using credentials outside their approved service/scope.
- Purchasing or upgrading plans.
- Adding paid services.
- Enabling live checkout.
- Changing prices.
- Sending customer-facing communications without the applicable approval.
- Expanding to a new product without approval.

## 7. Decision Rules

1. **No test pass → no production deployment.**
2. **No approved deployment target → HUMAN_REVIEW.**
3. **Credentials required but unavailable → HUMAN_REVIEW.**
4. **Deployment succeeds but verification fails → do not declare LIVE.**
5. **A known-good previous deployment exists and rollback is authorized → recommend/execute rollback according to the approved rollback policy.**
6. **Provider reports success but HTTP verification fails → classify as FAILED, not LIVE.**
7. **A production deployment that changes payment, pricing, legal text, or customer commitments requires the corresponding approval gate before release.**
8. **Never infer that a deployment is live solely from a successful commit.**
9. **Never claim a smoke test passed when the verification environment could not reach the deployment.**
10. **All deployment attempts must produce an auditable result record.**

## 8. Verification Test Set

Minimum production smoke test:

- HTTP response is successful.
- Expected page title is present.
- Expected brand marker is present.
- Calculator inputs exist.
- Calculator produces deterministic output for a known test case.
- Negative-profit case remains visible.
- Toolkit CTA exists.
- Live payment collection is not active unless explicitly approved.
- No unexpected authentication wall.
- No obvious JavaScript initialization error.
- Production commit matches the intended release commit.

### HISABIQ release test cases

The deployed calculator must be checked against the approved pre-launch matrix:

- T01 Base
- T02 Zero returns
- T03 High CAC stress
- T04 Low margin price
- T05 No VAT scenario

Any mismatch between the HTML calculator formulas and the approved workbook/test matrix is a release defect and must be resolved before declaring the product production-ready.

## 9. Exception Handling

### Build failure
- Capture error.
- Identify file/stage.
- Attempt one bounded deterministic repair if within approved scope.
- Re-run tests.
- Escalate after bounded retries.

### Deployment failure
- Capture provider error.
- Do not repeatedly retry indefinitely.
- Check whether the failure is configuration, permission, quota, or provider-side.
- Escalate if credentials/plan changes are required.

### Verification failure
- Keep deployment marked NOT VERIFIED.
- Compare against previous known-good release.
- Diagnose.
- Roll back only if rollback authority was pre-approved.
- Otherwise request human review.

### Provider unavailable
- Do not claim deployment success.
- Preserve the release artifact and commit.
- Retry only according to the bounded retry policy.

## 10. Audit Record

Every deployment must record:

- task_id
- product
- repository
- source commit
- deployment target
- deployment timestamp
- deployment result
- deployment URL
- verification result
- test results
- errors
- rollback action if any
- approval reference where applicable

No secrets or raw credentials may appear in the audit record.

## 11. Current HISABIQ Deployment Record

Prototype deployment completed manually through the Cloudflare dashboard.

Known from the owner-provided deployment screen:
- Project: `hisabiq`
- Production branch: `main`
- Automatic deployments: enabled
- Public domain: `hisabiq.pages.dev`
- Production deployment shown as successful
- Current HISABIQ spend remains SAR 0

Important limitation:
The deployment is operationally reported by Cloudflare as successful, but the current execution environment could not independently fetch `https://hisabiq.pages.dev`. Therefore this specification does **not** mark the live site as independently verified until the production smoke test is executed from an environment with external access.

## 12. Acceptance Criteria for Astra Deployment Capability

Astra deployment capability is considered implemented only when all are true:

1. Astra can produce a versioned release candidate.
2. Deterministic QA passes.
3. Astra can submit the approved deployment without exposing credentials to the reasoning context.
4. Deployment status is machine-readable.
5. Astra can identify the deployed commit.
6. Astra can execute production smoke tests.
7. Astra refuses to declare LIVE when verification fails.
8. Astra records an audit result.
9. Astra can recover from a bounded deployment failure.
10. Human approval remains mandatory for money, legal commitments, new credentials, and scope expansion.

## 13. Factory-Level Goal

Once this capability is implemented, a new micro-product should follow:

`Idea → Research → Build → QA → Deploy → Verify → Market → Measure → Automate → Delegate → Archive → Next Product`

The owner should not be required to repeat routine deployment clicks for every product. Human intervention should be reserved for authority boundaries rather than ordinary execution.
