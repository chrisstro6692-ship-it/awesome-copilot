# Test and README Templates

## ServerTests.swift Template

```swift
import XCTest
@testable import MyMCPServer

final class ServerTests: XCTestCase {
    func testGreetTool() async throws {
        let params = CallTool.Params(
            name: "greet",
            arguments: ["name": .string("Swift")]
        )
        
        let result = handleGreet(params: params)
        
        XCTAssertFalse(result.isError ?? true)
        XCTAssertEqual(result.content.count, 1)
        
        if case .text(let message) = result.content[0] {
            XCTAssertTrue(message.contains("Swift"))
        } else {
            XCTFail("Expected text content")
        }
    }
    
    func testCalculateTool() async throws {
        let params = CallTool.Params(
            name: "calculate",
            arguments: [
                "operation": .string("add"),
                "a": .number(5),
                "b": .number(3)
            ]
        )
        
        let result = handleCalculate(params: params)
        
        XCTAssertFalse(result.isError ?? true)
        XCTAssertEqual(result.content.count, 1)
        
        if case .text(let message) = result.content[0] {
            XCTAssertTrue(message.contains("8"))
        } else {
            XCTFail("Expected text content")
        }
    }
    
    func testDivideByZero() async throws {
        let params = CallTool.Params(
            name: "calculate",
            arguments: [
                "operation": .string("divide"),
                "a": .number(10),
                "b": .number(0)
            ]
        )
        
        let result = handleCalculate(params: params)
        
        XCTAssertTrue(result.isError ?? false)
    }
}
```

## README.md Template

````markdown
# MyMCPServer

A Model Context Protocol server built with Swift.

## Features

- ✅ Tools: greet, calculate
- ✅ Resources: example data, configuration
- ✅ Prompts: code-review
- ✅ Graceful shutdown with ServiceLifecycle
- ✅ Structured logging with swift-log
- ✅ Full test coverage

## Requirements

- Swift 6.0+
- macOS 13+, iOS 16+, or Linux

## Installation

```bash
swift build -c release
```

## Usage

Run the server:

```bash
swift run
```

Or with logging:

```bash
LOG_LEVEL=debug swift run
```

## Testing

```bash
swift test
```

## Development

The server uses:
- [MCP Swift SDK](https://github.com/modelcontextprotocol/swift-sdk) - MCP protocol implementation
- [swift-log](https://github.com/apple/swift-log) - Structured logging
- [swift-service-lifecycle](https://github.com/swift-server/swift-service-lifecycle) - Graceful shutdown

## Project Structure

- `Sources/MyMCPServer/main.swift` - Entry point with ServiceLifecycle
- `Sources/MyMCPServer/Server.swift` - Server configuration
- `Sources/MyMCPServer/Tools/` - Tool definitions and handlers
- `Sources/MyMCPServer/Resources/` - Resource definitions and handlers
- `Sources/MyMCPServer/Prompts/` - Prompt definitions and handlers
- `Tests/` - Unit tests

## License

MIT
````
