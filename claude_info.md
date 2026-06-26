# Congress.gov API guide for agents

This repository documents and demonstrates how to use the Congress.gov API. It is primarily documentation and sample clients, not a service implementation. Use the repository docs and examples as the source of truth for endpoint shape, authentication, and response conventions.

## What this repo contains

- `README.md`: high-level overview, quickstart, authentication, JSON guidance, main endpoints, pagination, and rate-limit notes.
- `Documentation/`: endpoint-specific markdown docs and OpenAPI files (`swagger.yaml`, `swagger.json`).
- `python/` and `api_client/python/`: example Python clients and scripts.
- `java/` and `api_client/java/`: example Java clients.

## API basics

- Base URL: `https://api.congress.gov/v3`
- Current API version: `v3`
- API key required: sign up at https://api.congress.gov/sign-up/
- Always request JSON explicitly with `format=json` unless you intentionally want XML.
- The repo explicitly warns against relying on the default response format.

## Authentication

The public README shows the key passed as a query parameter:

```text
https://api.congress.gov/v3/bill?api_key=YOUR_API_KEY&format=json
```

The sample Python client uses the `x-api-key` header instead. In practice:

- For raw HTTP requests, include `api_key` as a query parameter.
- For the Python examples in this repo, the client sets the `x-api-key` header.

Preferred environment variable names in this repo:

```bash
CONGRESS_API_KEY=your_api_key_here
CONGRESS_API_BASE_URL=https://api.congress.gov/v3
```

## Requesting data

Use `format=json` for reliable programmatic use.

Example:

```bash
curl "https://api.congress.gov/v3/bill?api_key=$CONGRESS_API_KEY&format=json&limit=5"
```

## Common endpoint patterns

The main endpoint family is rooted at `/v3`.

### Bills

- `/bill` — latest bills
- `/bill/{congress}` — bills in a specific Congress
- `/bill/{congress}/{billType}` — bills in a Congress and bill type (for example `hr`, `s`, `hjres`, `sres`)
- `/bill/{congress}/{billType}/{billNumber}` — a specific bill
- `/bill/{congress}/{billType}/{billNumber}/actions`
- `/bill/{congress}/{billType}/{billNumber}/amendments`
- `/bill/{congress}/{billType}/{billNumber}/summaries`
- `/bill/{congress}/{billType}/{billNumber}/text`

### Members

- `/member/{bioguideId}`
- `/member/{bioguideId}/sponsored-legislation`
- `/member/{bioguideId}/cosponsored-legislation`

### Other collections

- `/committee-report`
- `/committee-report/{congress}/{reportType}/{reportNumber}`
- `/house-vote/{congress}/{sessionNumber}/{voteNumber}`
- `/amendment`
- `/treaty`
- `/crsreport`

## Pagination and limits

- Default page size: 20 results
- Maximum page size: 250 results
- Use `limit` to control page size and `offset` to move through pages.
- The API rate limit is 5,000 requests per hour.

Example pagination request:

```bash
curl "https://api.congress.gov/v3/bill?api_key=$CONGRESS_API_KEY&format=json&limit=250&offset=250"
```

## Response shape

List endpoints generally return:

- `request`: information about the request (format, content type, and related request metadata)
- `pagination`: count/next/previous links when applicable
- a collection object named for the endpoint (for example `bills`, `amendments`, `members`)

Example list response shape:

```json
{
  "request": {
    "contentType": "application/json",
    "format": "json"
  },
  "pagination": {
    "count": 1,
    "next": "https://api.congress.gov/v3/bill?offset=1&limit=1&format=json"
  },
  "bills": [
    {
      "congress": 118,
      "number": "3076",
      "title": "Postal Service Reform Act"
    }
  ]
}
```

## Example requests

### curl: list recent bills

```bash
curl "https://api.congress.gov/v3/bill?api_key=$CONGRESS_API_KEY&format=json&limit=10"
```

### curl: get a specific bill

```bash
curl "https://api.congress.gov/v3/bill/118/hr/3076?api_key=$CONGRESS_API_KEY&format=json"
```

### curl: get a member by Bioguide ID

```bash
curl "https://api.congress.gov/v3/member/L000174?api_key=$CONGRESS_API_KEY&format=json"
```

### Python: simple requests example

```python
import os
import requests

API_KEY = os.environ["CONGRESS_API_KEY"]
BASE_URL = "https://api.congress.gov/v3"

response = requests.get(
    f"{BASE_URL}/bill",
    params={"api_key": API_KEY, "format": "json", "limit": 5},
    timeout=30,
)
response.raise_for_status()
data = response.json()

for bill in data.get("bills", []):
    print(bill.get("number"), bill.get("title"))
```

## Repository-specific implementation notes

- The Python sample client (`python/cdg_client.py`) constructs requests against `/v3/` and sets `format` and `x-api-key` automatically.
- The example script (`python/bill_example.py`) demonstrates bill endpoints and XML parsing. If you are working with the sample client, be aware that it defaults to XML in the example script unless you change the client configuration.
- The OpenAPI files in `Documentation/` are useful if you need a more complete endpoint reference.

## Practical guidance for agents

- Treat `README.md` as the main usage reference.
- Always include `format=json` unless you are intentionally testing XML behavior.
- Prefer explicit endpoint paths over guessing; the endpoint structure is path-based and predictable.
- For list endpoints, use `limit/offset` rather than trying to fetch everything at once.
- Respect the 5,000 requests/hour limit and avoid repeated retries without backoff.
- When a response includes pagination metadata, follow `next`/`previous` links when needed.
