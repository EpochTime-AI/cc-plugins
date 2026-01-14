---
name: doc-crawler
description: Crawl and extract documentation from websites using inform tool - handles sitemaps, filters doc URLs, and organizes content into clean markdown skills
---

# Documentation Crawler Skill

This skill guides you through crawling documentation websites and converting them into well-structured skills for Claude Code.

## Overview

The documentation crawler follows a systematic 8-step approach:

1. **Prerequisites Check** - Verify inform is installed
2. **Initial Crawl** - Try crawling the documentation URL directly
3. **Sitemap Discovery** - Fetch and parse sitemap.xml if needed
4. **URL Filtering** - Extract documentation URLs, exclude blogs/marketing
5. **Batch Crawling** - Use inform to crawl all documentation pages
6. **Content Organization** - Analyze and structure the content
7. **Skill Creation** - Create a concise, well-structured skill file
8. **Repository Cleanup** - Remove intermediate artifacts

## Prerequisites Check

Before starting, verify inform is installed:

```bash
# Check if inform is available
which inform && inform --version
```

**If not installed:**

```bash
# Quick install (recommended)
curl -fsSL https://raw.githubusercontent.com/fwdslsh/inform/main/install.sh | sh

# Or via npm
npm install -g @fwdslsh/inform

# Or via Bun
bun install -g @fwdslsh/inform
```

## Step 1: Initial Crawl Attempt

Start with a direct crawl to understand site structure:

```bash
# Basic crawl with reasonable limits
inform https://docs.example.com \
  --output-dir ./docs-crawl \
  --limit 100 \
  --delay 500 \
  --concurrency 3
```

**Analyze results:**
- Check how many pages were crawled
- Review the directory structure
- Identify if important sections are missing

If the crawl captured most documentation, proceed to Step 6. Otherwise, continue to Step 3.

## Step 2: Sitemap Discovery

When direct crawling is limited, use the sitemap:

```bash
# Common sitemap locations
https://example.com/sitemap.xml
https://example.com/docs/sitemap.xml
```

**Fetch and extract URLs:**

Use WebFetch to extract URLs:
```
WebFetch the sitemap URL with prompt:
"Extract all URLs from this sitemap. List URLs under /docs/ or /documentation/ paths.
Exclude blog posts (/blog/), case studies, marketing pages, and changelog entries."
```

**Alternative: Parse manually**

```bash
curl https://example.com/sitemap.xml | grep -oP '(?<=<loc>)[^<]+' | grep '/docs/'
```

## Step 3: Filter Documentation URLs

Create a filtered list. Include patterns like:
- `/docs/`, `/documentation/`, `/guide/`, `/tutorial/`, `/reference/`, `/api/`

Exclude patterns like:
- `/blog/`, `/news/`, `/case-studies/`, `/changelog/`, `/pricing/`, `/about/`

**Create inform config:**

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
```

## Step 4: Batch Crawling

Execute the crawl:

```bash
# Using config file
inform docs-config.yaml

# Or crawl URLs individually
for url in $(cat doc-urls.txt); do
  inform "$url" --output-dir ./docs-full --delay 300
done
```

**Monitor progress:**
```bash
find ./docs-full -name "*.md" | wc -l
du -sh ./docs-full
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

Common documentation structures:
- Getting Started / Quick Start
- Core Concepts / Fundamentals
- Guides / Tutorials
- API Reference
- Troubleshooting / FAQ

## Step 6: Create Skill File

**DO NOT merge all files into one giant file!**

Create a **concise, well-structured skill** (5-15KB):

```markdown
---
name: tool-name
description: Brief description
---

# Tool Name

## Core Concepts
Brief explanation with essential details

## Installation
Essential installation commands

## Quick Start
Practical examples with real commands

## Common Use Cases
Use case with code example

## Configuration
Essential configuration examples

## Best Practices
1. Practice 1
2. Practice 2

## Troubleshooting
Common issues and solutions

## Resources
- Official docs link
- GitHub link
```

**Skill best practices:**
- ✅ Keep under 15KB (ideally 5-10KB)
- ✅ Focus on practical examples
- ✅ Use clear sections
- ✅ Include code examples
- ❌ Don't dump all docs
- ❌ Don't create files over 50KB

## Step 7: Validate Skill Quality

```bash
# Check file size (should be under 15KB)
ls -lh skill-name.md

# Check line count (ideally 200-500 lines)
wc -l skill-name.md
```

**Quality checklist:**
- [ ] Frontmatter with name and description
- [ ] Clear introduction
- [ ] Installation instructions
- [ ] Quick start examples
- [ ] Common use cases with code
- [ ] Best practices section
- [ ] Troubleshooting tips
- [ ] File size under 15KB

## Step 8: Clean Up Repository

After creating the skill file:

```bash
# Remove crawled content
rm -rf docs-full/ docs-crawl/ *-docs-full/

# Remove config files
rm -f *-config.yaml sitemap.xml doc-urls.txt

# Keep only skill files
mkdir -p skills/
mv *.md skills/
```

**Create .gitignore:**

```bash
cat > .gitignore << 'EOF'
# Crawl artifacts
docs-full/
docs-crawl/
*-docs-full/

# Config files
*-config.yaml
sitemap.xml
doc-urls.txt

# Temporary files
*.tmp
.DS_Store
EOF
```

## Example Workflow

Complete example crawling Atlas documentation:

```bash
# Step 1: Direct crawl
inform https://atlasgo.io/docs --output-dir ./atlas-docs --limit 100

# Step 2: Create config
cat > atlas-config.yaml << 'EOF'
globals:
  outputDir: ./atlas-docs-full
  delay: 500
  concurrency: 5

targets:
  - url: https://atlasgo.io/docs
  - url: https://atlasgo.io/getting-started
EOF

# Step 3: Batch crawl
inform atlas-config.yaml

# Step 4: Analyze
find ./atlas-docs-full -name "*.md" | wc -l

# Step 5: Create skill
# (Manually create concise atlas.md file)

# Step 6: Validate
ls -lh atlas.md

# Step 7: Clean up
rm -rf atlas-docs-full/ atlas-config.yaml
```

## Tips for Success

1. **Start small** - Test with a few pages first
2. **Respect rate limits** - Use appropriate delays and concurrency
3. **Check robots.txt** - Verify you're allowed to crawl
4. **Verify content quality** - Review extracted markdown
5. **Focus on value** - Prioritize most useful content
6. **Keep skills concise** - Quality over quantity
7. **Iterate** - Refine based on usage

## Advanced Techniques

For advanced scenarios like large sites, dynamic content, authentication, and handling special cases, see [advanced.md](./advanced.md).

## Common Issues & Troubleshooting

For detailed troubleshooting including robots.txt blocking, rate limiting, incomplete crawls, and poor content extraction, see [troubleshooting.md](./troubleshooting.md).

## Resources

- **inform GitHub**: https://github.com/fwdslsh/inform
- **Skill Development**: Use `/plugin-dev:skill-development` for skill best practices
- **Plugin Development**: Use `/plugin-dev:create-plugin` for end-to-end plugin creation

## Complete Workflow Checklist

- [ ] **Step 0:** Check inform installation
- [ ] **Step 1:** Try direct crawl
- [ ] **Step 2:** Fetch sitemap if needed
- [ ] **Step 3:** Filter documentation URLs
- [ ] **Step 4:** Batch crawl
- [ ] **Step 5:** Analyze content structure
- [ ] **Step 6:** Create concise skill (5-15KB)
- [ ] **Step 7:** Validate quality
- [ ] **Step 8:** Clean up repository

**Remember:** The goal is a **concise, practical skill**, not a complete documentation dump!
