# Claude Marketplace

A collection of Claude Code plugins for various frameworks and tools.

## Available Plugins

### claude-symfony-plugin

A comprehensive Claude Code plugin for Symfony 7.x development featuring:

- **symfony-7-developer agent**: An expert Symfony developer agent with deep knowledge of Symfony 7.x architecture, Doctrine ORM, testing, and best practices

**Installation:**
```bash
claude /install-plugin https://github.com/adamchrabaszcz/claude-marketplace/plugins/claude-symfony-plugin
```

### github-steward

Sets GitHub repositories up and keeps work in them tidy. Language-agnostic; uses the `gh` CLI.

- **deliver** skill: the working loop for any change.
  - A plan first, approved by you.
  - An issue for every task, and one branch and one small PR per change, each from the latest `main` and mergeable on its own.
  - Checks and a self-review before committing; Conventional Commits; PRs that close their issue.
  - A test merge of all open PRs before reporting back.
- **onboard** skill (coming): checks a repository and sets up what's missing:
  - AGENTS.md, docs, CONTRIBUTING.md and README;
  - labels, sidebar and branch protection;
  - CI with coverage, and releases from Conventional Commits.

**Installation:**
```
/plugin marketplace add adamchrabaszcz/claude-marketplace
/plugin install github-steward@adams-claude-marketplace
```

## Structure

```
claude-marketplace/
├── plugins/
│   ├── claude-symfony-plugin/    # Symfony 7.x development plugin
│   └── github-steward/           # GitHub repo setup and workflow (skills)
│       ├── .claude-plugin/
│       │   └── plugin.json
│       └── agents/
│           └── symfony-7-developer.md
└── README.md
```

## Contributing

To add a new plugin:

1. Create a new directory under `plugins/`
2. Add a `.claude-plugin/plugin.json` manifest
3. Add your agents, skills, hooks, or commands
4. Submit a pull request

## License

MIT
