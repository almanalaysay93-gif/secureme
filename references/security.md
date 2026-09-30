# Web & Supabase Security Master Action Plan
### Complete Hardening Checklist & Vibe-Coding Agent Prompts

This document combines the complete web security checklist, Supabase production hardening rules, and 3 prompt-engineered templates ready to paste directly into your vibe-coding tool (Claude Code, Cursor, Windsurf, Copilot, etc.).

---

## Part 1: Master Web & Supabase Security Checklist

### 1. Environment & Secret Management
- [ ] **Exclude `.env` and sensitive configurations from Git**
  - Verify `.env*` (except `.env.example`) is listed in `.gitignore`.
  - Scan commit history using `gitleaks` or `git-filter-repo` to verify historical commits are clean.
  - Immediately rotate any leaked credentials in production dashboards.
- [ ] **Keep API secrets strictly server-side**
  - Never prefix backend API secrets with `NEXT_PUBLIC_` or expose them in client bundles.
  - Proxy all external API requests through server routes or Supabase Edge Functions.
- [ ] **Sanitize and mask production logs**
  - Strip tokens, passwords, credit card info, and PII from logger payloads.
  - Disable raw payload request/response dumps in production.

### 2. Supabase-Specific Hardening
- [ ] **Verify `service_role` key isolation**
  - Restrict client environments to `anon` / `sb_publishable_` keys only.
  - Add `import 'server-only'` to internal Supabase admin modules in Next.js to fail builds if imported client-side.
- [ ] **Enforce Row Level Security (RLS) on all public tables**
  - Run `SELECT tablename FROM pg_tables WHERE schemaname = 'public' AND NOT rowsecurity;` to identify unshielded tables.
  - Run `ALTER TABLE <table_name> ENABLE ROW LEVEL SECURITY;` on all public tables.
- [ ] **Audit RLS policy logic & write operations**
  - Avoid unconditional `USING (true)` policies on private records.
  - Include `WITH CHECK ((SELECT auth.uid()) = user_id)` on `INSERT` and `UPDATE` policies to prevent user impersonation.
  - Wrap `(SELECT auth.uid())` in subqueries for Postgres optimizer index acceleration.
- [ ] **Secure `SECURITY DEFINER` functions**
  - Ensure all database functions running with definer rights explicitly assert:
    ```sql
    IF auth.uid() IS NULL THEN
      RAISE EXCEPTION 'Not authenticated';
    END IF;
    ```
  - Revoke default public execution rights: `REVOKE EXECUTE ON FUNCTION <func> FROM public;`.
- [ ] **Harden Supabase Auth & Storage configuration**
  - Eliminate wildcard (`*`) domains in **Authentication > URL Configuration > Redirect URLs**.
  - Keep Supabase Storage buckets private by default and control read/write via `storage.objects` RLS policies.
  - Enable email confirmations and leaked-password protection in Auth settings.

### 3. Input Validation, Injection & Session Security
- [ ] **Parameterized SQL queries & ORMs**
  - Use Prisma, Drizzle, or Supabase client query builders; avoid raw template-literal SQL concatenation.
- [ ] **Strict schema validation**
  - Validate all inbound API payloads using Zod or Valibot at boundary layers.
- [ ] **Cross-Site Scripting (XSS) prevention**
  - Sanitize user-submitted HTML with DOMPurify before rendering.
  - Rely on React/modern framework auto-escaping; avoid unescaped `dangerouslySetInnerHTML`.
- [ ] **Token and session management**
  - Keep authentication tokens out of `localStorage` / `sessionStorage`.
  - Transmit tokens over secure, `HttpOnly`, `SameSite=Lax` cookies.
- [ ] **Hash passwords securely**
  - Rely on Supabase Auth's managed bcrypt/Argon2 implementation, or use minimum cost factor 10-12 if running custom auth.

### 4. Network & Operational Defenses
- [ ] **Tighten CORS**
  - Restrict allowed origins to designated staging and production domains.
- [ ] **Configure HTTP security headers & CSP**
  - Enforce `Content-Security-Policy`, `Strict-Transport-Security` (HSTS), `X-Content-Type-Options: nosniff`, and `X-Frame-Options: DENY`.
- [ ] **Rate limit public routes**
  - Enforce IP/user-based rate limits on auth routes, AI generation endpoints, and webhook targets using Redis/Upstash.
- [ ] **Verify webhook signatures**
  - Always verify raw request HMAC signatures prior to executing fulfillment logic.
- [ ] **Disable production debugging**
  - Suppress verbose stack traces and development logs in production outputs.
- [ ] **Dependency auditing & AI code review**
  - Run automated dependency checks (`npm audit`) and run `claude-code-security-review` on AI-generated commits.

---

## Part 2: Vibe-Coding Agent Prompts

Copy and paste these prompts directly into your AI coding assistant (Cursor, Claude Code, Windsurf, GitHub Copilot) to implement the 3 essential protections demonstrated in the guide.

---

### Prompt 1: Secure File Upload Implementation
```text
Act as a Senior Application Security Engineer specializing in Next.js and Supabase.

TASK:
Audit and implement a secure file upload pipeline for this application. Ensure malicious executables (.php, .jsp, .exe, .sh, .html, etc.) cannot be uploaded or executed on the server, and files are stored safely in a private Supabase Storage bucket.

REQUIREMENTS & CONSTRAINTS:
1. Client-Side & Server-Side Dual Validation:
   - Do NOT rely on client-side validation or file extensions alone.
   - On the server/endpoint, inspect the file's binary magic bytes using a package like `file-type` to detect the true MIME type.
   - Explicitly enforce a strict MIME whitelist (e.g., image/jpeg, image/png, application/pdf). Reject anything else.
   - Explicitly blacklist executable extensions including .php, .jsp, .asp, .sh, .exe, .phtml, .svg (unless sanitized), and .html.
2. File Quotas & Renaming:
   - Enforce a strict file size limit (e.g., max 5MB).
   - Sanitize file names: generate a non-guessable UUID for the stored file name (e.g., `${crypto.randomUUID()}.${detectedExtension}`) to prevent path traversal and overwrite vulnerabilities.
3. Supabase Storage Integration:
   - Store files in a Supabase Storage bucket that is marked PRIVATE.
   - Store user uploads under scoped paths: `${userId}/${fileId}`.
   - Provide a SQL migration setting up the RLS policy on `storage.objects` allowing only authenticated users to upload and read their own files.
   - Serve private files using short-lived signed URLs (`supabase.storage.from(...).createSignedUrl(...)`), never permanent public URLs.

OUTPUT REQUIRED:
- The server route / edge function handling the upload.
- The validation helper functions (magic byte inspection, size limits).
- The SQL migration for the private bucket and RLS policies on storage.objects.
- Verification tests to confirm .php/.jsp files and oversized files are rejected.
```

---

### Prompt 2: Stripe Payment Webhook Signature Verification
```text
Act as a Senior Payment Infrastructure and Security Engineer.

TASK:
Implement an end-to-end, spoof-proof Stripe webhook handler using Next.js route handlers and Supabase. Prevent unauthorized users from fabricating "payment_intent.succeeded" or "checkout.session.completed" events to unlock paid features for free.

REQUIREMENTS & CONSTRAINTS:
1. Raw Body Signature Verification:
   - Do NOT parse the request body with `await req.json()` or any JSON body parser before verifying the signature.
   - Extract the raw request payload buffer using `await req.text()` (or raw buffer) and the `stripe-signature` header.
   - Validate the signature with `stripe.webhooks.constructEvent(rawBody, sig, process.env.STRIPE_WEBHOOK_SECRET!)`.
   - Wrap signature verification in a try/catch block and immediately return HTTP 400 with a generic error if validation fails.
2. Idempotency & Replay Attack Defense:
   - Webhook deliveries can be retried or duplicated. Create a table in Supabase called `processed_webhook_events (event_id text primary key, processed_at timestamptz default now())`.
   - Before processing the event, check if `event.id` already exists. If yes, return HTTP 200 immediately to prevent double processing.
   - Insert `event.id` within a transactional workflow or write record before granting access.
3. Secure Status Provisioning:
   - Handle `checkout.session.completed` and `customer.subscription.updated/deleted`.
   - Update user subscription/payment tier using an isolated server-side Supabase client (`service_role` key stored safely in private env vars, NEVER sent to frontend).
   - Link Stripe `customer_id` and `subscription_id` to the matching user record using metadata or client_reference_id.

OUTPUT REQUIRED:
- The complete Next.js App Router route handler (`app/api/webhooks/stripe/route.ts`).
- Supabase SQL migration for the idempotency tracking table.
- Detailed step-by-step instructions for testing locally using Stripe CLI (`stripe listen --forward-to localhost:3000/api/webhooks/stripe`).
```

---

### Prompt 3: End-to-End XSS Prevention & Session Protection
```text
Act as a Web Application Security Specialist.

TASK:
Audit and remediate Cross-Site Scripting (XSS) vulnerabilities across the codebase to prevent malicious script injection in user-facing inputs (e.g., comments, profiles, usernames, reviews) and protect user session authentication data from being stolen.

REQUIREMENTS & CONSTRAINTS:
1. Input Schema Parsing & Sanitization:
   - Use Zod schemas on all API boundaries to validate, trim, and enforce character constraints on text fields.
   - Reject or escape dangerous payload strings on submission.
2. Output Rendering Safeguards:
   - Audit the frontend for any occurrences of `dangerouslySetInnerHTML`, `v-html`, or manual DOM injection (`innerHTML`).
   - If rich text or HTML rendering is required, wrap it with `DOMPurify` (or `isomorphic-dompurify`) configured with a strict whitelist of safe tags (e.g., `<b>`, `<i>`, `<p>`, `<a>`) and attributes (`href`, `target`). Disallow `<script>`, `<iframe>`, `onload=`, `onerror=`, and `javascript:` URIs.
   - For all standard text inputs (usernames, comments), render exclusively through framework text nodes (e.g., `{comment.text}`) which auto-escape by default.
3. Content Security Policy (CSP) & Security Headers:
   - Add a robust `Content-Security-Policy` header in `next.config.js` or middleware.
   - Restrict `default-src 'self'`, disallow `unsafe-inline` scripts where feasible, and limit `connect-src` strictly to self and Supabase project endpoints.
   - Add `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, and `Referrer-Policy: strict-origin-when-cross-origin`.
4. Authentication Cookie Hardening:
   - Confirm authentication tokens are stored in `HttpOnly`, `Secure`, `SameSite=Lax` cookies so that even in the unlikely event of an XSS flaw, session tokens cannot be read via `document.cookie`.

OUTPUT REQUIRED:
- The Next.js middleware / headers configuration with strict CSP and security headers.
- An example input validation schema and sanitization helper for user comments/profiles.
- A secure rich-text rendering component with DOMPurify.
- Verification checklist to confirm injection strings like `<img src=x onerror=alert(1)>` are neutralized.
```
