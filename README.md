# Claude Code Quality Hooks

![Claude Code](https://img.shields.io/badge/Claude_Code-Compatible-4A90E2?logo=anthropic&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-Powered-3178C6?logo=typescript&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-Runtime-339933?logo=node.js&logoColor=white)
![Hooks](https://img.shields.io/badge/Hooks-PostToolUse-FF6B6B?logo=git&logoColor=white)

Quality checks for different project types.

[Official Documentation](https://docs.anthropic.com/en/docs/claude-code/hooks)

## Quick Start

### 1. Choose Your Project Type

```bash
# React/Next.js/Vite apps
.claude/hooks/react-app/

# VS Code extensions
.claude/hooks/vscode-extension/

# Node.js TypeScript projects
.claude/hooks/node-typescript/

# More coming soon...
```

### 2. Configure Claude Code

Add to `.claude/settings.local.json` and customize "command" quality-check.js path according to your project type (react-app, vscode-extension, node-typescript):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "node .claude/hooks/react-app/quality-check.js"
          }
        ]
      }
    ]
  }
}
```

### 3. Done

Hook runs automatically when you edit files.

## Features

- Catches errors before runtime
- Uses project-specific TypeScript configurations
- Auto-fixes trivial issues
- Project-aware rules (React hooks allow console.log, VS Code extensions don't)
- Immediate feedback during editing
- Prevents regressions

## Project Types

### React App

- Console allowed in components
- 'as any' warnings (not errors)
- JSX support

### VS Code Extension

- Console blocked in extension code
- Console allowed in webview
- Strict TypeScript

### Node.js TypeScript

- Console allowed (acceptable for CLI tools)
- 'as any' warnings (not errors)
- Optimized for server/client transport code

## Configuration

Each hook has `hook-config.json`:

```json
{
  "typescript": {
    "enabled": true,
    "showDependencyErrors": false,
    "jsx": "react" // For React hooks
  },
  "eslint": {
    "enabled": true,
    "autofix": true
  },
  "prettier": {
    "enabled": true,
    "autofix": true
  },
  "rules": {
    "console": {
      "severity": "info|warning|error",
      "allowIn": {
        "paths": ["src/components/"],
        "fileTypes": ["component", "test"]
      }
    }
  }
}
```

### Environment Variable Overrides

```bash
# Disable specific checks
export CLAUDE_HOOKS_PRETTIER_ENABLED=false

# Enable debug mode
export CLAUDE_HOOKS_DEBUG=true

# Show dependency errors (not recommended)
export CLAUDE_HOOKS_SHOW_DEPENDENCY_ERRORS=true
```

## Testing

```bash
# Test manually
echo '{"tool_name":"Edit","tool_input":{"file_path":"src/App.tsx"}}' | node .claude/hooks/react-app/quality-check.js

# Run with debug output
export CLAUDE_HOOKS_DEBUG=true
echo '{"tool_name":"Edit","tool_input":{"file_path":"src/App.tsx"}}' | node .claude/hooks/react-app/quality-check.js
```

## Creating New Hooks

1. Copy existing hook folder
2. Edit `hook-config.json`
3. Customize rules as needed

## Exit Codes

- **Exit 0**: All checks passed
  - File quality verified
  - Auto-fixes applied if enabled
  - Hook output suppressed by Claude Code

- **Exit 2**: Issues found
  - TypeScript compilation errors
  - ESLint errors that couldn't be auto-fixed
  - Prettier issues (if auto-fix disabled)
  - Full error report shown to agent

When running via Claude Code hooks:
- **Exit 0**: Silent success (no output shown)
- **Exit 2**: Full error report shown to user

When testing manually:
- All `[INFO]`, `[OK]`, `[WARN]` messages visible
- Useful for debugging hook behavior

## Under the Hook

### Hook Flow

```mermaid
graph TD
    A[File Edit Detected] --> B[Hook Triggered]
    B --> C{Parse Tool Input}
    C --> D[Extract File Path]
    D --> E[Load Configuration]
    E --> F[Smart Cache Check]
    F --> G{Config Changed?}
    G -->|Yes| H[Rebuild TS Mappings]
    G -->|No| I[Use Cached Mappings]
    H --> J[Run Quality Checks]
    I --> J
    J --> K[TypeScript Check]
    J --> L[ESLint Check]
    J --> M[Prettier Check]
    J --> N[Common Issues Check]
    K --> O{Has Errors?}
    L --> O
    M --> O
    N --> O
    O -->|Yes| P[Exit Code 2 - Block]
    O -->|No| Q[Exit Code 0 - Pass]
```

### Smart TypeScript Config Cache

The hook uses a caching system to handle complex TypeScript configurations:

**1. Config Discovery**

- Finds all `tsconfig*.json` files in your project
- Sorts by specificity (specific configs like `tsconfig.webview.json` take precedence)
- Builds a mapping of file patterns to configs

**2. SHA256 Change Detection**

```javascript
// Each config file gets a SHA256 hash
const hash = crypto.createHash('sha256').update(configFileContent).digest('hex');
```

This creates a unique fingerprint for each config:

- `tsconfig.json`: `c70342f6265640de2ba06a522870b4dc...`
- `tsconfig.webview.json`: `55e05e47dc57fbdab2a2d30704f9ab1f...`

**3. Cache Structure**

```json
{
  "hashes": {
    "tsconfig.json": "c70342f6265640de2ba06a522870b4dc1a4737818abe862c41108014cf442735",
    "tsconfig.webview.json": "55e05e47dc57fbdab2a2d30704f9ab1f6d9f312ee7c14e83b7a3613e73b4a230"
  },
  "mappings": {
    "src/webview/**/*": {
      "configPath": "tsconfig.webview.json",
      "excludes": ["node_modules", "test"]
    },
    "src/protocol/**/*": {
      "configPath": "tsconfig.webview.json",
      "excludes": ["node_modules", "test"]
    },
    "src/**/*": {
      "configPath": "tsconfig.test.json",
      "excludes": ["node_modules", "gui"]
    }
  }
}
```

**4. Cache Lifecycle**

```mermaid
graph LR
    A[Hook Starts] --> B{Cache Exists?}
    B -->|No| C[Build Cache]
    B -->|Yes| D[Load Cache]
    D --> E{Verify SHA256 Hashes}
    E -->|Changed| F[Rebuild Affected Mappings]
    E -->|Same| G[Use Cached Mappings]
    F --> H[Update Cache File]
    C --> H
    G --> I[Run Checks]
    H --> I
```

**5. Performance & Benefits**

- First run: ~100-200ms to build cache
- Subsequent runs: <5ms to verify hashes using SHA256
- Config change: Only rebuilds affected mappings
- Result: 95%+ faster on repeated runs
- SHA256 ensures cache validity (cryptographically secure, collision resistant)
- Each hook maintains its own cache
- Skip expensive file system operations

**Why manual hashing vs TypeScript project references?**

Manual SHA256 hashing provides <5ms cache lookups compared to 100-500ms for `tsc` incremental checks. The cache maintains reference maps to actual config paths while delivering superior performance for real-time editing scenarios.

### Check Execution

**1. TypeScript Compilation**

- Uses the correct config for the edited file
- Only shows errors for the edited file (not dependencies)
- Respects JSX settings from config

**2. ESLint Integration**

- Auto-fixes issues when possible
- Re-runs after fixes to verify
- Respects project ESLint config

**3. Prettier Formatting**

- Auto-formats on save
- Silent when successful
- Uses project Prettier config

**4. Common Issues Detection**

- Console usage (configurable per project type)
- `as any` usage (error vs warning)
- TODO/FIXME comments (informational)

### Troubleshooting

**Cache issues?**

```bash
# Clear the hook's cache
rm .claude/hooks/react-app/tsconfig-cache.json
```


**Hook not running?**

```bash
# Check if executable
chmod +x .claude/hooks/*/quality-check.js
```

## Questions

Check individual hook folders for more details.
