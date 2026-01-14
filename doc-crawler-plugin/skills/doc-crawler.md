---
name: doc-crawler
description: Crawl and extract documentation from websites using inform tool - handles sitemaps, filters doc URLs, and organizes content into clean markdown skills
---

# Documentation Crawler Skill

This skill guides you through crawling documentation websites and converting them into well-structured skills for Claude Code.

## Overview

The documentation crawler follows a systematic approach:

1. **Direct crawl attempt** - Try crawling the documentation URL directly
2. **Sitemap discovery** - If direct crawl is limited, fetch and parse sitemap.xml
3. **URL filtering** - Extract documentation URLs, exclude blogs/marketing pages
4. **Batch crawling** - Use inform to crawl all documentation pages
5. **Skill creation** - Organize content into a concise, well-structured skill

## Prerequisites Check

### Step 0: Verify inform Installation

**Before starting, check if inform is installed:**

```bash
# Check if inform is available
which inform

# Check version
inform --version
```

**If inform is not installed:**

1. **Ask the user** if they want to install inform
2. **Provide installation options:**

```bash
# Option 1: Quick install script (recommended)
curl -fsSL https://raw.githubusercontent.com/fwdslsh/inform/main/install.sh | sh

# Option 2: Via Bun (if Bun is installed)
bun install -g @fwdslsh/inform

# Option 3: Via npm
npm install -g @fwdslsh/inform

# Option 4: Download binary manually
# Visit: https://github.com/fwdslsh/inform/releases
```

3. **Verify installation:**

```bash
# After installation, verify
inform --version

# Test with a simple crawl
inform --help
```

**If user declines installation:**
- Inform them that inform is required for this workflow
- Suggest alternative: manual documentation download
- Provide link to inform documentation

## Step 1: Initial Crawl Attempt

Start with a direct crawl to understand the site structure:

```bash
# Basic crawl with reasonable limits
inform https://docs.example.com \
  --output-dir ./docs-crawl \
  --limit 100 \
  --delay 500 \
  --concurrency 3
```

**Analyze the results:**
- Check how many pages were crawled
- Review the directory structure
- Identify if important sections are missing

If the crawl captured most documentation, proceed to Step 5. Otherwise, continue to Step 2.

## Step 2: Sitemap Discovery

When direct crawling is limited by robots.txt or site structure, use the sitemap:

```bash
# Common sitemap locations
https://example.com/sitemap.xml
https://example.com/sitemap_index.xml
https://example.com/docs/sitemap.xml
```

**Fetch and analyze sitemap:**

Use WebFetch to extract URLs:

```
WebFetch the sitemap URL with prompt:
"Extract all URLs from this sitemap. List URLs under /docs/ or /documentation/ paths.
Exclude blog posts (/blog/), case studies, marketing pages, and changelog entries."
```

**Alternative: Parse sitemap manually**

```bash
# Download sitemap
curl https://example.com/sitemap.xml -o sitemap.xml

# Extract URLs (basic filtering)
grep -oP '(?<=<loc>)[^<]+' sitemap.xml | grep '/docs/' > doc-urls.txt
```

## Step 3: Filter Documentation URLs

Create a filtered list of documentation URLs:

**Include patterns:**
- `/docs/`
- `/documentation/`
- `/guide/`
- `/tutorial/`
- `/reference/`
- `/api/`
- `/getting-started/`

**Exclude patterns:**
- `/blog/`
- `/news/`
- `/case-studies/`
- `/changelog/`
- `/release-notes/`
- `/community/`
- `/pricing/`
- `/about/`

**Create a YAML config for inform:**

```yaml
# docs-config.yaml
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
  - url: https://example.com/docs/guides
  # Add more URLs from filtered list
```

## Step 4: Batch Crawling

Execute the crawl with the configuration:

```bash
# Using config file
inform docs-config.yaml

# Or crawl URLs individually
for url in $(cat doc-urls.txt); do
  echo "Crawling: $url"
  inform "$url" --output-dir ./docs-full --delay 300
done
```

**Monitor the crawl:**
- Watch for failed requests
- Check for rate limiting issues
- Verify content quality

**Crawl statistics:**
```bash
# Count crawled files
find ./docs-full -name "*.md" | wc -l

# Check total size
du -sh ./docs-full

# List directory structure
find ./docs-full -type d | sort
```

## Step 5: Organize Content Structure

Analyze the crawled content:

```bash
# List all markdown files
find ./docs-full -name "*.md" -type f | sort

# Check file sizes
find ./docs-full -name "*.md" -exec wc -l {} + | sort -n

# Identify main sections
find ./docs-full -type d -maxdepth 2
```

**Common documentation structures:**
- Getting Started / Quick Start
- Core Concepts / Fundamentals
- Guides / Tutorials
- API Reference / CLI Reference
- Integrations
- Best Practices
- Troubleshooting / FAQ

## Step 6: Create Skill File

**DO NOT merge all files into one giant file!**

Instead, create a **concise, well-structured skill** with:

### Skill Structure Template

```markdown
---
name: tool-name
description: Brief description covering main features and use cases
---

# Tool Name

Brief introduction (2-3 sentences)

## Core Concepts

### Key Concept 1
Brief explanation with essential details

### Key Concept 2
Brief explanation with essential details

## Installation

```bash
# Installation commands
```

## Quick Start

```bash
# Essential commands with examples
```

## Common Use Cases

### Use Case 1
Brief description with code example

### Use Case 2
Brief description with code example

## Key Features

- Feature 1 with brief explanation
- Feature 2 with brief explanation
- Feature 3 with brief explanation

## Configuration

Essential configuration examples

## Best Practices

1. Practice 1
2. Practice 2
3. Practice 3

## Troubleshooting

Common issues and solutions

## Resources

- Official docs link
- GitHub link
- Community link
```

### Skill Best Practices

**✅ DO:**
- Keep skill file under 15KB (ideally 5-10KB)
- Focus on practical examples and commands
- Use progressive disclosure (overview → details)
- Include real code examples
- Organize with clear sections
- Cover common use cases
- Add troubleshooting tips

**❌ DON'T:**
- Merge all documentation into one file
- Include every detail from docs
- Copy-paste entire API references
- Add redundant information
- Create files over 50KB

### Content Selection Priority

1. **Essential** (must include):
   - Installation instructions
   - Core concepts
   - Quick start commands
   - Common use cases

2. **Important** (should include):
   - Configuration examples
   - Best practices
   - Key features overview
   - Troubleshooting

3. **Optional** (include if space allows):
   - Advanced features
   - Integration examples
   - Performance tips

## Step 7: Validate Skill Quality

**Check skill file:**

```bash
# File size (should be under 15KB)
ls -lh skill-name.md

# Line count (ideally 200-500 lines)
wc -l skill-name.md

# Preview first 50 lines
head -50 skill-name.md
```

**Quality checklist:**

- [ ] Frontmatter with name and description
- [ ] Clear introduction
- [ ] Installation instructions
- [ ] Quick start examples
- [ ] Common use cases with code
- [ ] Best practices section
- [ ] Troubleshooting tips
- [ ] Resource links
- [ ] File size under 15KB
- [ ] Well-organized sections
- [ ] Practical, actionable content

## Step 8: Clean Up Repository

After creating the skill file, clean up intermediate files and organize the repository.

### Identify Files to Keep

**Keep:**
- Final skill file(s) (e.g., `atlas.md`, `doc-crawler.md`)
- README or documentation about the skills
- Configuration files if needed for updates

**Remove:**
- Raw crawled markdown files
- Temporary directories
- Config files used for crawling
- Intermediate processing files

### Cleanup Commands

```bash
# Navigate to repository root
cd /path/to/repository

# List what will be removed (dry run)
echo "Files and directories to remove:"
ls -d docs-full/ docs-crawl/ atlas-docs-full/ *.yaml 2>/dev/null

# Remove crawled content directories
rm -rf docs-full/
rm -rf docs-crawl/
rm -rf atlas-docs-full/
rm -rf */  # Remove all subdirectories if they're all intermediate

# Remove config files
rm -f *-config.yaml
rm -f sitemap.xml
rm -f doc-urls.txt

# Keep only skill files
# Assuming skills are in root or a skills/ directory
mkdir -p skills/
mv *.md skills/ 2>/dev/null || true

# Verify cleanup
echo "Remaining files:"
ls -la
```

### Organize Final Structure

**Recommended repository structure:**

```
repository/
├── skills/
│   ├── atlas.md           # Main skill file
│   ├── doc-crawler.md     # Documentation crawler skill
│   └── other-skill.md     # Other skills
├── README.md              # Repository documentation
└── .gitignore            # Ignore future crawl artifacts
```

### Create .gitignore

Prevent future crawl artifacts from being committed:

```bash
# Create .gitignore
cat > .gitignore << 'EOF'
# Crawl artifacts
docs-full/
docs-crawl/
*-docs-full/
*-crawl/

# Config files
*-config.yaml
sitemap.xml
doc-urls.txt

# Temporary files
*.tmp
.DS_Store
EOF
```

### Final Verification

```bash
# Check repository size
du -sh .

# List all files
find . -type f -not -path './.git/*'

# Verify only skills remain
ls -lh skills/

# Check git status (if using git)
git status
```

### Commit Changes (if using git)

```bash
# Stage skill files
git add skills/*.md
git add README.md
git add .gitignore

# Commit
git commit -m "Add [tool-name] documentation skill

- Created concise skill file from crawled documentation
- Cleaned up intermediate crawl artifacts
- Organized repository structure"

# Push to remote
git push
```

## Example Workflow

Here's a complete example of crawling Atlas documentation:

```bash
# Step 1: Try direct crawl
inform https://atlasgo.io/docs --output-dir ./atlas-docs --limit 100

# Step 2: Fetch sitemap (if needed)
# Use WebFetch to get sitemap URLs

# Step 3: Create config file
cat > atlas-config.yaml << 'EOF'
globals:
  outputDir: ./atlas-docs-full
  delay: 500
  concurrency: 5
  ignoreErrors: true

targets:
  - url: https://atlasgo.io/docs
  - url: https://atlasgo.io/getting-started
  - url: https://atlasgo.io/declarative/apply
  - url: https://atlasgo.io/versioned/intro
  # ... more URLs
EOF

# Step 4: Crawl all documentation
inform atlas-config.yaml

# Step 5: Analyze structure
find ./atlas-docs-full -type d | head -20
find ./atlas-docs-full -name "*.md" | wc -l

# Step 6: Create skill file (manually)
# Write a concise skill based on crawled content
# Focus on core concepts, commands, and examples

# Step 7: Validate
ls -lh atlas.md
wc -l atlas.md
```

## Advanced Techniques

### Handling Large Documentation Sites

For sites with 500+ pages:

1. **Prioritize sections** - Crawl most important sections first
2. **Create multiple skills** - Split into logical skills (e.g., "tool-basics", "tool-advanced")
3. **Use sampling** - Review a sample of pages to understand structure

### Dealing with Dynamic Content

Some sites use JavaScript rendering:

```bash
# inform handles static HTML well
# For JS-heavy sites, consider:
# 1. Check if site has a static docs version
# 2. Look for API documentation endpoints
# 3. Use browser dev tools to find data sources
```

### Handling Authentication

For private documentation:

```bash
# Some sites require authentication
# Options:
# 1. Use browser to download pages manually
# 2. Check if site offers documentation exports
# 3. Contact site owner for documentation access
```

## Common Issues

### Issue: robots.txt blocking

**Solution:**
```bash
# Only if you have permission!
inform https://example.com --ignore-robots
```

### Issue: Rate limiting

**Solution:**
```bash
# Increase delay between requests
inform https://example.com --delay 1000 --concurrency 2
```

### Issue: Incomplete crawl

**Solution:**
```bash
# Use sitemap approach
# Manually specify important URLs in config
```

### Issue: Poor content extraction

**Solution:**
```bash
# Use --raw flag to get HTML
inform https://example.com --raw

# Then manually convert important pages
```

## Tips for Success

1. **Start small** - Test with a few pages first
2. **Respect rate limits** - Use appropriate delays
3. **Check robots.txt** - Respect site policies
4. **Verify content quality** - Review extracted markdown
5. **Iterate** - Refine your approach based on results
6. **Focus on value** - Prioritize most useful content
7. **Keep skills concise** - Quality over quantity

## Integration with Claude Code

Once you have created your skill:

1. **Save skill file** - Place in appropriate directory
2. **Test skill** - Verify it loads correctly
3. **Iterate** - Refine based on usage
4. **Update** - Keep skill current with documentation changes

## Resources

- **inform GitHub**: https://github.com/fwdslsh/inform
- **inform Documentation**: See inform README for full options
- **Skill Development**: Use `/plugin-dev:skill-development` for skill best practices

## Complete Workflow Checklist

The full documentation crawler workflow:

- [ ] **Step 0:** Check inform installation, install if needed
- [ ] **Step 1:** Try direct crawl with inform
- [ ] **Step 2:** Fetch sitemap if needed
- [ ] **Step 3:** Filter documentation URLs
- [ ] **Step 4:** Batch crawl with config file
- [ ] **Step 5:** Analyze content structure
- [ ] **Step 6:** Create concise skill (5-15KB)
- [ ] **Step 7:** Validate quality
- [ ] **Step 8:** Clean up repository, keep only skills

## Summary

**Key principles:**

1. ✅ **Check prerequisites** - Verify inform is installed
2. ✅ **Systematic approach** - Follow the 7-step workflow
3. ✅ **Quality over quantity** - Create concise, practical skills
4. ✅ **Clean repository** - Remove intermediate artifacts
5. ✅ **Organize properly** - Use clear directory structure

**Remember:** The goal is a **concise, practical skill**, not a complete documentation dump!

## Quick Reference

### Essential Commands

```bash
# Check inform
which inform && inform --version

# Direct crawl
inform https://docs.example.com --output-dir ./docs --limit 100

# Batch crawl with config
inform config.yaml

# Analyze results
find ./docs -name "*.md" | wc -l
du -sh ./docs

# Clean up
rm -rf docs-full/ *-config.yaml
mkdir -p skills/
mv *.md skills/

# Verify
ls -lh skills/
```

### File Size Guidelines

- **Skill file:** 5-15KB (200-500 lines)
- **Maximum:** 50KB
- **If larger:** Split into multiple skills

### Quality Indicators

✅ Good skill:
- Concise and focused
- Practical examples
- Clear structure
- Under 15KB

❌ Poor skill:
- Dumps all documentation
- No examples
- Disorganized
- Over 50KB
