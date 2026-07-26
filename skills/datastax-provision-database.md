---
name: Provision an Astra database and download its secure bundle
description: Create a serverless Astra DB database, wait for it to become active, add a keyspace, and download the secure-connect bundle so a client can connect.
api: openapi/datastax-devops-openapi.json
operations:
- createDatabase
- listDatabases
- getDatabase
- addKeyspace
- generateSecureBundleURL
---

# Provision an Astra database

Use the Astra DevOps API (`https://api.astra.datastax.com`). Authenticate every
request with an application token: `Authorization: Bearer AstraCS:...`. The token
must carry the `org-db-create` and `db-keyspace-create` scopes (see
`scopes/datastax-scopes.yml`).

## Steps

1. **Create the database** — `POST /v2/databases` (`createDatabase`) with a name,
   keyspace, cloud provider, region, and tier. Returns `201` with the new
   `databaseID` in the `Location` header. A `409` means a database with that name
   already exists.
2. **Poll until active** — `GET /v2/databases/{databaseID}` (`getDatabase`) and read
   `status`. A newly created serverless database moves `PENDING` → `INITIALIZING` →
   `ACTIVE`. Poll with backoff until `status == ACTIVE`. You can also list all
   databases with `GET /v2/databases` (`listDatabases`), which paginates with
   `limit` + `starting_after`.
3. **Add a keyspace** (optional) — `POST /v2/databases/{databaseID}/keyspaces/{keyspaceName}`
   (`addKeyspace`). Returns `201`.
4. **Download the secure bundle** — `POST /v2/databases/{databaseID}/secureBundleURL`
   (`generateSecureBundleURL`). Returns a short-lived signed download URL for the
   secure-connect bundle used by drivers.

## Error handling

Errors use the envelope `{ "errors": [ { "ID": <int>, "message": <str> } ] }`
(not RFC 9457). Handle `401` (bad/expired token), `403` (missing scope), `404`
(unknown database), and `409` (name conflict). See `errors/datastax-problem-types.yml`.
