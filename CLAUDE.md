# MCP Server Maintenance Guide

This repo contains the **public MCP Registry metadata** for Propbar's UK Property Research MCP server.

## Architecture Overview

```
/Users/goran/Projects/propbar/mcp/           <- THIS REPO (public, for MCP Registry)
├── server.json                              <- Registry metadata
├── README.md                                <- Public documentation
└── CLAUDE.md                                <- This file

/Users/goran/Projects/propbar/pb-frontend/apps/api/src/services/research-platform/
├── mcp/
│   ├── server.ts                            <- MCP server implementation
│   ├── adapter.ts                           <- Tool registration adapter
│   └── auth.ts                              <- OAuth 2.1 authentication
└── tools/
    ├── resolvers.ts                         <- Coordinate/areaCode conversion utilities
    ├── registry.ts                          <- Tool metadata registry
    ├── area/
    │   ├── get-crime.tool.ts
    │   ├── get-schools.tool.ts
    │   └── get-area-details.tool.ts
    ├── demographics/
    │   └── get-demographics.tool.ts
    ├── property/
    │   ├── get-property-basic.tool.ts
    │   ├── get-property-full.tool.ts
    │   └── get-comparables.tool.ts
    └── search/
        ├── search-areas.tool.ts
        └── search-properties.tool.ts
```

## Publishing to MCP Registry

### Prerequisites

- **Publisher binary**: `/Users/goran/Library/CloudStorage/Dropbox/Vepler Code/propbar-mcp-registry/mcp-publisher`
- **Private key**: Same location, `mcp-propbar-key.pem`
- **DNS TXT record**: `propbar.co.uk` root with Ed25519 public key

### Authentication

```bash
# Extract hex key from PEM
HEX_KEY=$(openssl pkey -in "/Users/goran/Library/CloudStorage/Dropbox/Vepler Code/propbar-mcp-registry/mcp-propbar-key.pem" -outform DER 2>/dev/null | tail -c 32 | xxd -p -c 64)

# Login
"/Users/goran/Library/CloudStorage/Dropbox/Vepler Code/propbar-mcp-registry/mcp-publisher" login dns \
  -algorithm ed25519 \
  -domain propbar.co.uk \
  -private-key "$HEX_KEY"
```

### Publishing

```bash
cd /Users/goran/Projects/propbar/mcp
git add . && git commit -m "vX.Y.Z: Description" && git push
"/Users/goran/Library/CloudStorage/Dropbox/Vepler Code/propbar-mcp-registry/mcp-publisher" publish
```

## Adding/Updating Tools

### 1. Create/modify tool in API codebase

Location: `apps/api/src/services/research-platform/tools/[category]/[tool-name].tool.ts`

Each tool needs:
- **Zod schema** for input validation
- **Description** with ACCEPTS, USE WHEN, EXAMPLES sections (for AI discoverability)
- **Execute function** returning JSON-stringified result

### 2. Register in MCP server

In `apps/api/src/services/research-platform/mcp/server.ts`:
```typescript
import { newTool } from '../tools/[category]/new-tool.tool'
registerMCPTool('new_tool', newTool)
```

### 3. Update documentation

- Update tool table in `/Users/goran/Projects/propbar/mcp/README.md`
- Bump version in `server.json`
- Commit and publish

## Flexible Input Pattern

All area-based tools should accept EITHER `areaCode` OR `latitude`/`longitude`. Use the shared resolver:

```typescript
import { flexibleLocationSchema, resolveLocation, coordinatesToAreaCode } from '../resolvers'

const schema = flexibleLocationSchema.extend({
  // additional fields
})

// In execute:
if (input.areaCode) {
  // Use areaCode directly
} else if (input.latitude !== undefined && input.longitude !== undefined) {
  const area = await coordinatesToAreaCode(input.latitude, input.longitude)
  // Use area.areaCode
}
```

## Tool Description Template

```typescript
description: `[One-line summary]

ACCEPTS:
- areaCode: ONS code like "E06000014" (York)
- OR latitude + longitude: Coordinates

USE WHEN:
- [Trigger phrase 1]
- [Trigger phrase 2]

EXAMPLES:
- "[Query]" → tool_name({ param: "value" })

RETURNS: [Brief output description]`
```

## Important Notes

- **Vepler API quirk**: `lat` and `long` fields are SWAPPED. `.long` = latitude, `.lat` = longitude
- **LSOA21**: Census area codes used for demographics (e.g., "E01000001")
- **ONS codes**: Used for local authorities (e.g., "E06000014" = York)
- **Endpoint**: `https://mcp.propbar.co.uk` (not api.propbar.co.uk/mcp)
- **Namespace**: `uk.co.propbar/research` (reverse domain format)
