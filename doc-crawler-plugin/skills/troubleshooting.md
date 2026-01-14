# Troubleshooting Guide

Common issues and solutions for the documentation crawler skill.

## Installation Issues

### Issue: inform not found

**Symptoms:**
```
command not found: inform
```

**Solutions:**

1. **Quick install script (recommended)**
   ```bash
   curl -fsSL https://raw.githubusercontent.com/fwdslsh/inform/main/install.sh | sh
   ```

2. **Via npm**
   ```bash
   npm install -g @fwdslsh/inform
   which inform  # Verify installation
   ```

3. **Via Bun**
   ```bash
   bun install -g @fwdslsh/inform
   ```

4. **Verify installation**
   ```bash
   inform --version
   inform --help
   ```

### Issue: Permission denied

**Symptoms:**
```
Permission denied: /usr/local/bin/inform
```

**Solution:**
```bash
# Reinstall with proper permissions
curl -fsSL https://raw.githubusercontent.com/fwdslsh/inform/main/install.sh | sh

# Or check permissions
ls -la /usr/local/bin/inform
chmod +x /usr/local/bin/inform
```

## Crawl Issues

### Issue: robots.txt blocking

**Symptoms:**
```
Blocked by robots.txt
No content crawled
```

**Causes:**
- Site's robots.txt restricts crawling
- Overly aggressive crawling settings

**Solutions:**

1. **Check robots.txt first**
   ```bash
   curl https://example.com/robots.txt
   ```

2. **Use appropriate delays**
   ```bash
   inform https://example.com \
     --delay 1000 \
     --concurrency 2
   ```

3. **Only crawl if permitted!**
   - If robots.txt blocks crawling, respect that
   - Contact site owner if you need access
   - Look for publicly available exports

4. **Alternative: Find alternative sources**
   - GitHub repositories (often have docs)
   - API documentation
   - Official documentation mirrors

### Issue: Rate limiting

**Symptoms:**
```
HTTP 429: Too Many Requests
Blocked after some pages crawled
```

**Solutions:**

1. **Increase delay between requests**
   ```bash
   inform https://example.com \
     --delay 1000 \
     --concurrency 2
   ```

2. **Reduce concurrency**
   ```bash
   # Default is 5, reduce to 2-3
   inform https://example.com --concurrency 2
   ```

3. **Crawl in smaller batches**
   ```bash
   # Instead of crawling entire site at once
   inform https://example.com/docs/section1 --limit 50
   inform https://example.com/docs/section2 --limit 50
   ```

4. **Try later**
   ```bash
   # Crawl during off-peak hours
   # Run the crawl at a different time
   ```

5. **Contact site administrator**
   - Explain your use case
   - Ask for dedicated crawling time window
   - Request documentation export

### Issue: Incomplete crawl

**Symptoms:**
```
Only crawled 5 pages, expected 100+
Missing important sections
```

**Solutions:**

1. **Use sitemap approach**
   ```bash
   # Fetch sitemap
   curl https://example.com/sitemap.xml -o sitemap.xml

   # Extract documentation URLs
   grep '/docs/' sitemap.xml | grep -oP '(?<=<loc>)[^<]+' > doc-urls.txt

   # Crawl from URL list
   for url in $(cat doc-urls.txt); do
     inform "$url" --output-dir ./docs --delay 200
   done
   ```

2. **Increase crawl limit**
   ```bash
   # Default limit is usually low, increase it
   inform https://example.com --limit 500
   ```

3. **Manually specify important URLs**
   ```yaml
   # config.yaml
   globals:
     outputDir: ./docs
     limit: 500

   targets:
     - url: https://example.com/docs/getting-started
     - url: https://example.com/docs/concepts
     - url: https://example.com/docs/api
   ```

4. **Check for dynamic content**
   - Some pages load content via JavaScript
   - inform might not execute JavaScript
   - Check if there's a static version or API

### Issue: Poor content extraction

**Symptoms:**
```
HTML/JavaScript in output markdown
Formatting is broken
Tables not extracted properly
```

**Solutions:**

1. **Review sample output first**
   ```bash
   # Test with small sample
   inform https://example.com --limit 3

   # Check output quality
   head -50 docs/index.md
   ```

2. **Try with --raw flag**
   ```bash
   # Get raw HTML instead of markdown
   inform https://example.com --raw > output.html
   ```

3. **Manually convert important pages**
   ```bash
   # Use Pandoc to convert HTML to Markdown
   pandoc page.html -t markdown -o page.md
   ```

4. **Use WebFetch as alternative**
   ```
   # Claude can fetch and convert pages
   WebFetch: https://example.com/docs/page
   "Convert this documentation to clean markdown"
   ```

### Issue: Special characters or encoding issues

**Symptoms:**
```
Garbled text in output
Accents/unicode not displaying
Chinese/Japanese characters broken
```

**Solutions:**

1. **Check file encoding**
   ```bash
   file docs/page.md
   ```

2. **Convert to UTF-8**
   ```bash
   iconv -f ISO-8859-1 -t UTF-8 input.md > output.md
   ```

3. **Clean up characters in batch**
   ```bash
   find ./docs -name "*.md" -exec sh -c '
     iconv -f ISO-8859-1 -t UTF-8 "$1" -o "$1.utf8"
     mv "$1.utf8" "$1"
   ' _ {} \;
   ```

## Content Issues

### Issue: No documentation found

**Symptoms:**
```
Crawl completes but crawled 0 pages
Output directory is empty
```

**Solutions:**

1. **Verify the URL**
   ```bash
   # Test URL in browser first
   curl https://example.com/docs | head
   ```

2. **Try different URL formats**
   ```bash
   inform https://example.com/docs
   inform https://docs.example.com
   inform https://example.com/documentation
   ```

3. **Check if documentation exists**
   - Some projects only have README
   - Some have documentation behind login
   - Some don't have online documentation

4. **Look for alternatives**
   - GitHub repository docs
   - API documentation
   - WikiBooks or community docs
   - PDF documentation

### Issue: Too much non-documentation content

**Symptoms:**
```
Crawled navigation, footers, advertisements
Too much bloat in output
```

**Solutions:**

1. **Filter by URL pattern**
   ```bash
   # Only crawl /docs/ section
   inform https://example.com --filter '.*/docs/.*'
   ```

2. **Exclude patterns**
   ```bash
   # Exclude common non-doc paths
   # (inform may support exclusion, check --help)
   ```

3. **Manual cleanup after crawl**
   ```bash
   # Remove files from non-doc directories
   find ./docs -path '*/sidebar*' -delete
   find ./docs -path '*/navbar*' -delete
   find ./docs -path '*/footer*' -delete
   ```

4. **Use WebFetch for selective extraction**
   ```
   WebFetch: https://example.com/docs/page
   "Extract only the main documentation content, ignore navigation"
   ```

## Skill Creation Issues

### Issue: Skill file too large

**Symptoms:**
```
Skill file exceeds 50KB
Takes too long to load
Uses too much context
```

**Solutions:**

1. **Split into multiple skills**
   ```bash
   # Instead of one 100KB skill
   # Create multiple smaller skills
   - tool-basics.md (10KB)
   - tool-advanced.md (12KB)
   - tool-api.md (15KB)
   ```

2. **Remove less important content**
   - Delete obsolete examples
   - Remove deprecated features
   - Shorten verbose explanations

3. **Use references**
   ```markdown
   # My Tool Basics

   For advanced usage, see [advanced.md](advanced.md)
   For API reference, see [api.md](api.md)
   ```

4. **Create external documentation**
   - Keep detailed docs in separate files
   - Link to them from skill
   - Keep skill concise and focused

### Issue: Skill file too small

**Symptoms:**
```
Skill file is only 1-2KB
Not enough content to be useful
```

**Solutions:**

1. **Add more examples**
   - Include more use cases
   - Add code examples
   - Show common patterns

2. **Expand explanations**
   - Explain key concepts better
   - Add context for beginners
   - Include troubleshooting section

3. **Add practical content**
   - Best practices
   - Performance tips
   - Integration examples

4. **Combine related sections**
   - If creating 3 files under 5KB, consider combining
   - Keep related content together
   - Balance between focused and comprehensive

## File System Issues

### Issue: Permission denied when writing files

**Symptoms:**
```
Permission denied: ./docs
Cannot create directory
```

**Solutions:**

```bash
# Check current directory
pwd

# Ensure you have write permissions
ls -ld .

# Create directory with proper permissions
mkdir -p docs
chmod 755 docs

# Or use sudo (less preferred)
sudo inform https://example.com --output-dir ./docs
```

### Issue: Disk space full

**Symptoms:**
```
No space left on device
Crawl stops mid-way
```

**Solutions:**

1. **Check disk space**
   ```bash
   df -h
   du -sh docs/
   ```

2. **Delete old crawls**
   ```bash
   rm -rf docs-old/
   rm -rf previous-crawls/
   ```

3. **Reduce crawl size**
   ```bash
   # Crawl smaller sections
   inform https://example.com/docs --limit 100
   ```

4. **Use external drive**
   ```bash
   inform https://example.com --output-dir /mnt/external/docs
   ```

## Git & Version Control Issues

### Issue: Committing large binary files

**Symptoms:**
```
Git object is too large
Push fails
```

**Solutions:**

1. **Don't commit crawled content**
   ```bash
   # Add to .gitignore
   echo "docs-crawl/" >> .gitignore
   echo "*-docs-full/" >> .gitignore

   # Remove if already committed
   git rm -r docs-crawl/
   git commit -m "Remove crawled artifacts"
   ```

2. **Commit only final skill files**
   ```bash
   git add skills/*.md
   git add README.md
   git commit -m "Add tool documentation skill"
   ```

3. **Use Git LFS for large files** (if absolutely needed)
   ```bash
   git lfs install
   git lfs track "*.pdf"
   ```

## Getting More Help

If issues persist:

1. **Check inform documentation**
   ```bash
   inform --help
   ```

2. **Look at inform GitHub issues**
   - https://github.com/fwdslsh/inform/issues
   - Search for your error message

3. **Use Claude Code features**
   - Use `/plugin-dev:skill-development` for skill guidance
   - Ask Claude for help processing documentation
   - Use WebFetch for manual content extraction

4. **Manual Approach**
   - If automated crawling fails, manually download docs
   - Use browser developer tools to inspect pages
   - Ask site owner for documentation export
