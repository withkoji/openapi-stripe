# Stripe Integration TODO

This repository is a fork of Stripe's generated OpenAPI specification. It is not the VELORA application server. The actual Checkout Session endpoint lives in `withkoji/velora-booking-commerce` at `integrations/stripe-server/app.rb`.

## Integration status

VELORA-specific Checkout Studio settings are recorded in:

- [velora/checkout-studio-profile.yaml](velora/checkout-studio-profile.yaml)

The generated Stripe specification files under `latest/`, `preview/`, and `openapi/` are intentionally unchanged so upstream Stripe updates can continue without VELORA-specific drift.

## Values to Replace / Verify

There are no secret values in this repository.

The runtime application must provide and verify:

| Field | Runtime source | Required action |
| --- | --- | --- |
| `mode` | VELORA product logic | Keep `subscription` for the recurring Wellness Membership. Use `payment` only for one-time products/services. |
| `success_url` | `STRIPE_SUCCESS_URL` | Use the real post-payment success page and preserve `{CHECKOUT_SESSION_ID}`. |
| `cancel_url` | `STRIPE_CANCEL_URL` | Use the real cancel/return page. |
| `line_items[].price` | `STRIPE_MEMBERSHIP_PRICE_ID` | Use the actual recurring Stripe Price ID configured for the runtime account. |
| Stripe secret key | runtime secret store | Never commit it here. |
| webhook signing secret | runtime secret store | Never commit it here. |

## Configured Checkout parameters

The VELORA profile records the Checkout Studio values supplied for the hosted mobile Checkout flow:

| Parameter | Value |
| --- | --- |
| `ui_mode` | `hosted` for Ruby Stripe gem `~> 19.6` |
| `billing_address_collection` | `auto` |
| `phone_number_collection.enabled` | `false` |
| `automatic_tax.enabled` | `false` |
| `allow_promotion_codes` | `false` |
| `payment_method_collection` | `always` for subscription mode |
| `submit_type` | `book` |
| `consent_collection.terms_of_service` | `required` |
| `consent_collection.promotions` | `auto` |
| `name_collection.individual.enabled` | `true` |
| `locale` | `en` |
| `saved_payment_method_options.payment_method_save` | `enabled` |
| `integration_identifier` | `hosted_mobile_app_0001` |
| `origin_context` | `mobile_app` |
| `branding_settings.background_color` | `#0c0702` |
| `branding_settings.button_color` | `#5b4d31` |
| `branding_settings.font_family` | `default` |
| `branding_settings.border_style` | `rounded` |
| `branding_settings.icon.file` | `file_1UWqQv2f2UX4QyfJFw9yJG4N` |

## Architecture rule

Do not add an Express, Sinatra, Rails, or other payment server to this repository. This repository contains Stripe API specifications only. Runtime payment logic belongs in `withkoji/velora-booking-commerce`.

Do not edit generated files under `latest/`, `preview/`, or `openapi/` to encode VELORA business defaults. Those files represent Stripe's API contract and are periodically replaced by upstream specification updates.

## Next validation steps

1. Keep `velora/checkout-studio-profile.yaml` synchronized with the Checkout Studio configuration used by VELORA.
2. Validate `withkoji/velora-booking-commerce/integrations/stripe-server/app.rb` against this profile after Stripe SDK upgrades.
3. When the Stripe Ruby SDK is upgraded to 21.0.0 or newer, change the runtime/profile `ui_mode` from `hosted` to `hosted_page` if required by the supplied versioning rule.
4. Run a sandbox Checkout end-to-end from the runtime application and verify the resulting Session reflects the profile.
5. Keep all credentials and webhook secrets only in the runtime hosting platform's secret store.

This file is the single source of truth for how the VELORA configuration is represented inside `withkoji/openapi-stripe`.
