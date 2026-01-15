# Claude Code Documentation Plugin

> Quick access to official Claude Code documentation within your development workflow

## Overview

The `claude-code-doc-plugin` provides instant access to all official Claude Code documentation. This plugin includes a comprehensive skill that gives Claude quick reference to setup guides, plugin development docs, configuration options, and troubleshooting resources.

## Features

- **Complete Documentation Index** - All Claude Code docs organized by category
- **Quick Links** - Fast access to most commonly used documentation
- **Full Documentation** - Complete markdown docs available in references/
- **Categorized Reference** - Docs grouped by topic (setup, plugins, enterprise, etc.)
- **Usage Examples** - Common documentation lookup patterns

## Installation

### From Marketplace

```bash
# Add the Epochtime AI marketplace
claude plugin marketplace add EpochTime-AI/cc-plugins

# Install the plugin
claude plugin install claude-code-doc-plugin
```

### Manual Installation

```bash
# Clone the repository
git clone https://github.com/EpochTime-AI/cc-plugins.git

# Install the plugin
cd cc-plugins
claude plugin install ./claude-code-doc-plugin
```

## Usage

Once installed, Claude will automatically have access to the Claude Code documentation through the `/claude-code-doc` skill. The skill is triggered when you ask questions about:

- Claude Code features and capabilities
- Setup and installation
- Plugin development
- Configuration and settings
- MCP integration
- Hooks and subagents
- IDE integration (VS Code, JetBrains)
- Enterprise deployment
- Troubleshooting

### Example Queries

```
How do I set up Claude Code?
→ Claude references: Setup, Quickstart

How do I create a plugin?
→ Claude references: Plugins, Plugins Reference, Skills, Hooks

How do I configure Claude Code for enterprise?
→ Claude references: IAM, Network Config, Security, Monitoring

What MCP servers can I use?
→ Claude references: MCP, Plugins

How do I use Claude Code in VS Code?
→ Claude references: VS Code, Interactive Mode, Settings
```

## Documentation Structure

```
claude-code-doc/
├── SKILL.md                    # Quick reference (5.2KB)
└── references/                 # Full documentation (808KB)
    ├── quickstart.md
    ├── setup.md
    ├── plugins.md
    ├── skills.md
    ├── mcp.md
    └── ... (48 total docs)
```

## Documentation Categories

The plugin organizes documentation into these categories:

- **Getting Started** - Overview, quickstart, setup, workflows
- **Core Features** - Skills, plugins, hooks, MCP, subagents
- **IDE Integration** - VS Code, JetBrains, Chrome
- **Configuration** - Settings, models, network, terminal
- **Interactive Features** - Keyboard shortcuts, commands, memory
- **Advanced Features** - Checkpointing, sandboxing, cost management
- **Deployment** - Desktop, web, headless, containers
- **Cloud Providers** - AWS, GCP, Azure
- **CI/CD** - GitHub Actions, GitLab CI/CD
- **Enterprise** - IAM, analytics, security, compliance

## What's Included

All 48 official Claude Code documentation pages:

- amazon-bedrock.md
- analytics.md
- checkpointing.md
- chrome.md
- claude-code-on-the-web.md
- cli-reference.md
- common-workflows.md
- costs.md
- data-usage.md
- desktop.md
- devcontainer.md
- discover-plugins.md
- github-actions.md
- gitlab-ci-cd.md
- google-vertex-ai.md
- headless.md
- hooks.md
- hooks-guide.md
- iam.md
- interactive-mode.md
- jetbrains.md
- legal-and-compliance.md
- llm-gateway.md
- mcp.md
- memory.md
- microsoft-foundry.md
- model-config.md
- monitoring-usage.md
- network-config.md
- output-styles.md
- overview.md
- plugin-marketplaces.md
- plugins.md
- plugins-reference.md
- quickstart.md
- sandboxing.md
- security.md
- settings.md
- setup.md
- skills.md
- slack.md
- slash-commands.md
- statusline.md
- sub-agents.md
- terminal-config.md
- third-party-integrations.md
- troubleshooting.md
- vs-code.md

## Benefits

1. **No Context Switching** - Claude can reference docs without you leaving your workflow
2. **Accurate Guidance** - Claude uses official documentation for correct information
3. **Time Saving** - No need to manually search docs for common questions
4. **Always Updated** - Plugin includes complete documentation snapshot
5. **Comprehensive Coverage** - All official docs in one place

## Version

- **Plugin Version**: 1.0.0
- **Documentation Source**: https://code.claude.com/docs/
- **Last Updated**: 2026-01-15

## Author

Epochtime AI

## License

MIT License - See repository LICENSE file

## Support

For issues or questions:
- GitHub Issues: https://github.com/EpochTime-AI/cc-plugins/issues
- Documentation: https://code.claude.com/docs/

## Related Plugins

- **doc-crawler-plugin** - Create documentation skills from any website
- **plugin-dev** - Official Claude Code plugin development tools

---

Made with Claude Code
