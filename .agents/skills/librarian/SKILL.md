---
name: librarian
description: Read-only research agent searching GitHub (public/private) for code samples/patterns. NEVER implements/edits files. NOTE: If you are ampcode, skip this skill.
---

# Librarian

A **read-only research agent** via `amp` CLI to search GitHub codebases.

## Rules & Capabilities
- **DO**: Search GitHub, analyze patterns, prioritize popular repos (⭐100+), provide code samples with links.
- **DO NOT**: Create/edit files, implement code, or run non-research commands.
- **Use for**: Finding reference implementations, library internals, or cross-repo analysis.

## Prerequisites
1. **Amp CLI** installed.
2. **GitHub Connection** for private repos.

## Usage

**Interactive Prompts:**
- "Use Librarian to lookup React's useEffect cleanup implementation"
- "Explain deployment process by searching our docs and infra repos"
- "Find TypeScript singleflight patterns with Redis"

**Agent Command:**
```bash
amp -m rush -x "Use Librarian to [QUERY]. Prioritize ⭐100+ repos. RESEARCH ONLY - no implementation."
```
