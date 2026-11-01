# VELORA Stripe/OpenAPI Integration

This directory contains the VELORA-specific Stripe Checkout configuration and operational documentation layered on top of the upstream Stripe OpenAPI fork.

## Purpose

`withkoji/openapi-stripe` is a specification repository. It is **not** the VELORA payment runtime. The executable Checkout endpoint remains in `withkoji/velora-booking-commerce`, under `integrations/stripe-server/app.rb`.

The goal of this directory is to keep VELORA configuration explicit, reviewable, and separate from Stripe-generated files.

## Sources of truth

| Concern | Source of truth |
| --- | --- |
| Stripe API schema | `latest/openapi.spec3.{json,yaml}` |
| VELORA Checkout Studio values | `velora/checkout-studio-profile.yaml` |
| Runtime Checkout Session creation | `withkoji/velora-booking-commerce/integrations/stripe-server/app.rb` |
| Remaining runtime setup | `withkoji/velora-booking-commerce/STRIPE_INTEGRATION_TODO.md` |
| Launch-health acceptance | `withkoji/velora-booking-commerce` issue #20 |
| Integration work breakdown | `withkoji/velora-booking-commerce` issue #29 |

## Repository boundary

Do not insert VELORA business values into generated Stripe specification files under `latest/`, `preview/`, or `openapi/`. Those files should remain compatible with upstream Stripe updates.

Do not store secret keys, webhook secrets, bearer tokens, customer payloads, or live payment credentials in this repository.

## Checkout ownership

The canonical VELORA profile describes intended Checkout parameters. Runtime-owned values remain in the application environment:

- `mode` is determined by product type.
- `success_url` is provided by the runtime configuration.
- `cancel_url` is provided by the runtime configuration.
- `line_items[].price` comes from the configured Stripe Price ID.
- Stripe secret keys and webhook secrets remain server-side only.

## Current runtime

The current VELORA membership flow uses Ruby/Sinatra with Stripe gem `~> 19.6`. Under the supplied versioning rule, that runtime uses `ui_mode: hosted`. The runtime is intentionally sandbox-only until live fulfillment, authorization, reconciliation, and operational controls are implemented.

## Change flow

1. Update `velora/checkout-studio-profile.yaml` when Checkout Studio settings change.
2. Validate every configured field against the current GA Stripe OpenAPI specification.
3. Align `velora-booking-commerce` runtime parameters with the canonical profile.
4. Run runtime tests and configuration checks.
5. Complete end-to-end sandbox Checkout and webhook acceptance.
6. Review generated-spec integrity, secrets, release order, and rollback plan.
7. Merge documentation/spec-reference changes before or with the matching runtime change.
8. Update the parent integration tracker and launch-health issue with final evidence.

See [FLOW.md](FLOW.md) for the detailed sequence and [CHANGE_CONTROL.md](CHANGE_CONTROL.md) for review and rollback rules.

## Issue hierarchy

Operational tracking lives in `withkoji/velora-booking-commerce` because Issues are disabled in this OpenAPI repository.

Parent tracker: `#29`.

The child issue set covers documentation, OpenAPI validation, runtime alignment, CI/release guardrails, sandbox acceptance, PR coordination, labels/triage, handoff, regression monitoring, source-of-truth verification, compatibility review, and sandbox security.

## Definition of done

The integration is complete only when:

- the canonical profile and runtime are aligned;
- the configured fields are valid for the supported Stripe API/SDK combination;
- no generated Stripe specification file contains VELORA-specific configuration;
- sandbox Checkout and signed webhook delivery are verified;
- duplicate or unpaid events cannot produce booking or membership fulfillment;
- release and rollback steps are documented;
- remaining operational work is accurately reflected in `STRIPE_INTEGRATION_TODO.md`.
