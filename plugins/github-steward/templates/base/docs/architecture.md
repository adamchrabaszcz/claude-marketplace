# Architecture

How {{PROJECT}} is put together. Why it's built this way is in `docs/adr/`.

Keep this to what a newcomer needs before changing something: the parts, where each lives, how data flows between them. Update it in the pull request that changes the structure.

## Layout

| Path | What |
| --- | --- |
| `.github/workflows/` | CI, the format check, releases and the Claude review |
| `docs/` | This file and the decision records |
