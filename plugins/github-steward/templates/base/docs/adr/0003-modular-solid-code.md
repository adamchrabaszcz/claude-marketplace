# 0003. Modular code that follows SOLID

- Status: Accepted
- Date: {{DATE}}

## Context
Code is easiest to change and test while each piece is small and does one thing. Without a stated standard, files and functions grow one feature at a time, settings get hard-coded where they're used, the same shape gets written in two places, and I/O ends up mixed into the logic, until a change needs a refactor first.

## Decision
Code follows SOLID, applied to whatever the language uses (classes or plain functions):

- **Single responsibility:** a module, class or function does one job. When it starts doing several (parsing and deciding and formatting), split it into named steps.
- **Open/closed:** rules and thresholds that will be tuned live in one settings object or config file, not scattered through `if` chains; adding a case shouldn't mean editing unrelated code.
- **Liskov substitution:** anything passed in place of an interface behaves like it, including in tests (fakes keep the real contract).
- **Interface segregation:** a piece takes only the input it needs, not a large shared object, so it can be used and tested on its own.
- **Dependency inversion:** I/O, storage, time, randomness and platform APIs sit at the edges behind small interfaces and are passed in; the core logic doesn't reach out for them. Settings come in as a parameter with defaults.

Two habits follow from it:

- **One source of truth per shape:** a type, a schema and a config format aren't written out twice; one is generated from or checks the other.
- **Files don't keep growing:** when a file or function passes a screenful, or a change has to touch a long one, split it first.

## Consequences
- Each piece can be tested on its own, with fakes at the edges and fixed inputs.
- Reviews point at a module doing too much, a hard-coded setting or a duplicated shape, and those get fixed in the pull request that caused them.
- Some small indirection (a settings parameter, an interface) appears before it's strictly needed; that's the price of not refactoring later.
