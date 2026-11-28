# 0002. Plans stay out of the repository

- Status: Accepted
- Date: {{DATE}}

## Context
Planning produces a lot of text: options, open questions, half-made decisions, a build order. Kept as a file in the repository (a `docs/plan.md`), it repeats what the decision records and the issues already say, and it goes stale as soon as the work moves on. Readers can't tell which of the three is true.

## Decision
Working plans live in `.plan/`, which git ignores. What comes out of a plan goes where it belongs:

- **decisions** that are costly to reverse into `docs/adr/`;
- **the roadmap and requirements** into GitHub issues, with the details (tables, rules, examples) in the issue that builds them;
- **structure** into `docs/architecture.md`;
- **how something behaves** into docs only once the code exists.

## Consequences
- Each fact has one home, and the review can check pull requests against it.
- Plans can be as rough as they need to be; nothing in them is binding until it's an ADR or an issue.
- A pull request never adds a plan file.
