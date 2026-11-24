# Security

## Reporting a vulnerability

Please don't open a public issue. Report it privately through GitHub: **Security → Report a vulnerability** on this repository. Include what you found, how to reproduce it, and which version you use.

You'll get an answer within a week. Fixes are released as a patch version and credited in the release notes unless you prefer otherwise.

## How the repository is kept safe

- No secrets in the repository. CI's tokens, including `CLAUDE_CODE_OAUTH_TOKEN` for the AI review, are repository secrets.
- GitHub Actions are pinned to commit SHAs; Dependabot proposes updates and security fixes.
- The AI review can comment but not push, and only the owner, members and collaborators can start it with `@claude`. Workflows from forks wait for a maintainer's approval.

## Supported versions

Only the latest release gets fixes.
