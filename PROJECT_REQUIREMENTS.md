# BV PAN Digital Store: Project Requirements

**Status:** Initial product and technical baseline  
**Prepared:** 2026-11-26  
**Evidence note:** No reference image was available in the workspace. All business-specific facts not supplied by the owner remain marked as placeholders.

## 1. Product Overview

BV PAN Digital Store is a responsive storefront for browsing and purchasing digital products. The experience must make the catalog, prices, currency, availability, delivery details, support route, and store identity straightforward for the owner to maintain. This document does not establish that any particular product, price, legal term, or business contact exists.

### Goals

- Let customers discover products, understand their details, and complete an order.
- Provide order status, confirmation, and access-controlled digital delivery after payment is verified.
- Give authorized staff a protected interface for catalog, category, order, and basic store-setting management.
- Keep payment, identity, email, database, and storage secrets out of browser code.
- Run locally in a clearly labeled demo mode before owner credentials and business details are available.

### Target Users

- **Customers:** Visitors browsing or buying digital goods and returning customers checking order history or requesting support.
- **Store administrators:** Owner/staff managing products, categories, orders, store details, and operational settings.

### Confirmed Requirements

- Store name: **BV PAN Digital Store**.
- The site sells digital products.
- Required stack considerations: Neon, Clerk, Resend, optional Cloudflare R2, and either Cloudflare Pages or Vercel.
- Email preference: **Resend**.
- Payment preference: **UPI gateway**.
- Do not invent business data or legal terms. Never collect/store raw card data. Do not send marketing email without later consent-based configuration.

### Assumptions and Placeholders

- **[ASSUMPTION]** Next.js App Router with TypeScript and server-side route handlers is suitable for the storefront and API.
- **[ASSUMPTION]** Vercel is the primary host because it is a direct fit for Next.js server rendering and route handlers. Cloudflare Pages is not a second deployment target in this baseline.
- **[ASSUMPTION]** A hosted payment checkout is preferred. Razorpay is a candidate for UPI, pending the owner’s country, merchant account eligibility, provider terms, fees, and credentials. Do not present Razorpay or any gateway as free. A payment must be verified server-side before order fulfillment.
- **[ASSUMPTION]** Clerk provides customer authentication and an admin role/allowlist. Admin status is enforced server-side, not inferred from UI visibility.
- **[PLACEHOLDER]** Products, categories, prices, tax treatment, currency, stock/availability, delivery instructions/files, logo, colors, domain, contact details, support hours, and business address.
- **[PLACEHOLDER]** Terms of service, privacy notice, refund/cancellation rules, digital-delivery terms, tax/invoice language, and any required business disclosures. Owner/legal review is required; this project must not invent them.
- **[PLACEHOLDER]** Owner’s UPI/payment provider account, settlement country/currency, payment methods, and provider-approved fee disclosures.
- **[PLACEHOLDER]** Resend sending domain, verified sender, recipient addresses, and production credentials.
- The reference image mentioned by the requester was not present in the workspace; no visual-brand details are inferred from it.

## 2. Roles and Permissions

| Role | Permissions |
| --- | --- |
| Anonymous visitor | Browse published products/categories, search/filter/sort, view public store/support information, maintain a local cart, and begin checkout. Cannot view private orders or downloads. |
| Customer | All visitor actions; view only their own orders and eligible downloads; submit support requests; manage their own profile through Clerk. |
| Administrator | Customer permissions plus manage products/categories, update order status where permitted, review orders, configure non-secret store settings, and view operational status. Admin access is checked on every protected server action/route using the verified Clerk identity and role/allowlist. |
| System/provider | Create payment sessions, verify signed provider webhooks, persist order/payment state, issue scoped delivery access, and send transactional email. Provider secrets remain server-only. |

No customer-facing route may expose an order or download by guessing an ID. Admin actions require authorization independent of navigation or hidden controls.

## 3. Site Map and Page Requirements

| Route | Requirements |
| --- | --- |
| `/` | Store introduction using owner-provided identity placeholders, featured/published catalog entries only, and clear paths to browse and support. |
| `/products` | Search, category filter, sort, responsive product listing, empty/loading/error states, and no fabricated inventory. |
| `/products/[slug]` | Product description, owner-provided price/currency, delivery/compatibility details, availability, and add-to-cart action. Missing data must display a conspicuous setup placeholder in demo/admin contexts and never imply a real purchasable offer. |
| `/cart` | Editable quantities where relevant, item removal, subtotal and clearly marked tax/shipping treatment; digital-only assumption must be confirmed. |
| `/checkout` | Authenticated/customer identity as required, order review, hosted payment redirect when configured, and a visibly labeled non-charging demo path when not configured. Never request card details directly. |
| `/checkout/return` and `/orders/[id]` | Safe payment-status refresh and order confirmation; pending is distinct from paid. Show next steps only after verified state. |
| `/account/orders` | Signed-in customer’s own order history and status. Signed-out, empty, loading, and error states are required. |
| `/downloads/[token]` or equivalent | Authenticated, order-owned, time-limited access to eligible files; no public permanent object URL. |
| `/support` | Accessible support form with server validation and Resend notification when configured; demo/local fallback states must be explicit. Display contact placeholders until supplied. |
| `/admin` | Protected dashboard and links to management sections. Unauthorized access must be denied server-side. |
| `/admin/products`, `/admin/categories` | Create/edit/archive product and category content, including price/currency and publish state, with validation and confirmation. |
| `/admin/orders` | Search/review orders, payment state, delivery state, and permitted status changes with audit-relevant timestamps. Never allow staff UI to mark a payment paid without provider verification or a documented manual process. |
| `/admin/settings` | Basic store identity, contact, currency, and payment-mode configuration. Secrets are set through deployment environment variables, not this UI. |
| `/privacy`, `/terms`, `/refunds` | Owner-provided content only. Until reviewed and supplied, show “Store owner content required before launch,” not invented policy language. |

## 4. Core Customer Journeys

1. **Browse:** Customer opens the home page, searches or filters the catalog, sorts results, and opens a product detail page. Only published products appear in public listings.
2. **Purchase:** Customer adds a product to cart, reviews the total and currency, signs in if required, and starts checkout. If payment is configured, the server creates a hosted provider session and redirects the customer. If not, a clearly labeled demo flow completes without charging or implying a real payment.
3. **Payment and confirmation:** The customer returns to a status page. The UI shows pending until a verified provider webhook/status check confirms payment. A paid order is persisted once, confirmation email is attempted, and delivery becomes eligible. Duplicate webhooks must be idempotent.
4. **Receive product:** Customer follows authenticated order/download instructions. Download access checks payment state, customer ownership, and expiry/limits before issuing a short-lived URL or streaming the file.
5. **Order history:** Customer signs in and sees only their own orders, item summaries, status, and eligible download links.
6. **Support:** Customer submits a validated support form with a relevant order reference if available. The server sends a notification using Resend when configured and returns a safe confirmation; failures are logged without exposing provider details.

## 5. Functional Requirements and Acceptance Criteria

- The app starts locally from documented commands and builds without production service credentials. Missing integrations are represented as disabled/configuration-needed states, not false success.
- Public catalog supports case-insensitive search, category filtering, and deterministic sorting (e.g. name and price); filters work together and are keyboard accessible.
- Product and category CRUD is available to authorized administrators; invalid data, duplicate slugs, negative prices, and malformed inputs are rejected server-side.
- Product prices use integer minor units and an explicit ISO currency code. Currency and tax behavior are owner-configured, not inferred from UPI.
- Cart state survives page navigation and can be edited. The server recomputes prices from the database at order creation; browser totals are never authoritative.
- Checkout has two explicit modes: a configured hosted gateway and a demo mode that cannot charge. A missing/misconfigured gateway never falls through to a fake “paid” production order.
- Payment notification signatures are verified server-side; order/payment updates are idempotent. Return URLs alone do not prove payment.
- Order confirmation/history is scoped to the signed-in customer. Paid state is distinct from created, pending, failed, cancelled, and refunded/partially refunded (where supported).
- Digital file links are private and only issued for paid, customer-owned order items. R2 is optional; local/demo products can use non-downloadable sample entries.
- Resend templates cover order confirmation, digital delivery/instructions, payment/order-status updates, customer support notification, and optional admin order notification. No marketing sends are implemented.
- Every asynchronous page/form has loading, empty, success, and recoverable error states as applicable; errors do not reveal stack traces, SQL, tokens, or provider responses.
- Support submissions are validated and length-limited on the server, safely rendered in email, and rate-limited or protected against abuse before public launch.
- Automated tests cover input validation, catalog filters, price recomputation/order ownership, admin authorization, payment webhook verification/idempotency, and demo-mode non-charging behavior.

## 6. Admin and Store Management

- Product fields: name, unique slug, summary, full description, category, integer price in minor units, currency, published/archived state, delivery type, private file reference (optional), and timestamps.
- Category fields: name, unique slug, description, display order, published state, and timestamps.
- Order tools: filter/search by ID, customer, date, and state; inspect line-item price snapshots; see provider reference and fulfillment status; retry safe transactional notifications where appropriate.
- Basic store settings: display name, logo/image reference, currency, contact/support details, and public policy content/status. Do not store provider secrets in database settings.
- Admin membership comes from Clerk role metadata or a server-only allowlist environment variable. Role changes and all mutations are re-authorized server-side.
- Destructive archive/delete operations require confirmation; purchased product/order snapshots remain auditable and are not rewritten when catalog values change.

## 7. Email Requirements

Use Resend from server-only code with a verified sender domain and environment variables. Keep templates text-first, accessible, branded only with owner-approved assets, and safe against HTML injection. Delivery failure must not invalidate a verified paid order; record/retry notification status safely.

Required template types:

- Order received/confirmation, with order ID, item summary, status, and support route.
- Digital delivery/download instructions, only after payment and fulfillment eligibility are verified.
- Payment/order-status update for relevant transitions.
- Customer support/contact form notification to the configured owner/support inbox.
- Optional admin new-order notification, with minimal necessary customer data.

Example variables: `{{customer_name}}`, `{{order_id}}`, `{{order_status}}`, `{{items}}`, `{{total}}`, `{{currency}}`, `{{download_url}}`, and `{{support_url}}`. Use escaped values and expiring download URLs. Do not send marketing email without a separately configured consent-based mailing-list feature.

## 8. Data Model and Relationships

- **UserProfile**: Clerk user ID (external primary key), optional display name/email snapshot, role reference/metadata, created/updated timestamps. Clerk remains identity authority.
- **Category**: ID, name, unique slug, description, sort order, published state, timestamps.
- **Product**: ID, category ID (nullable/controlled), name, unique slug, descriptions, price minor units, ISO currency, published state, delivery type, optional private asset key, timestamps.
- **Order**: ID, customer Clerk ID, unique public-safe reference, state, payment state, subtotal/tax/total minor units, currency, provider name/reference, timestamps. Customer has many orders.
- **OrderItem**: ID, order ID, product ID nullable for historical retention, immutable product name/price/currency snapshot, quantity, delivery state. Order has many items; product changes do not rewrite purchased snapshots.
- **PaymentEvent**: ID, order ID, provider event ID (unique), event type, verified state, limited safe metadata, received timestamp. Used for idempotent webhook processing; do not retain raw payment credentials.
- **DownloadGrant**: ID, order item ID, customer ID, expiry, usage limit/count, revoked timestamp. Only references private R2 object keys; no public object URL.
- **SupportRequest**: ID, optional customer/order reference, sender name/email, subject/message, notification state, created timestamp. Apply retention and access rules once owner policy is supplied.
- **StoreSettings**: Public non-secret settings only (name, currency, contact/policy text). Provider credentials remain environment variables.

Relational integrity, indexes for catalog lookup/order ownership, migrations, and clearly labeled non-commercial sample seed data are required. Do not seed invented real-looking products or prices; use conspicuously named examples with `is_sample` or equivalent and disable purchasability by default.

## 9. Security, Privacy, Accessibility, Performance, SEO

### Security and Privacy

- Validate, normalize, and length-limit all untrusted input on the server; parameterize database access and safely escape HTML/email content.
- Enforce Clerk authentication and authorization on server-side actions/routes. Protect customer records by owner ID and admins by role/allowlist.
- Use secure, HTTP-only, same-site session cookies through Clerk; protect state-changing routes against CSRF as required by the framework/provider patterns.
- Keep Neon, Clerk secret, Resend, payment-provider, and R2 credentials in environment variables. Only publish explicitly public Clerk keys.
- Verify signed gateway webhooks, handle replay/idempotency, and avoid trusting browser return URLs for paid status.
- Do not collect/store card numbers or security codes. Use hosted checkout. Never put secrets, private file links, passwords, payment data, or sensitive email contents in logs.
- Minimize personal data in logs and notifications. Define retention/deletion/export behavior with the owner before launch; publish no privacy claims until reviewed.
- Apply rate limits/abuse controls to checkout creation and contact submission. Configure security headers and a restrictive Content Security Policy compatible with Clerk/payment embeds.

### Accessibility

- Target WCAG 2.2 AA: semantic landmarks/headings, keyboard operation, visible focus, sufficient contrast, labels/errors, screen-reader status announcements, reduced-motion support, and accessible dialogs/menus.
- Do not rely on color alone for payment/order state. Test responsive layouts and forms with keyboard and automated accessibility checks.

### Performance and SEO

- Server-render public product/category pages, optimize images, cache public catalog data with invalidation on admin edits, and avoid caching private account/order pages.
- Set explicit image dimensions, use responsive assets, limit client JavaScript, and monitor Core Web Vitals against agreed launch targets.
- Provide unique page titles/descriptions, canonical URLs, sitemap and robots policy, Open Graph metadata, structured product data only when complete/accurate, and noindex for account/admin/checkout pages.
- Do not index placeholder policies, private downloads, or unpublished products.

## 10. Recommended Architecture and Deployment

- **Web/application:** Next.js App Router + TypeScript; server components for public catalog; route handlers/server actions for validated mutations and provider callbacks.
- **Hosting:** Vercel as the single primary deployment target, selected for native Next.js deployment/serverless route handling. Use a Vercel project/domain only after owner approval. Cloudflare Pages is not duplicated as a second host.
- **Database:** Neon PostgreSQL with Drizzle ORM and checked-in SQL/Drizzle migrations. Use a pooled connection for serverless runtime and a direct connection for migrations if required by Neon guidance.
- **Authentication:** Clerk for sign-in/session management. Keep admin role assignment restricted to the owner; configure development and production instances separately.
- **Email:** Resend for transactional email, with verified sender domain, server-only API key, templates, and delivery status handling.
- **Payments:** A hosted gateway supporting UPI (Razorpay is a candidate, not confirmed). Keep provider integration disabled until merchant account, eligibility, fees, webhook secret, currency, and environment variables are supplied. Demo mode is non-charging. Manual QR/UPI fallback requires admin verification and must never auto-deliver based on a customer screenshot/claim.
- **File storage:** Cloudflare R2 only if private digital files are needed. Store object keys privately; issue short-lived signed access only after authorization. Otherwise omit R2 initially.
- **Testing/operations:** Vitest or Node test runner for unit/integration logic, Playwright for critical customer/admin journeys when feasible, lint/typecheck/build in CI, migration-before-deploy discipline, structured redacted logs, error monitoring, backups/restore exercises, and alerts for payment/email/webhook failures.

### Local Setup and Configuration

1. Install the supported Node.js LTS version and npm; install dependencies with `npm install`.
2. Copy `.env.example` to `.env.local`; local app must clearly run in demo mode with payment disabled and sample catalog disabled/non-purchasable by default.
3. Configure Neon and run documented migrations; seed only clearly marked sample records if explicitly requested.
4. Configure Clerk development keys and the server-side admin allowlist before testing authenticated routes.
5. Configure Resend test sender/domain and recipient before sending real email.
6. Configure payment test mode, webhook secret, and optional R2 only after owner supplies accounts. Keep live payment disabled until end-to-end verification.
7. Run tests, lint, typecheck, and production build before deployment.

### Service Plan Notes (checked 2026-11-26)

Plan names, quotas, eligibility, and prices can change; confirm the provider’s current terms and billing dashboard before launch. A free-tier label does not guarantee a permanently free deployment or that all required production features are included.

- **Neon:** Official pricing page currently advertises a free plan; usage can incur charges depending on compute/storage and plan. Verify current project, compute, storage, and autosuspend quotas: https://neon.tech/pricing
- **Clerk:** Official pricing page currently advertises Hobby at $0 with up to 50,000 monthly active users per app; custom-domain/production features and limits may differ by plan: https://clerk.com/pricing
- **Resend:** Official pricing page currently shows a free daily sending limit of 100 emails; higher volume and additional domains/features can cost extra: https://resend.com/pricing
- **Cloudflare R2:** Official pricing docs list a free tier and state no egress bandwidth charges; stored data and operation classes are metered and can become billable: https://developers.cloudflare.com/r2/pricing/
- **Vercel:** Official pricing page lists Hobby at $0/month; usage limits and plan eligibility apply, and a paid plan may be needed for production usage/features: https://vercel.com/pricing
- **UPI gateway:** Razorpay fees and merchant eligibility were not independently verified for this PRD. Obtain current written rates and settlement terms directly from the selected provider. Do not advertise a gateway as free.

### Deployment Plan

1. Owner supplies production identity, content, support, currency/tax decisions, approved policy text, and service accounts.
2. Create isolated development and production Neon/Clerk/Resend/payment/R2 configuration; use least-privilege keys.
3. Set Vercel environment variables separately for preview and production; keep live payment credentials out of previews.
4. Apply migrations, validate seed/sample state, deploy preview, and complete the post-deployment checklist below.
5. Verify provider webhook signatures and idempotency in test mode; validate email domain DNS and private download access.
6. Obtain owner sign-off, enable production credentials, run a low-value authorized payment/refund test if permitted, then monitor alerts and reconcile provider settlement.
7. Document rollback, migration recovery, secret rotation, backup retention, and incident contact procedures.

## 11. Risks, Dependencies, and Unresolved Details

- No owner-provided catalog, branding, currency, contact, legal policies, or operational rules exist yet; launch must be blocked until these are supplied and reviewed.
- UPI implies a market/payment context but does not establish merchant country, currency, tax rules, settlement account, payment provider, or provider availability. Razorpay remains only a candidate.
- No payment gateway can be assumed free. Provider transaction fees, settlement timing, refund/chargeback terms, KYC, and account approval are unresolved.
- Digital delivery creates fraud, link-sharing, and support risks; delivery rules, download limits, expiration, and replacement/refund handling need owner decisions.
- Free-tier quotas, commercial eligibility, data location, support, and service limits can change or require upgrades. Monitor usage and billing alerts.
- Clerk, payment gateway, email, and storage outages must not create false paid states or expose private data. Retry safely and offer support paths.
- Email deliverability depends on verified DNS records, domain reputation, consent/transaction classification, and provider policy.
- Applicable privacy, consumer, tax, invoice, and digital-content obligations depend on the business’s jurisdiction; obtain qualified review.

## 12. Owner Inputs Required Before Launch

- Brand logo, approved colors/type, domain, store description, and social/open-graph assets.
- Real product catalog, categories, descriptions, images, digital assets, delivery steps, availability, and approved prices.
- Merchant country, settlement currency, displayed currency, tax/invoice rules, and whether products are digital-only.
- Payment provider account, supported UPI methods, merchant verification, test/live credentials, webhook setup, and written fee/refund/settlement information.
- Clerk production application, approved admin identities/roles, sign-in methods, and domain configuration.
- Resend verified sending domain, sender identity, support/admin recipients, and approved email wording.
- Neon production project/region, backup/retention expectations, and data-processing preferences.
- R2 decision, bucket/region configuration, upload ownership, file-access policy, and download limits if applicable.
- Business contact details, support availability, escalation route, and expected response process.
- Owner/legal-approved terms, privacy, refund/cancellation, delivery, tax, and required disclosures. These must not be drafted as factual policy without owner/legal review.
- Data retention/deletion/export expectations, analytics/consent decisions, and operational contact for incidents.

## 13. Release Acceptance and Post-Deployment Checklist

- Confirm all visible placeholders are either replaced with owner-approved information or remain explicitly marked as setup-required.
- Confirm catalog pages, search, filters, sorting, cart, responsive layouts, and empty/error/loading states.
- Confirm sign-in, customer order isolation, admin authorization, and denied unauthenticated/unauthorized requests.
- Confirm checkout is visibly demo-only when live integration is absent; no charge can occur in demo mode.
- In gateway test mode, verify successful, failed, cancelled, duplicate, delayed, and invalid-signature payment events; only verified paid events permit delivery.
- Confirm Resend sender-domain verification, all transactional templates, failure handling, and no marketing sends.
- Confirm download links are inaccessible to other customers, expire as intended, and do not expose permanent public R2 URLs.
- Confirm privacy/security headers, HTTPS, redacted logs, secrets isolation, backups, restore procedure, alerts, and billing notifications.
- Run automated tests, lint/typecheck/build, critical browser journeys, keyboard checks, and accessibility scan on desktop and mobile.
- Verify titles, canonical URLs, sitemap/robots rules, social metadata, and noindex behavior for private routes.
- Record production service plan/quota and gateway fee terms with the owner; re-check current provider pricing immediately before launch.