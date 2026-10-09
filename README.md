# GitHub Actions Workflow Security Gate demo

Synthetic GitHub Actions workflows used to demonstrate the Kestra **GitHub Actions Workflow Security Gate** blueprint (`github-actions-workflow-security-gate`). The blueprint scans every workflow at two commits with [zizmor](https://docs.zizmor.sh), offline, and blocks only the findings the second commit introduces.

> **GitHub Actions is disabled on this repository.** Some workflows here are deliberately unsafe (the `greet` workflow expands an issue title into a shell script). They exist only to be scanned. Do not copy them, and do not enable Actions on a fork.

## Commits

| Commit | Change |
| --- | --- |
| `7239b08f60ff7618aa1a0ca44595c89cb87a8369` | README only, no workflows |
| `8fa1d8e1c6041248563513a84d07bd5889e2e82c` | `ci.yml`: hardened CI workflow (no permissions by default, `actions/checkout` pinned to a commit, credentials not persisted) |
| `43f64722bacb90b8fa1c65bf99522474f2b50a48` | `ci.yml` adds a lint step using `example-org/lint-action@v1`, an action pinned to a tag instead of a commit |
| `e28ccb0d2447f900e47cbf53899e006dc3f60b05` | adds `greet.yml`, which runs on new issues and puts `${{ github.event.issue.title }}` straight into a shell script |
| `cc9759b83b995deebb768e6eadf8b1609f478f0e` | adds two comment lines at the top of `greet.yml`, nothing else |
| `893ceea45dc2717c8614a0f8ed9a9f3ac9912b1b` (branch `broken-workflow`) | from the hardened CI commit, adds `release.yml`, which is not valid YAML |

`example-org/lint-action` does not exist; it only stands for any third-party action.

## Expected verdicts

Inputs: `repo_url: https://github.com/AyushVUpadhye/kestra-github-actions-security-gate-demo`, default thresholds (severity and confidence at least `medium`).

| `base_commit` | `candidate_commit` | Verdict | Why |
| --- | --- | --- | --- |
| `e28ccb0d2447f900e47cbf53899e006dc3f60b05` | `cc9759b83b995deebb768e6eadf8b1609f478f0e` | WARN: `EXISTING_FINDINGS` | both findings already existed; they only moved down two lines (the blueprint's default inputs) |
| `7239b08f60ff7618aa1a0ca44595c89cb87a8369` | `8fa1d8e1c6041248563513a84d07bd5889e2e82c` | PASS | the new workflow has no finding at or above the thresholds |
| `8fa1d8e1c6041248563513a84d07bd5889e2e82c` | `43f64722bacb90b8fa1c65bf99522474f2b50a48` | BLOCK: `NEW_FINDINGS` | new `unpinned-uses` finding |
| `43f64722bacb90b8fa1c65bf99522474f2b50a48` | `e28ccb0d2447f900e47cbf53899e006dc3f60b05` | BLOCK: `NEW_FINDINGS` | new `template-injection` finding; the `unpinned-uses` finding is reported as existing |
| `8fa1d8e1c6041248563513a84d07bd5889e2e82c` | `893ceea45dc2717c8614a0f8ed9a9f3ac9912b1b` | INCONCLUSIVE: `WORKFLOW_UNPARSEABLE` | `release.yml` cannot be parsed, so the scan is incomplete |

Use the full commit SHAs; the blueprint does not accept branch or tag names.
