---
name: java-mcp-server-generator
description: 'Generate a complete Model Context Protocol server project in Java using the official MCP Java SDK with reactive streams and optional Spring Boot integration.'
---

# Java MCP Server Generator

Generate a complete, production-ready MCP server in Java using the official Java SDK with Maven or Gradle.

## Project Generation

When asked to create a Java MCP server, generate a complete project with this structure:

```
my-mcp-server/
├── pom.xml (or build.gradle.kts)
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/mcp/
│   │   │       ├── McpServerApplication.java
│   │   │       ├── config/
│   │   │       │   └── ServerConfiguration.java
│   │   │       ├── tools/
│   │   │       │   ├── ToolDefinitions.java
│   │   │       │   └── ToolHandlers.java
│   │   │       ├── resources/
│   │   │       │   ├── ResourceDefinitions.java
│   │   │       │   └── ResourceHandlers.java
│   │   │       └── prompts/
│   │   │           ├── PromptDefinitions.java
│   │   │           └── PromptHandlers.java
│   │   └── resources/
│   │       └── application.properties (if using Spring)
│   └── test/
│       └── java/
│           └── com/example/mcp/
│               └── McpServerTest.java
└── README.md
```

## Build File Templates

Maven and Gradle build files. See [references/build-templates.md](references/build-templates.md) for:

- Maven pom.xml Template
- Gradle build.gradle.kts Template

## Java Source Templates

Application entry point, tool/resource/prompt definitions and handlers. See [references/source-templates.md](references/source-templates.md) for:

- McpServerApplication.java Template
- ToolDefinitions.java Template
- ToolHandlers.java Template
- ResourceDefinitions.java Template
- ResourceHandlers.java Template
- PromptDefinitions.java Template
- PromptHandlers.java Template

## Test and README Templates

JUnit test class and project README. See [references/test-and-readme-templates.md](references/test-and-readme-templates.md) for:

- McpServerTest.java Template
- README.md Template

## Generation Instructions

1. **Ask for project name and package**
2. **Choose build tool** (Maven or Gradle)
3. **Generate all files** with proper package structure
4. **Use Reactive Streams** for async handlers
5. **Include comprehensive logging** with SLF4J
6. **Add tests** for all handlers
7. **Follow Java conventions** (camelCase, PascalCase)
8. **Include error handling** with proper responses
9. **Document public APIs** with Javadoc
10. **Provide both sync and async** examples
