# 0002. Plugins follow main: no version numbers, no releases

- Status: Accepted
- Date: 2026-11-24

## Context
Claude Code decides whether an installed plugin has an update from its version. [Its docs](https://code.claude.com/docs/en/plugins-reference) give two ways:

- With a `version` in `plugin.json` (or the marketplace entry), users stay on that version until the string changes, however many commits are pushed. A release means bumping it.
- Without one, users track commits. For a plugin at a relative path inside a Git-hosted marketplace, like ours, [the version is the commit SHA of the installed directory](https://code.claude.com/docs/en/plugins/marketplace-reference): the last commit that changed that plugin's folder.

Both plugins here had a fixed `version` that was never bumped, so a day of fixes to `github-steward` reached nobody through `/plugin update`. A version in every pull request is easy to forget, and the repository-wide `release` action (#28) can't move per-plugin versions anyway (#30).

## Decision
No `version` in any plugin's `plugin.json` or marketplace entry. Every merge to `main` that changes a plugin's folder is that plugin's next version, delivered by `/plugin update` or auto-update. There are no repository releases or tags.

## Consequences
- A fix reaches users on their next update after the merge; nothing to remember.
- A change to one plugin doesn't move another's version, since each follows its own folder.
- `main` is what ships, so it only changes through pull requests that pass `claude plugin validate`, the format check and the review (the `main` ruleset).
- `claude plugin validate` warns "No version specified" for each plugin. That's general advice, not an error (it still passes); the warning is expected here and isn't fixed by adding a version.
- Users can't pin an older version. A plugin that needs that can add `version` back, superseding this record for it.
- No release notes: each pull request and its issue say what changed. Per-plugin tags with notes (the shared `release` action limited to a plugin's folder, with a tag prefix) can be added later without changing how updates work.
