# Epochtime AI - Claude Code Plugins

> Official Claude Code plugin marketplace by **Epochtime AI**

A curated collection of Claude Code plugins to enhance your AI-powered development workflow.

## 🚀 Quick Start

```bash
# Install from this repository
claude code plugin install https://github.com/epochtime-ai/cc-plugins/doc-crawler-plugin

# Or clone and install locally
git clone https://github.com/epochtime-ai/cc-plugins.git
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
claude code plugin install https://github.com/epochtime-ai/cc-plugins/doc-crawler-plugin
```

**Usage:**
```bash
/doc-crawler
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

1. Fork this repository
2. Create your plugin following the standard structure
3. Add documentation
4. Submit a pull request

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
