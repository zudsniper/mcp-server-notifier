# MCP Server Notifier

A lightweight notification service that integrates with MCP (Model Context Protocol) to send webhooks when AI agents complete tasks.

[简体中文文档](./docs/README_zh.md)

![MCP Server Notifier](./docs/images/banner.png)

## Authors
Originally created by [tuberrabbit@gmail.com](mailto:tuberrabbit@gmail.com).  
Currently maintained by [zudsniper](https://github.com/zudsniper).

## Features

- **Webhook Notifications**: Receive alerts when your AI agents complete tasks
- **Multiple Webhook Providers**: Support for Discord, Slack, Microsoft Teams, Feishu, Ntfy, and custom webhooks
- **Image Support**: Include images in notifications via Imgur
- **Multi-Project Support**: Efficiently manage notifications across different projects
- **Easy Integration**: Simple setup with AI tools like Cursor
- **Customizable Messages**: Send personalized notifications with title, body, and links
- **Discord Embed Support**: Send rich, customizable Discord embed notifications
- **Interactive Question System**: Ask questions and receive answers through a web interface
- **Discord Webhook Example**: Now includes a sample Discord webhook config (`discord_webhook.json`) and test script (`src/test-discord.js`)
- **Improved Discord/NTFY Logic**: Enhanced webhook handling and configuration types

## Installation

### Option 1: Using npm

```bash
npm install -g mcp-server-notifier
```

### Option 2: Using Docker

```bash
docker pull zudsniper/mcp-server-notifier:latest

# Run with environment variables
docker run -e WEBHOOK_URL=https://your-webhook-url -e WEBHOOK_TYPE=discord zudsniper/mcp-server-notifier
```

### Option 3: From Source

```bash
git clone https://github.com/zudsniper/mcp-server-notifier.git
cd mcp-server-notifier
npm install
npm run build
```

## Integration

### Cursor Integration

1. Go to your 'Cursor Settings'
2. Click `MCP` in the sidebar, then click `+ Add new global MCP server`
3. Add `mcp-server-notifier`.  
```json
{
   "mcpServers": {
      "notifier": {
         "command": "npx",
         "args": [
            "-y",
            "mcp-server-notifier"
         ],
         "env": {
            "WEBHOOK_URL": "https://ntfy.sh/webhook-url-example",
            "WEBHOOK_TYPE": "ntfy"
         }
      }
   }
}

```

## Configuration

By default, the notifier supports several webhook types:

- Discord
- Slack
- Microsoft Teams
- Feishu
- Ntfy
- Generic JSON

You can specify the webhook type and URL through environment variables:

```bash
env WEBHOOK_URL="https://your-webhook-url" WEBHOOK_TYPE="discord" npx -y mcp-server-notifier
```
### Authentication Tokens

`WEBHOOK_TOKEN` is an **optional** environment variable. When set, it will be included as a Bearer token in the `Authorization` header **only** for **ntfy** webhook requests. If `WEBHOOK_TOKEN` is not set, no Authorization header is sent.

- **Basic Authentication is not supported.**
- This token is **ignored** by all other webhook providers (Discord, Slack, Teams, Feishu, Generic JSON).

**Example:**

```bash
env WEBHOOK_URL="https://ntfy.sh/your-topic" WEBHOOK_TYPE="ntfy" WEBHOOK_TOKEN="your-secret-token" npx -y mcp-server-notifier
```

### Configuration File

For more advanced configuration, you can create a `webhook-config.json` file:

```json
{
  "webhook": {
    "type": "discord",
    "url": "https://discord.com/api/webhooks/your-webhook-url",
    "name": "My Notifier"
  },
  "imgur": {
    "clientId": "your-imgur-client-id"
  }
}
```

See the [Configuration Guide](./docs/CONFIGURATION.md) for full details and examples.

### Ask Functionality Configuration (Optional)

To enable the interactive question system, add the following to your environment variables:

```bash
# Enable the ask functionality
ASKING=true

# Configure the server URL (without trailing slash)
ASK_SERVER_URL=http://localhost

# Configure the server port
ASK_SERVER_PORT=3000
```

When enabled, this starts a small web server that allows you to ask questions and receive answers through a web interface.

## Usage

- Ask your AI agent to notify you with a custom message when a task is complete
- Configure it as a persistent rule in Cursor settings to avoid repeating the setup

For detailed usage instructions, see the [Usage Guide](./docs/USAGE.md).

### Available Tools

1. `notify`
   - **Purpose**: Send rich notifications to any configured webhook
   - **Input**: 
     - `message` - Text content of the notification
     - `title` (optional) - Title for the notification
     - `link` (optional) - URL to include in the notification (used as click action for ntfy)
     - `imageUrl` (optional) - URL of an image to include (legacy, use `image` or `attachments`)
     - `image` (optional) - Local file path of an image to upload to Imgur
     - `priority` (optional, ntfy only) - Notification priority (1-5)
     - `attachments` (optional, ntfy only) - Array of URLs to attach
     - `template` (optional, ntfy only) - Predefined template to use: status, question, progress, problem
     - `templateData` (optional, ntfy only) - Data to populate the chosen template
     - `actions` (optional, ntfy only) - Array of action button definitions (`view` or `http`)
   - **Best for**: General notification needs

> **Note:** Template functionality is currently under development and has limited support. Templates work best with ntfy.sh but may not be fully implemented for all webhook providers. See the ROADMAP.md file for future implementation plans.

2. `ask_question` (Requires ASKING=true)
   - **Purpose**: Ask a question and wait for an answer through a web interface
   - **Input**: 
     - `question` - The question to ask
     - `title` (optional) - Title for the question
     - `timeout` - Maximum time to wait for an answer in seconds (10-3600)
   - **Output**: The answer received or an error if timed out
   - **Best for**: Getting user input during AI workflows

### NTFY Templates

When using ntfy.sh as your webhook provider, you can use the following predefined templates:

1. **Status Template** (`status`)
   - **Purpose**: Send status updates about systems, processes, or tasks
   - **Data Fields**:
     - `status` - Current status (e.g., "online", "completed", "pending")
     - `details` (optional) - Additional information about the status
     - `timestamp` (optional) - When this status was recorded
     - `component` (optional) - The system component this status applies to

2. **Question Template** (`question`)
   - **Purpose**: Ask questions that require a response
   - **Data Fields**:
     - `question` - The main question being asked
     - `context` (optional) - Background information for the question
     - `options` (optional) - Possible answer options
     - `deadline` (optional) - When a response is needed by

3. **Progress Template** (`progress`)
   - **Purpose**: Track progress of long-running tasks
   - **Data Fields**:
     - `title` - Name of the task or process
     - `current` - Current progress value
     - `total` - Total value to reach completion
     - `percentage` (optional) - Explicit percentage value (calculated if not provided)
     - `eta` (optional) - Estimated time to completion
     - `details` (optional) - Additional information about the progress

4. **Problem Template** (`problem`)
   - **Purpose**: Report errors or issues
   - **Data Fields**:
     - `title` - Short description of the problem
     - `description` (optional) - Detailed information about the problem
     - `severity` (optional) - How severe the problem is (e.g., "critical", "warning")
     - `source` (optional) - Where the problem originated
     - `timestamp` (optional) - When the problem occurred
     - `solution` (optional) - Suggested ways to fix the problem

**Example Using Template**:
```javascript
// Send a progress notification
{
  "template": "progress",
  "templateData": {
    "title": "Database Backup",
    "current": 75,
    "total": 100,
    "eta": "2 minutes remaining",
    "details": "Compressing backup files"
  },
  "priority": 3
}
```

## Docker Support

The MCP Server Notifier is available as a Docker image:

```bash
docker pull zudsniper/mcp-server-notifier:latest
```

Run with environment variables:

```bash
docker run -e WEBHOOK_URL=https://your-webhook-url -e WEBHOOK_TYPE=discord zudsniper/mcp-server-notifier
```

## Example Configurations

Example webhook configurations are available in the [examples](./examples) directory.

A new example for Discord webhook configuration is provided in `discord_webhook.json` at the project root.

## Development

### Setting Up Development Environment

1. Clone the repository:
```bash
git clone https://github.com/zudsniper/mcp-server-notifier.git
cd mcp-server-notifier
```

2. Install dependencies:
```bash
npm install
```

3. Build the project:
```bash
npm run build
```

- Node version management is now supported via `.nvmrc` for consistent development environments.

### Testing Your Changes

1. Run the MCP server in development mode:
```bash
# Install the MCP Inspector if you haven't already
npm install -g @modelcontextprotocol/inspector

# Start the server with the Inspector
npx @modelcontextprotocol/inspector node build/index.js
```

2. The Inspector provides a web interface where you can:
   - Send requests to your tools
   - View request/response logs
   - Debug issues with your implementation

### Releasing New Versions

To release a new version:

1. Update version in `package.json`
2. Push changes to the `release` branch
3. GitHub Actions will automatically:
   - Run tests
   - Build and push Docker images
   - Publish to npm
   - Create a GitHub Release

Required repository secrets for CI/CD:
- `DOCKERHUB_USERNAME` - Docker Hub username
- `DOCKERHUB_TOKEN` - Docker Hub access token
- `NPM_TOKEN` - npm access token

## License

MIT License - see LICENSE file for details.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
