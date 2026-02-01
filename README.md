# Propbar MCP Server

[![MCP Registry](https://img.shields.io/badge/MCP-Registry-blue)](https://registry.modelcontextprotocol.io)
[![UK Property Data](https://img.shields.io/badge/UK-Property%20Data-green)](https://propbar.co.uk)

MCP server providing **UK property research tools** for AI assistants. Enables Claude, GPT, and other LLMs to search properties, analyse area safety, find nearby schools, explore demographics, and **build automated valuation models (AVMs)**.

Powered by [Propbar's](https://propbar.co.uk) comprehensive UK property database covering **30+ million properties**.

## Use Cases

### Build Your Own AVM

The `get_comparables` tool provides access to a **comprehensive property search** with 50+ filterable fields - perfect for building automated valuation models:

- **Transaction Data**: Sale history with dates and amounts, current asking prices
- **Market Status**: For sale, SSTC, under offer, reserved, back on market dates
- **Property Attributes**: Type, beds, baths, floor area, construction age
- **EPC Data**: Energy ratings (A-G), efficiency scores, potential ratings
- **Tenure & Charges**: Freehold/leasehold, lease years remaining, service charges, ground rent
- **Pricing Metrics**: Price per sqm, price per sqft, price changes over time
- **Listing Intelligence**: Days on market, price reductions, portal links

**Example AVM workflow:**
```
1. get_comparables({ latitude, longitude, propertyType: "flat", bedrooms: 2 })
2. Filter by sale date (last 12 months) and distance
3. Adjust for differences (floor area, EPC rating, condition)
4. Calculate £/sqft and apply to subject property
```

### Property Research for Buyers

Help users make informed decisions with comprehensive area data:

- **Crime & Safety** - Is this area safe? How does it compare?
- **Schools** - What schools are nearby? Ofsted ratings?
- **Demographics** - Who lives here? Age, tenure, household types?
- **Market Analysis** - What are similar properties selling for?

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
| `get_comparables` | **Comparable properties for valuation** | lat/lng + filters |

## Comparables Search Fields

The `get_comparables` tool queries an Elasticsearch index with **56 filterable fields**:

<details>
<summary><strong>Property Attributes</strong></summary>

| Field | Description |
|-------|-------------|
| `build.propertyType` | Detached, semi, terrace, flat, etc. |
| `roomDetails.beds` | Number of bedrooms |
| `roomDetails.baths` | Number of bathrooms |
| `build.totalFloorArea` | Total floor area (sqm) |
| `build.constructionAge.*` | Year built / construction period |
| `tenure.type` | Freehold, leasehold, share of freehold |
| `councilTax.taxBand` | Council tax band (A-H) |

</details>

<details>
<summary><strong>Pricing & Sales</strong></summary>

| Field | Description |
|-------|-------------|
| `pricing.currentSale` | Current asking price (sales) |
| `pricing.currentRent` | Current asking rent (pcm) |
| `saleHistory.date` | Historical sale dates |
| `saleHistory.amount` | Historical sale prices |
| `marketStatus.pricePerSqm` | Price per square metre |
| `marketStatus.pricePerSqft` | Price per square foot |

</details>

<details>
<summary><strong>Market Status</strong></summary>

| Field | Description |
|-------|-------------|
| `marketStatus.forSale` | Currently listed for sale |
| `marketStatus.forRent` | Currently listed for rent |
| `marketStatus.sstcDate` | Sold subject to contract date |
| `marketStatus.underOfferDate` | Under offer date |
| `marketStatus.reservedDate` | Reserved date (rentals) |
| `marketStatus.lastOTM` | Last on the market date |
| `marketStatus.backOTM` | Back on market flag |

</details>

<details>
<summary><strong>EPC & Energy</strong></summary>

| Field | Description |
|-------|-------------|
| `epc.currentRating` | Current EPC rating (A-G) |
| `epc.potentialRating` | Potential EPC rating |
| `epc.currentEfficiency` | Current efficiency score (1-100) |
| `epc.potentialEfficiency` | Potential efficiency score |

</details>

<details>
<summary><strong>Leasehold & Charges</strong></summary>

| Field | Description |
|-------|-------------|
| `leaseYearsRemaining` | Years remaining on lease |
| `charges.serviceCharge.current` | Annual service charge |
| `charges.groundRent.current` | Annual ground rent |

</details>

<details>
<summary><strong>Listings & Intelligence</strong></summary>

| Field | Description |
|-------|-------------|
| `listings.*` | Full listing data (nested) |
| `listings.publishedDate` | When listed |
| `listings.removedDate` | When removed |
| `listings.priceChange` | Price change history |

</details>

## Connection

**Endpoint:** `https://mcp.propbar.co.uk`

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
      "url": "https://mcp.propbar.co.uk"
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
  new URL('https://mcp.propbar.co.uk'),
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

// Get comparables for valuation
const result = await client.callTool('get_comparables', {
  coordinates: { latitude: 51.5074, longitude: -0.1278 },
  propertyType: 'flat',
  bedrooms: 2,
  radiusKm: 1,
  transactionType: 'sale'
})
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

**Flexible inputs:** Most tools accept EITHER `areaCode` OR `latitude`/`longitude` coordinates.

## Example Usage

### Get comparables for valuation
```json
{
  "tool": "get_comparables",
  "arguments": {
    "coordinates": { "latitude": 51.5074, "longitude": -0.1278 },
    "propertyType": "flat",
    "bedrooms": 2,
    "radiusKm": 1,
    "transactionType": "sale"
  }
}
```

**Returns:** Recent sales, SSTC properties, and current listings with prices, dates, floor areas, EPC ratings, and portal links.

### Check area safety
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
https://mcp.propbar.co.uk/.well-known/oauth-protected-resource
```

## Data Sources

- **Property Data:** [Propbar](https://propbar.co.uk) - 30M+ UK properties with sales history, listings, EPC data
- **Crime Statistics:** [Police UK](https://data.police.uk/) - Monthly crime data by location
- **Schools:** [Get Information About Schools](https://get-information-schools.service.gov.uk/) - Ofsted ratings, school types
- **Demographics:** [UK Census 2021](https://census.gov.uk/) - Population, households, tenure

## Related Products

- **Need automated marketing?** [EverySignal.ai](https://everysignal.ai) - AI-powered property marketing automation
- **Need consulting, enterprise solutions, or direct API access?** [Vepler.com](https://vepler.com) - UK property data consulting & enterprise platform

## Support

- **Website:** [propbar.co.uk](https://propbar.co.uk)
- **Email:** hello@propbar.co.uk
- **GitHub:** [Propbar/mcp](https://github.com/Propbar/mcp)

## License

Proprietary - Requires active Propbar subscription.

---

**Keywords:** UK property API, automated valuation model, AVM, property comparables, house price data, EPC ratings, UK house prices, property market analysis, MCP server, AI property tools, land registry data

*UK property research and valuation tools for AI assistants.*
