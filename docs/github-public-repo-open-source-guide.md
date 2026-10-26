# Creating a GitHub Public Repository for an Open-Source Project

**A step-by-step guide derived from the [GitHub Well-Architected Framework](https://learn.github.com/well-architected).**

---

## How this guide was built

The GitHub Well-Architected Framework structures every area of your engineering system on GitHub in a layered model:

1. **Pillars** — five: Productivity, Collaboration, Application Security, Governance, Architecture.
2. **Design principles** — the "how" behind each pillar (e.g., *Design for Openness*, *Design for Control*).
3. **Checklists** — self-assessment questions for each pillar.
4. **Recommendations** — concrete, implementable patterns (e.g., *Managing Repositories at Scale*, *Effective CI*).

Every design principle is a maturity ladder — **Start → Mature → Advance** — and every pillar ends with a **Keep it Simple** principle. Scenarios (anti-patterns, migrations, monorepos, NIST SSDF) apply the pillars to specific situations.

This guide applies that framework to one specific scenario: **creating and launching a public GitHub repository for an open-source project.** Each step below cites the framework layer it implements.

| Phase | Framework pillars used |
|---|---|
| 1. Plan | Governance (accountability, control), Architecture (repo architecture strategy), Scenarios (project planning) |
| 2. Create & push | Architecture (modularity, Git LFS), Scenarios (anti-patterns: commit/branching) |
| 3. Documentation & openness | Collaboration (openness, transparency, communication), Architecture (interoperability: docs) |
| 4. CI | Productivity (automation, feedback), Architecture (efficiency) |
| 5. Security | Application Security (security, proactivity, awareness, compliance) |
| 6. Releases | Architecture (change management), Productivity (automation) |
| 7. Governance & collaboration ops | Governance (auditability, accountability, control), Collaboration (flexibility, inclusivity) |
| 8. Launch & operate | All pillars (observability, continuous learning, engineering-system success) |

---

## 1. Plan the project (before creating anything)

### 1.1 Define scope, goals, and success criteria
*(Governance: Design for Control; Scenarios: Project Planning)*

- Write a one-paragraph project statement: what the software does, who it's for, and the outcome you want in the first 6 months.
- Define success metrics you'll actually measure later (e.g., contributors, release cadence, issue triage time, CI feedback time).
- Identify stakeholders and decision-makers; document who will be involved or informed at each stage.

### 1.2 Choose where the project lives: a dedicated organization
*(Governance: Design for Accountability — "delineate ownership for repositories, teams, and settings")*

- Create a dedicated **GitHub organization** for the project (even a free one) rather than scattering repos under a personal account. This gives you a stable home for teams, packages, pages, and discussions, and survives contributor turnover.
- If this is one of many projects, group related projects under one org and plan per-repo governance.

### 1.3 Decide the repository layout: monorepo vs. polyrepo
*(Architecture: Exploring Repository Architecture Strategy; Scenarios: Monorepos)*

- **Single project** → one repo (obvious choice).
- **Multiple tightly-coupled components** (web app + mobile + backend services that must change atomically) → **monorepo** with a modular directory layout; plan CI to trigger on path filters so builds stay fast.
- **Independent, independently-released components** → **polyrepo** (one repo per component) with consistent naming and directory conventions across repos; plan a shared "integration/meta" repo if you need cross-repo validation later.
- Heuristic from the framework: default to the simplest structure that meets your needs (Keep it Simple); split repos *only* when scale, access control, or release-cadence boundaries demand it. Document the decision in `docs/` so future maintainers understand why.

### 1.4 Choose a license (the defining decision for open source)
*(Application Security: Design for Compliance; Collaboration: Design for Openness)*

- Pick an **OSI-approved license** that matches your intent:
  - Permissive: MIT, Apache-2.0 (Apache-2.0 adds patent grant + notice requirements).
  - Copyleft: GPL-2.0/3.0, LGPL, MPL-2.0.
- Record the choice in the plan; you will add the license file in step 2.2 and 3.2.
- If the project will ever be used in a compliance-sensitive context, document which regulations/standards apply (the framework's Compliance principle: "document applicable regulations and standards").

### 1.5 Define ownership and roles
*(Governance: Design for Accountability; Design for Control)*

- Name a **project owner** (final decision-maker) and at least one backup maintainer — a project with exactly one busy person is a failure risk (Architecture: Resiliency).
- Define the role set you'll need on GitHub: **maintainers** (write access to the repo), **triagers** (label/close issues), **releasers** (publish packages), plus the read-only public contributor path (fork → pull request).
- Keep access to least privilege: grant write access only to maintainers (Governance: "implement robust access controls and permissions").

### 1.6 Define naming, branching, and versioning conventions
*(Architecture: Version Control Practices; Scenarios: Anti-patterns — "inconsistent branching strategy", "poor commit practices")*

- **Repo name**: short, lowercase, hyphenated; memorable and searchable.
- **Branching**: trunk-based development with short-lived feature branches (the framework lists trunk-based or Gitflow as options; trunk-based best fits continuous integration and fast feedback).
- **Branch naming**: `fix/123-login-bug`, `feature/45-dark-mode` — prefix + issue number.
- **Commit messages**: imperative, one-line summary ≤ 72 chars, reference the issue (`Fix null crash in parser (#123)`).
- **Versioning**: [Semantic Versioning](https://semver.org) (`MAJOR.MINOR.PATCH`), tags as `v1.2.3`.
- **Delete merged branches** automatically (keep the repo clean).

### 1.7 Write a short runbook and launch/communication plan
*(Scenarios: Project Planning — "create a migration plan runbook", "create a comprehensive communication plan")*

- One page: what you're building, the milestone plan, who does what, and how you'll announce the project (launch post, newsletter, HN/Reddit/forums) in step 8.

---

## 2. Create the repository and push the code

### 2.1 Create a **public** repository
- On github.com: **New repository** (or org name → New → Create a new one) → name it per 1.6 → set **Description** → visibility **Public** → initialize with:
  - ✔ *Add a README*
  - ✔ *Add a .gitignore* (pick the template for your language)
  - ✔ *Choose a license* (pick the license from 1.4 — GitHub inserts the correct file)
- Public visibility is what unlocks the open-source contributor model (anyone can fork/PR) **and** free security features (step 5).

### 2.2 (If the code already exists elsewhere) import it cleanly
*(Scenarios: Checklist for Repository Migrations — Appendix A)*

- **Fresh project**: push from your local clone.
  ```bash
  git init -b main
  git add . && git commit -m "Initial commit: <project name>"
  git remote add origin https://github.com/<org>/<repo>.git
  git push -u origin main
  ```
- **Existing project**: run the six-phase migration checklist (Plan → Assess → Set up → Test → Migrate → Post-migrate, Appendix A) rather than a blind `git push`. If migrating from another host with metadata (issues/PRs) you want to keep, use the **GitHub Enterprise Importer** or `gh migration start`.
- **Before the first push, audit what you're committing** (see 2.3) — public means *everything you push is public forever*. Verify no secrets, PII, or internal data are in the tree/history (Application Security: secret scanning will also catch past secrets — Appendix A step 6.1).

### 2.3 Audit file types before and after the first push
*(Architecture: When to Use Git LFS; Managing Large Git Repositories)*

- Apply the framework's LFS heuristics to anything large:
  - File **doesn't diff** (binary) and **doesn't compress well** and **changes often** → **use Git LFS**.
  - File **compresses well** (text/JSON/markup) → **keep in Git** (LFS would actually *increase* transfer size).
  - Rarely-changing or few-version files → either way is fine.
- Typical LFS targets: media/images, datasets, models, auto-generated manifests, many small generated files. Typical non-LFS: source, text, config, compressed archives.
- If the repo already has big binaries in history, plan cleanup with `git filter-repo` (back up first; rewriting history in a public repo requires a coordinated re-clone — do this *before* announcing the project).
- Keep `.gitignore` rules in the deepest sensible directories; prune stale branches/tags periodically to keep the repo healthy.

### 2.4 Verify the baseline
- Open the repo URL: README renders, license badge appears, code is visible, `main` branch exists.
- Confirm no secrets in the pushed content (search your old environments for anything you may have committed).

---

## 3. Build the open-source documentation layer

*(Collaboration: Design for Openness, Transparency, Communication; Governance: "develop a comprehensive set of contribution standards to steer open-source collaborative efforts")*

### 3.1 Write a README.md that answers the newcomer's questions in 60 seconds
- What it is, one command to try it, how to build it, where to ask for help, license, and badges (CI status, latest release, license).
- The framework's Openness principle: create a "comprehensive knowledge repository for documentation and guidelines" — the README is its front door.
- Add a link to every other document you create in this phase (CONTRIBUTING, SECURITY, discussions) so no information is hidden.

### 3.2 Commit the LICENSE file (already chosen in 1.4, inserted in 2.1)
- Without a license, open source means *all rights reserved* by default — this file is legally what makes the project open source.

### 3.3 Write CONTRIBUTING.md
*(Governance: Accountability — contribution standards; Collaboration: Inclusivity — onboarding)*

- How to set up the development environment (one-command setup if possible — Collaboration: Keep it Simple).
- The contribution flow: find an issue labeled `good first issue` → fork → branch → PR → review.
- Coding standards, commit/PR conventions (from 1.6), and what maintainers do with contributions.
- An onboarding path for new contributors is an explicit framework principle (Inclusivity: "is there an onboarding process that incorporates key tooling concepts?").

### 3.4 Add CODE_OF_CONDUCT.md
- Use an established template (e.g., Contributor Covenant). This is how a public, open community protects itself.

### 3.5 Add issue templates in `.github/ISSUE_TEMPLATE/`
*(Collaboration: Design for Effective Communication — "are there established channels for proactive engagement and feedback?")*

- `bug_report.md` (with a checklist: version, OS, repro steps, expected vs actual).
- `feature_request.md` (describe the problem, not just the solution).
- A "question → please use Discussions" nudge.

### 3.6 Add a pull request template in `.github/pull_request_template.md`
- What changed, why, linked issue, screenshots/CI evidence, and a "how was this tested" section. Structured PRs make code review (step 7.4) dramatically faster.

### 3.7 Add SECURITY.md — a vulnerability disclosure policy
*(Application Security: Design for Proactivity & Awareness — "integrate source code vulnerabilities into a vulnerability reporting and management system")*

- Tell researchers **not** to open public issues for vulnerabilities, and give them a private channel: GitHub's private vulnerability reporting (**Settings → Security → Secret scanning and security analysis / private vulnerability reporting** → "Start a private advisory") or `security@your-domain`.
- State your response SLA (e.g., acknowledge in 3 days) — public disclosure policies measurably improve reporting.

### 3.8 Enable GitHub Discussions
*(Collaboration: Openness — "evaluate the use of GitHub Discussions…"; repository recommendation: "enable Discussions… use for brainstorming, Q&A, archiving decisions")*

- **Settings → General → Features → Discussions: enable.** Categories: Announcements, Q&A, Ideas, Show & Tell.
- Archive important design decisions here (Transparency: "are there virtual spaces for exchanging ideas and deliberating issues?").

### 3.9 Set up release notes and a changelog
*(Architecture: Change Management; Collaboration: Transparency)*

- Choose a convention: GitHub Releases with auto-generated notes, or a `CHANGELOG.md` kept via a "keep a changelog" style.
- Document the decision in CONTRIBUTING.md so every release is consistent.

### 3.10 (Optional) Host documentation with GitHub Pages
*(Architecture: Interoperability — "provide comprehensive and clear documentation"; Repository Architecture: "host project documentation on GitHub Pages")*

- **Settings → Pages** → serve from `docs/` or the repo root; pair with a static site generator (Jekyll/Docusaurus/VitePress) built in CI (step 4).

---

## 4. Set up continuous integration (fail-fast by design)

*(Productivity: Design for Automation, Design for Feedback; Architecture: Design for Efficiency; Recommendation: Effective CI)*

### 4.1 Create the base CI workflow: `.github/workflows/ci.yml`
The framework's core CI pattern is **fail-fast**: run the checks most likely to fail and fastest to run *first*, so a typo is caught in 30 seconds, not 17 minutes.

```yaml
name: CI
on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]
permissions:
  contents: read        # least privilege — override per-job only where needed
jobs:
  lint:                 # stage 1: fast validation gate
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@<commit-SHA>
      - run: <install deps with cache>
      - run: <lint + typecheck>
  build:                # stage 2: independent jobs fan out after the gate
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@<commit-SHA>
      - run: <build>
  test:
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@<commit-SHA>
      - run: <test suite>
```

- Tests must be **isolated** (no shared state/ordering), **deterministic** (treat flaky tests as high-priority bugs — they train people to ignore CI), and **clearly named** (`test_creating_user_without_email_returns_400`, not `test_user_creation`).
- Give rich feedback: add job summaries/annotations so a failure tells the developer *what* failed, *why*, and *where* to start — not just a red ❌.

### 4.2 Optimize the pipeline
*(Effective CI: Practices; Architecture: Efficiency — "eliminate bottlenecks")*

- **Cache dependencies** in every job (`actions/setup-<lang>` with `cache:`) — every hosted runner starts clean; without caching you re-download everything.
- **Path filters**: only run each job when its directories changed (essential if you later go monorepo: "build only affected packages").
- **Concurrency**: cancel superseded runs (`concurrency: group: ci-${{ github.ref }}`).
- **Matrix/sharding** only when you genuinely need multiple platforms/versions; combine with the lint gate, don't multiply it.
- Reuse: if the pattern repeats, turn it into a **reusable workflow** — and treat it like a product: stable inputs contract, versioned tags, changelog, protected with CODEOWNERS (polyrepo recommendation).

### 4.3 Measure CI, don't just run it
*(Effective CI: "CI metrics are tracked — duration, flake rate, queue time"; Productivity: Engineering System Success — baseline your metrics)*

- Watch three numbers from day one: **time-to-first-feedback**, **total duration**, **flaky-failure rate**.
- When duration degrades, that's your optimization backlog (caching, sharding, parallelism).

---

## 5. Harden security (most of this is free on public repositories)

*(Application Security: Design for Security, Proactivity, Awareness, Compliance. Code scanning, secret scanning, and Dependabot are provided at no charge for public repos — no paid license needed.)*

### 5.1 Protect the default branch with a repository ruleset
*(Governance: Rulesets Best Practices; Recommendation: NIST SSDF — "implement repository rulesets on all default branches")*

Create a ruleset for `main` (Settings → **Rules → Rulesets** → New ruleset; or branch protection rules) with the framework's **baseline tier**:

- ✔ Block force pushes — protect history from accidental/malicious destruction.
- ✔ Block branch deletion of protected branches.
- ✔ **Require a pull request before merging** (even with no required approval at first — creates visibility and lets checks run).
- ✔ **Require status checks to pass** (your CI workflow from step 4, plus the security scans from 5.3–5.5).
- ✔ Require at least **1 approving review**, and **dismiss stale approvals** when new commits land.
- Later tightening (Production tier from the framework): require review from **CODEOWNERS**, require conversation resolution, require signed commits once maintainers sign.

Roll it out the framework way: draft in **Evaluate (non-blocking) mode** for a sprint, watch the "Rule Insights" friction data, then switch to **Active (blocking)** — don't land a wall of blocking rules on day one (Keep it Simple).

### 5.2 Add a CODEOWNERS file
*(Architecture: "use GitHub's CODEOWNERS to manage permissions and code reviews"; Governance: Control)*

- `.github/CODEOWNERS` mapping sensitive paths (CI configs, `.github/workflows/*`, security files, core library dirs) to the maintainer(s) who must review changes there.
- Pair with the ruleset rule "require review from CODEOWNERS" so *anyone's* change to a sensitive path needs a domain expert.

### 5.3 Enable code scanning (CodeQL)
*(AppSec: "perform proactive security of source code throughout the SDLC"; NIST SSDF checklist)*

- **Settings → Security → Code scanning → Set up a workflow** (pick the language analysis starter).
- It runs as a required status check in CI — vulnerabilities are caught *before* merge (shift left).

### 5.4 Enable secret scanning + push protection
*(AppSec: Resiliency — "use secrets management to reduce the failure rate"; NIST SSDF — "enable secret scanning and push protection")*

- **Settings → Security → Secret scanning and security analysis → Secret scanning: enable**, and enable **push protection** so accidental keys are blocked at push time instead of living in a public repo.
- Framework anti-pattern warning: keep custom secret patterns focused on *credentials* (detect-revoke-rotate). Do **not** use secret scanning patterns to track PII — that creates compliance exposure; use DLP tooling for PII.

### 5.5 Turn on Dependabot (dependencies *and* your Actions)
*(AppSec: "continuously protect against vulnerable components in the broader software supply chain"; Actions Security — "use Dependabot to protect the supply chain")*

- **Settings → Code security → Dependabot → Security updates: enable** + **Version updates: enable**.
- In `.github/dependabot.yml`, add **both** your package ecosystems **and** the `github-actions` ecosystem — your workflow files are a software supply chain too.

### 5.6 Secure your Actions workflows against the public-repo threat model
*(Application Security: Securing GitHub Actions — "anyone can suggest changes to public repositories")*

Apply every item; these are the framework's explicit rules for public repos:

1. **Never run self-hosted runners for a public repository.** Anyone who can fork can open a PR that runs code on your machine — malicious programs, cache poisoning, network access. Use **GitHub-hosted runners** (clean, isolated, destroyed per job).
2. **Require approval for outside collaborators**: Settings → Actions → General → "require approval for all outside collaborators and outside actors on pull request workflows from forks."
3. **Least-privilege tokens**: workflow-level `permissions: contents: read`; grant `packages: write` etc. only on the specific jobs that need it.
4. **Pin every third-party action to a commit SHA** (not `v2`, not `latest`); let Dependabot (5.5) keep the pins updated.
5. **Avoid `pull_request_target`** in public repos — it runs untrusted fork code with your base-repo permissions.
6. **Prevent workflow injection**: never interpolate untrusted data (`github.event.*`) into `run:` steps; sanitize user-controlled input; prefer `head.sha` over user-supplied branch names.
7. **Review every workflow trigger** — a public repo's workflows can be triggered by anyone who can open a PR.
8. (When operating multiple repos) restrict which actions/reusable workflows are allowed at the org level, managed as policy-as-code.

### 5.7 Build a vulnerability response process
*(AppSec: Design for Proactivity — "develop a security incident response plan relevant to development teams"; NIST SSDF — RV group)*

- Write one page: who triages Dependabot/CodeQL/secret alerts, the SLA by severity (e.g., critical < 24 h), and how advisories ship (security release → CVE → disclosure in SECURITY.md channel).
- Route security alerts to a private channel (team inbox or private advisory), never to the public issue tracker.

### 5.8 (Advance) Hardening the build & release supply chain
*(AppSec: Managing Dependency Threats — the 6-layer model; Architecture: Resiliency)*

Layer these on as the project matures:

1. **Disable package lifecycle scripts** in dependency managers (npm `ignore-scripts` / Yarn `enableScripts: false`) unless a script is explicitly allowlisted — scripts are a top supply-chain vector.
2. **Isolate builds** (ephemeral runners are already isolated on GitHub-hosted; use dev containers for local parity, running as non-root).
3. **Signed commits** for maintainers (GPG/SSH) — verifiable provenance; enforce via ruleset once signing adoption is high.
4. **Trusted publishing + artifact attestations**: publish packages via **OIDC** (no long-lived registry tokens; short-lived federated tokens) and attach build attestations; consumers can then verify provenance.
5. **Generate an SBOM** for every release (your manifest/lockfile or a tool) and attach it to the GitHub Release.
6. **Monitor continuously**: alerts → triage SLA (5.7), plus periodic review of dependency licenses (Dependency Review workflow blocks *introducing* vulnerable or license-incompatible dependencies).

---

## 6. Automate releases

*(Architecture: Change Management & Release Strategy; Productivity: Automation; Scenarios: Anti-patterns — "large releases", "manual deployment processes")*

### 6.1 Version and tag from `main`
- Releases are small and frequent (the framework's anti-patterns call out large, manual releases). Bump the version, tag `vX.Y.Z`, and let automation do the rest.

### 6.2 Create a release workflow
- `.github/workflows/release.yml` triggered on tag push (or a "Release" workflow_dispatch):
  - Bump/generate version → build → run full test suite → package artifacts → **create GitHub Release** with changelog notes → publish packages.
  - For package publishing, use **OIDC trusted publishing** (5.8.4) instead of storing registry tokens.
  - Attach the SBOM (5.8.5) to the release.
- Enforce "require deployments to succeed" in the ruleset if you have a deployment step (framework: sensitive-tier rule).

### 6.3 Keep releases independent of each other
- If you ever split into multiple repos (polyrepo): components release on their own cadence; system-level releases happen in an integration/meta repo with a pinned manifest (framework: "decouple individual component release cadences from system-level releases").

---

## 7. Configure governance and collaboration operations

### 7.1 Create teams and grant least privilege
*(Governance: "design a multi-tiered user role hierarchy"; Application Security: least privilege)*

- **Settings → Teams**: a `maintainers` team (repo write), optionally `triagers` (repo read + issue management via granular permissions).
- Public contributor path stays permissionless: fork → branch → PR. No write access for the community — that *is* the model.

### 7.2 Set up work tracking with GitHub Projects
*(Collaboration: "use tracking and project management features to assign tasks and track progress"; Governance: accountability dashboards)*

- Create a **Project** for the repo; set columns (e.g., Backlog → Triaged → In progress → Review → Done).
- Define a **label taxonomy** that automation can act on: `bug`, `enhancement`, `good first issue`, `help wanted`, `security` (reserved for the private channel), `area/*` per component.
- Add automation: new issue → label + assign a triager; PR merged → move card to Done (GitHub Actions can move Project items).

### 7.3 Keep the branching model trunk-based and boring
*(Collaboration: Design for Flexibility; Anti-patterns: "inconsistent branching strategy")*

- Short-lived feature branches → PR → merge to `main` → branch auto-deleted (configured in the ruleset).
- Publish long-lived `release/*` branches only if your consumers need them; document any deviation in CONTRIBUTING.md.

### 7.4 Formalize code review
*(Collaboration: "optimize and formalize the code review process"; Governance: "implement mandatory review and approval processes")*

- Ruleset: require ≥1 approval, dismiss stale approvals, require conversation resolution (framework production-tier rules).
- Reviewers: CODEOWNERS-mapped maintainers for sensitive paths (5.2); everyone else reviews for quality/kindness.
- Culture: framework's Communication principle — reviews are how a project *disseminates knowledge*, not a gate to be annoyed by. Pair experienced contributors with newcomers (Openness: "establish mentorship programs that pair experienced contributors with newcomers").

### 7.5 Configure the external contributor flow
*(Actions Security public-repo rules; Collaboration: Inclusivity)*

- **Settings → Actions → General**: ✔ *require approval for all outside collaborators* (fork PRs run in a sandbox until a maintainer approves — step 5.6.2).
- Optional: a **CLA bot** (e.g., CLA assistant) if your license/governance requires signed agreements.
- Document the exact contributor steps in CONTRIBUTING.md (3.3) — the framework is explicit that "new contributors can quickly become productive when clear guidelines are available, reducing the learning curve and fostering a welcoming environment."

### 7.6 Invest in community growth (Collaboration: Openness — Mature/Advance)
- Label genuinely good entry points `good first issue` and keep them fresh.
- Host **open contribution sprints / hackathons** — the framework's explicit Advance-stage move: "encourage intensive collaboration on specific features… harness the skills and creativity of a global talent pool."
- Pair newcomers with mentors (7.4); archive decisions in Discussions (3.8).
- Advocate for the project in your network — the framework lists "programs that advocate for open source… highlighting the value of community contributions" as a deliberate practice.

### 7.7 Keep it Simple (the framework's universal principle)
- A young public project needs **few, clear rules** — baseline ruleset + CODEOWNERS + one CI workflow + clear docs. Add governance layers (tiered rulesets, custom properties, policy-as-code) only when you have *multiple repos in the org* and real drift to prevent.
- If you do scale: tag repos with **custom properties** (e.g., `production`, `security_tier`) and attach tiered rulesets dynamically (Governance: Managing Repositories at Scale). **Public-repo caution from the framework: avoid custom properties that could expose sensitive information** (internal project names, security posture) — they're public metadata.

---

## 8. Launch and operate

### 8.1 Pre-flight verification
*(Scenarios: Post-Migration Activities — "confirm repository visibility, permissions… ensure CI/CD and integrations work")*

- [ ] Repo is public; README, LICENSE, CONTRIBUTING, CODE_OF_CONDUCT, SECURITY all render.
- [ ] CI is green on `main` and on a test PR from a **fork** (outside-collaborator approval flow works).
- [ ] CodeQL, secret scanning, Dependabot all report active; a planted test secret is caught by push protection.
- [ ] Branch ruleset blocks a force-push and a merge-without-PR.
- [ ] A release tag produces a full Release with artifacts/SBOM.
- [ ] Discussions enabled; issue/PR templates work.

### 8.2 Announce the project
- Launch via the communication plan (1.7): a post describing the project + link, followed by the appropriate venues (your channels, relevant communities, "show and tell" — Collaboration: Openness advocacy).
- Pin a "getting started" discussion and a "roadmap" issue.

### 8.3 Operate with feedback loops
*(Productivity: Design for Feedback & Continuous Learning; Architecture: Observability)*

- Review the three CI metrics weekly (4.3); tune the pipeline when feedback time degrades.
- Watch repo health: clone size, repo size, LFS usage (Architecture: Scaling Git Repositories) — when clones slow down, apply the large-repo playbook (modularity, LFS heuristics, shallow/partial clone guidance for contributors, history cleanup).
- Run the **pillar checklists** from the framework quarterly as a lightweight self-assessment (Collaboration, Productivity, AppSec, Governance, Architecture checklists); score each area, note one improvement per pillar per quarter — this is exactly the Getting-Started flow of the Well-Architected assessment, scaled to a project.
- Ask contributors for feedback (surveys/discussions) and act on it visibly — "regularly seek user feedback, analyze data, and update technology" (Keep it Simple, Productivity).

### 8.4 Iterate the architecture deliberately
- Revisit the monorepo/polyrepo decision, release cadence, and governance tiers as the community grows; record each decision in Discussions with context (Transparency: "document the decision-making processes for changes").
- Treat framework maturity ladders as your roadmap: you are at **Start** on most principles today; move each principle to **Mature** deliberately, one at a time.

---

## Appendix A — If you're migrating an existing project: the 6-phase repository checklist

*(Scenarios: Checklist for Repository Migrations — condensed)*

| Phase | Key actions |
|---|---|
| **A1. Plan** | Define scope/success criteria; identify stakeholders & decision-makers; timeline with milestones; risks + mitigations; runbook; communication plan; choose pace (big-bang vs phased); per-repo schedule; training needs |
| **A2. Assess** | Inventory current repos: sizes, commit history, submodules, **Git LFS usage**; document branch policies to recreate; document permissions/access patterns; document all integrations/webhooks |
| **A3. Set up target** | Identity/access on GitHub; org/team structure; governance & policy settings; security assessment of the target |
| **A4. Test** | Define success metrics; pick migration method per repo; fix repo-data issues first (large files, history, LFS); **dry runs**; pilot migrations + pilot-user feedback; failure/revert plan documented |
| **A5. Migrate** | Freeze source changes; dedicated support channel; execute runbook; track & fix errors; link identity attribution (for importer) |
| **A6. Post-migrate** | Confirm visibility/permissions; validate migrated data; convert pipelines to GitHub Actions; verify CI/CD integrations; validate governance & compliance settings; update integrations; gather developer feedback; monitor; post-mortem; plan source deprecation |

For moving *content plus* issues/PRs, use the **GitHub Enterprise Importer**; for source-only content, plain Git works.

---

## Appendix B — Anti-patterns to avoid (Scenarios: Anti-patterns)

| Area | Anti-patterns the framework calls out |
|---|---|
| Development | Poor commit practices · inconsistent branching strategy · accumulating technical debt · overengineering |
| Collaboration | **Bypassing code reviews** · delayed feedback cycles |
| CI | Insufficient test automation (and: flaky tests that train people to ignore failures) |
| CD | Large releases · manual deployment processes |
| AppSec | Neglecting application security measures · using secret-scanning custom patterns to detect PII (compliance exposure — use DLP tooling for PII) |
| Platform/Org | Fragmented org structure · vague requirements · ineffective work management |

Each has a direct counter in this guide: good commit/branch conventions (1.6), fail-fast CI (4.1), required reviews (5.1, 7.4), small automated releases (6), and the full security stack (5).

---

## Sources (from [learn.github.com/well-architected](https://learn.github.com/well-architected))

- [Layers of GitHub Well-Architected](https://learn.github.com/well-architected/overview/layers) — pillars, design principles, checklists, recommendations; maturity model.
- [Getting Started Checklist](https://learn.github.com/well-architected/overview/getting-started-checklist) — the assessment flow condensed in step 8.3.
- [Collaboration — Design Principles](https://learn.github.com/well-architected/collaboration/design-principles) (communication, inclusivity, openness, transparency, flexibility, keep it simple) · [Checklist](https://learn.github.com/well-architected/collaboration/checklist)
- [Productivity — Design Principles](https://learn.github.com/well-architected/productivity/design-principles) (automation, integration, continuous learning, feedback, engineering-system success) · [Checklist](https://learn.github.com/well-architected/productivity/checklist) · [Effective CI](https://learn.github.com/well-architected/productivity/recommendations/effective-ci)
- [Application Security — Design Principles](https://learn.github.com/well-architected/application-security/design-principles) (security, compliance, proactivity, awareness) · [Checklist](https://learn.github.com/well-architected/application-security/checklist) · [Managing Dependency Threats](https://learn.github.com/well-architected/application-security/recommendations/managing-dependency-threats) · [Securing GitHub Actions](https://learn.github.com/well-architected/application-security/recommendations/actions-security)
- [Governance — Design Principles](https://learn.github.com/well-architected/governance/design-principles) (auditability, accountability, adaptability, control) · [Checklist](https://learn.github.com/well-architected/governance/checklist) · [Rulesets Best Practices](https://learn.github.com/well-architected/governance/recommendations/managing-repositories-at-scale/rulesets-best-practices) · [Custom Properties Best Practices](https://learn.github.com/well-architected/governance/recommendations/managing-repositories-at-scale/custom-properties-best-practices)
- [Architecture — Design Principles](https://learn.github.com/well-architected/architecture/design-principles) (scalability, resiliency, efficiency, disaster recovery, modularity, interoperability, observability) · [Checklist](https://learn.github.com/well-architected/architecture/checklist) · [Repository Architecture Strategy](https://learn.github.com/well-architected/architecture/recommendations/scaling-git-repositories/repository-architecture-strategy) · [Managing Large Git Repositories](https://learn.github.com/well-architected/architecture/recommendations/scaling-git-repositories/large-git-repositories) · [When to Use Git LFS](https://learn.github.com/well-architected/architecture/recommendations/scaling-git-repositories/when-to-use-git-lfs) · [Implementing Polyrepo Engineering](https://learn.github.com/well-architected/architecture/recommendations/implementing-polyrepo-engineering)
- [Scenarios: Anti-patterns](https://learn.github.com/well-architected/scenarios/anti-patterns) · [Repository Migration Checklist](https://learn.github.com/well-architected/scenarios/migrations/repository-checklist) · [Monorepos](https://learn.github.com/well-architected/scenarios/monorepos) · [Implementing the NIST SSDF with GitHub](https://learn.github.com/well-architected/scenarios/nist-ssdf-implementation)
