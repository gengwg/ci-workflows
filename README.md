# ci-workflows

Reusable GitHub Actions workflows.

## ocr-review

AI review of pull requests with [Alibaba Open Code Review](https://github.com/alibaba/open-code-review). It posts inline comments on the changed lines and one summary comment that is updated in place on each push. The write-up is [Open Code Review on GitHub Actions](https://gengwg.substack.com/p/open-code-review-on-github-actions).

Add `.github/workflows/ocr-review.yml` to a repo:

```yaml
name: ocr-review

on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]

permissions:
  contents: read
  pull-requests: write

jobs:
  review:
    uses: gengwg/ci-workflows/.github/workflows/ocr-review.yml@main
    with:
      llm_url: ${{ vars.OCR_LLM_URL }}
      llm_model: ${{ vars.OCR_LLM_MODEL }}
    secrets:
      llm_auth_token: ${{ secrets.OCR_LLM_AUTH_TOKEN }}
```

Then set the repository variables `OCR_LLM_URL` (an OpenAI-compatible `/v1/chat/completions` endpoint) and `OCR_LLM_MODEL`, and the secret `OCR_LLM_AUTH_TOKEN`.

What it does:

- Reviews same-repo, non-draft PRs. Fork and Dependabot PRs skip, because they get no secrets. Marking a draft ready starts a review.
- A new push cancels the running review for that PR.
- The job goes red if the review fails or the summary comment never posts. When a review fails, times out or is cancelled, the summary is rewritten to say the current commit has no review, so an earlier all-clear never stays up.

Callers use `@main`, so fixes here reach every repo. PRs to this repo review themselves with the PR's copy of the workflow (`self-review.yml`) before a change reaches other repos.
