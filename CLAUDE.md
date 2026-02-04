# Zenn.dev AI Article Writer

This repository is auto-synced with zenn.dev for publishing Japanese technical articles.

## Project Structure

```
articles/     - Markdown articles (auto-published to zenn.dev)
books/        - Book content (chapters)
```

## Article Format

All articles must follow zenn.dev frontmatter format:

```markdown
---
title: "記事タイトル"
emoji: "🤖"
type: "tech"  # or "idea"
topics: ["ai", "llm", "claude"]
published: true  # false for drafts
---

本文...
```

## File Naming (CRITICAL)

**Zenn slug requirements - MUST follow:**
- Slug = filename without `.md` extension
- Length: **12-50 characters** (STRICT LIMIT)
- Characters: `a-z`, `0-9`, `-`, `_` only
- Format: `YYYY-MM-DD-short-topic.md`

**Examples:**
```
✅ 2026-01-20-multi-agent-systems.md (32 chars)
✅ 2026-01-20-claude-code-tips.md (28 chars)
❌ 2026-01-20-comprehensive-guide-to-building-multi-agent-systems-with-claude.md (TOO LONG)
```

**Before creating any article file:**
1. Count slug length (filename minus `.md`)
2. If > 50 chars, shorten the topic portion
3. Keep date prefix `YYYY-MM-DD-` (11 chars), leaving 39 chars max for topic

## Workflow

1. Create article in `articles/` directory
2. **Validate filename is 12-50 characters**
3. Commit and push to `main` branch
4. Zenn.dev auto-deploys within minutes

## Rate Limits

- Zenn has a 24-hour posting limit (exact number undisclosed)
- If rate-limited, articles deploy after 24 hours
- For bulk migrations, contact Zenn support for temporary increase

## Topics Focus

- AI Agents / マルチエージェント
- LLM Agents / LLMエージェント
- Claude Code techniques
- MCP (Model Context Protocol)
- Prompt engineering
- RAG systems

## Reference Links Format (CRITICAL for Qiita)

**DO NOT use markdown link format for references.**

Qiita does not render markdown links properly in some cases. Use plain text format:

```markdown
## 参考リンク

Article Title Here

https://example.com/full-url-here

Another Article Title

https://example.com/another-url
```

**WRONG (DO NOT USE):**
```markdown
- [Article Title](https://example.com/url)
```

**CORRECT:**
```markdown
Article Title

https://example.com/url
```

This ensures URLs are clickable on both Zenn and Qiita.

## Preview Locally

```bash
npx zenn preview
```

Opens http://localhost:8000 for local preview.
