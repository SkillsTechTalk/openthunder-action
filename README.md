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
- **Your Definition of Done.** If the repository has `.openthunder/done.yml`, each required item is
  decided from evidence for this exact change, and an open item keeps the answer at Needs review.

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
