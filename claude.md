# Epochtime AI - Claude Code Plugins Marketplace

> Official Claude Code plugin marketplace by **Epochtime AI**

## Repository Overview

This repository serves as a Claude Code plugin marketplace, providing a curated collection of plugins to enhance AI-powered development workflows. The marketplace follows the official Claude Code marketplace specification and can be added to Claude Code using Git.

### Repository Structure

```
cc-plugins/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace manifest (REQUIRED)
├── doc-crawler-plugin/            # Plugin: Documentation crawler
│   ├── plugin.json
│   ├── README.md
│   └── skills/
│       └── doc-crawler.md
├── claude.md                      # This file - development guide
├── README.md                      # User-facing documentation
└── LICENSE
```

## Marketplace Development Guide

### Understanding the Marketplace Structure

A Claude Code marketplace is a Git repository that contains:
1. **Marketplace manifest** - `.claude-plugin/marketplace.json` (required)
2. **Plugin directories** - Each plugin in its own directory
3. **Documentation** - README files for users and developers

### Marketplace Manifest Format

The `.claude-plugin/marketplace.json` file defines your marketplace:

```json
{
  "name": "marketplace-name",
  "description": "Description of your marketplace",
  "version": "1.0.0",
  "owner": {
    "name": "Organization Name"
  },
  "repository": "https://github.com/owner/repo",
  "plugins": [
    {
      "name": "plugin-name",
      "source": "./plugin-directory",
      "description": "Plugin description",
      "version": "1.0.0"
    }
  ]
}
```

### Plugin Source Formats

The `source` field in the marketplace.json supports three formats:

#### 1. Relative Path (Current Setup)

For plugins in the same repository:

```json
{
  "name": "doc-crawler-plugin",
  "source": "./doc-crawler-plugin",
  "description": "Documentation crawler plugin",
  "version": "1.0.0"
}
```

**Important**:
- Path MUST start with `./`
- Cannot contain `..` (parent directory references)
- Only works when users add marketplace via Git URL

#### 2. GitHub Repository

For plugins hosted in separate GitHub repositories:

```json
{
  "name": "external-plugin",
  "source": {
    "source": "github",
    "repo": "owner/plugin-repo"
  },
  "description": "External plugin from GitHub",
  "version": "1.0.0"
}
```

#### 3. Git Repository URL

For plugins hosted on other Git platforms:

```json
{
  "name": "gitlab-plugin",
  "source": {
    "source": "url",
    "url": "https://gitlab.com/team/plugin.git"
  },
  "description": "Plugin from GitLab",
  "version": "1.0.0"
}
```

## Adding New Plugins to This Marketplace

### Step 1: Create Plugin Directory

```bash
# Create plugin directory structure
mkdir -p new-plugin/{skills,commands,agents,hooks}

# Create plugin manifest
cat > new-plugin/plugin.json << 'EOF'
{
  "name": "new-plugin",
  "version": "1.0.0",
  "description": "Plugin description",
  "author": "Your Name"
}
EOF
```

### Step 2: Update Marketplace Manifest

Edit `.claude-plugin/marketplace.json` and add your plugin:

```json
{
  "plugins": [
    {
      "name": "doc-crawler-plugin",
      "source": "./doc-crawler-plugin",
      "description": "Documentation crawler plugin",
      "version": "1.0.0"
    },
    {
      "name": "new-plugin",
      "source": "./new-plugin",
      "description": "Your new plugin description",
      "version": "1.0.0"
    }
  ]
}
```

### Step 3: Validate

```bash
# Validate marketplace manifest
claude plugin validate .

# Should output:
# ✔ Validation passed
```

### Step 4: Commit and Push

```bash
git add .
git commit -m "Add new-plugin to marketplace"
git push
```

## Validation

### Manual Validation

```bash
# Validate marketplace manifest
claude plugin validate .

# Validate specific plugin
claude plugin validate ./doc-crawler-plugin
```

### What Gets Validated

The `claude plugin validate` command checks:

**For Marketplace (`.claude-plugin/marketplace.json`)**:
- Valid JSON syntax
- Required fields: `name`, `description`, `version`, `owner`, `repository`, `plugins`
- Plugin source format (relative path with `./`, GitHub repo, or Git URL)
- No parent directory references (`..`) in relative paths
- Version format (semantic versioning)

**For Plugin (`plugin.json`)**:
- Valid JSON syntax
- Required fields: `name`, `version`, `description`
- Optional fields: `author`, `repository`, `skills`, `commands`, `agents`, `hooks`
- Component path references are valid
- No duplicate component names

### Automated Validation (Pre-commit Hook)

This repository includes a pre-commit hook that automatically validates the marketplace manifest before each commit. See `.git/hooks/pre-commit`.

## Using This Marketplace

### Add Marketplace to Claude Code

```bash
# Via SSH
claude plugin marketplace add git@github.com:EpochTime-AI/cc-plugins.git

# Via HTTPS
claude plugin marketplace add https://github.com/EpochTime-AI/cc-plugins.git
```

### Install Plugins from Marketplace

```bash
# List available plugins
claude plugin marketplace list

# Install a plugin
claude plugin install doc-crawler-plugin
```

## Common Issues and Solutions

### Issue: "Marketplace file not found"

**Cause**: Missing `.claude-plugin/marketplace.json`

**Solution**: Ensure the file exists at the repository root:
```bash
ls -la .claude-plugin/marketplace.json
```

### Issue: "plugins.0.source: Invalid input"

**Cause**: Incorrect source format

**Solutions**:
- Relative paths must start with `./`
- Cannot use `..` in paths
- For objects, `source` must be `"github"` or `"url"`

### Issue: Validation fails

**Cause**: Invalid JSON or missing required fields

**Solution**: Run validation and fix errors:
```bash
claude plugin validate .
```

## Development Workflow

### 1. Create New Plugin

```bash
# Use the plugin-dev skill
/plugin-dev:create-plugin
```

### 2. Test Locally

```bash
# Install plugin locally for testing
cp -r new-plugin ~/.claude/plugins/

# Test the plugin
claude code
```

### 3. Add to Marketplace

```bash
# Update marketplace.json
# Validate
claude plugin validate .

# Commit
git add .
git commit -m "Add new-plugin"
git push
```

### 4. Users Install

```bash
# Users can now install from marketplace
claude plugin install new-plugin
```

## Getting Help with Plugin Development

### Plugin Development Questions

For any questions about plugin development or plugin architecture:

1. **First**: Use the plugin-dev skills available in Claude Code
   ```bash
   /plugin-dev:create-plugin      # End-to-end plugin creation
   /plugin-dev:plugin-structure   # Plugin architecture
   /plugin-dev:skill-development  # Creating skills
   /plugin-dev:command-development # Creating commands
   /plugin-dev:agent-development  # Creating agents
   /plugin-dev:hook-development   # Creating hooks
   /plugin-dev:mcp-integration    # MCP server integration
   ```

2. **If still unclear**: Ask the claude-code-guide agent
   - The guide has comprehensive knowledge about Claude Code features
   - Can answer questions about CLI tools, API usage, and best practices
   - Access via: Ask Claude to consult the claude-code-guide

3. **For specific issues**: Check the official documentation
   - [Claude Code Documentation](https://code.claude.com/docs)
   - [Plugin Development Guide](https://code.claude.com/docs/en/plugins)

## Plugin Development Best Practices

### Plugin Structure

Each plugin should follow this structure:

```
plugin-name/
├── plugin.json              # Plugin manifest (REQUIRED)
├── README.md                # Plugin documentation
├── skills/                  # Skills (.md files)
│   └── skill-name.md
├── commands/                # Commands (.md files)
│   └── command-name.md
├── agents/                  # Agents (.md files)
│   └── agent-name.md
└── hooks/                   # Hooks (.md files)
    └── hook-name.md
```

### Plugin Manifest (plugin.json)

The `plugin.json` file is the core manifest for your plugin. It defines metadata and component discovery.

#### Basic Structure

```json
{
  "name": "plugin-name",
  "version": "1.0.0",
  "description": "Clear, concise description",
  "author": {
    "name": "Your Name",
    "email": "your.email@example.com"
  },
  "repository": "https://github.com/owner/repo"
}
```

#### Required Fields

- **`name`** (string): Plugin identifier, lowercase with hyphens
- **`version`** (string): Semantic version (e.g., "1.0.0")
- **`description`** (string): Clear, concise plugin description

#### Optional Fields

- **`author`** (string | object): Author name or object with `name` and `email`
- **`repository`** (string): Git repository URL
- **`skills`** (string | object): Path to skills directory or skill definitions
- **`commands`** (string | object): Path to commands directory or command definitions
- **`agents`** (string | object): Path to agents directory or agent definitions
- **`hooks`** (string | object): Path to hooks directory or hook definitions

#### Component Discovery

**Auto-discovery (Recommended)**:
```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "My plugin",
  "skills": "./skills/",
  "commands": "./commands/",
  "agents": "./agents/",
  "hooks": "./hooks/"
}
```

Claude Code will automatically discover all `.md` files in these directories.

**Explicit Definition**:
```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "My plugin",
  "skills": {
    "skill-name": {
      "description": "Skill description",
      "path": "skills/skill-name.md"
    }
  }
}
```

Use explicit definition when you need fine-grained control over component metadata.

### Skill Guidelines

- Keep skills concise (5-15KB ideal, max 50KB)
- Focus on practical examples
- Use clear section structure
- Include troubleshooting tips
- See `/plugin-dev:skill-development` for details

### Documentation

- Each plugin needs a README.md
- Include installation instructions
- Provide usage examples
- Document all skills/commands/agents

## Resources

### Official Documentation

- [Claude Code Documentation](https://code.claude.com/docs)
- [Plugin Development Guide](https://code.claude.com/docs/en/plugins)
- [Marketplace Specification](https://code.claude.com/docs/en/plugin-marketplaces)

### Plugin Development Skills

This repository includes the `doc-crawler-plugin` which provides:
- `/doc-crawler` - Systematic documentation crawling workflow
- Sitemap parsing and URL filtering
- Quality guidelines for skill creation
- Repository cleanup automation

### Development Tools

```bash
# Validate marketplace
claude plugin validate .

# Validate plugin
claude plugin validate ./plugin-name

# List installed plugins
claude plugin list

# Remove plugin
claude plugin remove plugin-name
```

## Contributing

We welcome contributions! To add a plugin:

1. Fork this repository
2. Create your plugin following the structure above
3. Update `.claude-plugin/marketplace.json`
4. Validate: `claude plugin validate .`
5. Submit a pull request

## Maintenance

### Updating Plugins

```bash
# Update plugin version in plugin.json
# Update version in marketplace.json
# Validate and commit
claude plugin validate .
git add .
git commit -m "Update plugin-name to v1.1.0"
git push
```

### Removing Plugins

```bash
# Remove plugin directory
rm -rf plugin-name/

# Remove from marketplace.json
# Validate and commit
claude plugin validate .
git add .
git commit -m "Remove plugin-name from marketplace"
git push
```

## License

MIT License - See [LICENSE](./LICENSE) file for details.

---

**Maintained by Epochtime AI** | Last Updated: 2026-01-15
