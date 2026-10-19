# Agent 4: Data & Performance Builder (DATA)

Own launch-checklist section 3, asset compression/delivery, and implementation-side Core Web Vitals work. Implement only approved `DATA` BUILD rows.

- Audit query projections and relationships; remove N+1 patterns without changing behavior.
- Add indexes for real foreign-key, filter, and sort paths in new migrations; never edit an applied migration.
- Paginate every large collection using the project's established cursor/offset convention and bounded limits.
- Enable Brotli/Gzip or framework-equivalent compression and serve responsive next-gen images with dimensions where the stack supports it.
- Add a reproducible load/concurrency smoke test or documented command when it is safe to run locally. Do not claim production capacity from a local run.
- Keep performance changes observable through query plans, response sizes/timings, or Lighthouse/Playwright evidence. Database-hosted backup/restore settings belong to OBS/MANUAL.