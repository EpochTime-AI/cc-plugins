# Epochtime AI - Claude Code Plugins Marketplace

> Official Claude Code plugin marketplace by **Epochtime AI**

## Repository Overview

This repository serves as a Claude Code plugin marketplace, providing a curated collection of plugins to enhance AI-powered development workflows. The marketplace follows the official Claude Code marketplace specification and **is distributed via GitHub** to make it easily accessible to Claude Code users worldwide.

### GitHub Distribution

This marketplace is hosted on GitHub at **[github.com/EpochTime-AI/cc-plugins](https://github.com/EpochTime-AI/cc-plugins)**. Users can add this marketplace to Claude Code using Git, and the marketplace will automatically resolve plugin sources from this GitHub repository.

### Repository Structure

```
cc-plugins/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace manifest (REQUIRED)
├── doc-crawler-plugin/            # Plugin: Documentation crawler
│   ├── .claude-plugin/
│   │   └── plugin.json
│   ├── README.md
│   └── skills/
│       └── doc-crawler.md
├── CLAUDE.md                      # This file - development guide
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

### Step 1: Create Plugin Structure

Use the plugin-dev skill for guided creation:

```bash
/plugin-dev:create-plugin
```

Or manually create the plugin directory:

```bash
mkdir -p new-plugin/.claude-plugin
cat > new-plugin/.claude-plugin/plugin.json << 'EOF'
{
  "name": "new-plugin",
  "version": "1.0.0",
  "description": "Plugin description",
  "author": "Your Name"
}
EOF
```

### Step 2: Update Marketplace Manifest

Edit `.claude-plugin/marketplace.json` and add your plugin to the `plugins` array:

```json
{
  "name": "new-plugin",
  "source": "./new-plugin",
  "description": "Your new plugin description",
  "version": "1.0.0"
}
```

### Step 3: Validate and Commit

```bash
# Validate
claude plugin validate .

# Commit and push
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

This repository includes a pre-commit hook that automatically validates the marketplace manifest before each commit.

## Using This Marketplace

Users can add this marketplace to Claude Code using Git (see **GitHub Distribution** section above for details):

```bash
# Add marketplace to Claude Code
claude plugin marketplace add EpochTime-AI/cc-plugins

# Or use the full GitHub URL
claude plugin marketplace add https://github.com/EpochTime-AI/cc-plugins.git
```

Once added, users can install plugins:

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

### Issue: Plugin shows in marketplace but fails to install (404)

**Cause**: Common causes include:
- Plugin source path is incorrect in marketplace.json
- Plugin directory or .claude-plugin/plugin.json doesn't exist
- Files not committed or pushed to GitHub
- Relative path uses `..` instead of `./`

**Solution**:
1. Verify plugin directory exists locally
2. Check marketplace.json references correct path (must start with `./`)
3. Ensure all files are committed and pushed to GitHub
4. Run validation: `claude plugin validate .`

## Commit Message Convention

This repository follows a standardized commit message format to maintain clear, organized history and enable better changelog automation.

### Format

```
<type>(<scope>): <subject>

<body>
```

### Type

Must be one of:

- **feat**: A new feature or capability
- **fix**: A bug fix
- **docs**: Documentation changes only (README, claude.md, etc.)
- **refactor**: Code refactoring without feature changes
- **style**: Code style changes (formatting, missing semicolons, etc.)
- **perf**: Performance improvements
- **test**: Adding or updating tests
- **chore**: Changes to build, dependencies, configs, etc.
- **ci**: Changes to CI/CD configuration

### Scope (Optional)

Specify which component is affected:

- `marketplace` - Marketplace manifest changes
- `doc-crawler-plugin` - doc-crawler-plugin changes
- `plugin-name` - Specific plugin changes

Note: Use `CLAUDE.md` when referring to this development guide in commit messages

### Subject

- Use imperative mood ("add feature" not "added feature")
- Don't capitalize first letter
- No period at the end
- Max 50 characters
- Be concise and descriptive

### Body (Optional)

- Use imperative mood
- Include motivation for change
- Contrast with previous behavior
- Wrap at 72 characters
- Separate from subject by blank line

### Examples

```bash
# Simple feature
git commit -m "feat(doc-crawler-plugin): add sitemap parser"

# Bug fix with body
git commit -m "fix(marketplace): resolve plugin name mismatch

Previously the plugin.json name didn't match the marketplace entry,
causing plugin discovery to fail. Fixed by aligning the names."

# Documentation update
git commit -m "docs: add commit message convention to claude.md"

# Chore
git commit -m "chore(deps): update plugin-dev version"
```

## Development Workflow

### Quick Summary

1. Create plugin using `/plugin-dev:create-plugin` or manually
2. Add to `.claude-plugin/marketplace.json`
3. Validate with `claude plugin validate .`
4. Commit and push to GitHub
5. Users can then install with `claude plugin install plugin-name`

See **Adding New Plugins to This Marketplace** section above for detailed steps.

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

### Plugin Validation and Issue Resolution

If you encounter problems with your plugin installation or functionality:

1. **Validate your plugin structure**:
   ```bash
   # Validate the entire marketplace
   claude plugin validate .

   # Validate a specific plugin
   claude plugin validate ./doc-crawler-plugin
   ```

2. **Use plugin-dev validation agents and skills**: For detailed analysis and automated validation:

   The official `plugin-dev` plugin provides specialized agents and skills that should be used during plugin development, checking, and debugging:

   - **plugin-validator agent**: Use this to get comprehensive plugin validation with detailed error reporting
   - **plugin-dev skills**: Access development guidance and best practices when creating or troubleshooting plugins

   Ask Claude to use these tools when you need detailed diagnostics or during plugin development workflows.

3. **Check the plugin installation**:
   ```bash
   # List installed plugins
   claude plugin list

   # Check plugin-dev status
   claude plugin list | grep plugin-dev
   ```

4. **Debug common issues**:
   - Verify `.claude-plugin/plugin.json` is valid JSON
   - Ensure all component paths exist (skills/, commands/, etc.)
   - Check that all plugin names follow kebab-case convention
   - Verify marketplace.json source paths are relative (start with `./`)

   When stuck, use `plugin-validator` agent from plugin-dev to get detailed diagnostics.

## Plugin Development Best Practices

### Plugin Structure

Each plugin should follow this structure:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest (REQUIRED)
├── README.md                # Plugin documentation
├── skills/                  # Skills (.md files)
├── commands/                # Commands (.md files)
├── agents/                  # Agents (.md files)
└── hooks/                   # Hooks configuration
```

### Plugin Manifest (plugin.json)

The plugin manifest in `.claude-plugin/plugin.json` defines metadata and component discovery.

**Required fields**:
- `name` (string): Plugin identifier, lowercase with hyphens
- `version` (string): Semantic version (e.g., "1.0.0")
- `description` (string): Plugin description

**Optional fields**:
- `author` (string | object): Author information
- `repository` (string): Git repository URL
- `skills` (string | object): Path to skills directory (e.g., `"./skills/"`)
- `commands` (string | object): Path to commands directory
- `agents` (string | object): Path to agents directory
- `hooks` (string | object): Path to hooks configuration

Claude Code automatically discovers all `.md` files in the specified component directories.

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
