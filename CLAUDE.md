# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains quality check hooks for Claude Code that automatically validate and fix code when files are edited. The hooks integrate with Claude Code's PostToolUse event system to provide real-time feedback on TypeScript, ESLint, and Prettier issues.

## Key Architecture

### Hook System

- Hooks are triggered after Write/Edit/MultiEdit operations on TypeScript/JavaScript files
- Each project type (react-app, vscode-extension) has its own quality check implementation
- Exit code 0 = success (silent), exit code 2 = quality issues found (blocking)

### Core Components

1. **settings.local.json**: Configures which hooks run and when
2. **quality-check.js**: Main validation logic for each project type
3. **hook-config.json**: Defines rules, severity levels, and allowed patterns
4. **tsconfig-cache.json**: Auto-generated cache mapping project paths to TypeScript configs

### Quality Checks Performed

1. TypeScript compilation checks (using project's tsconfig.json)
2. ESLint with auto-fix capability
3. Prettier formatting with auto-fix
4. Custom rule validation (console usage, 'as any', debugger, TODOs)

## Development Commands

Since this is a hooks collection without a traditional build system:

```bash
# Test a hook manually
node .claude/hooks/react-app/quality-check.js /path/to/file.tsx

# View hook configuration
cat .claude/hooks/react-app/hook-config.json

# Clear TypeScript cache if needed
rm .claude/hooks/*/tsconfig-cache.json
```

## Working with Hooks

### Adding New Project Types

1. Create a new directory under `.claude/hooks/`
2. Copy `quality-check.js` and `hook-config.json` from an existing hook
3. Modify rules and configuration for the new project type
4. Update `settings.local.json` to include the new hook

### Modifying Rules

Edit `hook-config.json` to:

- Change rule severity (info/warning/error)
- Add allowed patterns for specific rules
- Adjust TypeScript/ESLint/Prettier behavior

### Cache Management

The hooks use SHA256 checksums to detect when TypeScript configurations change. The cache is automatically invalidated when:

- tsconfig.json files are modified
- New TypeScript projects are added
- Cache files are deleted

## Important Notes

- Hooks run synchronously and must complete quickly to avoid blocking Claude Code
- Exit code 2 will prevent Claude Code from saving the file changes
- The hooks expect the project to have its own TypeScript, ESLint, and Prettier configurations
- No npm install needed - hooks use the project's existing node_modules

## Writing Style Guidelines

**IMPORTANT**:

Write facts, not fluff. Every word must justify its existence.

Banned Elements

- No emojis (❌ 🚀 ✅ 📖 etc.)
- No marketing language or praise
- No "thank you" pleasantries
- No explanatory preambles ("Here is...", "This is...")
- No adjectives that don't add information

**Banned Words**:

<banned-words>
- powerful, seamless, comprehensive, robust, elegant
- enhanced, amazing, great, awesome, wonderful, excellent
- sophisticated, advanced, intuitive, user-friendly
- cutting-edge, state-of-the-art, innovative, revolutionary
</banned-words>

**What TO Write**:

<rules>
- Facts only - What it does, not why it's good
- Direct statements - Start with the point
- Concrete specifics - Numbers, not abstractions
- Technical accuracy - Precise, not approximate
</rules>

**Examples**:

**BAD**:

<bad>
- "Powerful semantic search for enhanced code understanding"
- "This elegantly handles parsing with optimal performance"
- "Thank you for making Codanna better!"
- "Here is the content of the file..."
</bad>

**GOOD**:

<good>
- "Search code using natural language queries"
- "Parse source code and extract symbols"
- "Contributing improves code intelligence performance"
- [Just show the content directly]
</good>

**Commit Messages**:

<rules>
- Short, factual subject line
- Bullet points for changes
- One line explaining critical impact (if any)
- No storytelling or justification
</rules>

**Documentation**

<rules>
- Lead with what it does
- Skip the "why it's special"
- Use headings without emojis
- End when done (no thank-yous or summaries)
</rules>

**Comments**

<rules>
- What the code does, not why it's clever
- No added comments unless requested
- Facts about behavior, not quality
</rules>

**The Secret**

<system-reminder>
Write like you're writing code comments for yourself. Facts only. No adjectives.
</system-reminder>
