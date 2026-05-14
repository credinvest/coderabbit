# credinvest CodeRabbit central configuration

This repo is the canonical source of truth for [CodeRabbit](https://www.coderabbit.ai/) configuration across the entire `credinvest` GitHub organization.

CodeRabbit's documented [Central Configuration](https://docs.coderabbit.ai/configuration/central-configuration) feature applies the `.coderabbit.yaml` in this repo to every `credinvest/*` repository that does NOT have its own `.coderabbit.yaml` override.

## Resolution order

When CodeRabbit reviews a PR, it picks configuration from the first source in this list that supplies a value (per [CodeRabbit's docs](https://docs.coderabbit.ai/configuration/central-configuration#how-configuration-resolution-works)):

| Priority | Source | Location |
| --- | --- | --- |
| 0 (highest) | Global overrides | CodeRabbit UI → Organization Settings → Global Overrides |
| 1 | Repository file | `<repo>/.coderabbit.yaml` in the PR's repo |
| 2 | **Central repository** | **`credinvest/coderabbit/.coderabbit.yaml` ← this repo** |
| 3 | Repository UI settings | CodeRabbit UI → Repository Settings |
| 4 | Organization UI settings | CodeRabbit UI → Organization Settings |
| 5 (lowest) | CodeRabbit defaults | per [Configuration reference](https://docs.coderabbit.ai/reference/configuration) |

## Current policy

See [`.coderabbit.yaml`](./.coderabbit.yaml) for the full policy. Summary:

- `reviews.auto_review.enabled: true` — CodeRabbit reviews every qualifying PR automatically.
- `reviews.auto_review.drafts: false` — **skip draft PRs.** Draft PRs are by definition not ready for review. The 2026-05-13 workspace rule (`Fan-out implementation PRs open in draft + must declare cross-PR dependencies`) explicitly marks fan-out implementation PRs as draft until `cred-author-agent[bot]` validates them. The internal cred-\* reviewer fleet (`cred-architecture-review.yml`, `cred-qa-review.yml`, `cred-test-agent.yml`, `cred-config-agent.yml` — all org-injected via ruleset `15444553`) already gates on `github.event.pull_request.draft == false`; this central config matches that policy.
- `reviews.auto_review.ignore_usernames: []` — **review PRs from every author, including `cred-author-agent[bot]`.** The internal cred-\* reviewer fleet excludes bot authors by design (cf. `cred-architecture-review.yml:212` / `cred-qa-review.yml:257`) because those agents coordinate with each other through the merge-agent loop. CodeRabbit is the **third-party external eye** that catches diffs the internal fleet skips — e.g. the 2026-05-12 cred-web-commercial#18695 silent ~5000-line deletion in a small-scope conflict resolution that no internal reviewer flagged.

## How to update the policy

1. Open a PR against this repo's default branch (`main`) modifying `.coderabbit.yaml`.
2. CodeRabbit will review the policy change itself (this repo has the CodeRabbit App installed too).
3. Merge once approved. The change applies to every consuming `credinvest/*` repo on the next PR they receive.

## Per-repo overrides

Individual repositories can override central configuration by adding their own `.coderabbit.yaml` file at their repo root. The repo-local file takes CodeRabbit priority 1, which beats this central config (priority 2). Repo-local examples in the org:

- `cred-web-commercial/.coderabbit.yaml` — overrides with repo-specific `path_instructions` for `.tsx` files (preserves the styling-intent guardrails) while inheriting `drafts: false` / `ignore_usernames: []` from this central config.
- `plan-and-design-repo/.coderabbit.yaml` — disables CodeRabbit entirely on that repo (`auto_review.enabled: false`, `commit_status: false`) because the plan-and-design pipeline is reviewed by `cred-plan-review-agent` / `cred-design-review-agent` / `cred-architecture-agent` — CodeRabbit commentary on plain markdown plan/design files adds noise without value.

## Related agent-policy docs

- **`cred-agent-ai/agent-ops/coderabbit-policy.md`** — workspace-side pointer to this repo, mirrors the `cursor-bugbot-policy.yaml` precedent for centralizing third-party reviewer policy under `cred-agent-ai`.
- **`cred-agent-ai/agent-ops/cursor-bugbot-policy.yaml`** — the analogous policy for Cursor Bugbot (currently `managed: true` / `enabled: false` org-wide; Bugbot was disabled 2026-05-07 because the internal cred-\* reviewer fleet supersedes it).

## CodeRabbit App installation

For this central config to take effect, the CodeRabbit GitHub App **must** be installed on this `credinvest/coderabbit` repo — CodeRabbit needs read access to load the file. Verify in [CodeRabbit's dashboard](https://app.coderabbit.ai/) under the credinvest org. If you observe a PR in a credinvest/\* repo where CodeRabbit's review comment header does NOT show `Repository: coderabbit/.coderabbit.yaml`, check the App installation first.

## Tickets

- [COM-39674](https://linear.app/credplatform/issue/COM-39674) — parent epic, centralize CodeRabbit config.
- [COM-39675](https://linear.app/credplatform/issue/COM-39675) — this repo (create central config).
- [COM-39676](https://linear.app/credplatform/issue/COM-39676) — fix `.coderabbitai.yaml` typo in `cred-web-commercial`.
- [COM-39677](https://linear.app/credplatform/issue/COM-39677) — document policy in `cred-agent-ai/agent-ops/`.
