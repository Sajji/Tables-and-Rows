# Row Counts Explorer (tables.html)

A single-file Collibra widget that discovers columns with a Row Count numeric attribute, groups them by table, and presents an interactive explorer with search (by name and description), expand/collapse, stats, and JSON/CSV export.

## What It Does

- Calls Collibra session API to fetch a CSRF token
- Queries Knowledge Graph GraphQL for assets that contain the Row Count attribute
- Maps each column asset to its related table via outgoing relations
- Pulls Description string attributes for both columns and their parent tables
- Groups columns under their parent table
- Shows per-table row count, number of columns, and table description
- Renders direct links to each table and column asset in Collibra
- Provides:
  - Search across table and column names
  - Separate search across table and column descriptions
  - Expand all / collapse all
  - Refresh
  - Download result as JSON or CSV

## File

- `tables.html` - Complete UI, styles, and JavaScript logic in one file

## Data Flow

1. Read base URL from `window.location.origin`.
2. GET session data to retrieve CSRF token.
3. POST paginated GraphQL requests (page size 100) until no more assets are returned.
4. Convert raw assets into:
   - Table name / id / description
   - Column name / id / description
   - Row count value
5. Group by table and render expandable cards with deep links to each asset.

## Endpoints Used

- `GET /rest/2.0/auth/sessions/current?include=csrfToken`
  - Retrieves CSRF token for GraphQL calls
- `POST /graphql/knowledgeGraph/v1`
  - Executes GraphQL query for assets with Row Count numeric attribute

## GraphQL Filter Logic

The query requests assets where numeric attributes include type name `Row Count`, then reads:
- Asset metadata (`id`, `displayName`, `fullName`)
- Numeric attribute value (`numericValue`) for `Row Count`
- String attribute (`stringValue`) for `Description` on the column
- Outgoing relations, including target id/name/type and the target's `Description` string attribute, used to identify the related Table asset

## UI Features

- Header actions: Refresh, Download JSON, Download CSV
- Stats bar: tables count, columns count, max row count
- Two search inputs:
  - Name search: filters on table and column names
  - Description search: filters on table and column descriptions
- Expand/collapse per card and globally
- Asset links: table names and column names link out to `/asset/{id}` in Collibra (open in a new tab)
- Description display: table descriptions render under the table name; column descriptions render under each column
- Progress bar while paging GraphQL results
- Error box for auth/query/network issues

## Export Shapes

### JSON (`row_counts.json`)

```json
[
  {
    "tableName": "Customer",
    "tableDescription": "Master customer table",
    "rowCount": 120034,
    "columns": [
      { "name": "Customer ID", "description": "Primary key" },
      { "name": "Name", "description": null },
      { "name": "Region", "description": "ISO region code" }
    ]
  }
]
```

### CSV (`row_counts.csv`)

Columns: `Table Name, Row Count, Table Description, Column Name, Column Description`

One row per column. Values are quoted and embedded quotes are escaped.

Notes:
- Exports intentionally strip internal asset IDs.
- Row count and descriptions can be null/empty when missing.

## Usage in Collibra

1. Upload `tables.html` under your resources/images path.
2. Add an HTML widget that points to this file.
3. Open the widget while authenticated in Collibra.

## Local Development

1. Serve from a local/static server (recommended), or open directly for UI-only testing.
2. For real data, run in an environment where:
   - The browser has an active Collibra session
   - The current origin can reach your Collibra APIs

## Permissions and Access

To return results, the active user must be able to:
- Access session endpoint
- Execute the Knowledge Graph query
- Read relevant assets, relations, and Description attributes

## Known Limitations

- Table detection depends on an outgoing relation to a target with type name `Table`.
- If relation modeling differs, rows may appear under `Unknown Table`.
- Progress bar is approximate during pagination and finalizes at completion.
- Large datasets can take time because all pages are fetched client-side.
- Description search only matches values present on the column or its parent table.

## Troubleshooting

- Error: Failed to fetch CSRF token
  - Verify active Collibra session and endpoint access
- Error: Request failed (4xx/5xx)
  - Check network access, auth, and proxy/CORS behavior in your environment
- Empty results
  - Confirm assets actually have numeric attribute type named exactly `Row Count`
- Missing descriptions
  - Confirm the string attribute type is named exactly `Description`

## License

MIT (or your repository standard license)
