# ReviewGuard — Deployable Prototype

A responsive, static prototype for a QR-based customer feedback SaaS for local businesses.

## Included features

- Owner dashboard with average feedback, response count, unresolved issues, resolved issues, and rating distribution.
- Feedback inbox with filters and resolve/reopen actions.
- Customer feedback journey with 1–5 star rating, optional category, and optional comments.
- Optional Google review link available to every customer regardless of rating.
- Business setup screen and a real QR code generated in the browser.
- Local browser persistence using `localStorage`.
- No build step and no backend required for the demo.

## Important limitations

This is a prototype, not production SaaS:
- Data is stored only in the current browser and is not shared across devices or accounts.
- Authentication, tenant isolation, database persistence, server-side validation, rate limiting, abuse prevention, and subscription billing are not implemented.
- The QR code library is loaded from a CDN, so QR generation requires internet access.
- The QR code becomes useful to customers after deployment to a public HTTPS URL.
- The default Google review URL is a placeholder. Replace it with the business's real review link.

## Run locally

1. Unzip the project.
2. Open `index.html` in a browser. Most features work immediately.
3. For a local web server, run from this folder:

   ```bash
   python3 -m http.server 8080
   ```

4. Open `http://localhost:8080`.

A QR code generated locally points to the local URL and will not work from another person's phone. Deploy first for a public QR code.

## Deploy free on Netlify

1. Visit https://app.netlify.com/drop
2. Drag the unzipped `reviewguard-prototype` folder (or the folder containing `index.html`) into the deploy area.
3. Wait for Netlify to provide a public HTTPS URL.
4. Open that URL, go to **Setup & QR**, set the real Google review URL, and download the QR PNG.
5. Scan the QR with a phone that is not on your computer's local network, then test feedback.

## Deploy on Vercel

The project includes `vercel.json`. You can import the folder into a Git repository and deploy it as a static site on Vercel, or use the Vercel CLI from the project folder:

   ```bash
   npm i -g vercel
   vercel
   ```

Accept the default settings for a static site. No build command is required.

## Google review policy

The customer journey does not gate Google review access based on the private rating. Every customer who submits feedback sees the same optional Google review invitation. Do not change this into a flow that redirects only positive ratings to Google or discourages unhappy customers from posting a public review.

## Suggested production roadmap

1. Validate with 5–10 local businesses.
2. Build an API using Java 21 + Spring Boot.
3. Use PostgreSQL for businesses, users, feedback, categories, QR identifiers, and issue resolution.
4. Add secure authentication and strict business-level authorization.
5. Generate stable public QR URLs such as `https://yourdomain.com/b/{publicSlug}`.
6. Add email alerts and audit logs.
7. Add billing only after customers validate willingness to pay.

## Proposed database tables

- `users(id, name, email, password_hash, created_at)`
- `businesses(id, owner_user_id, name, category, public_slug, google_review_url, created_at, updated_at)`
- `feedback(id, business_id, rating, comment, created_at, status)`
- `feedback_categories(id, business_id, name, active)`
- `feedback_category_selections(feedback_id, category_id)`
- `feedback_actions(id, feedback_id, actor_user_id, action, note, created_at)`
- `qr_codes(id, business_id, public_token, destination_url, active, created_at)`
- `subscriptions(id, business_id, plan, status, current_period_end)`

Use UUIDs or cryptographically random public tokens for public identifiers. Store passwords only as strong hashes using a reputable authentication framework; never store plaintext passwords.

## Proposed API endpoints

- `GET /api/public/businesses/{slug}` — public business profile and form configuration.
- `POST /api/public/feedback` — validate and save a feedback submission.
- `GET /api/business/dashboard` — dashboard summary for authenticated business.
- `GET /api/business/feedback?status=&rating=` — list feedback for the authenticated business.
- `PATCH /api/business/feedback/{id}` — update resolution status and owner notes.
- `GET /api/business/settings` — read current business settings.
- `PUT /api/business/settings` — update business name, category, and Google review URL.
- `POST /api/business/qr-codes` — create a stable QR destination.

All business endpoints must authorize the user against the business ID on the server. Never trust a business ID supplied by the browser alone.
