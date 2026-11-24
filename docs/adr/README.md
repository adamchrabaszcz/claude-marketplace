# Architecture decision records

One file per decision that is costly to reverse, or that people will ask "why?" about. `docs/architecture.md` says what the code does; these say why.

- **Name:** `NNNN-decision-title.md`, numbered in order; numbers are never reused.
- **Never deleted.** A changed decision gets a new record, and the old one's status becomes `Superseded by NNNN`.
- **Short:** a page at most, written in the pull request that makes the decision.

## Template

```markdown
# NNNN. Decision title

- Status: Proposed | Accepted | Superseded by NNNN
- Date: YYYY-MM-DD

## Context
What forces the decision: the problem, the constraints, the options considered.

## Decision
What we do, in one or two sentences, then any detail.

## Consequences
What becomes easier, what becomes harder, what we now have to keep doing.
```

## Records

- [0001. Record architecture decisions](0001-record-architecture-decisions.md)
