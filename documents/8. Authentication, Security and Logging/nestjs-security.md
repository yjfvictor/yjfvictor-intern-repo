# Reflection: Security Best Practices in NestJS

## Project Summary

The project [test-repo-for-security-best-practices-in-NestJS](https://github.com/yjfvictor/test-repo-for-security-best-practices-in-NestJS) is a small NestJS backend built to demonstrate security best practices. It runs on the Fastify adapter rather than the default Express adapter, so that security and rate-limiting behaviour can be wired in via Fastify plugins before any routes are registered. At bootstrap, the application registers `@fastify/helmet` on the underlying Fastify instance so that every HTTP response receives security-related headers (such as X-Content-Type-Options and X-Frame-Options), then registers `@fastify/rate-limit` with a fixed cap per client (for example 100 requests per minute) so that abuse and brute-force traffic are limited and excess requests receive HTTP 429. Configuration is handled through `@nestjs/config` with a Joi validation schema that runs at startup. Environment variables such as `NODE_ENV` and `PORT` are validated and optional entries such as `API_KEY` and `DATABASE_URL` are checked when present. Secrets are not hardcoded and are intended to be supplied via the environment or a secret manager in production. The root module loads this configuration globally so that the rest of the application can rely on validated settings. The application exposes a root route that returns a greeting and a health route that returns a JSON status for liveness and readiness checks. Both routes are subject to the same rate limit and security headers. Unit tests cover the root service, controller, and the configuration schema. End-to-end tests run the full application with the Fastify adapter and use Fastify’s inject API to send requests to these routes without opening a real port, so that the behaviour of the server under the applied security and rate-limiting layers can be asserted.

---

## Most Common Security Vulnerabilities in a NestJS Backend

- **Injection attacks:** If user input is concatenated into queries or commands (e.g. raw SQL or shell commands), the application is exposed to SQL injection, NoSQL injection, or command injection. NestJS does not remove this risk by itself. Use parameterised queries, ORMs correctly, and avoid passing user input directly to exec or similar.

- **CORS misconfigurations:** Allowing `*` or overly broad origins can let untrusted sites send authenticated requests to your API. Configure CORS with an explicit list of allowed origins (and methods/headers) in production.

- **Missing or weak security headers:** Without headers such as `X-Content-Type-Options`, `X-Frame-Options`, or Content-Security-Policy, the app is more exposed to XSS, clickjacking, and MIME sniffing. NestJS does not set these by default.

- **Secrets in code or in logs:** API keys, database URLs, or tokens hardcoded in source or logged in plain text can be leaked via version control or log aggregation. Configuration should come from the environment or a secret manager and must not be logged.

- **No rate limiting:** Endpoints (especially login or password reset) can be brute-forced or abused for denial of service. Without rate limiting, a single client can send unlimited requests.

- **Unvalidated configuration:** Wrong or missing environment variables can lead to insecure defaults (e.g. debug mode in production) or runtime failures. Validating configuration at startup avoids this.

---

## How @fastify/helmet Improves Application Security

`@fastify/helmet` is the Fastify plugin that wraps the Helmet library. It sets a set of HTTP response headers that harden the application against common client-side and browser-based attacks:

- **X-Content-Type-Options: nosniff:** Stops the browser from guessing content types and can reduce the impact of certain XSS or incorrect MIME handling.

- **X-Frame-Options:** Restricts whether the response can be embedded in frames, which helps prevent clickjacking.

- **Strict-Transport-Security (HSTS):** When enabled, tells the browser to use HTTPS only for the host, reducing downgrade and cookie theft over plain HTTP.

- **X-DNS-Prefetch-Control, X-Download-Options,** and similar headers further lock down browser behaviour.

Helmet does not fix bugs in your code but reduces the impact of mistakes (e.g. reflected XSS) and enforces safer defaults. In the project, Helmet is registered on the Fastify instance before any routes so that every response gets these headers. Content-Security-Policy is turned off in the demo. For apps that serve HTML, it should be enabled and tuned.

---

## Why Rate Limiting Is Important for Preventing Abuse

Rate limiting caps how many requests a client (e.g. per IP or per API key) can make in a given time window. Without it:

- **Brute force:** Attackers can try many passwords or tokens on login or password-reset endpoints.
- **Resource exhaustion:** A few clients can send a huge number of requests and exhaust CPU, memory, or downstream services.
- **Scraping and enumeration:** High request volumes can be used to scrape data or enumerate valid identifiers.

In the project, `@fastify/rate-limit` is registered with a limit (e.g. 100 requests per minute per client). When a client exceeds the limit, the plugin responds with HTTP 429 and appropriate headers (e.g. `Retry-After`). This protects both the application and other users. Rate limits should be chosen based on expected traffic and the sensitivity of the endpoints (e.g. stricter limits on auth endpoints).

---

## How Sensitive Configuration Values Can Be Protected in Production

- **Environment variables:** Keep API keys, database URLs, and other secrets in environment variables (or in a `.env` file that is never committed). The project uses `@nestjs/config` to load from the environment and validates them with Joi so that missing or invalid values fail at startup.

- **Secret managers:** In production, use a secret manager (e.g. AWS Secrets Manager, HashiCorp Vault, or the platform’s secret store) and inject values into the process environment or fetch them at startup. Do not store production secrets in code or in version control.

- **Least privilege:** Run the process with a dedicated user and grant only the permissions it needs. Restrict file system and network access where possible.

- **No logging of secrets:** Ensure configuration and error handlers never log secrets or full connection strings. Log only non-sensitive identifiers (e.g. “config loaded” or redacted values).

- **Secure defaults:** Validate and enforce safe defaults (e.g. `NODE_ENV=production`, HTTPS-only cookies) and reject invalid or dangerous combinations at startup. The project’s Joi schema is an example of validating env vars before the app runs.
