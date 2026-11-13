---
name: swift-mcp-server-generator
description: 'Generate a complete Model Context Protocol server project in Swift using the official MCP Swift SDK package.'
---

# Swift MCP Server Generator

Generate a complete, production-ready MCP server in Swift using the official Swift SDK package.

## Project Generation

When asked to create a Swift MCP server, generate a complete project with this structure:

```
my-mcp-server/
├── Package.swift
├── Sources/
│   └── MyMCPServer/
│       ├── main.swift
│       ├── Server.swift
│       ├── Tools/
│       │   ├── ToolDefinitions.swift
│       │   └── ToolHandlers.swift
│       ├── Resources/
│       │   ├── ResourceDefinitions.swift
│       │   └── ResourceHandlers.swift
│       └── Prompts/
│           ├── PromptDefinitions.swift
│           └── PromptHandlers.swift
├── Tests/
│   └── MyMCPServerTests/
│       └── ServerTests.swift
└── README.md
```

## Package and Source Templates

Package.swift, entry point, server, and tool/resource/prompt definitions and handlers. See [references/source-templates.md](references/source-templates.md) for:

- Package.swift Template
- main.swift Template
- Server.swift Template
- ToolDefinitions.swift Template
- ToolHandlers.swift Template
- ResourceDefinitions.swift Template
- ResourceHandlers.swift Template
- PromptDefinitions.swift Template
- PromptHandlers.swift Template

## Test and README Templates

XCTest suite and project README. See [references/test-and-readme-templates.md](references/test-and-readme-templates.md) for:

- ServerTests.swift Template
- README.md Template

## Generation Instructions

1. **Ask for project name and description**
2. **Generate all files** with proper naming
3. **Use actor-based state** for thread safety
4. **Include comprehensive logging** with swift-log
5. **Implement graceful shutdown** with ServiceLifecycle
6. **Add tests** for all handlers
7. **Use modern Swift concurrency** (async/await)
8. **Follow Swift naming conventions** (camelCase, PascalCase)
9. **Include error handling** with proper MCPError usage
10. **Document public APIs** with doc comments

## Build and Run

```bash
# Build
swift build

# Run
swift run

# Test
swift test

# Release build
swift build -c release

# Install
swift build -c release
cp .build/release/MyMCPServer /usr/local/bin/
```

## Integration with Claude Desktop

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "my-mcp-server": {
      "command": "/path/to/MyMCPServer"
    }
  }
}
```
