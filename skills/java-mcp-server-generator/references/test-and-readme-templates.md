# Test and README Templates

## McpServerTest.java Template

```java
package com.example.mcp;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.node.ObjectNode;
import io.mcp.server.McpServer;
import io.mcp.server.McpSyncServer;
import io.mcp.server.tool.ToolResponse;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class McpServerTest {
    
    private McpSyncServer syncServer;
    private ObjectMapper objectMapper;
    
    @BeforeEach
    void setUp() {
        McpServer server = createTestServer();
        syncServer = server.toSyncServer();
        objectMapper = new ObjectMapper();
    }
    
    private McpServer createTestServer() {
        // Same setup as main application
        McpServer server = McpServerBuilder.builder()
            .serverInfo("test-server", "1.0.0")
            .capabilities(cap -> cap.tools(true))
            .build();
        
        // Register handlers
        ToolHandlers.register(server);
        
        return server;
    }
    
    @Test
    void testGreetTool() {
        ObjectNode args = objectMapper.createObjectNode();
        args.put("name", "Java");
        
        ToolResponse response = syncServer.callTool("greet", args);
        
        assertFalse(response.isError());
        assertEquals(1, response.getContent().size());
        assertTrue(response.getContent().get(0).getText().contains("Java"));
    }
    
    @Test
    void testCalculateTool() {
        ObjectNode args = objectMapper.createObjectNode();
        args.put("operation", "add");
        args.put("a", 5);
        args.put("b", 3);
        
        ToolResponse response = syncServer.callTool("calculate", args);
        
        assertFalse(response.isError());
        assertTrue(response.getContent().get(0).getText().contains("8"));
    }
    
    @Test
    void testDivideByZero() {
        ObjectNode args = objectMapper.createObjectNode();
        args.put("operation", "divide");
        args.put("a", 10);
        args.put("b", 0);
        
        ToolResponse response = syncServer.callTool("calculate", args);
        
        assertTrue(response.isError());
    }
}
```

## README.md Template

````markdown
# My MCP Server

A Model Context Protocol server built with Java and the official MCP Java SDK.

## Features

- ✅ Tools: greet, calculate
- ✅ Resources: example data, configuration
- ✅ Prompts: code-review
- ✅ Reactive Streams with Project Reactor
- ✅ Structured logging with SLF4J
- ✅ Full test coverage

## Requirements

- Java 17 or later
- Maven 3.6+ or Gradle 7+

## Build

### Maven
```bash
mvn clean package
```

### Gradle
```bash
./gradlew build
```

## Run

### Maven
```bash
java -jar target/my-mcp-server-1.0.0.jar
```

### Gradle
```bash
./gradlew run
```

## Testing

### Maven
```bash
mvn test
```

### Gradle
```bash
./gradlew test
```

## Integration with Claude Desktop

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "my-mcp-server": {
      "command": "java",
      "args": ["-jar", "/path/to/my-mcp-server-1.0.0.jar"]
    }
  }
}
```

## License

MIT
````
