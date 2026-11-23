# OpenThunder for GitHub

One answer for every pull request: **Ready to merge**, **Needs review**, or **Blocked**, with the reason,
the next step, and the fix list. It is the same Check `openthunder check` prints in a terminal.

**Keyless.** No account and no AI key. The Check runs on your runner; your code does not leave it.

## Usage

```yaml
name: OpenThunder
on: [pull_request]

permissions:
  contents: read
  pull-requests: write     # the PR comment
  security-events: write   # findings in Code scanning (optional, see `upload`)

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0     # the Check compares with the base branch, so it needs history
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - uses: SkillsTechTalk/openthunder-action@v1
        with:
          gate: report       # report | block | require-ready
```

## What you get

- **One PR comment**, edited in place on every push: the answer, the reason, the next step, and the
  fix list (file:line, what must be true, the command that proves it). Hand the fix list to your
  coding agent; the comment counts the rounds until it is Ready.
- **The same answer in the job summary**, so it is visible even when a comment cannot be posted
  (pull requests from forks).
- **Findings in Code scanning** (SARIF), on the lines they are about.
- **A Definition of Done on every pull request**, decided from evidence for that exact change (see below).

## Definition of Done

Every pull request is checked against a Definition of Done: what your team requires before a change
is done. Each item is decided from evidence for that exact change (never a ticked checklist):
**Passed**, **Failed**, **Not checked**, **Outdated**, **Not applicable**, or **Exception approved**.

**Out of the box** (`definition-of-done: typical`, the default), a repository without its own file
gets this typical one:

| Required | What it means |
|---|---|
| No test weakened | No test was changed to prove less |
| No security finding on the changed lines | OpenThunder's scan of the lines this PR changes |
| Callers still work | Nothing still calls what the change removed or renamed |
| A person approved protected changes | Security-sensitive files (auth, payments, secrets) get a person's review |
| Nothing blocks the change, nothing waits for a person | Always part of it: the Check's own answer |

| Recommended (shown, never holds a change) | |
|---|---|
| Changed code has tests | Source changed with a test changed alongside |
| AI behavior change evaluated | Prompt, model, or tool changes are covered by an eval |

With `run-impacted-tests: true`, **Impacted tests ran and passed** is required too.

**Make it yours:** add `.openthunder/done.yml` to your default branch. It replaces the typical one,
and it is read from the base branch, so a pull request cannot change the rules that judge it.
Start from [the default](definition-of-done/typical.yml): every other option is already written
out, commented, with one line on what it does. Remove the `# ` in front of a line to turn it on:

```yaml
require:
  - check: openthunder.tests-not-weakened    # no test was changed to prove less
  - check: openthunder.no-security-findings  # no security finding on the changed lines
  - check: openthunder.contracts-intact      # nothing still calls what this change removed or renamed
  - check: openthunder.person-approval       # a person approves changes to security-sensitive files
  # - check: openthunder.impacted-tests      # the tests related to the change ran and passed (run with --run-tests)
  # - coverage: { min: 80, of: changed }     # changed lines covered, from your coverage report (lcov)
  # - tests                                  # your test report (JUnit): the tests ran and passed for this exact code
  # - security: { failOn: error }            # your scanner report (SARIF): no error-level finding on the changed files
```

The same default is what `openthunder done init` and the OpenThunder VS Code extension's
**Create Definition of Done** write.

An accepted risk is an exception with a person and an end date; the evidence underneath stays visible:

```yaml
exceptions:
  - requirement: coverage
    reason: "Legacy module; tests land in the follow-up"
    approver: your-github-username
    expires: 2026-12-31
```

With `gate: require-ready`, an open required item fails the check, so the Definition of Done becomes
a merge gate. Set `definition-of-done: none` to use only your own file.

## Enforcing it

| `gate` | The check fails when |
|---|---|
| `report` (default) | never; start here to see what it would do |
| `block` | the answer is Blocked |
| `require-ready` | the answer is anything other than Ready to merge |

Make the job a required status check in branch protection to enforce it. A repo admin or maintainer
who reviewed a Needs review change can approve that exact commit with a comment:
`/openthunder approve <commit> <what you checked>`. An approval never clears anything Blocked.

## Common inputs

| Input | Default | What it does |
|---|---|---|
| `gate` | `report` | See above. |
| `definition-of-done` | `typical` | `typical`: the typical Definition of Done when the repository has no `.openthunder/done.yml`. `none`: only your own file. |
| `base` | the PR base branch | The branch to compare with. |
| `run-impacted-tests` | `false` | Also run the tests related to the change. This runs your repository's own test commands: turn it on only for repositories you trust. |
| `see-it-run` | `false` | Start the app on the base and on the PR, replay the affected routes (GET/HEAD), and report what changed. |
| `upload` | `true` | Upload the findings to Code scanning (needs `security-events: write`). |
| `pr-comment` | `true` | Post the PR comment (needs `pull-requests: write`). |
| `cli-version` | `latest` | The `@skillstech/openthunder` version to run. Pin one for repeatable builds. |

Every input is described in [action.yml](action.yml).

## Outputs

`check` (`ready`, `review`, or `blocked`; empty when the Check could not run), `fix-list` (path to the
fix list JSON), and `rounds` (checked commits on this PR).

## What it is not

The Check reports what it examined and what it could not prove. It is not a promise that a change is
bug-free, and it does not replace code review.

## License

This action (the workflow wrapper in this repository) is Apache 2.0. It runs the OpenThunder CLI
(`@skillstech/openthunder`), which is free to use under its own license.

Learn more at https://openthunder.dev
