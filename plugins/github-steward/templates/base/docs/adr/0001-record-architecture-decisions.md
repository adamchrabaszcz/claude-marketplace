# 0001. Record architecture decisions

- Status: Accepted
- Date: {{DATE}}

## Context
Some choices are costly to reverse or look odd from outside. Without the reasons written down, a contributor or the AI review would reasonably propose undoing them.

## Decision
Record each such decision in `docs/adr/` as a short record (context, decision, consequences), in the format in this folder's README.

## Consequences
- A change against a recorded decision is a conscious one: it says so and supersedes the record.
- The AI review checks pull requests against the records.
