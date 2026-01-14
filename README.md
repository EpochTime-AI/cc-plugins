# Epochtime AI - Claude Code Plugins

> Official Claude Code plugin marketplace by **Epochtime AI**

A curated collection of Claude Code plugins to enhance your AI-powered development workflow.

## 🚀 Quick Start

**Add marketplace to Claude Code:**

```bash
# Via owner/repo format (recommended)
claude plugin marketplace add EpochTime-AI/cc-plugins

# Or via HTTPS URL
claude plugin marketplace add https://github.com/EpochTime-AI/cc-plugins.git
```

**Install plugins:**

```bash
# List available plugins
claude plugin marketplace list

# Install doc-crawler plugin
claude plugin install doc-crawler-plugin

# Test it
/doc-crawler
```

**Manual installation:**

```bash
git clone https://github.com/EpochTime-AI/cc-plugins.git
cd cc-plugins
cp -r doc-crawler-plugin ~/.claude/plugins/
```

## 📦 Available Plugins

### Doc Crawler Plugin

Systematic documentation crawler that converts online documentation into well-structured Claude Code skills.

**Features:**
- Automated documentation crawling with sitemap support
- Smart URL filtering (docs only, excludes blogs/marketing)
- Batch processing with configurable delays
- Quality guidelines for concise skills
- Automated cleanup

**Installation:**
```bash
# From marketplace
claude plugin install doc-crawler-plugin

# Or manually
git clone https://github.com/EpochTime-AI/cc-plugins.git
cp -r cc-plugins/doc-crawler-plugin ~/.claude/plugins/
```

**Usage:**
```bash
# Use the skill
/doc-crawler

# Follow the guided 8-step workflow:
# 1. Check inform installation
# 2. Try direct crawl
# 3. Discover sitemap URLs
# 4. Filter documentation URLs
# 5. Batch crawl
# 6. Organize content
# 7. Create skill file
# 8. Clean up repository
```

[View Documentation](./doc-crawler-plugin/README.md)

---

## 📖 Plugin Structure

Each plugin follows the standard Claude Code structure:

```
plugin-name/
├── plugin.json          # Plugin manifest
├── README.md            # Documentation
├── skills/              # Skills (.md files)
├── commands/            # Commands (optional)
├── agents/              # Agents (optional)
└── hooks/               # Hooks (optional)
```

## 🤝 Contributing

We welcome contributions! To add a plugin:

1. Read [CONTRIBUTING.md](.github/CONTRIBUTING.md) for guidelines
2. Fork this repository
3. Create your plugin following the standard structure
4. Validate with `claude plugin validate ./your-plugin`
5. Update `.claude-plugin/marketplace.json`
6. Submit a pull request

**Resources:**
- [CONTRIBUTING.md](.github/CONTRIBUTING.md) - Detailed contribution guide
- [CLAUDE.md](./CLAUDE.md) - Plugin development guide
- Use `/plugin-dev:create-plugin` in Claude Code for guided creation

## 📊 Statistics

- **Total Plugins**: 1
- **Total Skills**: 1
- **Total Commands**: 0
- **Total Agents**: 0

## 📄 License

MIT License - See [LICENSE](./LICENSE) file for details.

## 🔗 Resources

- [Claude Code Documentation](https://docs.anthropic.com/claude/docs)
- [Plugin Development Guide](https://github.com/anthropics/claude-code)

---

**Maintained by Epochtime AI** | Last Updated: 2026-01-15
