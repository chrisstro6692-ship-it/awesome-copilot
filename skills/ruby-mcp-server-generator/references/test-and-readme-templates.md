# Test and README Templates

## test/test_helper.rb Template

```ruby
# frozen_string_literal: true

$LOAD_PATH.unshift File.expand_path('../lib', __dir__)
require 'my_mcp_server'
require 'minitest/autorun'
```

## test/tools/greet_tool_test.rb Template

```ruby
# frozen_string_literal: true

require 'test_helper'

module MyMcpServer
  module Tools
    class GreetToolTest < Minitest::Test
      def test_greet_with_name
        response = GreetTool.call(
          name: 'Ruby',
          server_context: {}
        )
        
        refute response.is_error
        assert_equal 1, response.content.length
        assert_match(/Ruby/, response.content.first[:text])
        
        assert response.structured_content
        assert_equal 'Hello, Ruby! Welcome to MCP.', response.structured_content[:message]
      end
      
      def test_output_schema_validation
        response = GreetTool.call(
          name: 'Test',
          server_context: {}
        )
        
        assert response.structured_content.key?(:message)
        assert response.structured_content.key?(:timestamp)
      end
    end
  end
end
```

## test/tools/calculate_tool_test.rb Template

```ruby
# frozen_string_literal: true

require 'test_helper'

module MyMcpServer
  module Tools
    class CalculateToolTest < Minitest::Test
      def test_addition
        response = CalculateTool.call(
          operation: 'add',
          a: 5,
          b: 3,
          server_context: {}
        )
        
        refute response.is_error
        assert_equal 8, response.structured_content[:result]
      end
      
      def test_subtraction
        response = CalculateTool.call(
          operation: 'subtract',
          a: 10,
          b: 4,
          server_context: {}
        )
        
        refute response.is_error
        assert_equal 6, response.structured_content[:result]
      end
      
      def test_multiplication
        response = CalculateTool.call(
          operation: 'multiply',
          a: 6,
          b: 7,
          server_context: {}
        )
        
        refute response.is_error
        assert_equal 42, response.structured_content[:result]
      end
      
      def test_division
        response = CalculateTool.call(
          operation: 'divide',
          a: 15,
          b: 3,
          server_context: {}
        )
        
        refute response.is_error
        assert_equal 5.0, response.structured_content[:result]
      end
      
      def test_division_by_zero
        response = CalculateTool.call(
          operation: 'divide',
          a: 10,
          b: 0,
          server_context: {}
        )
        
        assert response.is_error
        assert_match(/Division by zero/, response.content.first[:text])
      end
      
      def test_unknown_operation
        response = CalculateTool.call(
          operation: 'modulo',
          a: 10,
          b: 3,
          server_context: {}
        )
        
        assert response.is_error
        assert_match(/Unknown operation/, response.content.first[:text])
      end
    end
  end
end
```

## README.md Template

````markdown
# My MCP Server

A Model Context Protocol server built with Ruby and the official MCP Ruby SDK.

## Features

- ✅ Tools: greet, calculate
- ✅ Prompts: code_review
- ✅ Resources: example-data
- ✅ Input/output schemas
- ✅ Tool annotations
- ✅ Structured content
- ✅ Full test coverage

## Requirements

- Ruby 3.0 or later

## Installation

```bash
bundle install
```

## Usage

### Stdio Transport

Run the server:

```bash
bundle exec bin/mcp-server
```

Then send JSON-RPC requests:

```bash
{"jsonrpc":"2.0","id":"1","method":"ping"}
{"jsonrpc":"2.0","id":"2","method":"tools/list"}
{"jsonrpc":"2.0","id":"3","method":"tools/call","params":{"name":"greet","arguments":{"name":"Ruby"}}}
```

### Rails Integration

Add to your Rails controller:

```ruby
class McpController < ApplicationController
  def index
    server = MyMcpServer::Server.new(
      server_context: { user_id: current_user.id }
    )
    render json: server.handle_json(request.body.read)
  end
end
```

## Testing

Run tests:

```bash
bundle exec rake test
```

Run linter:

```bash
bundle exec rake rubocop
```

Run all checks:

```bash
bundle exec rake
```

## Integration with Claude Desktop

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "my-mcp-server": {
      "command": "bundle",
      "args": ["exec", "bin/mcp-server"],
      "cwd": "/path/to/my-mcp-server"
    }
  }
}
```

## Project Structure

```
my-mcp-server/
├── Gemfile              # Dependencies
├── Rakefile             # Build tasks
├── lib/                 # Source code
│   ├── my_mcp_server.rb # Main entry point
│   └── my_mcp_server/   # Module namespace
│       ├── server.rb    # Server setup
│       ├── tools/       # Tool implementations
│       ├── prompts/     # Prompt templates
│       └── resources/   # Resource handlers
├── bin/                 # Executables
│   └── mcp-server       # Stdio server
├── test/                # Test suite
│   ├── test_helper.rb   # Test configuration
│   └── tools/           # Tool tests
└── README.md            # This file
```

## License

MIT
````
