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

## Structure

```
claude-marketplace/
├── plugins/
│   └── claude-symfony-plugin/    # Symfony 7.x development plugin
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
