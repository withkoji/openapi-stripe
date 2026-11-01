# VELORA Stripe/OpenAPI End-to-End Flow

## 1. Configuration source

Checkout Studio configuration is represented in `velora/checkout-studio-profile.yaml`.

The profile contains the VELORA-specific fixed Checkout values while leaving runtime-owned values such as URLs, Price IDs, and product mode under application control.

## 2. OpenAPI compatibility check

Use the GA Stripe specification under `latest/` as the compatibility reference.

For each Checkout field in the VELORA profile:

1. Confirm the create Checkout Session operation supports the field.
2. Confirm its type and accepted values.
3. Flag preview-only or unsupported fields.
4. Keep SDK-version-sensitive behavior documented separately from API schema compatibility.
5. Do not edit generated Stripe spec files to force compatibility.

## 3. Runtime handoff

The runtime implementation is owned by:

`withkoji/velora-booking-commerce/integrations/stripe-server/app.rb`

The runtime must:

- use the fixed profile values;
- preserve product-specific `mode`;
- read success and cancel URLs from server-owned configuration;
- read the Stripe Price ID from server-owned configuration;
- keep secret keys and webhook signing secrets outside source control;
- preserve idempotency and webhook verification safeguards.

## 4. Request flow

```text
VELORA client
    |
    | POST /api/create-checkout-session
    | Idempotency-Key: <stable attempt key>
    v
velora-booking-commerce
    |
    | validates local configuration
    | validates Stripe resources needed by runtime
    | constructs Checkout Session parameters
    v
Stripe Checkout Session API
    |
    | returns hosted Checkout URL
    v
VELORA client redirects to Stripe-hosted Checkout
    |
    | customer completes or cancels
    +----------------------------+
    |                            |
    v                            v
success_url                  cancel_url
    |
    | Stripe separately sends signed webhook events
    v
/api/webhook
    |
    | verify signature
    | reject live-mode events in sandbox service
    | reject invalid/unapproved account context
    | deduplicate stored events
    | record observation only
    v
No booking/membership fulfillment until authoritative fulfillment logic exists
```

## 5. Sandbox acceptance flow

A release candidate must verify all of the following:

1. Checkout Session creation succeeds with a sandbox key.
2. The recurring membership Price is correct.
3. Hosted Checkout renders the intended branding and fields.
4. Terms consent is required.
5. Configured optional collection behavior matches the profile.
6. Success and cancel redirects reach implemented pages.
7. `checkout.session.completed` reaches the signed webhook destination.
8. Duplicate webhook delivery is handled idempotently.
9. Unpaid or failed events never trigger booking or membership fulfillment.
10. Live keys and live events remain rejected by the sandbox service.

## 6. CI and drift control

Before merge:

- compare the VELORA profile to runtime Checkout parameters;
- confirm generated Stripe spec files are unchanged by VELORA business configuration;
- run the Ruby test suite in `integrations/stripe-server`;
- fail review when a configured field is unsupported or silently omitted;
- record remaining operational blockers in `STRIPE_INTEGRATION_TODO.md`.

## 7. Release order

Recommended order:

1. Merge OpenAPI-reference documentation/profile changes.
2. Merge matching runtime changes.
3. Configure deployment environment variables.
4. Run configuration validation.
5. Run end-to-end sandbox acceptance.
6. Verify launch-health signals and blockers.
7. Close completed child issues only after evidence is captured.

## 8. Rollback

Rollback when any of the following occurs:

- Checkout Session creation fails after a configuration change;
- a configured field is unsupported by the current API/SDK combination;
- redirects point to an incorrect destination;
- webhook verification or deduplication regresses;
- VELORA configuration appears inside generated Stripe spec files;
- live-mode behavior becomes reachable from the sandbox service.

Rollback procedure:

1. Stop release promotion.
2. Revert the runtime/profile change pair to the last verified state.
3. Keep generated Stripe specs at their upstream-derived revision.
4. Re-run the sandbox test suite and webhook checks.
5. Record the failure and remediation in the parent integration tracker.
6. Update `STRIPE_INTEGRATION_TODO.md` if operator action is required.

## 9. Tracking

Operational tracking is maintained in `withkoji/velora-booking-commerce`:

- Parent integration tracker: #29
- Launch-health tracker: #20
- Related launch-readiness work: #12

Issues are intentionally tracked there because GitHub Issues are disabled in `withkoji/openapi-stripe`.
