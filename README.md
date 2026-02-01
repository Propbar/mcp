# Propbar MCP Server

[![MCP Registry](https://img.shields.io/badge/MCP-Registry-blue)](https://registry.modelcontextprotocol.io)

MCP server providing **UK property research tools** for AI assistants. Enables Claude, GPT, and other LLMs to search properties, analyse area safety, find nearby schools, explore demographics, and get property valuations.

Powered by [Propbar's](https://propbar.co.uk) comprehensive UK property database.

## Features

- **Crime & Safety Analysis** - UK Police crime statistics with safety ratings
- **School Finder** - Ofsted-rated schools with distance calculations
- **Demographics** - UK Census 2021 data for any location
- **Property Valuations** - Comparable sales and market analysis
- **Flexible Inputs** - Use area codes OR coordinates for most tools

## Available Tools

| Tool | Description | Input |
|------|-------------|-------|
| `search_areas` | Search UK areas by name or postcode | query |
| `search_properties` | Search UK properties by address | query |
| `get_area_details` | Get coordinates and area codes | areaCode or path |
| `get_crime_stats` | Crime statistics and safety rating | areaCode OR lat/lng |
| `get_schools` | Schools with Ofsted ratings | areaCode OR lat/lng |
| `get_demographics` | Census demographics | areaCode OR lat/lng |
| `get_property_basic` | Basic property details | propertyId |
| `get_property_full` | Full property with history | propertyId |
| `get_comparables` | Comparable properties | lat/lng or propertyId |

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

## Tool Workflow

```
┌─────────────────────┐     ┌─────────────────────┐
│   search_areas      │     │  search_properties  │
│   (get areaCode)    │     │  (get propertyId)   │
└─────────┬───────────┘     └──────────┬──────────┘
          │                            │
          ▼                            ▼
┌─────────────────────┐     ┌─────────────────────┐
│  Area Tools:        │     │  Property Tools:    │
│  • get_crime_stats  │     │  • get_property_*   │
│  • get_schools      │     │  • get_comparables  │
│  • get_demographics │     │                     │
└─────────────────────┘     └─────────────────────┘
```

**Flexible inputs:** Most area tools accept EITHER `areaCode` OR `latitude`/`longitude` coordinates.

## Example Usage

### Check area safety
```json
{
  "tool": "get_crime_stats",
  "arguments": {
    "areaCode": "E06000014"
  }
}
```

Or with coordinates:
```json
{
  "tool": "get_crime_stats",
  "arguments": {
    "latitude": 53.958,
    "longitude": -1.080
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

### Get demographics
```json
{
  "tool": "get_demographics",
  "arguments": {
    "areaCode": "E06000014",
    "topics": ["ageBands", "ethnicGroup", "tenure"]
  }
}
```

## OAuth Discovery

Protected Resource Metadata (RFC 9728):
```
https://api.propbar.co.uk/mcp/.well-known/oauth-protected-resource
```

## Data Sources

- **Crime Statistics:** [Police UK](https://data.police.uk/)
- **Schools:** [Get Information About Schools](https://get-information-schools.service.gov.uk/)
- **Demographics:** [UK Census 2021](https://census.gov.uk/)
- **Property Data:** [Propbar](https://propbar.co.uk)

## Support

- **Website:** [propbar.co.uk](https://propbar.co.uk)
- **Email:** hello@propbar.co.uk

## License

Proprietary - Requires active Propbar subscription.

---

*UK property research made simple for AI assistants.*
