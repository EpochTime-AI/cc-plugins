---
name: doc-crawler
description: This skill should be used when the user asks to "crawl documentation", "extract docs from website", "convert website to markdown", "download documentation site", "scrape docs", "use inform tool", "create skill from docs", or wants guidance on systematically crawling documentation websites using inform tool, handling sitemaps, filtering URLs, and organizing content into structured markdown or skills.
---

# Documentation Crawler Skill

This skill guides you through crawling documentation websites and organizing them either as structured documentation or as Claude Code skills.

## Overview

The documentation crawler follows a systematic workflow:

1. **Prerequisites Check** - Verify inform is installed
2. **Initial Crawl** - Try crawling the documentation URL directly
3. **Sitemap Discovery** - Fetch and parse sitemap.xml if needed
4. **URL Filtering** - Extract documentation URLs, exclude blogs/marketing
5. **Batch Crawling** - Use inform to crawl all documentation pages
6. **Content Organization** - Analyze and structure the content
7. **Choose Output Format** - Create skills or organize as documentation
8. **Repository Cleanup** - Remove intermediate artifacts

## Prerequisites Check

**Important: Choose the right download method**

Before using inform, determine if you need it:

- **✅ Direct download (no inform needed)**: If the documentation is provided as plain text files (.txt, .md, .rst, etc.), you can download them directly using `curl`, `wget`, or WebFetch
  ```bash
  # Example: Download markdown documentation directly
  curl -O https://example.com/docs/guide.md
  wget https://example.com/docs/api-reference.txt
  ```

- **⚠️ Use inform (required)**: If the documentation is served as HTML web pages, you must use inform to crawl and convert to markdown
  ```bash
  # Example: Crawl HTML documentation
  inform https://example.com/docs --output-dir ./docs
  ```

**Quick decision rule:**
- Plain text formats (.md, .txt, .rst, .adoc) → Direct download with curl/wget
- HTML web pages → Use inform tool

### Install inform (if needed)

Before starting with inform, verify it's installed:

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

## Step 6: Choose Output Format

**⚠️ CRITICAL: Always ask the user first!**

Before proceeding, you **MUST** ask the user what they want to do with the crawled documentation:

1. **What is the desired output?**
   - Just organize documentation files?
   - Create a Claude Code skill?
   - Create a plugin?

2. **If creating skills or plugins:**
   - Read [references/skills-guide.md](references/skills-guide.md) to understand the complete skill creation process
   - Determine whether they need:
     - A personal skill in `~/.claude/skills/` (user-specific)
     - A project skill in `.claude/skills/` (shared with team)
     - A plugin skill in a plugin directory (distributed via marketplace)

3. **Clarify the structure based on the documentation:**
   - Understand the target location from skills-guide.md
   - Follow the appropriate pattern for that location
   - Ensure proper structure and frontmatter

**DO NOT assume what the user wants. Always ask and confirm before proceeding.**

---

After clarifying with the user, choose the appropriate path:

### Option A: Organize as Documentation Only

Simply organize the crawled markdown files into a clean directory structure:

```bash
# Create organized documentation structure
mkdir -p docs/
cp -r ./docs-full/* docs/

# Optional: Organize by category
mkdir -p docs/{getting-started,guides,api-reference,troubleshooting}
# Move files to appropriate directories based on content
```

**Use this when:**
- You want to keep the full documentation for reference
- The documentation will be accessed directly as markdown files
- You're building a documentation repository

**Then proceed to Step 8 (Cleanup)**

### Option B: Create Claude Code Skills

**📖 READ [references/skills-guide.md](references/skills-guide.md) for complete skill structure details.**

After reading the guide, create skills based on the target location:

**Skill types summary:**

1. **Personal skills** (`~/.claude/skills/`): User-specific, self-contained (10-20KB)
2. **Project skills** (`.claude/skills/`): Team-shared, self-contained (10-20KB)
3. **Plugin skills** (plugin directory): Marketplace distribution, concise SKILL.md (5-15KB) + references/

**Key principles:**
- Personal/Project skills: Self-contained, no references directory
- Plugin skills: Concise main file + detailed references/
- All types: Clear frontmatter, practical examples, focused content
- Refer to skills-guide.md for detailed structure and best practices

## Step 7: Validate Quality

### For Documentation Organization:

```bash
# Check directory structure
tree docs/ -L 2

# Check total size
du -sh docs/
```

### For Skills:

```bash
# Check SKILL.md size (plugin: under 15KB, .claude/: under 20KB)
ls -lh skills/tool-name/SKILL.md

# Check line count (plugin: 200-500 lines, .claude/: 300-700 lines)
wc -l skills/tool-name/SKILL.md

# Validate plugin structure if creating plugin skill
claude plugin validate .
```

**Quality checklist for skills:**
- [ ] ⚠️ Asked user about desired output format and skill type
- [ ] 📖 Read skills-guide.md and followed structure requirements
- [ ] Frontmatter with name and description
- [ ] Practical examples and use cases with code
- [ ] Appropriate file size (plugin: 5-15KB, personal/project: 10-20KB)
- [ ] (Plugin only) References directory with detailed docs

## Step 8: Clean Up Repository

After organizing documentation or creating skills:

```bash
# Remove temporary crawl directories
rm -rf docs-full/ docs-crawl/ *-docs-full/ *-crawl/

# Remove temporary config files
rm -f *-config.yaml sitemap.xml doc-urls.txt *.tmp

# Keep final output
# - If Option A: keep docs/ directory
# - If Option B: keep skills/ directory
```

**Update .gitignore:**

```bash
cat > .gitignore << 'EOF'
# Crawl artifacts (temporary)
docs-full/
docs-crawl/
*-docs-full/
*-crawl/

# Config files (temporary)
*-config.yaml
sitemap.xml
doc-urls.txt

# Temporary files
*.tmp
.DS_Store

# System files
Thumbs.db
EOF
```

## Example Workflow

Complete example for creating a plugin skill:

```bash
# 1. Crawl documentation
inform https://example.com/docs --output-dir ./docs-crawl --limit 200

# 2. Analyze and create structure
find ./docs-crawl -name "*.md" | head -20
mkdir -p skills/tool-name/references

# 3. Create concise SKILL.md (refer to skills-guide.md for structure)
# Write SKILL.md with quick reference, common use cases, links to references

# 4. Organize full docs as references
cp -r ./docs-crawl/* skills/tool-name/references/

# 5. Validate and clean up
ls -lh skills/tool-name/SKILL.md
rm -rf docs-crawl/
```

For documentation-only or personal/project skills, adjust paths accordingly and refer to skills-guide.md.

## Tips for Success

1. **Ask first** - Always ask user what they want (docs vs skills, personal vs project vs plugin)
2. **Read skills-guide.md** - Essential for understanding skill structure and best practices
3. **Start small** - Test crawl with a few pages first
4. **Respect rate limits** - Use appropriate delays and concurrency
5. **Verify content quality** - Review extracted markdown before organizing
6. **Keep skills concise** - Quality over quantity, follow size guidelines from skills-guide.md

## Decision Tree

```
Crawled documentation
    │
    ⚠️ ASK USER: What do you want to do with the documentation?
    │
    ├─→ Just organize as documentation?
    │   └─→ Copy to docs/ directory → Cleanup → Done
    │
    └─→ Create a skill or plugin?
        │
        📖 READ skills-guide.md to understand skill types
        │
        ⚠️ ASK USER: Which type of skill?
        │
        ├─→ Personal skill (~/.claude/skills/)?
        │   ├─→ Read skills-guide.md for structure
        │   ├─→ Create ~/.claude/skills/tool-name/SKILL.md
        │   └─→ Self-contained (10-20KB)
        │
        ├─→ Project skill (.claude/skills/)?
        │   ├─→ Read skills-guide.md for structure
        │   ├─→ Create .claude/skills/tool-name/SKILL.md
        │   └─→ Self-contained (10-20KB), shared with team
        │
        └─→ Plugin skill (for marketplace)?
            ├─→ Read skills-guide.md for plugin structure
            ├─→ Create plugin/skills/tool-name/SKILL.md (5-15KB)
            ├─→ Create plugin/skills/tool-name/references/ (detailed docs)
            └─→ Link from SKILL.md to references
```

## Advanced Techniques

For advanced scenarios like large sites, dynamic content, authentication, and handling special cases, see [references/advanced.md](references/advanced.md).

## Common Issues & Troubleshooting

For detailed troubleshooting including robots.txt blocking, rate limiting, incomplete crawls, and poor content extraction, see [references/troubleshooting.md](references/troubleshooting.md).

## Resources

- **inform GitHub**: https://github.com/fwdslsh/inform
- **Skills Guide**: See [references/skills-guide.md](references/skills-guide.md) - **MUST READ** for understanding where and how to create skills
- **Skill Development**: Use `/plugin-dev:skill-development` for skill best practices
- **Plugin Development**: Use `/plugin-dev:create-plugin` for end-to-end plugin creation

## Complete Workflow Checklist

- [ ] **Step 0:** Check inform installation
- [ ] **Step 1:** Try direct crawl
- [ ] **Step 2:** Fetch sitemap if needed
- [ ] **Step 3:** Filter documentation URLs
- [ ] **Step 4:** Batch crawl
- [ ] **Step 5:** Analyze content structure
- [ ] **Step 6:** Choose output format (docs vs skills)
  - [ ] ⚠️ **ASK USER** what they want to do with the documentation
- [ ] **If creating skills:**
  - [ ] 📖 **READ skills-guide.md** to understand skill types and structure
  - [ ] ⚠️ **ASK USER** which type: personal/project/plugin?
  - [ ] Confirm target location with user
  - [ ] Determine skill type based on skills-guide.md
  - [ ] Create appropriate structure
  - [ ] (Personal/Project) Create self-contained SKILL.md (10-20KB)
  - [ ] (Plugin) Create concise SKILL.md (5-15KB) + references/
- [ ] **If organizing docs:**
  - [ ] Create organized directory structure
  - [ ] Copy and categorize files
- [ ] **Step 7:** Validate quality
- [ ] **Step 8:** Clean up temporary files

**Remember:**
- ⚠️ **ALWAYS ASK THE USER FIRST** - Never assume what they want
- 📖 **READ skills-guide.md** before creating skills - Essential for correct structure
- Clarify skill type: personal/project/plugin (each has different structure and size)
