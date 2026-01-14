# Contributing to Claude Code Plugins Marketplace

Thank you for your interest in contributing to the Epochtime AI plugins marketplace! This guide will help you add high-quality plugins to our collection.

## Plugin Structure Requirements

All plugins must follow the standard Claude Code plugin structure:

```
your-plugin/
├── .claude-plugin/
│   └── plugin.json           # Plugin manifest (REQUIRED)
├── README.md                 # Plugin documentation
├── skills/                   # Optional: skill files
│   └── skill-name.md
├── commands/                 # Optional: command files
│   └── command-name.md
├── agents/                   # Optional: agent files
│   └── agent-name.md
└── hooks/                    # Optional: hook configuration
    └── hooks.json
```

## Plugin Manifest (plugin.json)

**Required fields:**
- `name` - Plugin identifier (kebab-case, lowercase)
- `version` - Semantic version (e.g., "1.0.0")
- `description` - Clear, concise description

**Optional but recommended:**
- `author` - Author information
- `repository` - Git repository URL
- `skills` - Path to skills directory (e.g., `"./skills/"`)
- `commands` - Path to commands directory
- `agents` - Path to agents directory

**Example:**
```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "Plugin description",
  "author": {
    "name": "Your Name"
  },
  "repository": "https://github.com/yourname/my-plugin",
  "skills": "./skills/"
}
```

## Plugin Quality Standards

### Code Quality

- ✅ **Valid JSON** - All JSON files must be properly formatted
- ✅ **Clear naming** - Use descriptive names (kebab-case for directories/files)
- ✅ **Documentation** - Include comprehensive README
- ✅ **Examples** - Provide working examples in skills/commands
- ✅ **Error handling** - Handle edge cases gracefully

### Documentation

- ✅ **README.md** - Required at plugin root
- ✅ **Installation** - Clear setup instructions
- ✅ **Usage** - Practical examples
- ✅ **Prerequisites** - List any required tools/libraries
- ✅ **License** - Include LICENSE file

### File Size Limits

- **Skills:** Keep under 15KB (ideally 5-10KB)
- **Plugin total:** Reasonable size (under 100MB)
- **If larger:** Split into multiple plugins

### Best Practices

- ✅ Use progressive disclosure (core content + references)
- ✅ Include multiple practical examples
- ✅ Provide troubleshooting section
- ✅ Link to official documentation
- ✅ Keep content up-to-date
- ✅ Version your components

## Adding Your Plugin

### Step 1: Create Your Plugin

Create plugin in the standard structure above. Use `/plugin-dev:create-plugin` for guided creation.

### Step 2: Create Plugin Directory

```bash
# Fork the repository on GitHub
git clone https://github.com/yourusername/cc-plugins.git
cd cc-plugins

# Create your plugin directory
mkdir -p your-plugin/.claude-plugin

# Create plugin.json
cat > your-plugin/.claude-plugin/plugin.json << 'EOF'
{
  "name": "your-plugin",
  "version": "1.0.0",
  "description": "What your plugin does",
  "author": {
    "name": "Your Name"
  },
  "skills": "./skills/"
}
EOF
```

### Step 3: Add Plugin Content

- Create README.md with installation and usage instructions
- Add skills, commands, or agents as needed
- Follow the examples in existing plugins

### Step 4: Validate Your Plugin

```bash
# Validate plugin structure
claude plugin validate ./your-plugin

# You should see: ✓ Validation passed
```

If validation fails, fix errors before proceeding.

### Step 5: Update Marketplace Manifest

Edit `.claude-plugin/marketplace.json` and add your plugin:

```json
{
  "name": "your-plugin",
  "source": "./your-plugin",
  "description": "Short description of your plugin",
  "version": "1.0.0"
}
```

### Step 6: Test Locally

```bash
# Install your plugin locally
cp -r your-plugin ~/.claude/plugins/

# Test in Claude Code
# Verify skills load, commands work, etc.
```

### Step 7: Create Pull Request

1. Commit your changes:
   ```bash
   git add your-plugin/ .claude-plugin/marketplace.json
   git commit -m "feat: add your-plugin to marketplace

   - Brief description of what the plugin does
   - Any specific features or use cases"
   ```

2. Push to your fork:
   ```bash
   git push origin your-plugin-branch
   ```

3. Create a pull request on GitHub with:
   - Clear title (e.g., "Add your-plugin to marketplace")
   - Description of what the plugin does
   - Link to any relevant documentation

## Skill Development Guidelines

### Skill File Format

Skills should be concise, practical, and well-organized:

```markdown
---
name: skill-name
description: Brief description of what this skill does
---

# Skill Title

Introduction (2-3 sentences)

## Core Concepts
Explain key ideas with practical context

## Installation
Step-by-step setup instructions

## Quick Start
Essential commands with examples

## Common Use Cases
Real-world examples

## Best Practices
1. Practice 1
2. Practice 2

## Troubleshooting
Common issues and solutions

## Resources
Links to official documentation
```

### Skill Best Practices

**✅ DO:**
- Keep under 15KB (ideally 5-10KB)
- Focus on practical examples
- Use clear, concise language
- Include real code examples
- Provide troubleshooting section
- Link to official docs

**❌ DON'T:**
- Dump entire documentation
- Create files over 50KB
- Use overly technical language
- Include outdated examples
- Omit prerequisite information

## Command Development Guidelines

Commands should be:
- **Focused** - Do one thing well
- **Documented** - Clear help text
- **Efficient** - Fast execution
- **Robust** - Handle errors gracefully

Example command structure:
```bash
---
name: my-command
description: What this command does
---

Instructions for Claude...
```

## Validation Checklist

Before submitting your PR, verify:

- [ ] Plugin structure matches standard
- [ ] `plugin.json` is valid JSON
- [ ] `plugin.json` has required fields
- [ ] README.md is complete and clear
- [ ] All file paths are correct
- [ ] Skills are under 15KB
- [ ] `claude plugin validate` passes
- [ ] Plugin installs locally without errors
- [ ] All examples work correctly
- [ ] Marketplace.json is properly formatted
- [ ] No hardcoded credentials or secrets
- [ ] LICENSE file included
- [ ] No large binary files

## Review Process

Your PR will be reviewed for:

1. **Structure** - Follows plugin standards
2. **Quality** - Well-written, clear documentation
3. **Functionality** - Plugin works as described
4. **Security** - No vulnerabilities or secrets
5. **Uniqueness** - Doesn't duplicate existing plugins

## Commit Message Convention

Use standardized commit messages:

```
<type>(<scope>): <subject>

<body>
```

**Types:**
- `feat` - New feature/plugin
- `fix` - Bug fix
- `docs` - Documentation
- `refactor` - Code refactoring
- `style` - Code style
- `test` - Tests
- `chore` - Maintenance

**Example:**
```
feat(my-plugin): add documentation crawler skill

Adds ability to crawl documentation websites and convert them
to Claude Code skills using the inform tool.

Includes:
- 8-step systematic workflow
- Sitemap support
- URL filtering
- Quality guidelines
```

## Getting Help

- **Plugin development questions:** Use `/plugin-dev:create-plugin` skill
- **Architecture questions:** Use `/plugin-dev:plugin-structure` skill
- **Skill writing:** Use `/plugin-dev:skill-development` skill
- **Validation issues:** Use the plugin-validator agent
- **General questions:** Open an issue on GitHub

## Code of Conduct

- Be respectful and constructive
- Provide helpful feedback
- Focus on improving the marketplace
- No spam or inappropriate content
- Follow all Claude Code guidelines

## License

All plugins in this marketplace must be compatible with open-source licenses:
- MIT (recommended)
- Apache 2.0
- BSD
- GPL (v3+)

Include a LICENSE file in your plugin.

## Additional Resources

- [Claude Code Plugin Development](https://code.claude.com/docs/en/plugins)
- [Plugin Best Practices](https://code.claude.com/docs/en/plugin-best-practices)
- [existing plugins](../) - Study examples in this marketplace

## Questions?

If you have questions or need clarification:
1. Check existing plugins for examples
2. Review Claude Code documentation
3. Open an issue on GitHub
4. Ask for help in pull request comments

Thank you for contributing! 🎉
