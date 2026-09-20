# HISABIQ

**Simple business math.**

HISABIQ is a focused toolkit for Saudi e-commerce sellers to understand order-level profit, pricing, CAC ceilings, and break-even economics.

## Current release
Production deployment V1.3 is active on Cloudflare Pages at `hisabiq.pages.dev`. Independent live verification remains pending in the current execution environment.

The public prototype currently provides a free calculator. Live payment is intentionally disabled until business/payment eligibility and production checkout are verified.

## Product path
Free calculator -> decision insight -> paid E-commerce Profit & Pricing Toolkit.

## Design
Minimalist black + warm gold, mobile-first.

## Assumptions
- VAT treatment is an editable assumption and must match the seller's actual tax position.
- Return rate is an expected-value assumption.
- Platform/payment fees should be replaced with the seller's actual terms.
- The toolkit is decision support, not tax, accounting, or legal advice.

## Status
Current HISABIQ spend: SAR 0.

See the docs directory for operating documentation and release gates.


## Astra execution
The deployment/execution target is documented in `docs/ASTRA_EXECUTION_SPEC_V1.md`, including states, permissions, deployment contracts, verification rules, exception handling, and acceptance tests.
