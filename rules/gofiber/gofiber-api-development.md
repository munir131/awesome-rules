---

name: GoFiber API Development
globs: "**/*.go"
alwaysApply: false
description: Best practices for building production-grade APIs with Go and Fiber
---

You expert in Go backend engineering and deep expertise in the Fiber framework.

## Project Structure

* Structure the service using layered or hexagonal architecture
  Controllers → Services → Repositories → External Providers
* Maintain clear separation between HTTP handlers and domain logic.
* Use Fiber’s app.Group for organizing domain-specific route segments.
* Keep configuration, bootstrap logic, and dependency injection isolated from business logic.
* Store environment configuration in dedicated config structs and load via Viper or envconfig.

## Middleware Best Practices

* Register global middleware in a deterministic order:
  Logging → Request ID → Recovery → CORS → Security Headers → Authentication → Route Handlers
* Use Fiber’s built-in middleware where appropriate:
  logger, recover, cors, requestid, limiter, compress, helmet, etag
* Encapsulate concerns such as auth, tenant resolution, tracing, and request validation as Fiber handlers.
* Prefer local, route-level middleware for domain-specific logic.
* Use fiber.Router groups to compose middleware stacks cleanly.
* Integrate validation using go-playground/validator or similar libraries, wrapping validation errors consistently.

## Error Handling

* Implement a unified error response contract for the API.
* Use custom error types to represent domain, validation, and infrastructure failures.
* Leverage Fiber’s centralized error handler (app.Settings.ErrorHandler) to produce consistent output.
* Map domain errors to appropriate HTTP status codes.
* Log errors with context (request ID, user claims, correlation IDs) using structured logging (zerolog, slog, zap).
* Ensure the error handler never leaks internal error messages.

## Security Best Practices

* Apply fiber/middleware.Helmet to set secure headers.
* Enable rate limiting with fiber/middleware.Limiter for both global and route-specific protection.
* Sanitize and validate all request payloads before passing them to services.
* Enforce strict CORS settings scoped by environment.
* Use fiber/middleware.CSRF for stateful apps when applicable.
* Secure configuration using environment variables and centralized secrets management.
* Zero sensitive data in logs, including JWTs, passwords, and tokens.

## Performance Optimization

* Use fiber/middleware.Compress for response compression.
* Make use of in-memory caching layers (ristretto, groupcache) or Fiber’s built-in storage for specific use cases.
* Prefer connection pooling for database drivers and configure pool size explicitly.
* Use asynchronous task runners (workers, goroutines, background processors) for non-critical paths.
* Reduce allocations in handlers by reusing buffers and leveraging Fiber’s low-level APIs only when needed.
* Make use of Fiber’s prefork mode only after benchmarking under expected traffic and knowing its tradeoffs.
* Monitor performance with tools like Prometheus, OpenTelemetry, and pprof.

## Storage and Session Management

* Use Fiber’s session middleware with a pluggable storage backend (Redis, Memcached, Badger).
* Configure TTLs and secure cookie attributes (HttpOnly, Secure, SameSite).
* Avoid embedding heavy state in sessions; keep them lean and validate session integrity.
* For caching, choose storage based on workload characteristics, not convenience.

## API Design

* Follow RESTful semantics and consistent naming conventions.
* Version APIs using top-level namespaces (e.g., `/v1`, `/v2`).
* Implement pagination and filtering uniformly across list endpoints.
* Return typed JSON responses with explicit structs; avoid map[string]any for public contracts.
* Provide OpenAPI/Swagger documentation using tools like Fiber Swagger, oapi-codegen, or go-swagger.
* Use appropriate HTTP status codes and avoid overloading 200 OK for all responses.
* Include trace IDs and request IDs in responses for observability.

## Operational Readiness

* Ensure graceful shutdown using context-aware cleanup.
* Wrap Fiber startup in an application lifecycle manager for consistent bootstrap behavior.
* Provide health, readiness, and liveness endpoints.
* Run the server in a minimal container image (distroless or Alpine) with non-root user.
* Automate code scanning, linting, and formatting (golangci-lint, staticcheck, gofumpt).