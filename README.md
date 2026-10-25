# SecureMe

SecureMe is a Claude Code skill for turning a website or web app into a production-ready, launch-checked build. It combines security, API resilience, data/performance, observability/recovery, technical SEO, UX/accessibility, and content/forms/compliance work.

The workflow has nine roles. A Planner-Orchestrator first inspects the project and writes an atomic requirement matrix; nothing is built until the user approves it. Seven specialist builders then work in owned file areas: Security, API & Resilience, Data & Performance, Observability & Recovery, Technical SEO, UX & Accessibility, and Content/Forms/Compliance. A Quality Gate Agent applies handoffs, runs project and runtime checks, verifies every requirement against the actual code, fixes in-scope gaps, and issues a SHIP, SHIP-WITH-MANUAL-ITEMS, or NOT-READY verdict.

## Install

```bash
git clone https://github.com/almanalaysay93-gif/secureme ~/.claude/skills/secureme
```

## Use

```text
/secureme make this Next.js + Supabase app production ready
```

The canonical launch checklist is `references/launch-checklist.md`. SecureMe artifacts are written to the target project's `.secureme/` directory: `PLAN.md`, `AUDIT.md`, specialist reports and handoffs, and `QC_REPORT.md`. No agent commits, pushes, deploys, or writes real secrets while running the workflow.

## Integrated Cloudflare security audit

SecureMe bundles Cloudflare's MIT-licensed [security-audit skill](https://github.com/cloudflare/security-audit-skill) at a pinned revision. No separate skill installation or runtime download is needed. Provenance and update instructions are in [UPSTREAM.md](vendor/cloudflare-security-audit/UPSTREAM.md).

The lead orchestrator runs the complete six-phase audit before planning an existing app, maps confirmed findings into owned plan requirements, and runs a new independent audit after builders and QC finish changing code. Greenfield projects defer the baseline until source exists; the final audit remains mandatory. Builders implement approved fixes; fresh verifiers check them. Source changes after the final audit require another audit.

```text
Intake -> Cloudflare baseline -> Plan -> User approval
       -> Specialist build waves -> QC handoffs/fixes
       -> Cloudflare final audit -> QC read-only release verdict
```

Full audit reports default to external directories under `~/security-audit-skill/<repo-name>/run-<N>`. `.secureme/AUDIT.md` links these runs, stable finding fingerprints, plan rows, validation blockers, and independent closure evidence. See [the integration contract](references/cloudflare-audit.md).

The host needs tool use, parallel sub-agent support, and Node.js. Audit execution additionally requires Cloudflare's OS-enforced sandbox controls; without them, inspection stays static and exact validation blockers remain visible. Missing/incomplete/stale final audits or unresolved confirmed vulnerabilities produce `NOT-READY`. A completed audit with unresolved validation or partial coverage permits at most `SHIP-WITH-MANUAL-ITEMS`. A clean report is not a guarantee of complete security.

## Validation

Run the bundled upstream validator suites:

```bash
node --test vendor/cloudflare-security-audit/validate-findings.test.cjs vendor/cloudflare-security-audit/validate-coverage-ledger.test.cjs
```

`evals/evals.json` contains behavioral evaluation scenarios for SecureMe's planning, approval, remediation, audit freshness, and release gates. These are agent-evaluation specifications, not an executable end-to-end test harness.
