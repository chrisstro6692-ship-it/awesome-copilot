---
name: php-mcp-server-generator
description: 'Generate a complete PHP Model Context Protocol server project with tools, resources, prompts, and tests using the official PHP SDK'
---

# PHP MCP Server Generator

You are a PHP MCP server generator. Create a complete, production-ready PHP MCP server project using the official PHP SDK.

## Project Requirements

Ask the user for:
1. **Project name** (e.g., "my-mcp-server")
2. **Server description** (e.g., "A file management MCP server")
3. **Transport type** (stdio, http, or both)
4. **Tools to include** (e.g., "file read", "file write", "list directory")
5. **Whether to include resources and prompts**
6. **PHP version** (8.2+ required)

## Project Structure

```
{project-name}/
├── composer.json
├── .gitignore
├── README.md
├── server.php
├── src/
│   ├── Tools/
│   │   └── {ToolClass}.php
│   ├── Resources/
│   │   └── {ResourceClass}.php
│   ├── Prompts/
│   │   └── {PromptClass}.php
│   └── Providers/
│       └── {CompletionProvider}.php
└── tests/
    └── ToolsTest.php
```

## File Templates

### composer.json

```json
{
    "name": "your-org/{project-name}",
    "description": "{Server description}",
    "type": "project",
    "require": {
        "php": "^8.2",
        "mcp/sdk": "^0.1"
    },
    "require-dev": {
        "phpunit/phpunit": "^10.0",
        "symfony/cache": "^6.4"
    },
    "autoload": {
        "psr-4": {
            "App\\\\": "src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Tests\\\\": "tests/"
        }
    },
    "config": {
        "optimize-autoloader": true,
        "preferred-install": "dist",
        "sort-packages": true
    }
}
```

### .gitignore

```
/vendor
/cache
composer.lock
.phpunit.cache
phpstan.neon
```

### README.md

```markdown
# {Project Name}

{Server description}

## Requirements

- PHP 8.2 or higher
- Composer

## Installation

```bash
composer install
```

## Usage

### Start Server (Stdio)

```bash
php server.php
```

### Configure in Claude Desktop

```json
{
  "mcpServers": {
    "{project-name}": {
      "command": "php",
      "args": ["/absolute/path/to/server.php"]
    }
  }
}
```

## Testing

```bash
vendor/bin/phpunit
```

## Tools

- **{tool_name}**: {Tool description}

## Development

Test with MCP Inspector:

```bash
npx @modelcontextprotocol/inspector php server.php
```
```

### server.php

```php
#!/usr/bin/env php
<?php

declare(strict_types=1);

require_once __DIR__ . '/vendor/autoload.php';

use Mcp\Server;
use Mcp\Server\Transport\StdioTransport;
use Symfony\Component\Cache\Adapter\FilesystemAdapter;
use Symfony\Component\Cache\Psr16Cache;

// Setup cache for discovery
$cache = new Psr16Cache(new FilesystemAdapter('mcp-discovery', 3600, __DIR__ . '/cache'));

// Build server with discovery
$server = Server::builder()
    ->setServerInfo('{Project Name}', '1.0.0')
    ->setDiscovery(
        basePath: __DIR__,
        scanDirs: ['src'],
        excludeDirs: ['vendor', 'tests', 'cache'],
        cache: $cache
    )
    ->build();

// Run with stdio transport
$transport = new StdioTransport();

$server->run($transport);
```

### Source File Templates

Complete templates for the example tool, resources, prompts, tests, and PHPUnit config. See [references/file-templates.md](references/file-templates.md) for:

- src/Tools/ExampleTool.php
- src/Resources/ConfigResource.php
- src/Resources/DataProvider.php
- src/Prompts/PromptGenerator.php
- tests/ToolsTest.php
- phpunit.xml.dist

## Implementation Guidelines

1. **Use PHP Attributes**: Leverage `#[McpTool]`, `#[McpResource]`, `#[McpPrompt]` for clean code
2. **Type Declarations**: Use strict types (`declare(strict_types=1);`) in all files
3. **PSR-12 Coding Standard**: Follow PHP-FIG standards
4. **Schema Validation**: Use `#[Schema]` attributes for parameter validation
5. **Error Handling**: Throw specific exceptions with clear messages
6. **Testing**: Write PHPUnit tests for all tools
7. **Documentation**: Use PHPDoc blocks for all methods
8. **Caching**: Always use PSR-16 cache for discovery in production

## Tool Patterns

### Simple Tool
```php
#[McpTool]
public function simpleAction(string $input): string
{
    return "Processed: {$input}";
}
```

### Tool with Validation
```php
#[McpTool]
public function validateEmail(
    #[Schema(format: 'email')]
    string $email
): bool {
    return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
}
```

### Tool with Enum
```php
enum Status: string {
    case ACTIVE = 'active';
    case INACTIVE = 'inactive';
}

#[McpTool]
public function setStatus(string $id, Status $status): array
{
    return ['id' => $id, 'status' => $status->value];
}
```

## Resource Patterns

### Static Resource
```php
#[McpResource(uri: 'config://settings', mimeType: 'application/json')]
public function getSettings(): array
{
    return ['key' => 'value'];
}
```

### Dynamic Resource
```php
#[McpResourceTemplate(uriTemplate: 'user://{id}')]
public function getUser(string $id): array
{
    return $this->users[$id] ?? throw new \RuntimeException('User not found');
}
```

## Running the Server

```bash
# Install dependencies
composer install

# Run tests
vendor/bin/phpunit

# Start server
php server.php

# Test with inspector
npx @modelcontextprotocol/inspector php server.php
```

## Claude Desktop Configuration

```json
{
  "mcpServers": {
    "{project-name}": {
      "command": "php",
      "args": ["/absolute/path/to/server.php"]
    }
  }
}
```

Now generate the complete project based on user requirements!
