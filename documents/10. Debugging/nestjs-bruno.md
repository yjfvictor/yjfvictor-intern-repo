# Reflection: API Debugging with Bruno

## Summary of tasks

The tasks were completed in the repository [test-repo-for-api-debugging-with-bruno](https://github.com/yjfvictor/test-repo-for-api-debugging-with-bruno). The work included researching Bruno and how it differs from Postman, creating a new API collection, adding and testing public endpoints, and exploring headers and authentication tokens.

The repo contains a README that documents each step: what Bruno is, a comparison with Postman, how to create a collection, how to add and run a request (using public APIs and a local NestJS app), and how to set headers and Bearer (or other) authentication. A sample Bruno collection in the `bruno` folder demonstrates multiple HTTP methods (GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS, TRACE), query parameters, and a range of content types: `application/json`, `application/json; charset=utf-8`, `text/plain`, `text/csv`, `text/html`, `application/x-www-form-urlencoded`, `multipart/form-data`, and `application/xml`. It includes both generic and login-style form-urlencoded requests and basic and extended multipart requests. All 20 sample requests have been run and verified with the Bruno CLI (`bru run`) against [JSONPlaceholder](https://jsonplaceholder.typicode.com/) and [httpbin.org](https://httpbin.org/). An example request using Bearer token auth is also included to show how tokens and headers are configured.

---

## How Bruno helps with API testing compared to Postman or cURL

**Compared to Postman:** Bruno is open source and stores collections as plain text files (Bru or OpenCollection YAML) on disk. You do not need an account or cloud sync. Collaboration is done through Git instead of Postman workspaces, so the same review and permission model as your code applies. Bruno stays focused on being an API client rather than a platform, so there is less tooling and no limit on local collection runs. Data never leaves your machine unless you choose to commit and push it.

**Compared to cURL:** cURL is ideal for one-off commands and scripts. Bruno gives you a visual interface to edit and re-run requests, save them in a collection, attach headers and auth in a structured way, and inspect responses (status, headers, body) without re-typing commands. Collections can still be versioned and shared via Git, and you can run them repeatedly without copying long cURL strings.

---

## How to send an authenticated request in Bruno

Authentication can be set per request or for the whole collection.

1. Open the request (or the collection) and go to the **Auth** tab.
2. Choose an **Auth Type** (for example Bearer Token, Basic Auth, OAuth 2.0, or AWS Sig v4).
3. Fill in the required fields. For **Bearer Token**, enter the token in the Token field. Bruno then adds the `Authorization: Bearer <token>` header for you.
4. For security, store secrets in environment variables and reference them (e.g. `{{bearerToken}}`) instead of typing them into the request. Use Bruno environments so that tokens are not committed to the repo.

If auth is set at the collection level, requests can inherit it via the **Auth** tab by selecting **Inherit from collection** (or the equivalent option in your Bruno version).

---

## Advantages of organising API requests in collections

- **Reuse:** Requests are saved and can be run again with one click instead of re-entering URLs and options.
- **Structure:** Folders and naming make it easy to group requests by feature or resource (e.g. by NestJS controller or module).
- **Sharing:** Because collections are files, they can be committed to Git and shared with the team like code. Changes go through the same review process.
- **Consistency:** Collection-level headers and auth ensure every request in that collection uses the same base URL, content type, or token without repeating them.
- **Documentation:** Request names and optional docs in Bruno double as living documentation of how to call the API.
- **Environments:** Switching between local, staging, and production is done by changing the active environment (e.g. base URL and tokens) without duplicating requests.

---

## How to structure a Bruno collection for a NestJS backend project

A practical approach is to mirror the backend structure and group by resource or module.

1. **One collection per project** (or one per “service” if you have several). Name it after the app (e.g. “MyApp API”). Store it in the repo (e.g. `bruno/` or `api-collection/`) so it lives next to the code.

2. **Folders by domain or module:** Create a folder per NestJS module or resource (e.g. `person`, `auth`, `orders`). This matches how NestJS organises controllers and makes it clear which part of the API each request hits.

3. **Requests per endpoint:** Add one request per route. Name them by method and action (e.g. “Create person”, “Get person by id”, “Update person”, “Delete person”). Use path parameters and query params in the request config so URLs stay readable and reusable.

4. **Environments:** Define at least `local`, and optionally `staging` and `production`. Set variables such as `baseUrl` (e.g. `http://localhost:3000`) and `bearerToken` (or other auth). Use `{{baseUrl}}/person` in requests so switching environment changes the host without editing each request.

5. **Collection-level defaults:** In the collection config, set default headers (e.g. `Content-Type: application/json`) and, if most endpoints are protected, set Bearer (or other) auth at collection level so new requests inherit it. Override only where needed (e.g. login or health check).

6. **Sensitive data:** Keep tokens and secrets in environment files that are listed in `.gitignore` (Bruno can generate this when you create the collection). Commit only the structure and non-sensitive variables so the collection is safe to share.

This structure keeps the collection easy to navigate, aligned with the NestJS backend, and safe to version in Git.
