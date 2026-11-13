---
name: rust-mcp-server-generator
description: 'Generate a complete Rust Model Context Protocol server project with tools, prompts, resources, and tests using the official rmcp SDK'
---

# Rust MCP Server Generator

You are a Rust MCP server generator. Create a complete, production-ready Rust MCP server project using the official `rmcp` SDK.

## Project Requirements

Ask the user for:
1. **Project name** (e.g., "my-mcp-server")
2. **Server description** (e.g., "A weather data MCP server")
3. **Transport type** (stdio, sse, http, or all)
4. **Tools to include** (e.g., "weather lookup", "forecast", "alerts")
5. **Whether to include prompts and resources**

## Project Structure

Generate this structure:

```
{project-name}/
├── Cargo.toml
├── .gitignore
├── README.md
├── src/
│   ├── main.rs
│   ├── handler.rs
│   ├── tools/
│   │   ├── mod.rs
│   │   └── {tool_name}.rs
│   ├── prompts/
│   │   ├── mod.rs
│   │   └── {prompt_name}.rs
│   ├── resources/
│   │   ├── mod.rs
│   │   └── {resource_name}.rs
│   └── state.rs
└── tests/
    └── integration_test.rs
```

## File Templates

### Cargo.toml

```toml
[package]
name = "{project-name}"
version = "0.1.0"
edition = "2021"

[dependencies]
rmcp = { version = "0.8.1", features = ["server"] }
rmcp-macros = "0.8"
tokio = { version = "1", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
anyhow = "1.0"
tracing = "0.1"
tracing-subscriber = "0.3"
schemars = { version = "0.8", features = ["derive"] }
async-trait = "0.1"

# Optional: for HTTP transports
axum = { version = "0.7", optional = true }
tower-http = { version = "0.5", features = ["cors"], optional = true }

[dev-dependencies]
tokio-test = "0.4"

[features]
default = []
http = ["dep:axum", "dep:tower-http"]

[[bin]]
name = "{project-name}"
path = "src/main.rs"
```

### .gitignore

```gitignore
/target
Cargo.lock
*.swp
*.swo
*~
.DS_Store
```

### README.md

```markdown
# {Project Name}

{Server description}

## Installation

```bash
cargo build --release
```

## Usage

### Stdio Transport

```bash
cargo run
```

### SSE Transport

```bash
cargo run --features http -- --transport sse
```

### HTTP Transport

```bash
cargo run --features http -- --transport http
```

## Configuration

Configure in your MCP client (e.g., Claude Desktop):

```json
{
  "mcpServers": {
    "{project-name}": {
      "command": "path/to/target/release/{project-name}",
      "args": []
    }
  }
}
```

## Tools

- **{tool_name}**: {Tool description}

## Development

Run tests:

```bash
cargo test
```

Run with logging:

```bash
RUST_LOG=debug cargo run
```
```

### src/main.rs

```rust
use anyhow::Result;
use rmcp::{
    protocol::ServerCapabilities,
    server::Server,
    transport::StdioTransport,
};
use tokio::signal;
use tracing_subscriber;

mod handler;
mod state;
mod tools;
mod prompts;
mod resources;

use handler::McpHandler;

#[tokio::main]
async fn main() -> Result<()> {
    // Initialize tracing
    tracing_subscriber::fmt()
        .with_max_level(tracing::Level::INFO)
        .with_target(false)
        .init();
    
    tracing::info!("Starting {project-name} MCP server");
    
    // Create handler
    let handler = McpHandler::new();
    
    // Create transport (stdio by default)
    let transport = StdioTransport::new();
    
    // Build server with capabilities
    let server = Server::builder()
        .with_handler(handler)
        .with_capabilities(ServerCapabilities {
            tools: Some(Default::default()),
            prompts: Some(Default::default()),
            resources: Some(Default::default()),
            ..Default::default()
        })
        .build(transport)?;
    
    tracing::info!("Server started, waiting for requests");
    
    // Run server until Ctrl+C
    server.run(signal::ctrl_c()).await?;
    
    tracing::info!("Server shutting down");
    Ok(())
}
```

### Source File Templates

Complete templates for the handler, state, tools, prompts, resources, and integration tests. See [references/file-templates.md](references/file-templates.md) for:

- src/handler.rs
- src/state.rs
- src/tools/mod.rs
- src/tools/example.rs
- src/prompts/mod.rs
- src/resources/mod.rs
- tests/integration_test.rs

## Implementation Guidelines

1. **Use rmcp-macros**: Leverage `#[tool]`, `#[tool_router]`, and `#[tool_handler]` macros for cleaner code
2. **Type Safety**: Use `schemars::JsonSchema` for all parameter types
3. **Error Handling**: Return `Result` types with proper error messages
4. **Async/Await**: All handlers must be async
5. **State Management**: Use `Arc<RwLock<T>>` for shared state
6. **Testing**: Include unit tests for tools and integration tests for handlers
7. **Logging**: Use `tracing` macros (`info!`, `debug!`, `warn!`, `error!`)
8. **Documentation**: Add doc comments to all public items

## Example Tool Patterns

### Simple Read-Only Tool

```rust
#[derive(Debug, Deserialize, JsonSchema)]
pub struct GreetParams {
    pub name: String,
}

#[tool(
    name = "greet",
    description = "Greets a user by name",
    annotations(read_only_hint = true, idempotent_hint = true)
)]
async fn greet(params: Parameters<GreetParams>) -> String {
    format!("Hello, {}!", params.inner().name)
}
```

### Tool with Error Handling

```rust
#[derive(Debug, Deserialize, JsonSchema)]
pub struct DivideParams {
    pub a: f64,
    pub b: f64,
}

#[tool(name = "divide", description = "Divides two numbers")]
async fn divide(params: Parameters<DivideParams>) -> Result<f64, String> {
    let p = params.inner();
    if p.b == 0.0 {
        Err("Cannot divide by zero".to_string())
    } else {
        Ok(p.a / p.b)
    }
}
```

### Tool with State

```rust
#[tool(
    name = "increment",
    description = "Increments the counter",
    annotations(destructive_hint = true)
)]
async fn increment(state: &ServerState) -> i32 {
    state.increment().await
}
```

## Running the Generated Server

After generation:

```bash
cd {project-name}
cargo build
cargo test
cargo run
```

For Claude Desktop integration:

```json
{
  "mcpServers": {
    "{project-name}": {
      "command": "path/to/{project-name}/target/release/{project-name}",
      "args": []
    }
  }
}
```

Now generate the complete project based on the user's requirements!
