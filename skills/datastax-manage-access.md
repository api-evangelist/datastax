---
name: Manage Astra organization access — tokens, roles, and users
description: Issue and revoke scoped application tokens, review organization roles, and invite users to a DataStax Astra organization.
api: openapi/datastax-devops-openapi.json
operations:
- getOrganizationRoles
- createToken
- getTokensForOrg
- deleteToken
- inviteUserToOrganization
---

# Manage Astra organization access

Use the Astra DevOps API with an `Authorization: Bearer AstraCS:...` token that has
`org-admin` scope. All access-management operations are organization-scoped.

## Steps

1. **List roles** — `GET /v2/organizations/roles` (`getOrganizationRoles`) to find the
   role id(s) you want to bind to a token or user. Roles carry the permission scopes
   in `scopes/datastax-scopes.yml`.
2. **Issue a scoped token** — `POST /v2/tokens` (`createToken`) with the target
   `roles[]`. The response returns the `clientId`, `secret`, and the full `token`
   (`AstraCS:...`) exactly once — store it immediately, it cannot be retrieved again.
3. **Audit tokens** — `GET /v2/tokens` (`getTokensForOrg`) lists every active token and
   its bound roles for the organization.
4. **Revoke a token** — `DELETE /v2/tokens` (`deleteToken`) with the `clientId` to
   revoke a compromised or unused token.
5. **Invite a user** — `PUT /v2/organizations/users` (`inviteUserToOrganization`) with
   the user email and `roles[]`. Users can also be provisioned via SCIM 2.0.

## Rules

- Grant the narrowest role/scope set that satisfies the use case (least privilege).
- Treat the token `secret`/`token` as write-once secrets.
- Errors use `{ "errors": [ { "ID", "message" } ] }`; handle `401`/`403`/`409`.
