# CAP Mockoon AI Agent Instructions (Lean)

This guide instructs AI agents to produce a Mockoon environment for CAP APIs using the provided OpenAPI contract.

## Overview
- Goal: Create realistic local mocks for CAP endpoints to unblock UI flows.
- Input: OpenAPI JSON contract at [docs/cap-prodsvcadm-proddealacctadm-10xacctsvcng-v1-api (1).json](docs/cap-prodsvcadm-proddealacctadm-10xacctsvcng-v1-api%20(1).json).
- Output: Mockoon environment JSON (e.g., [galaxy-accountpac-microfrontend/api/env/sit-w1c2/mockoon-data-cap.json](galaxy-accountpac-microfrontend/api/env/sit-w1c2/mockoon-data-cap.json)).

## Prerequisites
- Node 22 on macOS: `nvm use 22`.
- Contract accessible and validated.
- Know the consuming base path (e.g., `/exp/galaxy-accountpac`) and required headers: `x-brandId`, `x-channelType`, `x-channelRequestId`.

## Minimal Conventions
- Success responses: align with contract schemas; include only fields needed by UI.
- Error template (reuse across 4xx/5xx):
  {
    "error": {
      "code": "BAD_REQUEST|NOT_FOUND|CONFLICT|UNPROCESSABLE_ENTITY|SERVER_ERROR",
      "message": "Summary",
      "details": [{ "field|header|query|param": "name" }]
    }
  }
- Use Mockoon rules to switch variants by headers/queries/params.

## Lean Workflow
1. Enumerate endpoints from the contract
   - List `paths` + HTTP methods; capture required headers and key params.
   - Note standard success status (usually 200/201) and any documented errors.

2. Build the environment JSON
   - One route per operation: unique `uuid`, `method`, `endpoint` (no leading `/`).
   - Provide a 2xx success response body that matches schema minimally.
   - Add error responses and rules for missing/invalid headers/queries/params.

3. Common rules (copy/paste and tweak)
   - Missing header `x-brandId` (400):
     {
       "target": "headers", "modifier": "x-brandId",
       "value": ".+", "invert": true, "operator": "regex"
     }
   - Invalid query `BSB` (400):
     {
       "target": "query", "modifier": "BSB",
       "value": "^\\d{6}$", "invert": true, "operator": "regex"
     }
   - Path param match `caseId` (404 variant selection):
     {
       "target": "params", "modifier": "caseId",
       "value": "CASE-99999", "invert": false, "operator": "equals"
     }

4. Register routes
   - Add each route `uuid` under the environment `rootChildren` as `{ "type": "route", "uuid": "<uuid>" }`.

## Sample Route Snippets
- 200 success (example fields; adapt to CAP schema):
  {
    "uuid": "...",
    "statusCode": 200,
    "label": "200 OK",
    "bodyType": "INLINE",
    "body": "{\n  \"result\": \"OK\",\n  \"referenceId\": \"CAP-001\"\n}"
  }
- 400 missing `x-brandId`:
  {
    "uuid": "...",
    "statusCode": 400,
    "label": "400 Bad Request - Missing x-brandId",
    "bodyType": "INLINE",
    "body": "{\n  \"error\": {\n    \"code\": \"BAD_REQUEST\",\n    \"message\": \"Missing x-brandId\"\n  }\n}",
    "rules": [ { "target": "headers", "modifier": "x-brandId", "value": ".+", "invert": true, "operator": "regex" } ]
  }

## Endpoint Coverage (derive from contract)
- Use the contract to drive the list. Typical CAP surfaces include account details, product packages, routing, health.
- Always include health: `GET /healthcheck` with `{ "status": "UP" }`.

## Run & Verify
- Place the environment file under the target MFE, e.g. [galaxy-accountpac-microfrontend/api/env/sit-w1c2/mockoon-data-cap.json](galaxy-accountpac-microfrontend/api/env/sit-w1c2/mockoon-data-cap.json).
- Start mocks (from MFE root):

```bash
nvm use 22
npm run mockoon
```

- Quick curl checks (adjust port/base path per environment):

```bash
# Healthcheck success
curl -i 'http://localhost:3106/exp/galaxy-accountpac/healthcheck'

# Example endpoint with headers
curl -i -H 'x-brandId: WBC' -H 'x-channelType: WEB' -H 'x-channelRequestId: req-123' \
  'http://localhost:3106/exp/galaxy-accountpac/<cap-endpoint>'

# Header-missing error selection
curl -i 'http://localhost:3106/exp/galaxy-accountpac/<cap-endpoint>'
```

## Maintenance Tips
- Keep bodies minimal, schema-aligned; expand only when UI needs fields.
- Prefer `regex + invert` for required header/query checks.
- Reuse the error template for consistency.
- Document base path/port differences if any.

## Optional: Automate Generation
- If needed, implement a small Node script to parse the OpenAPI JSON and emit a Mockoon environment (place under `tools/` and write to `api/env/<env>/mockoon-data-cap.json`).
- Keep rule generation simple (required headers, basic query/param validation); refine per UI needs.
