# Reflection: Authentication in NestJS with Auth0 and JWT

## Summary of the project

The project [test-repo-for-authentication-in-NestJS-with-Auth0-JWT](https://github.com/yjfvictor/test-repo-for-authentication-in-NestJS-with-Auth0-JWT) is a small NestJS API that shows how to secure routes using Auth0 and JWT. It includes:

- A Passport JWT strategy that validates access tokens issued by Auth0
- Use of **jwks-rsa** to load Auth0’s public keys from the JWKS endpoint (by key id in the token)
- A guard that protects selected routes so only requests with a valid Bearer token are allowed
- Example endpoints: one public and two protected (messages)
- Inline comments and tooling to generate HTML documentation from source

The app reads Auth0 configuration from environment variables: `AUTH0_ISSUER_BASE_URL`, `AUTH0_AUDIENCE`, and optionally `CLIENT_ORIGIN_URL` and `PORT`. All code is TypeScript with explicit types.

---

## How Auth0 handles authentication compared to traditional username/password auth

With traditional username/password auth, the app (or its backend) stores and checks credentials itself. The user sends a username and password, the server checks them against its own store, and then creates a session or token. The app is responsible for password hashing, storage and security.

Auth0 is an identity provider (IdP). The app does not handle passwords. Instead, Auth0 hosts the login UI and logic. The user signs in on Auth0’s (or your custom) login page. Auth0 checks the credentials and then issues tokens (e.g. access token and optionally ID token and refresh token). The app only checks that the access token is valid and trusts Auth0 for “who the user is.” So the app gets tokens, not passwords. Benefits include: no password storage in your app, built-in flows (e.g. OAuth2, OIDC), social logins, MFA and centralised user management. The trade-off is dependence on Auth0’s service and configuration (e.g. tenant, API, audience).

---

## Role of JWT in API authentication

A JWT (JSON Web Token) is a compact, signed (and optionally encrypted) token. In API authentication it is often used as an **access token**: the client gets a JWT from Auth0 (or another IdP) and sends it on each request (e.g. in the `Authorization: Bearer <token>` header). The API does not look up a session in a store. Instead it **verifies the token**: checks the signature with the IdP’s public key, and checks claims such as audience, issuer and expiration. If valid, the API treats the token as proof that the request is authorised for that audience and subject. So the role of the JWT is to carry identity and authorisation information in a way the API can verify itself (stateless), without calling Auth0 on every request. The API only needs the public key (or key set, e.g. JWKS) to verify signatures.

---

## How jwks-rsa and public/private key verification work in Auth0

Auth0 signs JWTs with a **private key** (kept secret). The API must verify the signature using the matching **public key**. Auth0 exposes its public keys at a JWKS endpoint, e.g. `https://<tenant>.auth0.com/.well-known/jwks.json`. That URL returns a JSON Web Key Set (JWKS): a list of public keys, each with a key id (`kid`). A JWT’s header contains the `kid` of the key that was used to sign it.

**jwks-rsa** is a library that fetches that JWKS and, given a `kid`, returns the right public key. The API then uses that key to verify the JWT signature (e.g. with RS256). So the flow is: decode the JWT header to get `kid`, ask jwks-rsa for the key with that `kid` (it may cache and rate-limit requests to the JWKS URL), get the public key, and verify the token’s signature. Auth0 holds the private key and signs tokens; your API only has the public keys from the JWKS and never sees the private key. That is how public/private key verification works with Auth0 and jwks-rsa.

---

## How to protect an API route so that only authenticated users can access it

In this NestJS project, protection is done with a **guard** and the **Passport JWT strategy**:

1. **JWT strategy**  
   Configure a Passport strategy that reads the Bearer token from the request, uses jwks-rsa (via `secretOrKeyProvider`) to get the public key for the token’s `kid`, and then verifies signature, audience and issuer. If verification succeeds, the strategy’s `validate()` runs and its return value is attached to the request (e.g. as `request.user`).

2. **Guard**  
   Use a guard that runs the Passport JWT strategy (e.g. extend `AuthGuard('jwt')`). If the strategy fails (no token, invalid token, or wrong audience/issuer), the guard throws (e.g. `UnauthorizedException`) and the route handler is not run. If it succeeds, the request is considered authenticated and the handler runs.

3. **Apply the guard to the route**  
   On any controller or method that should require authentication, add `@UseGuards(JwtAuthGuard)` (or your guard’s name). Only requests that present a valid JWT in the `Authorization: Bearer <token>` header will reach the handler. Others receive a 401 response.

So to protect a route: register the JWT strategy (with Auth0 issuer, audience and JWKS), use a guard that invokes that strategy, and annotate the route with that guard. No extra session store is needed; validity is determined from the JWT and the JWKS public keys.
