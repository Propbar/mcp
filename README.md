# Propbar MCP Server

MCP server providing UK property research tools for AI assistants. Enables Claude, GPT, and other LLMs to search properties, analyse area safety, find nearby schools, explore demographics, and get property valuations.

Powered by [Propbar's](https://propbar.co.uk) comprehensive UK property database.

## Available Tools

| Tool | Description |
|------|-------------|
| `search_areas` | Search UK areas by name or postcode |
| `search_properties` | Search UK properties by address |
| `get_area_details` | Get coordinates and area codes for a location |
| `get_crime_stats` | Get crime statistics and safety ratings for an area |
| `get_schools` | Get schools near a location with Ofsted ratings |
| `get_property_basic` | Get basic property details |
| `get_property_full` | Get full property details with price history |
| `get_comparables` | Get comparable properties for valuation |
| `get_demographics` | Get census demographics for an area |

## Connection

**Endpoint:** `https://api.propbar.co.uk/mcp/streamable`

**Transport:** Streamable HTTP (MCP 2025-03-26)

**Authentication:** OAuth 2.1 via Supabase

## Requirements

- **Subscription:** Pro or Plus subscription required
- **Account:** [Sign up at propbar.co.uk](https://propbar.co.uk)

## Quick Start

### Claude Desktop

Add to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "propbar": {
      "url": "https://api.propbar.co.uk/mcp/streamable"
    }
  }
}
```

You'll be prompted to authenticate via OAuth when first connecting.

### Programmatic Access

For custom integrations, use the MCP SDK:

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js'
import { StreamableHTTPClientTransport } from '@modelcontextprotocol/sdk/client/streamableHttp.js'

const transport = new StreamableHTTPClientTransport(
  new URL('https://api.propbar.co.uk/mcp/streamable'),
  {
    requestInit: {
      headers: {
        Authorization: `Bearer ${accessToken}`,
      },
    },
  }
)

const client = new Client({ name: 'my-app', version: '1.0.0' })
await client.connect(transport)

// List available tools
const { tools } = await client.listTools()

// Call a tool
const result = await client.callTool('search_areas', { query: 'York' })
```

## OAuth Discovery

Protected Resource Metadata (RFC 9728):
```
https://api.propbar.co.uk/mcp/.well-known/oauth-protected-resource
```

## Example Usage

### Search for an area
```json
{
  "tool": "search_areas",
  "arguments": {
    "query": "York",
    "limit": 5
  }
}
```

### Get crime statistics
```json
{
  "tool": "get_crime_stats",
  "arguments": {
    "areaCode": "E06000014"
  }
}
```

### Find nearby schools
```json
{
  "tool": "get_schools",
  "arguments": {
    "latitude": 53.9583,
    "longitude": -1.0803,
    "radiusMetres": 2000
  }
}
```

## Support

- **Website:** [propbar.co.uk](https://propbar.co.uk)
- **Email:** hello@propbar.co.uk

## License

Proprietary - Requires active Propbar subscription.
