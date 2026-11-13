---
name: ruby-mcp-server-generator
description: 'Generate a complete Model Context Protocol server project in Ruby using the official MCP Ruby SDK gem.'
---

# Ruby MCP Server Generator

Generate a complete, production-ready MCP server in Ruby using the official Ruby SDK.

## Project Generation

When asked to create a Ruby MCP server, generate a complete project with this structure:

```
my-mcp-server/
├── Gemfile
├── Rakefile
├── lib/
│   ├── my_mcp_server.rb
│   ├── my_mcp_server/
│   │   ├── server.rb
│   │   ├── tools/
│   │   │   ├── greet_tool.rb
│   │   │   └── calculate_tool.rb
│   │   ├── prompts/
│   │   │   └── code_review_prompt.rb
│   │   └── resources/
│   │       └── example_resource.rb
├── bin/
│   └── mcp-server
├── test/
│   ├── test_helper.rb
│   └── tools/
│       ├── greet_tool_test.rb
│       └── calculate_tool_test.rb
└── README.md
```

## Project and Source Templates

Gemfile, Rakefile, server, tools, prompts, resources, and executable. See [references/source-templates.md](references/source-templates.md) for:

- Gemfile Template
- Rakefile Template
- lib/my_mcp_server.rb Template
- lib/my_mcp_server/server.rb Template
- lib/my_mcp_server/tools/greet_tool.rb Template
- lib/my_mcp_server/tools/calculate_tool.rb Template
- lib/my_mcp_server/prompts/code_review_prompt.rb Template
- lib/my_mcp_server/resources/example_resource.rb Template
- bin/mcp-server Template

## Test and README Templates

Minitest helper, tool tests, and project README. See [references/test-and-readme-templates.md](references/test-and-readme-templates.md) for:

- test/test_helper.rb Template
- test/tools/greet_tool_test.rb Template
- test/tools/calculate_tool_test.rb Template
- README.md Template

## Generation Instructions

1. **Ask for project name and description**
2. **Generate all files** with proper naming and module structure
3. **Use classes for tools and prompts** for better organization
4. **Include input/output schemas** for type safety
5. **Add tool annotations** for behavior hints
6. **Include structured content** in responses
7. **Implement comprehensive tests** for all tools
8. **Follow Ruby conventions** (snake_case, modules, frozen_string_literal)
9. **Add proper error handling** with is_error flag
10. **Provide both stdio and HTTP** usage examples
