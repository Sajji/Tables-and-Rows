# Row Counts Explorer (tables.html)

A single-file Collibra widget that discovers columns with a Row Count numeric attribute, groups them by table, and presents an interactive explorer with search, expand/collapse, stats, and JSON export.

## What It Does

- Calls Collibra session API to fetch a CSRF token
- Queries Knowledge Graph GraphQL for assets that contain the Row Count attribute
- Maps each column asset to its related table via outgoing relations
- Groups columns under their parent table
- Shows per-table row count and number of columns
- Provides:
  - Search across table names and column names
  - Expand all / collapse all
  - Refresh
  - Download result as JSON

## File

- `tables.html` - Complete UI, styles, and JavaScript logic in one file

## Data Flow

1. Read base URL from `window.location.origin`.
2. GET session data to retrieve CSRF token.
3. POST paginated GraphQL requests (page size 100) until no more assets are returned.
4. Convert raw assets into:
   - Table name/id
   - Column name/id
   - Row count value
5. Group by table and render expandable cards.

## Endpoints Used

- `GET /rest/2.0/auth/sessions/current?include=csrfToken`
  - Retrieves CSRF token for GraphQL calls
- `POST /graphql/knowledgeGraph/v1`
  - Executes GraphQL query for assets with Row Count numeric attribute

## GraphQL Filter Logic

The query requests assets where numeric attributes include type name `Row Count`, then reads:
- Asset metadata (`id`, `displayName`, `fullName`)
- Numeric attribute value (`numericValue`)
- Outgoing relation targets to identify the related Table asset

## UI Features

- Header actions: Refresh, Download JSON
- Stats bar: tables count, columns count, max row count
- Search filter: table + column text match
- Expand/collapse per card and globally
- Progress bar while paging GraphQL results
- Error box for auth/query/network issues

## JSON Export Shape

Downloaded filename: `row_counts.json`

```json
[
  {
    "tableName": "Customer",
    "rowCount": 120034,
    "columns": ["Customer ID", "Name", "Region"]
  }
]
```

Notes:
- Export intentionally strips internal asset IDs.
- Row count can be null/empty when the attribute is missing.

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
- Read relevant assets and relations

## Known Limitations

- Table detection depends on an outgoing relation to a target with type name `Table`.
- If relation modeling differs, rows may appear under `Unknown Table`.
- Progress bar is approximate during pagination and finalizes at completion.
- Large datasets can take time because all pages are fetched client-side.

## Troubleshooting

- Error: Failed to fetch CSRF token
  - Verify active Collibra session and endpoint access
- Error: Request failed (4xx/5xx)
  - Check network access, auth, and proxy/CORS behavior in your environment
- Empty results
  - Confirm assets actually have numeric attribute type named exactly `Row Count`

## License

MIT (or your repository standard license)
