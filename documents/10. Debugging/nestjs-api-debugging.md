# Reflection: Inspecting API Requests and Responses

## Summary of the tasks

The work was carried out in the GitHub repository [test-repo-for-inspecting-api-requests-and-responses](https://github.com/yjfvictor/test-repo-for-inspecting-api-requests-and-responses). The project provides a small NestJS Person CRUD API that serves as a target for practising inspection of HTTP requests and responses with tools such as Bruno, Postman, and curl.

Most of the effort focused on automated tests and debugging workflows instead of complex business logic. Unit tests exercise controllers, services, and the Content-Type middleware to confirm that success and error paths behave as expected. End to end tests drive the real HTTP API and verify status codes, validation rules, and the JSON response envelope across normal, invalid, and edge case scenarios.

Two client side suites make it easy to practise with inspection tools in a repeatable way. A Bruno collection covers the main flows for root, create, read, update, delete, and Content-Type enforcement and includes scripted assertions over status codes and response bodies. A curl and jq script reproduces the same scenarios from the command line and checks HTTP status codes and key response fields. Together with server side logging and editor debug configurations these tests form a practical process for reproducing issues and confirming fixes when working with Bruno, Postman, or curl.

---

## How can logging request payloads help with debugging?

Logging request payloads helps in several ways.

First, you can confirm that the client is sending the expected data. If a create or update fails validation, the log shows the exact body that arrived, so you can see missing fields, wrong types, or extra properties that were stripped or rejected.

Second, when a bug is reported (for example, "updating the phone number does not work"), you can compare the logged payload with the API contract. You might find that the client is sending the field under a different name or in a nested structure that the server does not handle.

Third, in distributed or async setups, request logs give you a trace of what was received at a given time. That makes it easier to reproduce issues and to check that the right version of a client or integration is calling the API with the expected shape.

Care must be taken not to log sensitive data (such as passwords or tokens) in plain text, and to comply with data protection and retention policies.

---

## What tools can you use to inspect API requests and responses?

Three common options are Bruno, Postman, and curl.

**Bruno** is an open-source API client that works offline and stores collections as plain-text files (for example, `.bru`). You can version-control these files and inspect response body, headers, status code, and timeline without sending data to a third party. It is well suited to teams that want a Postman-like experience with a Git-based workflow.

**Postman** is a popular graphical client. You set the URL, method, headers, and body, send the request, and view status code, headers, and body in the response panel. Environments and saved examples make it easy to share and repeat requests. It can sync to the cloud if you choose.

**curl** is a command-line tool available on most systems. You can script it and use it in CI. For example, you can send a GET or POST and print the response and the HTTP status code (for example, with `-w` to show the code). It does not provide a GUI but is very flexible for quick checks and automation.

In addition, the NestJS application itself logs request headers and payload in the controller and response status and type in the interceptor, so server logs are another way to inspect what was received and what status was sent.

---

## How would you debug an issue where an API returns the wrong status code?

A practical approach is as follows.

1. **Reproduce the case**
   Use the same URL, method, headers, and body (via Bruno, Postman, curl, or a test) so that the wrong status code is returned consistently.

2. **Inspect the response**
   Check the actual status code and body. In this project, the global interceptor wraps JSON responses in an envelope that includes `statusCode`, so you can confirm what the server is sending. Also check response headers if relevant.

3. **Trace the flow**
   Follow the request from the controller to the service and then to the database or external calls. The wrong code might come from:
   - The controller (for example, returning 200 instead of 201 for a create)
   - The service throwing an exception that is mapped to the wrong code (for example, 500 instead of 404 when a resource is not found)
   - A global exception filter or interceptor changing the status

4. **Check exception mapping**
   In NestJS, HTTP exceptions (for example, `NotFoundException`, `ConflictException`) set the status code. If the wrong code is returned, the wrong exception type may be thrown, or an exception filter may be overriding it. Search for where that route or error is handled and which exception is used.

5. **Add or use existing logs**
   The middleware and interceptor in this project log method, URL, and response status. Use these logs to see which handler ran and what status was set before the response was sent.

6. **Write or run tests**
   Add a unit or e2e test that calls the endpoint with the same inputs and asserts the expected status code. That both documents the correct behaviour and prevents regressions once the bug is fixed.

---

## What are some security concerns when logging request data?

Logging request data can create security and compliance risks if not handled carefully.

### Sensitive data in the body

Request bodies may contain passwords, tokens, payment details, or personal data. Logging them in full can expose them to anyone with access to the logs (including in log aggregation or support tools). Logs may be retained longer than necessary or stored in less secure environments. Mitigations include redacting or hashing sensitive fields, or not logging the body at all for certain endpoints (for example, login or payment).

### Headers

Headers such as `Authorization` or `Cookie` carry session or API credentials. Logging them can leak credentials. In this project, only a small set of headers (content-type, accept, user-agent) is logged, and Authorization is not included. In production, the list of logged headers should remain minimal and never include credentials.

### Personal data and compliance

Request payloads may contain personal data (for example, names, addresses, id card numbers). Logging them may conflict with data minimisation and retention rules (for example, under GDPR or the Australian Privacy Principles). You should only log what is needed for debugging or operations, restrict access to logs, and define retention and deletion policies.

### Injection and abuse

If logs are viewed in a vulnerable viewer or forwarded to a system that interprets them (for example, some monitoring tools), malicious input in the request could cause injection or unexpected behaviour. Sanitising or limiting the size of logged content can reduce this risk.

### Operational exposure

Logs are often available to developers, support, or third-party services. A compromise of any of these can expose logged request data. So it is important to treat logs as sensitive, limit who can access them, and avoid logging data that is not needed for a legitimate purpose.

In summary, logging request data is useful for debugging but should be done in a way that avoids logging secrets, minimises personal data, and aligns with security and privacy policies.
