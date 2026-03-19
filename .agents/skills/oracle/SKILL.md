---
name: oracle
description: Consult the Oracle - an AI advisor powered by OpenAI's GPT-5.2 reasoning model that can plan, review, and provide expert guidance. NOTE: If you are ampcode, skip this skill.
---

# Oracle

Consult the Oracle - an AI advisor powered by OpenAI's GPT-5.2 reasoning model that can plan, review, and provide expert guidance.

The Oracle acts as your senior engineering advisor and can help with:

## WHEN TO USE THE ORACLE:
- Code reviews and architecture feedback
- Finding a bug in multiple files
- Planning complex implementations or refactoring
- Analyzing code quality and suggesting improvements
- Answering complex technical questions that require deep reasoning

## WHEN NOT TO USE THE ORACLE:
- Simple file reading or searching tasks (use Read or Grep directly)
- Codebase searches (use finder)
- Web browsing and searching (use `perplexity` MCP: `mcp__perplexity__search`, `mcp__perplexity__reason`, `mcp__perplexity__deep_research`)
- Basic code modifications and when you need to execute code changes (do it yourself or use Task)

## USAGE GUIDELINES:
1. Be specific about what you want the Oracle to review, plan, or debug
2. Provide relevant context about what you're trying to achieve. If you know that 3 files are involved, list them and they will be attached.

## Examples

### Review the authentication system architecture and suggest improvements
```json
{"task":"Review the authentication architecture and suggest improvements","files":["src/auth/index.ts","src/auth/jwt.ts"]}
```

### Plan the implementation of real-time collaboration features
```json
{"task":"Plan the implementation of real-time collaboration feature"}
```

### Analyze the performance bottlenecks in the data processing pipeline
```json
{"task":"Analyze performance bottlenecks","context":"Users report slow response times when processing large datasets"}
```

### Review this API design and suggest better patterns
```json
{"task":"Review API design","context":"This is a REST API for user management","files":["src/api/users.ts"]}
```

### Debug failing tests after refactor
```json
{"task":"Help debug why tests are failing","context":"Tests fail with \"undefined is not a function\" after refactoring the auth module","files":["src/auth/auth.test.ts"]}
```

### Preferred Models
1. GPT-5.2 reasoning
2. OPOS 4-5 thinking
3. Gemini 3 Pro
