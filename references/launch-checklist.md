# Website Pre-Launch, Architecture & UX Checklist (v4)

A production-grade pre-launch checklist consolidating frontend UX, technical SEO, backend resilience, API cost control, database optimization, and disaster recovery.

---

## 1. Security, Secrets & Access Control
- [ ] **Remove `noindex` Tags:** Ensure production pages do not have accidental `noindex` meta tags or HTTP headers blocking search engine indexing.
- [ ] **Move Secrets Off Frontend:** Audit client bundles to ensure no private API keys, database credentials, or secret environment variables are exposed.
- [ ] **Enforce HTTPS:** Ensure automatic HTTP-to-HTTPS redirects and valid SSL/TLS certificates across all domains and subdomains.
- [ ] **Add Rate Limiting:** Protect public endpoints, auth routes, and form submissions from brute force and automated scraping.
- [ ] **Upload Size Limits:** Restrict maximum file upload sizes on both client and server to prevent resource exhaustion and denial of service.
- [ ] **Spam & Bot Protection:** Implement honeypot fields, Cloudflare Turnstile, or CAPTCHA on all user-facing submission forms.
- [ ] **Confirmation Modals:** Require explicit confirmation before executing destructive or irreversible actions.

---

## 2. API Safety, Cost Control & Backend Resilience
- [ ] **Set Spending Caps & Hard Limits:** Configure billing alert thresholds and hard usage caps on third-party APIs (OpenAI, Anthropic, payment gateways).
- [ ] **Set API Rate & Concurrency Limits:** Enforce per-user and per-IP throttling on expensive computational routes.
- [ ] **Handle API Timeouts Gracefully:** Define strict request timeout thresholds so hanging external calls do not freeze client requests.
- [ ] **Handle Failed Requests & Retries:** Implement exponential backoff retry strategies for transient network and API failures.
- [ ] **Cache Repeat Requests:** Utilize in-memory caches (e.g., Redis, HTTP cache-control headers, stale-while-revalidate) for redundant external and heavy queries.
- [ ] **Prevent Duplicate Submissions:** Disable submit buttons immediately upon click and implement client-side request debouncing.
- [ ] **Prevent Duplicate Payments (Idempotency):** Send unique idempotency keys with checkout and payment mutation requests to eliminate double charges.

---

## 3. Database & Performance Optimization
- [ ] **Optimize Database Queries:** Audit slow queries, avoid N+1 query patterns, and selectively project required columns.
- [ ] **Add Database Indexes:** Index frequently queried foreign keys, filter columns, and sorting fields to maintain low query latency under load.
- [ ] **Paginate Large Result Sets:** Implement cursor-based or offset pagination on lists and tables to avoid memory spikes.
- [ ] **Compress Files & Assets:** Enable server-level Gzip/Brotli compression and compress static media assets.
- [ ] **Test Simultaneous Users (Load & Concurrency Testing):** Stress-test endpoints to identify concurrency bottlenecks before public traffic arrives.

---

## 4. Monitoring, Error Handling & Recovery
- [ ] **Centralized Error Logging:** Integrate error tracking (e.g., Sentry, LogRocket, Datadog) to capture unhandled runtime exceptions in real time.
- [ ] **Set Up Uptime Monitoring:** Configure synthetic ping monitoring (e.g., BetterStack, UptimeRobot) with multi-channel alerts (email, SMS, Slack).
- [ ] **Comprehensive Error Handling:** Implement UI error boundaries and structured fallback views rather than blank screens or raw stack traces.
- [ ] **Design Meaningful Empty States:** Provide helpful illustrations and guidance when collections, searches, or feeds return zero items.
- [ ] **Test Backup & Restore Procedures:** Verify automated database snapshots and perform a trial restore drill to confirm recovery viability.

---

## 5. Technical SEO & Performance
- [ ] **Generate `sitemap.xml`:** Validate XML sitemaps containing all canonical indexable URLs.
- [ ] **Configure `robots.txt`:** Provide explicit crawl directives allowing search bots while disallowing private/admin paths.
- [ ] **Canonical Tags:** Add self-referencing canonical tags to prevent duplicate content penalties.
- [ ] **Meta Titles & Descriptions:** Craft unique titles (50–60 chars) and meta descriptions (150–160 chars) tailored for target search intent.
- [ ] **Favicon Package:** Include complete multi-device icons (`favicon.ico`, PNGs, `apple-touch-icon.png`, `site.webmanifest`).
- [ ] **Open Graph / Social Preview Cards:** Configure `og:image`, `og:title`, `og:description`, and Twitter Cards for social sharing.
- [ ] **Schema Markup (Structured Data):** Embed JSON-LD structured data (`Organization`, `Article`, `FAQPage`, `BreadcrumbList`).
- [ ] **Single `<h1>` & Header Hierarchy:** Enforce one `<h1>` per page with a logical semantic tree (`<h2>`, `<h3>`).
- [ ] **Image Alt Text:** Provide descriptive, accessible `alt` attributes on all images.
- [ ] **Image Compression & WebP/AVIF:** Serve next-gen responsive image formats with explicit width and height attributes.
- [ ] **Core Web Vitals:** Audit and pass LCP (Largest Contentful Paint), INP (Interaction to Next Paint), and CLS (Cumulative Layout Shift).
- [ ] **Clean URL Slugs:** Ensure clean, lowercase, hyphenated URL paths.
- [ ] **Fix Broken Links:** Eliminate 404s across internal links, external links, and media assets.
- [ ] **Google Search Console & Web Analytics:** Verify domain ownership in Search Console and deploy privacy-compliant analytics.
- [ ] **Backlink Strategy:** Plan initial directory submissions, PR, and domain authority outreach.

---

## 6. Core UX Laws & Behavioral Design
- [ ] **Fitts's Law (Target Sizing & Reachability):** Ensure tap targets meet minimum 44×44px standards and place primary actions in easy-to-reach zones.
- [ ] **Hick's Law (Minimizing Cognitive Load):** Limit menu choices, field counts, and decision branches to reduce friction and bounce rate.
- [ ] **Jakob's Law (Familiar Mental Models):** Follow proven web conventions for navigation, cart workflows, search bars, and iconography.
- [ ] **Law of Proximity (Visual Grouping):** Place related inputs, labels, and copy in distinct visual clusters using deliberate whitespace.
- [ ] **Von Restorff Effect (Visual Isolation):** Make primary CTAs and critical status messages stand out distinctly through color, contrast, and elevation.

---

## 7. UI, Navigation & Accessibility
- [ ] **Dark Mode Toggle:** Implement a smooth light/dark toggle respecting system preferences (`prefers-color-scheme`).
- [ ] **Responsive Mobile Menu:** Provide an accessible off-canvas drawer or navigation menu for small viewports.
- [ ] **Sticky Header:** Keep navigation and primary actions visible during page scroll.
- [ ] **Interactive Hover & Focus States:** Add explicit visual states (`:hover`, `:focus-visible`, `:active`) to all clickable components.
- [ ] **Scroll Progress Bar:** Display reading progress on long-form articles.
- [ ] **Scroll-to-Top Button:** Add an unobtrusive back-to-top shortcut on extended pages.
- [ ] **Loading & Skeleton States:** Replace abrupt layout shifts with skeleton placeholders and clear spinners during async loads.
- [ ] **Skip-to-Content Link:** Provide a top-of-page keyboard bypass link for screen reader and keyboard accessibility.
- [ ] **Color Contrast Verification:** Ensure text and UI elements meet WCAG AA/AAA contrast guidelines.
- [ ] **One Clear Primary CTA:** Position a distinct, high-converting call-to-action per landing view.
- [ ] **Mobile Responsiveness Check:** Verify responsive behavior across phone, tablet, and desktop viewports.

---

## 8. Content, Forms & Compliance
- [ ] **Full-Site Search:** Provide responsive search with query debouncing and clear results.
- [ ] **Expandable FAQ Accordion:** Build accessible accordions with ARIA attributes for common questions.
- [ ] **Newsletter Signup:** Include validated email inputs with dedicated loading, success, and error feedback states.
- [ ] **Form Validation:** Pair instant client-side input feedback with strict server-side schema verification.
- [ ] **Password Visibility Toggle:** Include an interactive show/hide toggle on password fields.
- [ ] **Floating Contact Button:** Offer quick access to support channels or inquiry forms.
- [ ] **Custom 404 Page:** Create a branded 404 page directing users back to useful pages or search.
- [ ] **Legal Pages:** Publish comprehensive Privacy Policy and Terms & Conditions pages.
- [ ] **Cookie Consent Banner:** Present a compliant cookie notice with granular preference controls.
- [ ] **Print Stylesheet (`@media print`):** Strip out headers, nav menus, and dark backgrounds for clean physical printing.
- [ ] **UTM Parameter Tracking:** Tag outbound marketing campaigns with consistent UTM parameters.
- [ ] **Clipboard Copy Buttons:** Add one-click copy buttons with state changes on code blocks and links.
- [ ] **"Last Updated" Timestamps:** Display revision dates on documentation and blog articles.
