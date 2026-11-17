# Milestone 0: charter and engineering governance

Date: 2026-11-17. Owner: coordinator. Scope authority: user's Prompt 1 and subsequent
repository instructions. Risk: documentation with consequential research interpretation;
separate evidence review required, no final architecture or protocol approval.

## Plan and acceptance

Inspect the existing workspace and destination; read the supplied paper in full;
document purpose, P01-P12 boundaries, permanent rules, execution/review process,
initial decisions, and architectural implications; validate documents and evidence;
commit on a feature branch and publish a PR to the verified repository; stop.

Acceptance: documents are consistent, all twelve capabilities are represented,
STAIR claims are qualified, implementation is explicitly deferred, source locations
are recorded, and checks/publication limitations are reported accurately.

## Initial inspection

The parent Research LAB repository had no commits or remote and contained unrelated
untracked lab/portfolio work. It was preserved. The user created STAIR_bot publicly
with README.md and an MIT license, initial commit
`a20f5c24fa6b29eb508c5638793ed10218a52b6e`. Clone inspection confirmed that origin is
`https://github.com/1SonOfGlory/STAIR_bot.git`. Work branch: `codex/project-charter`.

The initial clone failed using Windows Schannel credentials; a command-scoped OpenSSL
backend succeeded without disabling TLS verification. Bare `python`, `pdftotext`, and
`gh` were unavailable. Bundled Python/pypdf and Poppler supported extraction/rendering;
no packages were installed. All eleven PDF pages were read, including references;
page 8's table and plots were also visually inspected.

## Validation and review

- Local document validation: bundled Python check found seven nonempty Markdown files,
  ten resolving relative links, and all twelve requirement IDs; pass. An initial inline
  check failed on PowerShell quoting; the corrected file-based check executed successfully.
- `git diff --check`: pass before staging; staged diff checked again before commit.
- Source metrics and qualifications were checked against the extracted paper and its
  rendered page 8. No benchmark was run.
- M0-REVIEW, 2026-11-17: separate research-scientist agent `/root/charter_review`
  reviewed all seven Markdown artifacts against supplied extracted paper text and
  bounded primary-source checks. Disposition: pass for charter/evidence scope.
  Research-review SHA-256 at review:
  `21a3461afc98ccc16dce29b5f06de8e8c20c2e6f6cde60e9d812ee5b8cb9fb12`.
  No material objection. Low-severity finding: finalize validation/reviewer/publication
  records; response: record them here and in the decision log. Scientific review
  limitation: figure pixels, PDF byte identity, remote controls and publication were
  not independently verified. No final architecture or protocol was approved.
- GitHub confirmed creation of a classic protection rule for main, applying to one
  branch, after the human completed email verification. Configured: PR required,
  one approval, dismiss stale approvals, resolve conversations, no administrator
  bypass, force pushes disabled, deletions disabled. No CI status checks are required
  yet because no application/CI suite exists. Another eligible reviewer will be needed
  for the owner-authored PR; this internal review is not a GitHub approval.

Application tests and benchmarks are not applicable: no application was implemented.
No claim of production readiness follows from documentation checks.

## Remaining limitations and stopping condition

No selected framework/database/model provider, approved retrieval architecture,
benchmark reproduction, running application, deployment, or paid service use.
The supplied PDF has not been proven byte-identical to the hosted arXiv v1 artifact.
Requirements and measurable product targets remain a future human-approved milestone.
Stop after the charter documentation is delivered for review; do not begin that work.
