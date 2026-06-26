# Congress.gov API

The [Congress.gov Application Programming Interface (API)](https://api.congress.gov/) provides a way for Congress, researchers, developers, journalists, civic technologists, and the public to retrieve and reuse machine-readable data from Congress.gov collections.

This repository contains information on accessing and using the Congress.gov API, including examples, endpoint references, change management notes, and support information.

## Quickstart

1. Request an API key from [api.congress.gov/sign-up](https://api.congress.gov/sign-up/).
2. Copy `.env.example` to `.env`.
3. Add your API key:

```bash
CONGRESS_API_KEY=your_api_key_here
```

4. Make a request against the v3 API root.

```bash
curl "https://api.congress.gov/v3/bill?api_key=$CONGRESS_API_KEY&format=json&limit=5"
```

The API supports both XML and JSON responses. API clients should explicitly request JSON by adding `format=json` to requests. Do not rely on the default response format.

## Base URL

```text
https://api.congress.gov/v3
```

Example endpoint:

```text
https://api.congress.gov/v3/bill
```

## Authentication

An API key is required for access. Sign up for a key here:

```text
https://api.congress.gov/sign-up/
```

Pass the API key using the `api_key` query parameter:

```text
https://api.congress.gov/v3/bill?api_key=YOUR_API_KEY&format=json
```

Learn more about API key usage on [api.data.gov](https://api.data.gov/docs/api-key/).

## Requesting JSON

Congress.gov API responses are available in XML and JSON. The API has historically supported multiple response formats, and the changelog notes that the default response format was corrected to XML across endpoints.

For reliable application behavior, always add this parameter when you want JSON:

```text
format=json
```

Recommended:

```text
https://api.congress.gov/v3/bill?api_key=YOUR_API_KEY&format=json
```

Not recommended:

```text
https://api.congress.gov/v3/bill?api_key=YOUR_API_KEY
```

## curl Example

Fetch the five most recent bills:

```bash
curl "https://api.congress.gov/v3/bill?api_key=$CONGRESS_API_KEY&format=json&limit=5"
```

Fetch bill details:

```bash
curl "https://api.congress.gov/v3/bill/118/hr/3076?api_key=$CONGRESS_API_KEY&format=json"
```

## Python Example

```python
import os
import requests

API_KEY = os.environ["CONGRESS_API_KEY"]
BASE_URL = "https://api.congress.gov/v3"

response = requests.get(
    f"{BASE_URL}/bill",
    params={
        "api_key": API_KEY,
        "format": "json",
        "limit": 5,
    },
    timeout=30,
)

response.raise_for_status()
data = response.json()

for bill in data.get("bills", []):
    print(bill.get("number"), bill.get("title"))
```

## JavaScript Example

```javascript
const API_KEY = process.env.CONGRESS_API_KEY;
const BASE_URL = "https://api.congress.gov/v3";

async function main() {
  const url = new URL(`${BASE_URL}/bill`);
  url.searchParams.set("api_key", API_KEY);
  url.searchParams.set("format", "json");
  url.searchParams.set("limit", "5");

  const response = await fetch(url);

  if (!response.ok) {
    throw new Error(`Request failed: ${response.status} ${response.statusText}`);
  }

  const data = await response.json();

  for (const bill of data.bills ?? []) {
    console.log(`${bill.number}: ${bill.title}`);
  }
}

main().catch((error) => {
  console.error(error);
  process.exit(1);
});
```

## Major Endpoints

| Use case | Endpoint | Description |
| --- | --- | --- |
| Latest bills | `/bill` | Returns a list of bills. Supports pagination with `limit` and `offset`. |
| Bills by Congress | `/bill/{congress}` | Returns bills from a specific Congress. |
| Bills by Congress and type | `/bill/{congress}/{billType}` | Returns bills from a specific Congress and bill type, such as `hr`, `s`, `hjres`, or `sres`. |
| Bill details | `/bill/{congress}/{billType}/{billNumber}` | Returns details for a specific bill. |
| Bill actions | `/bill/{congress}/{billType}/{billNumber}/actions` | Returns actions for a specific bill. |
| Bill amendments | `/bill/{congress}/{billType}/{billNumber}/amendments` | Returns amendments for a specific bill. |
| Bill summaries | `/bill/{congress}/{billType}/{billNumber}/summaries` | Returns summaries for a specific bill. |
| Bill text | `/bill/{congress}/{billType}/{billNumber}/text` | Returns text versions for a specific bill. |
| Member by Bioguide ID | `/member/{bioguideId}` | Returns details for a member of Congress. |
| Sponsored legislation | `/member/{bioguideId}/sponsored-legislation` | Returns legislation sponsored by a member. |
| Cosponsored legislation | `/member/{bioguideId}/cosponsored-legislation` | Returns legislation cosponsored by a member. |
| Committee reports | `/committee-report` | Returns committee reports. |
| Committee report details | `/committee-report/{congress}/{reportType}/{reportNumber}` | Returns details for a specific committee report. |
| House votes | `/house-vote/{congress}/{sessionNumber}/{voteNumber}` | Returns details for a House roll call vote. |
| Amendments | `/amendment` | Returns amendments. |
| Treaties | `/treaty` | Returns treaties. |
| CRS reports | `/crsreport` | Returns Congressional Research Service reports available through the API. |

## Common Use Cases

### Latest bills

```bash
curl "https://api.congress.gov/v3/bill?api_key=$CONGRESS_API_KEY&format=json&limit=10"
```

### Bill details

```bash
curl "https://api.congress.gov/v3/bill/118/hr/3076?api_key=$CONGRESS_API_KEY&format=json"
```

### Member by Bioguide ID

```bash
curl "https://api.congress.gov/v3/member/L000174?api_key=$CONGRESS_API_KEY&format=json"
```

### Committee reports

```bash
curl "https://api.congress.gov/v3/committee-report?api_key=$CONGRESS_API_KEY&format=json&limit=10"
```

### House votes

```bash
curl "https://api.congress.gov/v3/house-vote/118/1/1?api_key=$CONGRESS_API_KEY&format=json"
```

## Sample JSON Response

List endpoints generally return request metadata, pagination metadata, and a data container named for the collection being returned.

Example shape for `/bill?format=json&limit=1`:

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
      "originChamber": "House",
      "title": "Postal Service Reform Act",
      "type": "HR",
      "updateDate": "2024-01-01",
      "url": "https://api.congress.gov/v3/bill/118/hr/3076?format=json"
    }
  ]
}
```

Exact fields vary by endpoint and collection.

## Response Structure

For every request, three general elements are returned:

- The **Request** element contains information about the API request itself. This includes the format and content type.
- The **Pagination** element contains the total count of data items in the response, a URL for the next page of results, and, when applicable, a URL for the previous page of results.
- The **Data** element changes depending on the endpoint. For example, the bill endpoint returns a `bills` collection, while the amendment endpoint returns an `amendments` collection.

When responses are returned in XML, an `<api-root>` element will be visible.

## Versioning

The current version of the API is version 3 (`v3`). Prior versions were used by the Government Publishing Office (GPO) for its [Bulk Data Repository](https://www.govinfo.gov/bulkdata) and other clients.

## Rate Limit

The rate limit is set to 5,000 requests per hour.

## Limit and Offset

By default, the API returns 20 results starting with the first record. The `limit` parameter can be adjusted up to 250 results. If `limit` is greater than 250, only 250 results will be returned.

Use `offset` to request later pages of results.

Example:

```bash
curl "https://api.congress.gov/v3/bill?api_key=$CONGRESS_API_KEY&format=json&limit=250&offset=250"
```

## Coverage and Estimated Update Times for Congress.gov Collections

Coverage information for Congress.gov collections data in the API can be found at [Coverage Dates for Congress.gov Collections](https://www.congress.gov/help/coverage-dates) on Congress.gov. This page also provides estimated update times for Congress.gov collections.

## Change Management

Congress.gov staff issue change management communication through the [ChangeLog](https://github.com/LibraryOfCongress/api.congress.gov/blob/main/ChangeLog.md) so consumers can adjust accordingly. The changelog contains information on updates to the API, impacted endpoints, and expected production release dates. Milestones are also used to tag issues with expected production release date information.

## Support

Congress.gov staff monitor and respond to [issues](https://github.com/LibraryOfCongress/api.congress.gov/issues) created in this repository and initiate actions as necessary. Before creating a new issue, review existing issues and add a comment to any issue relevant to yours.

## Relevant Privacy Policies

1. API keys and user registration follow the data.gov privacy policy. Read more [here](https://data.gov/privacy-policy/).
2. API content follows the Library of Congress privacy policy. Read more [here](https://www.loc.gov/legal/privacy-policy/).
