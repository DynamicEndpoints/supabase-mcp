# Supabase MCP Server

[![smithery badge](https://smithery.ai/badge/supabase-server)](https://smithery.ai/server/supabase-server)
A Model Context Protocol (MCP) server that provides comprehensive tools for interacting with Supabase databases, storage, and edge functions. This server enables seamless integration between Supabase services and MCP-compatible applications.

<a href="https://glama.ai/mcp/servers/vwi6nt8i80"><img width="380" height="200" src="https://glama.ai/mcp/servers/vwi6nt8i80/badge" alt="supabase-mcp MCP server" /></a>

## Overview

The Supabase MCP server acts as a bridge between MCP clients and Supabase's suite of services, providing:

- Database operations with rich querying capabilities
- Storage management for files and assets
- Edge function invocation
- Project and organization management
- User authentication and management
- Role-based access control

## Architecture

The server is built using TypeScript and follows a modular architecture:

```
supabase-server/
├── src/
│   ├── index.ts              # Main server implementation
│   └── types/
│       └── supabase.d.ts     # Type definitions
├── package.json
├── tsconfig.json
├── config.json.example       # Example configuration file
└── .env.example             # Environment variables template
```

### Key Components

- **Server Class**: Implements the MCP server interface and handles all client requests
- **Type Definitions**: Comprehensive TypeScript definitions for all operations
- **Environment Configuration**: Secure configuration management via environment variables
- **Error Handling**: Robust error handling with detailed error messages

## Prerequisites

- Node.js 16.x or higher
- A Supabase project with:
  - Project URL
  - Service Role Key (for admin operations)
  - Access Token (for management operations)
- MCP-compatible client

## Installation

### Installing via Smithery

To install Supabase Server for Claude Desktop automatically via [Smithery](https://smithery.ai/server/supabase-server):

```bash
npx -y @smithery/cli install supabase-server --client claude
```

1. Clone the repository:
```bash
git clone https://github.com/DynamicEndpoints/supabase-mcp.git
cd supabase-mcp
```

2. Install dependencies:
```bash
npm install
```

3. Create environment configuration:
```bash
cp .env.example .env
```

4. Configure environment variables:
```bash
SUPABASE_URL=your_project_url_here
SUPABASE_KEY=your_service_role_key_here
SUPABASE_ACCESS_TOKEN=your_access_token_here  # Required for management operations
```

5. Create server configuration:
```bash
cp config.json.example config.json
```

6. Build the server:
```bash
npm run build
```

## Configuration

The server supports extensive configuration through both environment variables and a config.json file. Here's a detailed breakdown of the configuration options:

### Server Configuration
```json
{
  "server": {
    "name": "supabase-server",    // Server name
    "version": "0.1.0",           // Server version
    "port": 3000,                 // Port number (if running standalone)
    "host": "localhost"           // Host address (if running standalone)
  }
}
```

### Supabase Configuration
```json
{
  "supabase": {
    "project": {
      "url": "your_project_url",
      "key": "your_service_role_key",
      "accessToken": "your_access_token"
    },
    "storage": {
      "defaultBucket": "public",           // Default storage bucket
      "maxFileSize": 52428800,            // Max file size in bytes (50MB)
      "allowedMimeTypes": [               // Allowed file types
        "image/*",
        "application/pdf",
        "text/*"
      ]
    },
    "database": {
      "maxConnections": 10,               // Max DB connections
      "timeout": 30000,                   // Query timeout in ms
      "ssl": true                         // SSL connection
    },
    "auth": {
      "autoConfirmUsers": false,          // Auto-confirm new users
      "disableSignup": false,             // Disable public signups
      "jwt": {
        "expiresIn": "1h",               // Token expiration
        "algorithm": "HS256"              // JWT algorithm
      }
    }
  }
}
```

### Logging Configuration
```json
{
  "logging": {
    "level": "info",                      // Log level
    "format": "json",                     // Log format
    "outputs": ["console", "file"],       // Output destinations
    "file": {
      "path": "logs/server.log",          // Log file path
      "maxSize": "10m",                   // Max file size
      "maxFiles": 5                       // Max number of files
    }
  }
}
```

### Security Configuration
```json
{
  "security": {
    "cors": {
      "enabled": true,
      "origins": ["*"],
      "methods": ["GET", "POST", "PUT", "DELETE", "OPTIONS"],
      "allowedHeaders": ["Content-Type", "Authorization"]
    },
    "rateLimit": {
      "enabled": true,
      "windowMs": 900000,                 // 15 minutes
      "max": 100                          // Max requests per window
    }
  }
}
```

### Monitoring Configuration
```json
{
  "monitoring": {
    "enabled": true,
    "metrics": {
      "collect": true,
      "interval": 60000                   // Collection interval in ms
    },
    "health": {
      "enabled": true,
      "path": "/health"                   // Health check endpoint
    }
  }
}
```

See `config.json.example` for a complete example configuration file.

## MCP Integration

Add the server to your MCP settings (cline_mcp_settings.json):

```json
{
  "mcpServers": {
    "supabase": {
      "command": "node",
      "args": ["path/to/supabase-server/build/index.js"],
      "env": {
        "SUPABASE_URL": "your_project_url",
        "SUPABASE_KEY": "your_service_role_key",
        "SUPABASE_ACCESS_TOKEN": "your_access_token"
      },
      "config": "path/to/config.json"  // Optional: path to configuration file
    }
  }
}
```

## Available Tools

### Database Operations

#### create_record
Create a new record in a table with support for returning specific fields.

```typescript
{
  table: string;
  data: Record<string, any>;
  returning?: string[];
}
```

Example:
```typescript
{
  table: "users",
  data: {
    name: "John Doe",
    email: "john@example.com"
  },
  returning: ["id", "created_at"]
}
```

#### read_records
Read records with advanced filtering, joins, and field selection.

```typescript
{
  table: string;
  select?: string[];
  filter?: Record<string, any>;
  joins?: Array<{
    type?: 'inner' | 'left' | 'right' | 'full';
    table: string;
    on: string;
  }>;
}
```

Example:
```typescript
{
  table: "posts",
  select: ["id", "title", "user.name"],
  filter: { published: true },
  joins: [{
    type: "left",
    table: "users",
    on: "posts.user_id=users.id"
  }]
}
```

#### update_record
Update records with filtering and returning capabilities.

```typescript
{
  table: string;
  data: Record<string, any>;
  filter?: Record<string, any>;
  returning?: string[];
}
```

Example:
```typescript
{
  table: "users",
  data: { status: "active" },
  filter: { email: "john@example.com" },
  returning: ["id", "status", "updated_at"]
}
```

#### delete_record
Delete records with filtering and returning capabilities.

```typescript
{
  table: string;
  filter?: Record<string, any>;
  returning?: string[];
}
```

Example:
```typescript
{
  table: "posts",
  filter: { status: "draft" },
  returning: ["id", "title"]
}
```

### Storage Operations

#### upload_file
Upload files to Supabase Storage with configurable options.

```typescript
{
  bucket: string;
  path: string;
  file: File | Blob;
  options?: {
    cacheControl?: string;
    contentType?: string;
    upsert?: boolean;
  };
}
```

Example:
```typescript
{
  bucket: "avatars",
  path: "users/123/profile.jpg",
  file: imageBlob,
  options: {
    contentType: "image/jpeg",
    upsert: true
  }
}
```

#### download_file
Download files from Supabase Storage.

```typescript
{
  bucket: string;
  path: string;
}
```

Example:
```typescript
{
  bucket: "documents",
  path: "reports/annual-2023.pdf"
}
```

### Edge Functions

#### invoke_function
Invoke Supabase Edge Functions with parameters and custom options.

```typescript
{
  function: string;
  params?: Record<string, any>;
  options?: {
    headers?: Record<string, string>;
    responseType?: 'json' | 'text' | 'arraybuffer';
  };
}
```

Example:
```typescript
{
  function: "process-image",
  params: {
    url: "https://example.com/image.jpg",
    width: 800
  },
  options: {
    responseType: "json"
  }
}
```

### User Management

#### list_users
List users with pagination support.

```typescript
{
  page?: number;
  per_page?: number;
}
```

#### create_user
Create a new user with metadata.

```typescript
{
  email: string;
  password: string;
  data?: Record<string, any>;
}
```

#### update_user
Update user details.

```typescript
{
  user_id: string;
  email?: string;
  password?: string;
  data?: Record<string, any>;
}
```

#### delete_user
Delete a user.

```typescript
{
  user_id: string;
}
```

#### assign_user_role
Assign a role to a user.

```typescript
{
  user_id: string;
  role: string;
}
```

#### remove_user_role
Remove a role from a user.

```typescript
{
  user_id: string;
  role: string;
}
```

## Error Handling

The server provides detailed error messages for common scenarios:

- Invalid parameters
- Authentication failures
- Permission issues
- Rate limiting
- Network errors
- Database constraints

Errors are returned in a standardized format:

```typescript
{
  code: ErrorCode;
  message: string;
  details?: any;
}
```

## Development

### Running Tests
```bash
npm test
```

### Building
```bash
npm run build
```

### Linting
```bash
npm run lint
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## License

MIT License - see LICENSE for details

## Support

For support, please:

1. Check the [issues](https://github.com/DynamicEndpoints/supabase-mcp/issues) for existing problems/solutions
2. Create a new issue with detailed reproduction steps
3. Include relevant error messages and environment details
