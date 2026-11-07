# PUBBA Freight Quote Confirmation Setup

The website uses Netlify Forms as the authoritative quote receiver. Netlify stores accepted submissions and sends the existing internal notification to PUBBA Logistics. After the browser receives a successful response from Netlify, it redirects the customer to the branded `/quote-confirmation/` page.

No customer confirmation email service, Netlify Function, API key, sending-domain configuration, new dependency, or recurring third-party email cost is required.

## Production audit

The production audit confirmed that:

- The active form is named `quote`.
- Netlify honeypot protection is enabled.
- The existing `submission_created` notification sends to `dispatch@pubbalogistics.com` with subject `QUOTE SUBMISSION`.
- This implementation does not replace or duplicate that internal notification.

## Submission experience

The form validates required fields and email formatting in the browser. While a request is being submitted, the button is disabled and displays a loading label. A second click cannot create another request. If the network or Netlify request fails, the customer's entries remain in the form and a helpful retry message is displayed.

Only a successful Netlify response creates the private confirmation state and redirects to `/quote-confirmation/`. No customer details or shipment information appear in the URL or confirmation-page content.

Direct visits to the confirmation URL without a successful submission are redirected to the quote form. The confirmation page is marked `noindex` and is not included in `sitemap.xml`.

## GA4 setup

Netlify's existing post-processing configuration injects GA4 measurement ID `G-P1CSWZLTC3` into published pages. The source does not add a second GA4 loader. After Netlify accepts the quote, the browser stores only a one-time random token and normalized equipment category in session storage. The confirmation page sends:

- Event: `generate_lead`
- `lead_type`: `freight_quote`
- `form_name`: `freight_quote`
- `equipment_type`: `cargo_van`, `box_truck_26ft`, `tractor_trailer`, or `not_sure`
- `event_id`: the random one-time submission token

No name, email, phone, address, or free-text shipment description is sent to GA4. The event is marked as sent before the tracking call, so refreshing the page does not generate another lead. Analytics behavior continues to follow the site's existing Netlify-injected configuration.

In GA4, open **Admin → Events** or **Key events**, locate `generate_lead`, and mark it as a key event. Use Realtime or DebugView to validate a test submission.

## End-to-end production test

Use a real business email and realistic shipment text so Netlify's spam filtering does not classify the submission as spam.

1. Submit one valid quote and confirm that the button disables with a loading label.
2. Confirm the browser reaches `/quote-confirmation/` only after the form POST succeeds.
3. In Netlify **Forms**, confirm one verified `quote` submission.
4. Confirm the existing notification reaches `dispatch@pubbalogistics.com` once.
5. Verify `generate_lead` in GA4 Realtime or DebugView and confirm its parameters contain no PII.
6. Refresh the confirmation page and verify that a second event is not sent.
7. Open `/quote-confirmation/` in a new private window and confirm it redirects to `/#quote` without generating an event.
8. Double-click Submit and confirm only one request is created.
9. Test missing required fields and an invalid email; browser validation should prevent submission.
10. Simulate a network failure in browser developer tools; the form must retain its values and allow retry.
11. Test the form and confirmation page at mobile viewport sizes.

## Rollback

Use Netlify **Deploys → previous successful production deploy → Publish deploy**. Existing Netlify form submissions are not deleted by a rollback.
