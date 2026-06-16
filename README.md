# Callshop-Radio/.github

Org-wide GitHub defaults for [Callshop-Radio](https://github.com/Callshop-Radio).
Currently the base for **Dependabot** automation across all repos in the org.

Notifications run through Dependabot's built-in email channel (GitHub
Notifications → *Dependabot*). No Slack, no AI/OpenRouter integration —
those are intentionally out of scope here.

## Update strategy

Identical to the `backendforth/next-sanity-starter` setup the rest of the
toolchain was modelled on:

- **Monthly cadence** — first Monday of the month, 06:00 Europe/Berlin.
- **One catch-all group** (`all-non-major`) bundles every minor + patch
  bump across the workspace into a single PR per run.
- **Majors stay individual** so each one gets a human eye (e.g. Next
  16→17, React 19→20).
- **`@types/node` major bumps are ignored** — pinned to whatever Node
  LTS the repo runs on.
- **Security advisories bypass the schedule** and arrive immediately,
  independently of the monthly cadence.

Auto-merge runs only for **patch + minor** PRs and only after **all
required checks pass**. Anything else stays open until a human acts.

## Onboarding a repo

Per-repo files (Dependabot reads `dependabot.yml` from the default
branch only):

### `.github/dependabot.yml`

```yaml
version: 2

updates:
  - package-ecosystem: npm        # swap for the ecosystem in use
    directory: "/"
    schedule:
      interval: monthly
      day: monday
      time: "06:00"
      timezone: "Europe/Berlin"
    open-pull-requests-limit: 5
    groups:
      all-non-major:
        patterns:
          - "*"
        update-types:
          - "minor"
          - "patch"
    ignore:
      - dependency-name: "@types/node"
        update-types: ["version-update:semver-major"]
    labels:
      - "dependencies"
```

If a repo ships multiple parallel branches (à la `next-sanity-starter`'s
`main` + `variant/document-level`), duplicate the entry and set
`target-branch:` on the second one — keep both in lockstep.

### `.github/workflows/dependabot-auto-merge.yml`

Thin caller for the reusable workflow in this repo. **Pin to a commit
SHA** (not `@main`) so a compromised push to this repo cannot silently
change what runs in your CI:

```yaml
name: Dependabot auto-merge

on:
  pull_request:
    branches: [main]   # add other long-lived branches if relevant

permissions:
  contents: write
  pull-requests: write

jobs:
  auto-merge:
    uses: Callshop-Radio/.github/.github/workflows/dependabot-auto-merge.yml@<SHA>
    secrets: inherit
```

Replace `<SHA>` with the full commit hash from this repo's `main` after
any change to the reusable workflow. Bump deliberately — treat it like a
dependency update.

## Email notifications

No setup beyond GitHub defaults — Dependabot opens PRs as the
`dependabot[bot]` user, and anyone watching the repo (or with
*Notifications → Dependabot* enabled in their GitHub account) gets the
email. To audit who's reachable, check each repo's *Settings → Code
security → Dependabot alerts* recipients.
