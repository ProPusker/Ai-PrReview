# Ai-PrReview

Ai-PrReview reviews GitHub pull requests for infrastructure/security issues using
AWS Bedrock. It fetches PR metadata, changed-file diffs and text file contents,
then writes a local report. Posting a review comment is opt-in.

This is a standalone Python CLI for reviewing GitHub PRs. It does not configure
GitHub Actions or publish a repository automatically.

It preserves the original tool's Bedrock-based review, editable standards,
local report, and optional PR comment. The original Tkinter GUI and
company-specific Confluence tagging-policy integration are intentionally not
included; put standards suitable for your public project in
`rules/default_rules.md` instead.

## Requirements

- Python 3.10+
- Access to the GitHub repository and its pull request
- AWS credentials with permission to invoke the selected Bedrock model
- A Bedrock model enabled for the selected AWS account and region

## Install

From the project root:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

## Keep credentials safe

Never commit access tokens, AWS credentials, `settings.json`, or real `.env`
files. This project does not store credentials in a settings file.

Recommended approaches:

- **Local GitHub access:** authenticate with GitHub CLI (`gh auth login`), then
  set `GITHUB_TOKEN` for the current PowerShell session with
  `$env:GITHUB_TOKEN = gh auth token`. The token is not written into this
  repository. Use a fine-grained token restricted to the target repository.
- **GitHub Actions or another CI system:** save the GitHub and AWS credentials
  in the platform's encrypted secret store, and expose them to the job only
  when needed. Do not place secrets in workflow YAML or repository files.
- **AWS:** prefer an AWS CLI profile/SSO session locally and the CI platform's
  short-lived role credentials in automation. boto3 uses the standard AWS
  credential chain. Avoid long-lived AWS access keys where possible.

A GitHub token is optional for reading a public repository, but recommended to
avoid low unauthenticated API limits. Private repositories require a token.
GitHub limits the pull-request files API to 300 changed files per PR; split
larger PRs to ensure the full changed-file list is reviewed.
For a fine-grained token, grant only the target repository:

- `Contents: read`
- `Pull requests: read`
- `Issues: write` only when using `--post-comment` (GitHub stores PR conversation
  comments through the Issues API)

For classic PATs, use the narrowest available private-repository scope. Never
paste a token into a command-line argument, source file, screenshot, issue, or
commit. If a credential is exposed, revoke/rotate it immediately.

## Configure AWS Bedrock

Use AWS SSO/profile or the standard AWS credential chain:

```powershell
aws sso login --profile my-profile
$env:AWS_PROFILE = "my-profile"
$env:AWS_REGION = "us-east-1"
$env:BEDROCK_MODEL_ID = "us.anthropic.claude-sonnet-4-6"
```

Optional environment variables:

- `BEDROCK_API_MODE`: `invoke_model` (default) or `converse`
- `AWS_DEFAULT_REGION`: fallback region if `AWS_REGION` is unset

Do not commit `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, or
`AWS_SESSION_TOKEN`. boto3 recognizes these standard variables when required,
but an SSO/profile or short-lived role is safer.

## Run a review

From this folder:

```powershell
$env:GITHUB_TOKEN = gh auth token
python github_pr_review.py --pr-url "https://github.com/owner/repository/pull/123"
```

The report is written to `PRReport/`. Rules are maintained in
`rules/default_rules.md`; pass `--rules-file` to use another rules file.

To post the review output as a PR comment, explicitly opt in:

```powershell
python github_pr_review.py --pr-url "https://github.com/owner/repository/pull/123" --post-comment
```

The token must have comment-write permission for that repository. Without the
flag, the tool only writes a local report.

## Public repository checklist

1. Publish this folder's contents as the root of the public `Ai-PrReview`
   repository.
2. Keep `.gitignore`; do not add tokens, local reports, caches, virtual
   environments, or credential files.
3. Review `rules/default_rules.md` and replace organization-specific standards
   before sharing.
4. Add a license file if you intend to grant reuse rights; no license is
   included by default.

The reviewer sends the pull request diff, PR description, commit messages, and
changed text-file contents to the configured Bedrock model. Do not use it on
repositories or data that are not approved for processing by that AWS account.
# Ai-PrReview
