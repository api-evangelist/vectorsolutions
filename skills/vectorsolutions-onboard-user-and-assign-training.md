---
name: Onboard a user and assign training
description: Create a TargetSolutions user in a site, verify the record, and assign
  a course as a training assignment using the TargetSolutions API v1.
api: openapi/vectorsolutions-targetsolutions-openapi.yml
operations: [getSites, createSiteUser, getUser, getSiteCourses, createTrainingAssignment, getUserTrainingAssignments]
generated: '2026-07-21'
method: generated
---

# Onboard a user and assign training

Operating instructions for an agent driving the TargetSolutions API v1
(`https://api.targetsolutions.com/v1`; sandbox `https://devsandbox.targetsolutions.com/v1`).

## Auth

Send the customer-specific token in the `AccessToken` header on every request over
HTTPS. Tokens come from the customer's TargetSolutions account manager and may be
read-only or feature-scoped — creating users and training assignments requires a
token with write access (error 800/801/802 means missing or invalid token, or a
resource you do not own).

## Steps

1. **Find the site** — `getSites` (`GET /sites`), pick the target `siteid`
   (or confirm it with `getSite`).
2. **Create the user** — `createSiteUser` (`POST /sites/{siteid}/users`) with a JSON
   body. Expect application code 701 (created); 705 means duplicate data, 1000/1001
   mean missing or invalid parameters.
3. **Verify** — `getUser` (`GET /users/{userid}`). In the sandbox the platform UI is
   not front-facing: always verify writes with a GET, never by looking at the app.
4. **Pick the course** — `getSiteCourses` (`GET /sites/{siteid}/courses`); filter
   collections with the JSON `q` parameter, e.g. `q={"key":"value"}` (invalid search
   JSON returns 902, unsupported search returns 904).
5. **Assign training** — `createTrainingAssignment`
   (`POST /trainingassignments/user/{userid}/course/{courseid}`).
6. **Confirm** — `getUserTrainingAssignments` (`GET /users/{userid}/trainingassignments`).

## Rules

- There is no idempotency-key contract: never blind-retry a POST; on timeout,
  re-check with the corresponding GET before re-creating (705/1002 signal duplicates).
- Back off on HTTP 429 (documented as too many simultaneous requests); no
  rate-limit headers are published.
- Errors are JSON with application codes 700-1006 — see
  `errors/vectorsolutions-problem-types.yml`; conventions in
  `conventions/vectorsolutions-conventions.yml`.
