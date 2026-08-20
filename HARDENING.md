<!-- markdownlint-disable -->

# Hardening Report: nick-fields--retry/v4.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **nick-fields--retry/v4.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow uses multiple actions pinned to mutable version tags instead of immutable 40-character SHA digests, making the workflow vulnerable to supply-chain attacks if any of those tags are moved. Failing references include: `actions/checkout@v4`, `actions/setup-node@v4`, `codecov/codecov-action@v5`, `nick-fields/assert-action@v2`, and `cycjimmy/semantic-release-action@v4`. Each should be replaced with a full commit SHA (e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`).

Locations:

- `.github/workflows/ci_cd.yml:17`
- `.github/workflows/ci_cd.yml:19`
- `.github/workflows/ci_cd.yml:25`
- `.github/workflows/ci_cd.yml:35`
- `.github/workflows/ci_cd.yml:37`
- `.github/workflows/ci_cd.yml:43`

### permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` block on any of its jobs. Without explicit permissions, the GITHUB_TOKEN is granted its default (often broad) permissions. A minimal permissions block (e.g. `permissions: read-all` at the top level, then elevated only where needed) should be added.

Locations:

- `.github/workflows/ci_cd.yml:1`

### script-injection (severity: high)

Sub-rule (b) violation: The `Tag` step in the `cd` job expands `${MAJOR_VERSION}` unquoted inside a `run:` shell command. `MAJOR_VERSION` is sourced from `${{ steps.semantic.outputs.new_release_major_version }}` (a `steps.*.outputs.*` value, which is workflow-controllable). An attacker who can influence the semantic-release output could inject shell metacharacters. The offending line is: `run: git tag -f v${MAJOR_VERSION} && git push -f origin v${MAJOR_VERSION}`. Fix: quote the variable — `"v${MAJOR_VERSION}"`.

Locations:

- `.github/workflows/ci_cd.yml:461`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings in .github/workflows/ci_cd.yml: (1) Pinned all 5 unpinned action references to full 40-char SHAs: actions/checkout@11d5960a326750d5838078e36cf38b85af677262, actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020, codecov/codecov-action@0fb7174895f61a3b6b78fc075e0cd60383518dac, nick-fields/assert-action@aa0067e01f0f6545c31755d6ca128c5a3a14f6bf, cycjimmy/semantic-release-action@16ca923e6ccbb50770c415a0ccd43709a8c5f7a4. (2) Added top-level `permissions: {}` and job-level permissions blocks: `contents: read` for all CI jobs, `contents: write` for the cd job (needs to push tags), `permissions: {}` for ci_all_tests_passed. (3) Fixed script injection in the Tag step by quoting MAJOR_VERSION: changed `v${MAJOR_VERSION}` to `"v${MAJOR_VERSION}"` in both the git tag and git push commands.

