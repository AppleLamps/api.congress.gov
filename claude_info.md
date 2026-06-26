# Congress.gov API guide for agents

This repository documents and demonstrates how to use the Congress.gov API. It is primarily documentation and sample clients, not a service implementation. Use the repository docs and examples as the source of truth for endpoint shape, authentication, and response conventions.

## What this repo contains

- `README.md`: high-level overview, quickstart, authentication, JSON guidance, main endpoints, pagination, and rate-limit notes.
- `Documentation/`: endpoint-specific markdown docs and OpenAPI files (`swagger.yaml`, `swagger.json`). The OpenAPI spec is the most complete endpoint reference.
- `python/` and `api_client/python/`: example Python clients and scripts.
- `java/` and `api_client/java/`: example Java clients.
- `ChangeLog.md`: record of API changes, impacted endpoints, and release dates.

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

## Complete endpoint reference

The main endpoint family is rooted at `/v3`. All paths below are relative to `https://api.congress.gov/v3`. For full parameter details, see `Documentation/swagger.yaml` or the endpoint-specific markdown files in `Documentation/`.

### Bills

Bill types: `hr`, `s`, `hjres`, `sjres`, `hconres`, `sconres`, `hres`, `sres`

- `/bill` — latest bills (sorted by latest action date)
- `/bill/{congress}` — bills from a specific Congress
- `/bill/{congress}/{billType}` — bills filtered by Congress and bill type
- `/bill/{congress}/{billType}/{billNumber}` — a specific bill
- `/bill/{congress}/{billType}/{billNumber}/actions` — legislative actions
- `/bill/{congress}/{billType}/{billNumber}/amendments` — amendments to the bill
- `/bill/{congress}/{billType}/{billNumber}/committees` — committee assignments
- `/bill/{congress}/{billType}/{billNumber}/cosponsors` — bill cosponsors
- `/bill/{congress}/{billType}/{billNumber}/relatedbills` — related bills
- `/bill/{congress}/{billType}/{billNumber}/subjects` — legislative subjects
- `/bill/{congress}/{billType}/{billNumber}/summaries` — CRS summaries
- `/bill/{congress}/{billType}/{billNumber}/text` — text versions
- `/bill/{congress}/{billType}/{billNumber}/titles` — bill titles

### Laws

Law types: `pub` (public law), `priv` (private law)

- `/law/{congress}` — laws from a specific Congress
- `/law/{congress}/{lawType}` — laws filtered by type
- `/law/{congress}/{lawType}/{lawNumber}` — a specific law

### Members

- `/member` — all members
- `/member/{bioguideId}` — a specific member by Bioguide ID
- `/member/{bioguideId}/sponsored-legislation` — bills sponsored by a member (sortable by `introducedDate` or `updateDate`)
- `/member/{bioguideId}/cosponsored-legislation` — bills cosponsored by a member (excludes withdrawn cosponsorships; sortable by `introducedDate` or `updateDate`)
- `/member/{stateCode}` — members by two-letter state code (e.g., `VT`)
- `/member/{stateCode}/{district}` — House members by state and district
- `/member/congress/{congress}` — members who served in a specific Congress
- `/member/congress/{congress}/{stateCode}/{district}` — member for a specific Congress, state, and district

**Filtering notes for members:**
- When retrieving members from prior Congresses via `/member/congress/{congress}`, include `currentMember=false` to get the most complete historical data.
- When you need only the current representative for a specific district, use `currentMember=true` (e.g., `/member/congress/118/TX/15?currentMember=true`).

### Amendments

Amendment types: `hamdt` (House), `samdt` (Senate), `suamdt` (Senate unamended — 97th/98th Congress only)

- `/amendment` — latest amendments (sorted by latest action date)
- `/amendment/{congress}` — amendments from a specific Congress
- `/amendment/{congress}/{amendmentType}` — filtered by type
- `/amendment/{congress}/{amendmentType}/{amendmentNumber}` — a specific amendment
- `/amendment/{congress}/{amendmentType}/{amendmentNumber}/actions` — amendment actions
- `/amendment/{congress}/{amendmentType}/{amendmentNumber}/amendments` — amendments to the amendment
- `/amendment/{congress}/{amendmentType}/{amendmentNumber}/cosponsors` — cosponsors
- `/amendment/{congress}/{amendmentType}/{amendmentNumber}/text` — text versions

### Summaries

- `/summaries` — latest CRS bill summaries
- `/summaries/{congress}` — summaries from a specific Congress
- `/summaries/{congress}/{billType}` — summaries filtered by bill type

### Congress

- `/congress` — list of all Congresses
- `/congress/current` — the current Congress
- `/congress/{congress}` — a specific Congress including session dates

### Committees

Chamber values: `house`, `senate`, `joint`

- `/committee` — all committees
- `/committee/{congress}` — committees active in a specific Congress
- `/committee/{congress}/{chamber}` — filtered by Congress and chamber
- `/committee/{congress}/{chamber}/{committeeCode}` — a specific committee in a Congress
- `/committee/{chamber}` — committees by chamber
- `/committee/{chamber}/{committeeCode}` — a specific committee
- `/committee/{chamber}/{committeeCode}/bills` — bills referred to the committee
- `/committee/{chamber}/{committeeCode}/reports` — committee reports
- `/committee/{chamber}/{committeeCode}/nominations` — nominations referred to the committee
- `/committee/{chamber}/{committeeCode}/house-communication` — House communications
- `/committee/{chamber}/{committeeCode}/senate-communication` — Senate communications

### Committee reports

Report types: `hrpt` (House), `srpt` (Senate), `erpt` (Executive)

- `/committee-report` — all committee reports (sortable by `updateDate`)
- `/committee-report/{congress}` — reports from a specific Congress
- `/committee-report/{congress}/{reportType}` — filtered by type
- `/committee-report/{congress}/{reportType}/{reportNumber}` — a specific report
- `/committee-report/{congress}/{reportType}/{reportNumber}/text` — text versions

### Committee meetings

- `/committee-meeting` — all committee meetings
- `/committee-meeting/{congress}` — meetings from a specific Congress
- `/committee-meeting/{congress}/{chamber}` — filtered by chamber
- `/committee-meeting/{congress}/{chamber}/{eventId}` — a specific meeting

### Committee prints

- `/committee-print` — all committee prints
- `/committee-print/{congress}` — prints from a specific Congress
- `/committee-print/{congress}/{chamber}` — filtered by chamber
- `/committee-print/{congress}/{chamber}/{jacketNumber}` — a specific print
- `/committee-print/{congress}/{chamber}/{jacketNumber}/text` — text versions

### Hearings

- `/hearing` — all hearings
- `/hearing/{congress}` — hearings from a specific Congress
- `/hearing/{congress}/{chamber}` — filtered by chamber
- `/hearing/{congress}/{chamber}/{jacketNumber}` — a specific hearing

### Nominations

Nomination numbers may include a partition suffix (e.g., `230-1`, `230-2`) when the Senate partitions a multi-nominee nomination.

- `/nomination` — all nominations
- `/nomination/{congress}` — nominations from a specific Congress
- `/nomination/{congress}/{nominationNumber}` — a specific nomination
- `/nomination/{congress}/{nominationNumber}/{ordinal}` — a specific nominee within a nomination
- `/nomination/{congress}/{nominationNumber}/actions` — nomination actions
- `/nomination/{congress}/{nominationNumber}/committees` — committee referrals
- `/nomination/{congress}/{nominationNumber}/hearings` — associated hearings

### Treaties

- `/treaty` — all treaties
- `/treaty/{congress}` — treaties from a specific Congress
- `/treaty/{congress}/{treatyNumber}` — a specific treaty
- `/treaty/{congress}/{treatyNumber}/actions` — treaty actions
- `/treaty/{congress}/{treatyNumber}/committees` — committee referrals
- `/treaty/{congress}/{treatyNumber}/{treatySuffix}` — a partitioned treaty
- `/treaty/{congress}/{treatyNumber}/{treatySuffix}/actions` — partitioned treaty actions

### House roll call votes (Beta)

Beta coverage: all legislation-related votes in the 118th and 119th Congresses.

- `/house-vote` — all House roll call votes
- `/house-vote/{congress}` — votes from a specific Congress
- `/house-vote/{congress}/{session}` — filtered by session (1 or 2)
- `/house-vote/{congress}/{session}/{voteNumber}` — a specific vote
- `/house-vote/{congress}/{session}/{voteNumber}/members` — individual member votes

### Communications

- `/house-communication` — House executive communications and petitions
- `/house-communication/{congress}` — filtered by Congress
- `/house-communication/{congress}/{communicationType}` — filtered by type
- `/house-communication/{congress}/{communicationType}/{communicationNumber}` — a specific communication
- `/house-requirement` — House requirements
- `/house-requirement/{requirementNumber}` — a specific requirement
- `/house-requirement/{requirementNumber}/matching-communications` — communications matching a requirement
- `/senate-communication` — Senate executive communications and petitions
- `/senate-communication/{congress}` — filtered by Congress
- `/senate-communication/{congress}/{communicationType}` — filtered by type
- `/senate-communication/{congress}/{communicationType}/{communicationNumber}` — a specific communication

### Congressional Record

- `/congressional-record` — Congressional Record issues
- `/daily-congressional-record` — daily edition issues
- `/daily-congressional-record/{volumeNumber}` — filtered by volume
- `/daily-congressional-record/{volumeNumber}/{issueNumber}` — a specific issue
- `/daily-congressional-record/{volumeNumber}/{issueNumber}/articles` — articles in an issue
- `/bound-congressional-record` — bound edition volumes
- `/bound-congressional-record/{year}` — filtered by year
- `/bound-congressional-record/{year}/{month}` — filtered by month
- `/bound-congressional-record/{year}/{month}/{day}` — a specific day

### CRS Reports

- `/crsreport` — Congressional Research Service reports
- `/crsreport/{reportNumber}` — a specific CRS report

## Pagination and limits

- Default page size: 20 results
- Maximum page size: 250 results
- Use `limit` to control page size and `offset` to move through pages.
- The API rate limit is 5,000 requests per hour.

Example pagination request:

```bash
curl "https://api.congress.gov/v3/bill?api_key=$CONGRESS_API_KEY&format=json&limit=250&offset=250"
```

## Sorting

Many list endpoints support a `sort` parameter. Common sort options:

- `updateDate+asc` / `updateDate+desc`
- `introducedDate+asc` / `introducedDate+desc` (sponsored/cosponsored legislation)
- `latestAction.date+asc` / `latestAction.date+desc`

The default sort for bill and amendment list endpoints is by latest action date (descending).

## Response shape

List endpoints generally return:

- `request`: information about the request (format, content type, and related request metadata)
- `pagination`: count/next/previous links when applicable
- a collection object named for the endpoint (for example `bills`, `amendments`, `members`)

Each item in a list response typically includes a `url` field pointing to the detail endpoint for that item.

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
      "originChamber": "House",
      "title": "Postal Service Reform Act",
      "type": "HR",
      "updateDate": "2024-01-01",
      "url": "https://api.congress.gov/v3/bill/118/hr/3076?format=json"
    }
  ]
}
```

When responses are returned in XML, an `<api-root>` element is present as the root wrapper.

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

### curl: list committee reports

```bash
curl "https://api.congress.gov/v3/committee-report?api_key=$CONGRESS_API_KEY&format=json&limit=10"
```

### curl: get a House roll call vote

```bash
curl "https://api.congress.gov/v3/house-vote/118/1/1?api_key=$CONGRESS_API_KEY&format=json"
```

### curl: get the current Congress

```bash
curl "https://api.congress.gov/v3/congress/current?api_key=$CONGRESS_API_KEY&format=json"
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

### JavaScript: fetch example

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

main().catch((error) => { console.error(error); process.exit(1); });
```

## Repository-specific implementation notes

- The Python sample client (`python/cdg_client.py`) constructs requests against `/v3/` and sets `format=json` and the `x-api-key` header automatically.
- The example script (`python/bill_example.py`) demonstrates bill endpoints. It uses the sample client, which defaults to JSON.
- The OpenAPI files in `Documentation/swagger.yaml` and `Documentation/swagger.json` provide a complete, machine-readable endpoint reference including all parameters and response schemas.
- Coverage dates (how far back data goes for each collection) are documented at https://www.congress.gov/help/coverage-dates.
- API changes are tracked in `ChangeLog.md` in this repository.

## Practical guidance for agents

- Treat `README.md` as the main usage reference; use `Documentation/swagger.yaml` for complete parameter details.
- Always include `format=json` unless you are intentionally testing XML behavior.
- Prefer explicit endpoint paths over guessing; the endpoint structure is path-based and predictable.
- For list endpoints, use `limit/offset` rather than trying to fetch everything at once.
- Respect the 5,000 requests/hour limit and avoid repeated retries without backoff.
- When a response includes pagination metadata, follow `next`/`previous` links when needed.
- Use the `url` field returned in list items to navigate directly to detail endpoints without reconstructing paths.
- The House roll call vote endpoint is marked Beta and currently covers only legislation-related votes in the 118th and 119th Congresses.
