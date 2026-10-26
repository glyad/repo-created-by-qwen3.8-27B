# Repository Configuration Plan — `glyad/repo-created-by-qwen3.8-27B`

**Target:** [github.com/glyad/repo-created-by-qwen3.8-27B](https://github.com/glyad/repo-created-by-qwen3.8-27B) — public repo, personal account `glyad`, GitHub Free plan, single project, default branch `main`.

**Sources (in priority order):**
1. [github-public-repo-open-source-guide.md](github-public-repo-open-source-guide.md) — the phase/step numbering used below (G = guide step)
2. [GitHub Well-Architected Framework](https://learn.github.com/well-architected) — pillars, design principles, checklists, recommendations (WA = framework page)
3. [docs.github.com](https://docs.github.com) — current (2026) Settings UI and rule inventory (D = docs page, URLs in §9)

**Current state (queried via GitHub API, 2026-10-26):** `public` · default branch `main` · Issues ✓ · Projects ✓ · Wiki ✓ · **Discussions ✗** · merge methods: merge+squash+rebase all allowed · **no rulesets** · **Dependabot alerts off** · security features unverified · Actions enabled (default), fork-PR approval = "first-time contributors" (default) · GITHUB_TOKEN = read-only contents/packages (default for new personal repos).

---

## 0. Standing rules (user) & execution log

**Standing rules (override guide/framework defaults):**

1. **Branching strategy = Git Flow** (2026-10-26): long-lived `main` (release) + `develop` (integration); `feature/*` → `develop`; `release/*` → `main` + `develop`; `hotfix/*` → both. Overrides the guide's trunk-based default (G 1.6/4.1).
2. **Merge commits only** (2026-10-27): squash + rebase merge methods **disabled** in repo settings; every integration is a real merge commit.
3. **Agent artefacts live in the repo** (2026-10-27): all agent-generated artefacts are stored in the repo under `docs/` (this plan and the open-source guide included); the repo copy is the **source of truth** and is updated as part of the project.

**Execution log** — settings are applied step by step with user approval (details in §3):

| Step | Scope (§3.x) | Status |
|---|---|---|
| 1 | General + Git Flow foundation (3.1) | ✅ 10-26: discussions on, wiki off, `develop` created; merge methods → **merge-only** 10-27. ⚠ Manual UI pending: push policy = 5; auto-delete head branches |
| 2 | Access: code review limits (3.3) | ⏳ approved 10-27 — **UI-only** (no REST): Settings → Access → Moderation options → *Code review limits* → check *Limit to users explicitly granted read or higher access* → Save — **awaiting your click** |
| 3 | Security & quality: Advanced Security (3.4) | ⬜ queued |
| 4 | Actions settings (3.5) | ⬜ queued |
| 5 | Config files: LICENSE, CODEOWNERS, SECURITY.md, templates, dependabot.yml, CI + CodeQL (3.6) | ⬜ queued |
| 6 | Ruleset `baseline-gitflow` (3.2): Disabled dry-run → Active + status checks (§4 Phase 2) | ⬜ queued |
| 7 | Verification suite (§5) + governance cadence (§6) | ⬜ queued |

---

## 1. Target state

A **public open-source repository configured at the framework's "Start" maturity, with the load-bearing security controls of "Mature" already in place** — and deliberately *not* over-governed (every pillar's universal **Keep it Simple** principle). Concretely: anyone can read/fork/PR safely; maintainers' work is protected by a single baseline ruleset; secrets can't enter; dependencies are watched; CI gates merges; the community has clear, documented channels (docs, discussions, issues, security reporting).

> **Branching-strategy rule (user, 2026-10-26): Git Flow.** Long-lived `main` (release) + `develop` (integration); `feature/*` → `develop`, `release/*` → `main` + `develop`, `hotfix/*` → both. This **overrides the guide's trunk-based default** (G 1.6/4.1); all merge-type, history, and ruleset-targeting config below follows Git Flow.

## 2. Design principles applied (from the framework)

| # | Principle (WA) | How it shapes this plan |
|---|---|---|
| P1 | **Keep it Simple** (all pillars) | One repo, one baseline ruleset, no tiering, no org-level machinery, no push rulesets (not available for public on Free anyway). Add complexity only at §8 triggers. |
| P2 | **Evaluate before enforce** (Governance: Rulesets Best Practices) | Roll out rules in observe-then-block stages (§4 Phase 2). Note: the current ruleset UI offers **Active/Disabled** (plus *Evaluate* specifically for metadata restrictions) — use **Disabled** as the dry-run state, then flip to **Active**. |
| P3 | **Public-repo threat model** (AppSec: Securing GitHub Actions — "use caution with public repositories") | Never self-hosted runners; require approval for *all* outside collaborators on fork PRs; restrict allowed actions + full-SHA pinning; least-privilege GITHUB_TOKEN. |
| P4 | **Defense in depth** (AppSec: NIST SSDF) | No single control is the only barrier: ruleset (PR required + approvals + status checks) **and** CODEOWNERS **and** secret scanning/push protection **and** code review limits **and** action allow-list. |
| P5 | **Free GHAS for public repos** (AppSec / docs) | Dependabot alerts, secret scanning, push protection, and code scanning cost nothing on public — enable all of them (the docs' "at a minimum, enable" list). |
| P6 | **Design for Accountability** (Governance) | CODEOWNERS + code review limits + attribution: random users can comment but not *approve*; every change is traceable. |
| P7 | **Design for Openness** (Collaboration) | Discussions on, issue/PR templates, SECURITY.md, code review *limits* (not a closed repo) — open, but quality-gated. |

## 3. Configuration by settings area

UI paths per current docs (2026 sidebar groups: **General** · **Code, planning, and automation** · **Access** · **Security and quality** · **Actions**).

### 3.1 General *(D: managing-your-repositorys-settings-and-features; docs page: managing-your-repositorys-settings-and-features)*

| Setting | Current | Target | Why (source) | How |
|---|---|---|---|---|
| Description | set (sample text) | Real one-line description of the project | G 3.1 — README/identity | UI General; `gh api -X PATCH … -f description="…"` |
| Homepage | — | Project URL **only after** a site exists (G 3.10) | G 3.1 | UI General |
| Topics | — | 3–6 descriptive topics (discoverability) | G 8.2 (launch/visibility) | UI General → Topics |
| **Features: Issues** | on | **on** | G 3.5 — core OSS channel | keep |
| **Features: Projects** | on | **on** (work tracking board, G 7.2) | G 7.2; Governance: accountability | keep |
| **Features: Discussions** | **off** | **on** (categories: Announcements, Q&A, Ideas) | G 3.8; WA Collaboration: Openness | UI General → Features; `gh api -X PATCH … -f has_discussions=true` |
| **Features: Wiki** | on | Decide: **off** if docs live in-repo + Pages (avoid two doc homes); **on** only if used | P1; Architecture: Interoperability (single doc home) | UI Features |
| **Default branch** | `main` | Keep `main` (Git Flow release branch); ruleset targets `main` + `develop` | Git Flow rule | keep (D: managing-the-default-branch-name) |
| **`develop` branch** | — | **Create from `main`** (Git Flow integration branch) | Git Flow rule | `git push origin main:develop` (Step 1) |
| **Merge methods** (General → *Pull request merges* or Rules) | all 3 | **Merge commits only** — squash + rebase **disabled** (Rule 2: strict Git Flow; every integration is a real merge commit) | Rules 1+2; D: available-rules (merge type rule) | `gh api -X PATCH … -f allow_squash_merge=false -f allow_rebase_merge=false` — ✅ applied 2026-10-27 |
| **Auto-delete head branches** | default | **On** — in Git Flow, merged `feature/*` branches are deleted after merging into `develop` | WA Governance (rulesets doc); Git Flow hygiene | UI branch settings |
| **Push policy** (General → Pushes) | unlimited | **Limit branch/tag updates per push to 5** (docs-recommended default) | WA Governance: auditability/control; blocks `git push --mirror` accidents (D: push policy) | UI General → Pushes |
| **Visibility** | public | **Keep public** — note: going public makes Actions logs public; public→private later costs free GHAS features (D: visibility) | G context; P5 | no change; document the trade-off in the repo |
| Autolinks | — | None (no external tracker) | P1 | no change |
| Delete repository | default | Keep default confirmation; **archive** (not delete) when done | D: archiving repos | no change |

### 3.2 Rulesets — Settings → *Code, planning, and automation* → Rulesets *(D: creating-rulesets, available-rules)*

Create **one baseline branch ruleset** — `baseline-gitflow` (WA Governance: start with a minimal baseline ruleset; rulesets are available on Free for public repos):

- **Target:** branches `main` **and** `develop` (Git Flow long-lived branches; two include entries in the ruleset)
- **Enforcement:** create **Disabled** (dry-run) → flip **Active** after the observation week (§4 Phase 2)
- **Bypass list:** `glyad` (repository owner/admin) — the only break-glass actor; monitor its use (WA: bypass SOP + audit)

| Rule | Value | Why |
|---|---|---|
| Block force pushes | **on** (default) | WA Governance: "protect against accidental or malicious destruction of history" (both branches) |
| Restrict deletions | **on** (default) | protect `main` + `develop` from deletion |
| **Restrict creation** | `release/*` and `hotfix/*` (bypass users only) | Git Flow discipline: only maintainers cut release/hotfix branches; `feature/*` stays open to contributors |
| Restrict updates | off | Git Flow: maintainers may push directly to `develop` for maintenance; community changes go via PR (below) |
| Require a pull request before merging | **on** | WA Governance baseline tier; NIST SSDF: "require code review approval before merging" |
| Required approvals | **1** (from a user with write permission) | WA Governance production tier. *(Team-based "required reviewers" is org-only — N/A on a personal account.)* |
| Dismiss stale approvals when new commits pushed | **on** | WA: "prevents outdated approvals from being applied to revised code" |
| Require approval of the most recent reviewable push | **on** | approvals correspond to the *latest* commit (WA sensitive-tier rule) |
| Require review from CODEOWNERS | **on** | WA Architecture: CODEOWNERS manages reviews; P6 |
| Require all comments resolved before merging | **on** | WA: "ensures reviewer feedback is addressed" |
| Require a merge type | **merge** | Git Flow integrations are real merge commits (`develop`→`main`, `release/*`/`hotfix/*` back-merges) — pairs with merge-only merge methods (§3.1, Rule 2) |
| Require linear history | **off** | Git Flow *needs* merge commits; linear history is incompatible with the strategy |
| Require status checks to pass | **on** — add `CI` (guide §4 workflow name) + code-scanning check; "branches up to date" **on** | WA Governance baseline: "require workflows to pass"; NIST SSDF: "configure required status checks for security scans" (add in Phase 2 once CI exists) |
| Require signed commits | **off now** | WA Advance-stage rule — enable later, once maintainers sign (G 5.8.3) — *trigger: §8* |
| Require secret scanning alerts resolved | **off now** (public preview) | Revisit when stable — P1 |
| Require code scanning results | **off now** → on with threshold *error* in Phase 2 | WA production-tier: "severity threshold tuned to avoid overly cautious blocks" |

**Explicitly NOT configured** (P1 + docs availability): push rulesets (private/internal-only on Team plan), tag ruleset (until release automation exists, §8), commit-metadata restrictions, file-size/extension restrictions, code-coverage threshold (needs GitHub Code Quality), license-compliance rule (needs dependency data; revisit once the project has real dependencies — then it's a *nice* OSS rule).

### 3.3 Access *(D: teams-people; pr-reviews)*

| Setting | Current | Target | Why |
|---|---|---|---|
| Collaborators | owner only | Owner only for now; **add maintainers as `write`**, triagers as `maintain`/custom role when they appear | WA Governance: least-privilege role hierarchy (G 7.1). Personal account ⇒ no teams — document as future org move (§8) |
| **Code review limits** (Access → Moderation options) | default (anyone can approve) | **On — "Limit to users explicitly granted read or higher access"** | P6: in public repos *any user* can submit approving reviews by default; limits keep quality gates meaningful while allowing everyone to comment (D: managing-pull-request-reviews) |

> **How:** UI only — Settings → Access → Moderation options → *Code review limits* → check *Limit to users explicitly granted read or higher access* → Save. No REST endpoint exists (probed 2026-10-27).

### 3.4 Security & quality → Advanced Security *(D: managing-security-and-analysis; free for public repos)*

| Feature | Current | Target | Why |
|---|---|---|---|
| Dependabot **alerts** | off | **on** | P5; WA AppSec: "continuously protect against vulnerable components"; NIST SSDF checklist |
| Dependabot **security updates** + **version updates** | off | **on** (backed by `.github/dependabot.yml` incl. `github-actions` ecosystem, G 5.5) | WA: updates the supply chain, incl. Actions |
| **Secret scanning** | verify | **on** | P5; WA Resiliency; NIST SSDF |
| **Push protection** | verify | **on** | blocks secrets *at push time* — the public-repo-critical one (NIST SSDF: "enable secret scanning and push protection") |
| **Code scanning** | off | **on** (choose language starter workflow, G 5.3) | NIST SSDF: "enable and configure code scanning" |
| **Private vulnerability reporting** (Security Advisories) | off | **on** + `SECURITY.md` pointing to it | G 3.7; WA Proactivity: "integrate source code vulnerabilities into a vulnerability reporting and management system" |
| Access to security alerts | default (write+) | owner + future maintainers only | WA: alerts are sensitive; least privilege |
| Custom secret patterns | — | **None** — credentials only; never PII patterns | WA anti-pattern: PII via secret scanning = compliance exposure (use DLP) |

### 3.5 Actions → General *(D: managing-github-actions-settings)*

| Setting | Current | Target | Why |
|---|---|---|---|
| Actions permission | allow all | **Allow OWNER, and select non-OWNER actions and reusable workflows** → tick *Allow actions created by GitHub* (+ *verified creators* if a needed action isn't GitHub's) | P3/P4 — restrict the supply chain at the repo level (WA: "restrict allowed actions") |
| **Require actions pinned to a full-length commit SHA** | off | **on** | WA Actions Security: "pin actions to a specific commit hash… enforce via policy" — now enforceable at repo level (D). If using a workflow dependency lockfile, locked deps satisfy this automatically (D) |
| Workflow permissions (GITHUB_TOKEN) | read contents/packages | **keep restricted (read-only default)**; do *not* enable "Allow GitHub Actions to create and approve pull requests" | P3 least privilege (WA effective-ci security fundamentals); prevent bot self-approval loops |
| **Fork PR workflow approval** | first-time contributors | **Require approval for ALL outside contributors** | P3 — WA's explicit public-repo rule; the docs warn the weaker "first-time" options can be gamed via one accepted typo (D: actions settings) |
| Self-hosted runners | — | **Never** for this public repo | P3 — WA warning: malicious code on your machine, cache poisoning, network exposure. Hosted runners only |
| Cache settings | default (7 d / 10 GB) | keep defaults; revisit if CI grows (public max 90 d / 10 TB) | P1 |
| Retention (runs/artifacts/logs) | 90 d | keep 90 d (public max); public ⇒ logs visible to all — another reason for SHA pinning + restricted actions | D note: from Oct 1 2026 retention policies also apply to checks/status |
| Environments | none | none until a deploy workflow exists (then: `staging`/`production` with required reviewers) | P1; WA Architecture: deployment gates |
| OIDC | — | configure only when publishing to a cloud registry (G 5.8.4) | short-lived tokens, no long-lived secrets (WA Actions Security) |

### 3.6 Repository files that are configuration *(G phase 3/4/5; D: code-owners)*

These live in the repo, not the Settings UI, but are part of the configuration target — on **`develop`** (CODEOWNERS only works from the PR base branch, D; feature PRs target `develop`), flowing to `main` at the first release cut (Git Flow):

```
.github/CODEOWNERS                 ← `* @glyad` plus explicit paths: .github/workflows/**, .github/SECURITY.md, etc. (owners need write access; team entries are org-only)
.github/dependabot.yml             ← ecosystems: <package mgr> + github-actions; schedule weekly
.github/workflows/ci.yml           ← fail-fast: lint → build/test, path filters, concurrency, SHA-pinned actions (G 4); triggers on PRs to `main` + `develop` and pushes to `develop` (Git Flow)
.github/workflows/codeql.yml       ← language starter (G 5.3)
LICENSE                            ← chosen in G 1.4 (add via Settings → *Licensing* or file)
SECURITY.md                        ← private advisory channel + SLA (G 3.7)
CONTRIBUTING.md, CODE_OF_CONDUCT.md, issue/PR templates (G 3.3–3.6)
```

Commit sign-off policy (General → *Commits*): **off for now** — DCO sign-off makes sense once external contributors are real (P1). Revisit at §8 trigger.

## 4. Rollout plan (evaluate → enforce, per P2)

**Phase 1 — Day 1, zero-friction changes** (none of these block any workflow):
- Create `develop` branch from `main` (Git Flow foundation)
- §3.1 (description, discussions on, wiki decision, merge methods, auto-delete, push policy, topics)
- §3.3 code review limits
- §3.4 all Advanced Security toggles + private vulnerability reporting
- §3.5 Actions settings
- §3.6 files (LICENSE, CODEOWNERS, SECURITY.md, templates, dependabot.yml, CI + CodeQL workflows)
- *Exit check:* a test PR from a **fork** runs CI but the workflow requires maintainer approval; a planted secret push is blocked; README/docs all render.

**Phase 2 — After first CI run (observe, then enforce):**
1. Create `baseline-gitflow` ruleset (targets `main` + `develop`) with **all rules above except status-check/code-scanning gates**, enforcement **Disabled**.
2. Observe 1 week: attempt the merge scenarios from §5 while it's Disabled; confirm no false positives (WA: "use Evaluate to surface potential merge/push friction before enforcement"; track rule insights/violations).
3. Flip to **Active**; add the required status checks (`CI` + code scanning) and the code-scanning merge-protection rule with severity *error*.
4. *Exit check:* §5 verification matrix passes fully.

**Phase 3 — At scale (only at §8 triggers):** signed commits + tag ruleset for release tags, environments + deployment gates, commit sign-off, license-compliance rule, move to a dedicated **organization** (unlocks teams, tiered org rulesets, custom properties with the public-repo metadata caution, SSO) — the WA "Managing Repositories at Scale" tiering model (properties → rulesets → phased rollout → metrics).

## 5. Verification suite (run at end of each phase)

| # | Test | Expected |
|---|---|---|
| V1 | Push a branch; `git push -f` a rewritten history | blocked (force-push rule) |
| V2 | Direct push to `main` | blocked (PR required) |
| V3 | Open PR from a fork | workflow run **needs approval**; cannot run unattended (all-outside policy) |
| V4 | PR with no approval / with 1 "requested changes" review | cannot merge |
| V5 | PR with 1 write-permission approval + green CI + resolved comments | merges via **merge commit** (Git Flow) |
| V6 | Push a file containing a fake AWS-format key | push protection blocks + secret alert |
| V7 | Random outside user attempts an approving review | not allowed (code review limits) |
| V8 | Delete `main` via API/push | blocked (deletion rule) |
| V9 | After Active: check **Rule Insights** + audit log for bypass usage | no unexpected violations; any bypass was deliberate (owner) |

## 6. Ongoing governance cadence

- **Monthly:** review Rule Insights + bypass usage + Dependabot/CodeQL alert backlog (WA: "monitor bypass exceptions… look for patterns").
- **Quarterly:** run the five **pillar checklists** (WA Getting-Started flow) as a 30-min self-assessment; one improvement per pillar; move one design principle up its Start→Mature→Advance ladder (G 8.3/8.4).
- **On every change to rules:** keep the ruleset in the repo as JSON where practical (prebuilt recipes: `github/ruleset-recipes`, D: creating-rulesets) so config is reviewable in PRs — Governance: "auditability".

## 7. Explicit non-actions (Keep it Simple)

| Not doing | Why / when to revisit |
|---|---|
| Custom properties / tiered rulesets | Personal account, single repo; org move (§8). WA caution: on public repos, never expose internal names/security posture in properties |
| Push ruleset | Private/internal-only on Free/Team (D: available-rules) |
| SSO / advanced identity controls | Org-level, later |
| Merge queues, auto-merge, merge-train | P1 until PR volume justifies |
| Commit sign-off (DCO), signed-commit rule, license-compliance rule, coverage threshold | Triggers below |

## 8. Evolution triggers (revisit §3/§4 when…)

- **First external contribution lands** → enable commit sign-off if the community asks for attribution guarantees; verify CODEOWNERS coverage of new paths.
- **First release** → Git Flow release flow: cut `release/*` from `develop`, stabilize, merge to `main` + `develop`, tag `vX.Y.Z` on `main` → tag ruleset (e.g., `v*`: restrict deletions, require signed commits, require status checks) + release workflow (G 6) + SBOM.
- **10+ regular contributors or 2nd repo** → move to a dedicated **org**; then apply WA Managing-Repositories-at-Scale (custom properties → tiered rulesets → phased rollout → dashboards) and GitHub **community profile** completion (D: visibility page notes the community profile for public OSS repos).
- **CI minutes/duration degrading** → path filters, matrices, reusable workflows, cache tuning (WA Effective CI; G 4.2/4.3).
- **Secret or vulnerability incident** → tighten: code-scanning severity, secret-resolution merge rule, private-advisory drill (WA: incident response plan, G 5.7).

---

## 9. Sources

**Guide:** [github-public-repo-open-source-guide.md](github-public-repo-open-source-guide.md)

**Well-Architected:** [Rulesets Best Practices](https://learn.github.com/well-architected/governance/recommendations/managing-repositories-at-scale/rulesets-best-practices) · [Custom Properties Best Practices](https://learn.github.com/well-architected/governance/recommendations/managing-repositories-at-scale/custom-properties-best-practices) · [NIST SSDF Implementation](https://learn.github.com/well-architected/scenarios/nist-ssdf-implementation) · [Securing GitHub Actions](https://learn.github.com/well-architected/application-security/recommendations/actions-security) · [Managing Dependency Threats](https://learn.github.com/well-architected/application-security/recommendations/managing-dependency-threats) · [Effective CI](https://learn.github.com/well-architected/productivity/recommendations/effective-ci) · [Collaboration Design Principles](https://learn.github.com/well-architected/collaboration/design-principles) · [Governance Design Principles](https://learn.github.com/well-architected/governance/design-principles) · [AppSec Design Principles](https://learn.github.com/well-architected/application-security/design-principles) · [Anti-patterns](https://learn.github.com/well-architected/scenarios/anti-patterns)

**docs.github.com (current, fetched 2026-10-26):**
[Managing security and analysis settings](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-security-and-analysis-settings-for-your-repository) · [Managing GitHub Actions settings](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository) · [Creating rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/creating-rulesets-for-a-repository) · [Available rules for rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets) · [Managing pull request reviews](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/managing-pull-request-reviews-in-your-repository) · [Push policy](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/managing-the-push-policy-for-your-repository) · [Teams & people access](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/managing-teams-and-people-with-access-to-your-repository) · [About code owners](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners) · [Commit signoff policy](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/managing-the-commit-signoff-policy-for-your-repository) · [Setting repository visibility](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/setting-repository-visibility) · [Enabling Discussions](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/enabling-or-disabling-github-discussions-for-a-repository) · [About secret scanning](https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning)
