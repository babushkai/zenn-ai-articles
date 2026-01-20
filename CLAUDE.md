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

## File Naming

- Articles: `articles/{slug}.md` (slug: lowercase, hyphens, no spaces)
- Example: `articles/2026-01-20-multi-agent-systems.md`

## Workflow

1. Create article in `articles/` directory
2. Commit and push to `main` branch
3. Zenn.dev auto-deploys within minutes

## Topics Focus

- AI Agents / マルチエージェント
- LLM Agents / LLMエージェント
- Claude Code techniques
- MCP (Model Context Protocol)
- Prompt engineering
- RAG systems

## Preview Locally

```bash
npx zenn preview
```

Opens http://localhost:8000 for local preview.
