---
name: Track and assign credentials
description: Look up credential categories and credentials, then create and maintain
  credential assignments for users with the TargetSolutions API v1.
api: openapi/vectorsolutions-targetsolutions-openapi.yml
operations: [getCredentialCategories, getCategoryCredentials, getCredential, getUserCredentials, createCredentialAssignment, updateCredentialAssignment, deleteCredentialAssignment]
generated: '2026-07-21'
method: generated
---

# Track and assign credentials

Operating instructions for managing certification/credential records in
TargetSolutions (API v1, `AccessToken` header auth — see
`authentication/vectorsolutions-authentication.yml`).

## Steps

1. **Browse categories** — `getCredentialCategories` (`GET /sites/categories/credential`).
2. **List credentials in a category** — `getCategoryCredentials`
   (`GET /sites/categories/credential/{categoryid}/credentials`), or inspect one
   with `getCredential` (`GET /credentials/{credentialid}`).
3. **Check what a user already holds** — `getUserCredentials`
   (`GET /users/{userid}/credentials`) before assigning, to avoid duplicates
   (application code 705 DUPLICATE / 1002 CONFLICT on non-unique creates).
4. **Assign** — `createCredentialAssignment`
   (`POST /credentials/{credentialid}/assignments`) with the JSON body for the user.
5. **Maintain** — `updateCredentialAssignment`
   (`PUT /credentials/{credentialid}/assignments/{assignmentid}`) for renewals or
   corrections; `deleteCredentialAssignment` (`DELETE`) to remove. Code 1004
   DEPENDENCY means dependent resources must be removed first.

## Rules

- No idempotency keys: on write timeouts, re-check with
  `getCredentialAssignments` before re-posting.
- Collection GETs accept the JSON `q` search parameter
  (`q={"key":"value"}`); 902/903/904 signal search errors.
- Back off on HTTP 429; verify all writes with GETs (the sandbox host
  `devsandbox.targetsolutions.com` refreshes nightly and is not viewable in the UI).
