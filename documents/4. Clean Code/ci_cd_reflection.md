# CI/CD Reflection: Static Analysis Checks

## What is the purpose of CI/CD?

**Continuous Integration (CI)** and **Continuous Deployment/Delivery
(CD)** automate and streamline the software development lifecycle.
Their core purpose is to enable teams to deliver changes **frequently**
and **reliably** whilst maintaining quality.

- **CI** means frequently committing code to a shared repository and
  automatically **building** and **testing** each change. This surfaces
  integration issues early, reduces debugging time, and makes it easier to
  merge work from multiple contributors. CI answers: *"Does this change break
  the build or tests?"*
- **CD** extends this by automating the **release** process. Code that passes
  CI can be deployed to staging or production quickly and consistently. Some
  teams ship many times per day. CD answers: *"Can we release this change
  safely?"*
Together, CI/CD moves quality checks and deployment from manual,
error‑prone steps into repeatable, automated pipelines. This supports faster
feedback, fewer production incidents, and a more predictable delivery cadence.

---

## How does automating style checks improve project quality?

Automating style checks (e.g. Markdown linting, spell checking) in CI/CD
improves quality in several ways:

1. **Consistency**: Linters and spell checkers enforce formatting and writing
   standards across all contributors. Documentation and prose stay uniform
   without relying on memory or manual review.
2. **Catching issues before review**: Problems like broken heading structure,
   trailing spaces, or typos are flagged by automation instead of human
   reviewers. Reviewers can focus on content, structure, and clarity.
3. **No "it works on my machine"**: Checks run in a clean, consistent
   environment (e.g. GitHub Actions). Everyone gets the same rules and
   results, independent of local setup.
4. **Lower barrier to contribution**: New contributors get immediate, neutral
   feedback from the pipeline. They can fix style issues before requesting
   review, which speeds up iteration and reduces back‑and‑forth.
5. **Documentation as code**: Markdown and docs are treated like source code:
   they are checked, versioned, and improved over time. This raises the
   baseline quality of project documentation.

---

## What are some challenges with enforcing checks in CI/CD?

1. **False positives and noise**: Linters and spell checkers can flag valid
   names, technical terms, or intentional patterns. Teams must tune configs
   (e.g. `cspell` word lists, markdownlint rule overrides) to reduce noise
   whilst keeping useful checks.
2. **Slower feedback loops**: CI jobs take time. If workflows are slow or
   flaky, developers may ignore or work around them. Optimising job duration
   and reliability is important.
3. **Configuration drift**: Different tools (local editors, pre‑commit hooks,
   CI) can use different configs. Keeping `.markdownlint.json`, `cspell.json`,
   and editor settings aligned requires discipline and occasional updates.
4. **Friction for contributors**: Strict checks can frustrate contributors who
   are unfamiliar with the tools or who disagree with the rules. Clear
   documentation, sensible defaults, and a way to add exceptions (e.g.
   dictionary words) help.
5. **Maintenance overhead**: Actions, linter versions, and configs need
   updates. Pipelines can break when dependencies change, so someone must own
   and maintain the CI setup.
6. **Branch vs. main**: Checks on PRs only run when a PR exists. Pushes
   directly to `main` may bypass PR checks unless branch protection rules
   enforce them, which requires repo admin configuration.

---

## How do CI/CD pipelines differ between small projects and large teams?

<!-- markdownlint-disable MD013 -->
| Aspect | Small projects | Large teams |
| ------ | -------------- | ----------- |
| **Scope** | Often a single app or repo; few services. | Many repos, microservices, shared libraries; orchestration across teams. |
| **Complexity** | Simple pipelines: lint → test → (optional) deploy. Few stages, minimal branching. | Multi‑stage pipelines: build, test, security scans, staging, canaries, production. Many environments and approval steps. |
| **Tooling** | GitHub Actions, simple YAML; maybe a single workflow file. | Dedicated CI/CD platforms (e.g. Jenkins, GitLab CI, Terraform); configuration as code; shared templates and custom tooling. |
| **Enforcement** | Relies on CI status; may use branch protection loosely. | Strict branch protection, required status checks, code owners, compliance and audit requirements. |
| **Ownership** | One or a few people own the pipeline and config. | Dedicated platform or DevOps teams; each team may own their pipelines but follow org standards. |
| **Speed vs. safety** | Favours speed; rapid iterations, direct deploys. | Favours safety; more gates, reviews, and rollback procedures; deployment windows and change management. |
<!-- markdownlint-enable MD013 -->

In small projects, CI/CD is often minimal and focused on "don't break the
build." In large teams, it becomes part of governance, security, and release
management, with more process and tooling to coordinate many people and
systems.

---

## Summary of what was set up

- **CI workflow**
  ([Static-Analysis-Checks-in-CI-CD-Test](https://github.com/yjfvictor/Static-Analysis-Checks-in-CI-CD-Test)):
  GitHub Actions workflow that runs **Markdown lint** (markdownlint‑cli2)
  and **spell check** (cspell, en‑GB) on all `**/*.md` files on push and pull
  request to `main`.
- **Config files**: `.markdownlint.json` (e.g. MD041 disabled), `cspell.json`
  (en‑GB, project word list).
- **Pre‑commit hooks**: Husky + lint‑staged run markdownlint and cspell on
  staged `.md` files before each commit.

**Expected results:**

- ✅ Markdown Lint job: Should pass (`README.md` follows markdownlint rules)
- ✅ Spell Check job: Should pass (no spelling errors found)
- Both jobs typically complete in 10–30 seconds

The workflow runs automatically when the PR is created because it triggers on
`pull_request` events targeting `main`.
