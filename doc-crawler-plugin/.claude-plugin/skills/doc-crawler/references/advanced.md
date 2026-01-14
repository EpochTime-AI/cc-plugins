# Advanced Techniques

This document covers advanced scenarios for the documentation crawler skill.

## Handling Large Documentation Sites

For sites with 500+ pages:

1. **Prioritize sections** - Crawl most important sections first
   ```bash
   # Crawl only essential sections
   inform https://example.com/docs/getting-started --limit 50
   ```

2. **Create multiple skills** - Split into logical skills
   - `tool-basics.md` - Core functionality
   - `tool-advanced.md` - Advanced features
   - `tool-api.md` - API reference

3. **Use sampling** - Review a sample of pages first
   ```bash
   # Test with small limit first
   inform https://example.com/docs --limit 10
   ```

## Dealing with Dynamic Content

Some sites use JavaScript rendering:

```bash
# inform handles static HTML well
# For JS-heavy sites, consider:
# 1. Check if site has a static docs version
# 2. Look for API documentation endpoints
# 3. Use browser dev tools to find data sources
```

## Handling Authentication

For private documentation:

```bash
# Options:
# 1. Use browser to download pages manually
# 2. Check if site offers documentation exports
# 3. Contact site owner for documentation access
# 4. Look for public API documentation
```

## Optimizing Crawl Performance

### Concurrency Settings

```yaml
globals:
  concurrency: 10  # Higher = faster, but more load on target
  delay: 200       # Lower delay = faster, but may trigger rate limiting
```

### Content Filtering

For large crawls, filter early:

```bash
# Only crawl docs/ paths
inform https://example.com \
  --filter '.*/docs/.*' \
  --limit 500
```

### Memory Management

For very large sites:

```bash
# Crawl in batches by section
inform https://example.com/docs/section1 --output-dir ./batch1
inform https://example.com/docs/section2 --output-dir ./batch2
```

## Creating Multiple Related Skills

When documentation is large, create a skill structure:

**Example: Python Django Skills**

```
skills/
├── django-basics.md
├── django-models.md
├── django-views.md
└── django-api.md
```

**Link between skills:**

In `django-basics.md`:
```markdown
## Related Skills
- [Django Models Guide](django-models.md)
- [Django Views Guide](django-views.md)
- [Django REST API](django-api.md)
```

## Incremental Updates

Keep skills current by periodically re-crawling:

```bash
# First crawl (new skill)
inform https://docs.example.com > docs-v1.txt

# Later, check for updates
inform https://docs.example.com > docs-v2.txt

# Compare changes
diff docs-v1.txt docs-v2.txt
```

## Handling Different Documentation Formats

### Markdown-based Docs

Works directly with inform:
```bash
inform https://example.com/docs --output-dir ./docs
```

### Wiki-based Docs (Confluence, MediaWiki)

Check for export options:
```bash
# Confluence: Look for PDF export
# MediaWiki: Use API to export pages
# GitHub Wiki: Clone the wiki repo
```

### Sphinx-generated Docs

Works well with inform:
```bash
inform https://docs.example.com/sphinx-docs --limit 200
```

### API-based Documentation

Look for OpenAPI/Swagger endpoints:
```bash
# Find OpenAPI spec
curl https://api.example.com/openapi.json | jq '.' > openapi.json

# Create skill from spec
# (Consider using dedicated tools for API docs)
```

## Preprocessing Before Skill Creation

### Clean Up Content

```bash
# Remove duplicate content
find ./docs -name "*.md" -exec sh -c '
  sort "$1" | uniq > "$1.tmp" && mv "$1.tmp" "$1"
' _ {} \;

# Remove empty files
find ./docs -name "*.md" -size 0 -delete

# Remove boilerplate
# Manually review and remove common navigation/footer content
```

### Reformat Content

```bash
# Convert HTML to Markdown if needed
for file in docs/*.html; do
  pandoc "$file" -t markdown -o "${file%.html}.md"
done
```

## Integration with Version Control

### Track Documentation Changes

```bash
# Create documentation version tag
git tag -a docs-v1.0 -m "Documentation snapshot from https://example.com"

# Compare future crawls against baseline
git diff docs-v1.0 HEAD -- skills/
```

### Create Documentation Branches

```bash
# Create branch for major documentation updates
git checkout -b docs-update

# Crawl and update skills
inform https://example.com/docs > ./skills/tool.md

# Review and commit
git commit -m "docs: update tool documentation from latest version"
```

## Performance Considerations

### File Size Limits

- **Minimum skill size:** 1KB (too small is not useful)
- **Recommended:** 5-15KB
- **Maximum:** 50KB (if larger, split into multiple skills)

### Total Repository Size

```bash
# Check current size
du -sh .

# Check size by component
du -sh skills/
du -sh crawled-docs/
```

### Context Management

Smaller skills = better context usage:
- 5KB skill ≈ 1,000-2,000 words
- 15KB skill ≈ 3,000-5,000 words
- 50KB skill ≈ 10,000-15,000 words

## Version Management

### Semantic Versioning for Skills

```markdown
---
name: tool-skill
version: 1.0.0
---
```

**Version format:** Major.Minor.Patch
- **1.0.0** → Initial release
- **1.1.0** → New features added
- **1.0.1** → Bug fixes/updates
- **2.0.0** → Major breaking changes

### Update Strategy

```bash
# When updating skills:
1. Update version in frontmatter
2. Document changes in commit message
3. Test with new tool version
4. Commit with semantic message

git commit -m "feat: update tool-skill to v1.1.0 with new examples"
```

## Automation Ideas

### Scheduled Updates

```bash
# Create a script to periodically update documentation
#!/bin/bash
# update-docs.sh

SITE="https://docs.example.com"
SKILL="skill-name.md"

# Crawl fresh documentation
inform "$SITE" --output-dir ./docs-fresh

# Compare and update if changed
if ! diff -q ./docs-old ./docs-fresh > /dev/null; then
  # Update skill based on new content
  echo "Documentation changed, updating skill..."
  # Create new skill from fresh docs
fi
```

### CI/CD Integration

Integrate documentation crawling into CI/CD:

```yaml
# Example: GitHub Actions
name: Update Documentation Skills
on:
  schedule:
    - cron: '0 0 * * 0'  # Weekly on Sunday

jobs:
  update-docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: npm install -g @fwdslsh/inform
      - run: bash update-docs.sh
      - run: git commit -am "docs: automated documentation update"
      - run: git push
```
