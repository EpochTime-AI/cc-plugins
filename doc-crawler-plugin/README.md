# Doc Crawler Plugin

A Claude Code plugin for crawling and extracting documentation from websites using the inform tool.

## Overview

This plugin provides a systematic workflow for converting online documentation into well-structured Claude Code skills. It handles sitemap discovery, URL filtering, batch crawling, and content organization.

## Features

- **Systematic Documentation Crawling**: Step-by-step workflow from initial crawl to final skill creation
- **Sitemap Support**: Automatically discover and parse sitemap.xml files
- **Smart URL Filtering**: Include documentation pages, exclude blogs and marketing content
- **Batch Processing**: Crawl multiple URLs efficiently with configurable delays
- **Quality Guidelines**: Best practices for creating concise, practical skills
- **Repository Cleanup**: Automated cleanup of intermediate crawl artifacts

## Installation

### Option 1: Install from GitHub

```bash
claude code plugin install https://github.com/yourusername/cc-plugins/doc-crawler-plugin
```

### Option 2: Manual Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/cc-plugins.git
cd cc-plugins

# Copy plugin to Claude Code plugins directory
cp -r doc-crawler-plugin ~/.claude/plugins/
```

## Prerequisites

This plugin requires the [inform](https://github.com/fwdslsh/inform) tool for web crawling.

**Install inform:**

```bash
# Quick install (recommended)
curl -fsSL https://raw.githubusercontent.com/fwdslsh/inform/main/install.sh | sh

# Or via npm
npm install -g @fwdslsh/inform

# Or via Bun
bun install -g @fwdslsh/inform
```

**Verify installation:**

```bash
inform --version
```

## Usage

Invoke the doc-crawler skill:

```bash
/doc-crawler
```

The skill will guide you through:

1. **Prerequisites Check**: Verify inform installation
2. **Initial Crawl**: Try direct crawling with reasonable limits
3. **Sitemap Discovery**: Fetch and parse sitemap.xml if needed
4. **URL Filtering**: Extract documentation URLs, exclude non-doc pages
5. **Batch Crawling**: Crawl all documentation pages efficiently
6. **Content Organization**: Analyze and structure crawled content
7. **Skill Creation**: Create concise, well-structured skill files
8. **Repository Cleanup**: Remove intermediate artifacts

## Example Workflow

```bash
# Step 1: Start the skill
/doc-crawler

# Step 2: Follow the guided workflow
# The skill will help you:
# - Check if inform is installed
# - Crawl documentation from a website
# - Filter and organize content
# - Create a concise skill file
# - Clean up temporary files

# Step 3: Result
# You'll have a clean, well-structured skill file ready to use
```

## Skill Best Practices

The plugin emphasizes creating **concise, practical skills**:

**✅ DO:**
- Keep skill files under 15KB (ideally 5-10KB)
- Focus on practical examples and commands
- Use progressive disclosure (overview → details)
- Include real code examples
- Cover common use cases

**❌ DON'T:**
- Merge all documentation into one file
- Include every detail from docs
- Create files over 50KB
- Add redundant information

## Configuration

The skill supports various inform configurations:

```yaml
# Example: docs-config.yaml
globals:
  outputDir: ./docs-full
  delay: 500
  concurrency: 5
  ignoreErrors: true
  limit: 500

targets:
  - url: https://example.com/docs/getting-started
  - url: https://example.com/docs/concepts
  - url: https://example.com/docs/api
```

## Troubleshooting

### Issue: inform not found

**Solution:**
```bash
# Install inform
curl -fsSL https://raw.githubusercontent.com/fwdslsh/inform/main/install.sh | sh
```

### Issue: Rate limiting

**Solution:**
```bash
# Increase delay between requests
inform https://example.com --delay 1000 --concurrency 2
```

### Issue: Incomplete crawl

**Solution:**
- Use the sitemap approach
- Manually specify important URLs in config file

## Resources

- [inform GitHub](https://github.com/fwdslsh/inform)
- [Claude Code Documentation](https://docs.anthropic.com/claude/docs)
- [Plugin Development Guide](https://github.com/anthropics/claude-code)

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## License

MIT License - See LICENSE file for details

## Author

c_w_xiaohei

## Version

1.0.0
