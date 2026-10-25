# Cloudflare audit integration

SecureMe's lead orchestrator owns this integration. Cloudflare is an independent,
read-only auditor; SecureMe's seven builders own remediation. Read the bundled
`vendor/cloudflare-security-audit/SKILL.md` and its referenced phase documents
before auditing. Use full audit mode, all six phases, and the upstream evidence,
coverage, independent-verification, sandbox, and artifact rules unchanged.
Do not replace this workflow with the Security Builder's checklist.

## Setup and scheduling

- Resolve the skill directory and target application to absolute paths. Audit the
  application, not the SecureMe installation or this vendored dependency.
- Check tool-use/sub-agent support and Node.js before starting. Queue independent
  assignments within the host's concurrency limit; the nine SecureMe roles are
  logical roles, not a requirement for nine simultaneous agents.
- The lead acts as Cloudflare's parent and owns shared audit state. It launches
  fresh hunters/verifiers using upstream prompts; never give a builder independent
  verification of its own work. Do not put the entire audit in a worker that cannot
  itself delegate. Audit stages and builder/QC mutations run sequentially.
- Default to the upstream `standard` profile. Preserve an explicit user scope,
  profile, or strict invocation budget, counting both audit stages and SecureMe
  agents against any overall budget. Apply upstream budget reservations before
  each stage; report an unfunded stage as blocked, never silently downgrade it.
- Use a new external output directory for each audit, defaulting to
  `~/security-audit-skill/<repo-name>/run-<N>`. An in-repository directory requires
  an explicit user-selected path and verification that the entire directory is
  ignored by version control. SecureMe's `.secureme/` default is not that consent.
- Record baseline/final paths and state in `.secureme/AUDIT.md`. Do not modify
  upstream `findings.json` or its schema to store SecureMe statuses.

## Execution boundary

Apply Cloudflare's universal execution safety to every audit reproduction or
validation, including a test requested through a builder or QC. Only run target
code in the specified OS-enforced sandbox: no external network, allowlisted empty
environment, read-only target/toolchain, per-agent writable scratch, and resource
limits. Use dummy data. Never probe production or provider dashboards. Missing
controls mean static inspection only, with exact `needs_validation` blockers;
never run a test unsandboxed to obtain an audit verdict. Normal SecureMe build
checks do not count as independently sandboxed audit evidence.

If Node.js, delegation, bundled files, or other mandatory capabilities are absent,
record a blocked stage and its exact reason in `AUDIT.md`; do not fabricate
upstream reports or claim the full workflow ran. If a run started, preserve the
upstream terminal-state contract: validated Phase 6 outputs or explicit incomplete
run metadata and a report disclosing the gap. Planning may continue with the gap
visible. Building still requires plan approval; final release rules below apply.

## Baseline: after intake, before planning

1. For an existing app, freeze source mutations and run the complete Cloudflare
   workflow. Capture the commit, dirty state, scope, and a content digest manifest
   for all reviewed source/configuration files, including relevant untracked
   files. Exclude generated run artifacts. This distinguishes two dirty worktrees
   on the same commit; never retain secrets or raw credential values in evidence.
2. Validate `coverage-ledger.json` after every update and `findings.json` after
   structured output and every material replacement, using the bundled validators.
3. Supply the planner with `run-metadata.json`, `findings.json`, coverage ledger,
   reports, exact blockers, and `AUDIT.md`. Findings are input to the plan, never
   permission to change source before approval.
4. For a greenfield target with no application source, mark baseline `DEFERRED:
   no application source` and plan normally. The final audit is still required
   after the application exists. A missing capability is `BLOCKED`, not greenfield
   deferral. If source changes before approval, refresh affected audit evidence
   and the draft plan before obtaining approval.

## Finding-to-plan contract

- Preserve each upstream fingerprint and verdict. Assign stable SecureMe IDs
  `CFA-001`, `CFA-002`, etc. in `AUDIT.md`; retain those mappings across runs.
- `confirmed`: add an atomic remediation `BUILD` row with one owner, affected
  files, exact boundary invariant, and regression/done-when evidence. Route to
  the actual file owner, not automatically SEC. Deduplicate existing requirements
  with `DUP -> <canonical ID>` and retain the fingerprint link.
- `needs_validation`: create a `VERIFY` row for obtainable local evidence or a
  `MANUAL` row for an external fact. Record the exact missing fact and safe
  validation plan, with no severity. Do not turn it into a confirmed defect or an
  automatic implementation task.
- `rejected`: retain for traceability; no remediation row unless separately
  justified by a launch-checklist requirement.
- A confirmed vulnerability cannot disappear into `N-A`, `FLAG`, or an excluded
  scope to obtain a release verdict. Document exclusions and coverage limitations.
- Builders receive mapped findings and may fix only approved `BUILD` rows. New
  out-of-plan findings require a plan amendment and approval before remediation.

## Final audit: after all source mutations, before release verdict

1. QC first applies handoffs, checks requirements, and completes any in-plan
   fixes. Then stop all source writers. Run a new full audit against the resulting
   source, supplying prior ledgers/findings as upstream requires. Default scope
   remains the whole app; a user-requested scoped run must remain visibly partial.
2. Independently revalidate baseline fingerprints and changed boundaries. A
   builder's `DONE`, a passing build, or disappearance from a new report does not
   establish closure. In `AUDIT.md`, record each finding's current disposition,
   evidence path, and matching source digest. Keep upstream verdicts unchanged;
   SecureMe closure is a separate mapping supported by fresh verifier evidence.
3. If the final audit finds an in-plan gap, QC may fix it within its two-pass
   limit. Any source/configuration mutation invalidates the final audit: perform
   another final run before the release verdict. Out-of-plan fixes require an
   approved amendment. At the limit or budget exhaustion, stop and report
   `NOT-READY` with remaining work rather than looping indefinitely.
4. QC's final action is read-only consumption of validated, current-source audit
   evidence and calculation of the verdict. A concurrent change or unmatched
   digest makes the final stage stale and blocks release until re-audited.

Run both validators on every completed baseline/final run:

```bash
node <skill-dir>/vendor/cloudflare-security-audit/validate-coverage-ledger.cjs <audit-dir>/coverage-ledger.json
node <skill-dir>/vendor/cloudflare-security-audit/validate-findings.cjs <audit-dir>/findings.json
```

Schema validity checks structure; it does not establish exploitability, closure,
source freshness, or complete coverage. QC must check those facts independently.

## Release rules (take precedence over the generic QC verdict rules)

| Audit result | SecureMe verdict |
|---|---|
| Any unresolved confirmed vulnerability, regardless of owner or plan inclusion | `NOT-READY` |
| Final audit missing, blocked, incomplete, stale, or validators fail | `NOT-READY` |
| Complete current-source audit with unresolved `needs_validation`, deferred/blocked coverage, or explicitly partial scope/profile | At best `SHIP-WITH-MANUAL-ITEMS`; disclose exact gaps |
| Complete current-source standard/deep audit, confirmed findings independently closed, no unresolved audit gaps | Eligible for `SHIP` only if all other QC requirements pass |

An incomplete baseline does not permanently block release if the final completed
run independently covers and resolves its candidates and gaps. Never describe a
single run or a clean report as a guarantee that no vulnerabilities remain.

## AUDIT.md template (lead-owned)

```markdown
# SecureMe Cloudflare Audit

Upstream revision: <pinned SHA from vendor UPSTREAM.md>
Profile / scope / total budget: <values and budget accounting>
Baseline: COMPLETE | DEFERRED | BLOCKED | INCOMPLETE
Baseline output: <absolute external path or reason>
Final: PENDING | COMPLETE | BLOCKED | INCOMPLETE | STALE
Final output: <absolute external path or reason>
Reviewed source: <commit, dirty state, digest manifest path>
Validators: <commands, exit statuses, evidence paths>

## Finding mapping
| SecureMe ID | Fingerprint | Run / upstream verdict | Plan ID / owner | Current disposition | Independent evidence / source digest |
|------------|-------------|------------------------|-----------------|---------------------|--------------------------------------|

## Coverage and blockers
- <unresolved fact, affected unit, safe next action, responsible owner>

## Release assessment
- <eligible verdict and reasons; final QC also checks all launch requirements>
```
