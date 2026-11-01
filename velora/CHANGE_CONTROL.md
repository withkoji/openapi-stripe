# VELORA Stripe/OpenAPI Change Control

Use this checklist for every VELORA Stripe/OpenAPI change.

## Before implementation

- [ ] Confirm whether the change belongs in the OpenAPI reference layer or the runtime application.
- [ ] Confirm the current canonical values in `velora/checkout-studio-profile.yaml`.
- [ ] Identify affected runtime fields in `withkoji/velora-booking-commerce`.
- [ ] Confirm the work is tracked under parent issue #29 or a linked child issue.
- [ ] Confirm no secret value needs to be committed.

## OpenAPI/specification safety

- [ ] Do not place VELORA business values in `latest/`, `preview/`, or `openapi/`.
- [ ] Use `latest/` GA specifications for compatibility checks unless a preview feature is intentionally being evaluated.
- [ ] Treat generated Stripe specifications as upstream-derived content.
- [ ] Do not change generated schemas to make an unsupported VELORA parameter appear valid.

## Runtime safety

- [ ] Keep Checkout Session creation in `withkoji/velora-booking-commerce`.
- [ ] Keep Stripe secret keys and webhook signing secrets server-side.
- [ ] Preserve stable idempotency behavior.
- [ ] Preserve signed webhook verification.
- [ ] Preserve duplicate-event protection.
- [ ] Preserve sandbox-only restrictions until live fulfillment is intentionally implemented.
- [ ] Do not mark a booking or membership fulfilled from an unverified or unpaid event.

## Configuration review

- [ ] Fixed Checkout Studio values match the canonical profile.
- [ ] `mode` matches the actual product type.
- [ ] `success_url` is implemented and keeps the Checkout Session template when required.
- [ ] `cancel_url` is implemented.
- [ ] `line_items[].price` references the intended Stripe Price from server configuration.
- [ ] SDK-version-sensitive values such as `ui_mode` match the installed Stripe SDK.
- [ ] Unsupported or preview-only fields are explicitly blocked or documented.

## Test evidence

- [ ] Runtime unit/regression tests pass.
- [ ] Sandbox Checkout Session creation succeeds.
- [ ] Hosted Checkout displays the intended fields and branding.
- [ ] Terms consent behavior is verified.
- [ ] Success and cancel redirects are verified.
- [ ] Signed webhook delivery is verified.
- [ ] Duplicate delivery is verified.
- [ ] Failed/unpaid events are verified not to fulfill anything.
- [ ] Live-mode credentials/events remain rejected by the sandbox implementation.

## Pull request review

- [ ] PR links parent tracker #29 and relevant child issues.
- [ ] PR explains why generated Stripe specs are unchanged.
- [ ] PR lists all VELORA-owned files changed.
- [ ] PR identifies any corresponding runtime change.
- [ ] PR includes a release order.
- [ ] PR includes a rollback trigger and rollback steps.
- [ ] No secrets, personal customer data, or payment credentials appear in the diff.

## Release

- [ ] Merge the specification/reference change and matching runtime change in the documented order.
- [ ] Configure deployment variables outside source control.
- [ ] Run end-to-end sandbox acceptance after deployment.
- [ ] Update `STRIPE_INTEGRATION_TODO.md` to remove completed items and retain unresolved blockers.
- [ ] Update launch-health tracking in `withkoji/velora-booking-commerce` issue #20 when measurable signals change.

## Rollback triggers

Rollback immediately if:

- Checkout Session creation regresses;
- a field is rejected or silently ignored in a way that changes business behavior;
- redirects are incorrect;
- webhook verification/deduplication fails;
- generated Stripe specs receive VELORA-specific business configuration;
- sandbox/live boundaries weaken;
- fulfillment can occur without authoritative paid-state verification.
