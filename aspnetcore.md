# ASP.NET Core

Rules for ASP.NET Core web APIs. This file builds on [csharp.md](csharp.md) (raw: https://raw.githubusercontent.com/dzaba1/instructions/main/csharp.md), which in turn builds on general_coding.md. Read both first; where they conflict, this file wins.

## Configuration

- Environment-specific settings go in `appsettings.{Environment}.json`.
- Secrets never go in committed files. Use user secrets locally, and environment variables or Azure Key Vault in deployed environments.

## Authentication and Authorization

- Authenticate with JWT bearer tokens (`AddAuthentication().AddJwtBearer()`). With Microsoft Entra ID, use `Microsoft.Identity.Web`.
- Authorize with named policies. Express role checks as policy requirements, not as `[Authorize(Roles = ...)]` scattered over endpoints.
- Secure controllers and Minimal APIs the same way: set a fallback policy that requires an authenticated user, and mark public endpoints explicitly with `[AllowAnonymous]` / `.AllowAnonymous()`.

## Validation and Error Handling

- Use data annotations for simple per-field checks (`[Required]`, `[Range]`, `[StringLength]`). Use FluentValidation when a rule spans several fields, needs services, or is otherwise complex. Don't mix both on the same type.
- Validation failures return `400` with `ValidationProblemDetails`.
- All error responses are Problem Details (RFC 9457): register `AddProblemDetails()` and handle unhandled exceptions in a global `IExceptionHandler`.
- Never return exception details or stack traces outside the Development environment.

## API Design

- Collection endpoints are paginated, with a default and a maximum page size. Add filtering and sorting where clients need them, applied in the database.
- Version APIs with `Asp.Versioning.Http` (Minimal APIs) or `Asp.Versioning.Mvc` (controllers), using a URL segment (`/v1/...`).

## OpenAPI

- Generate OpenAPI documents with `Microsoft.AspNetCore.OpenApi` (`AddOpenApi()` / `MapOpenApi()`), not Swashbuckle. Each API version gets its own document.
- Every endpoint declares a summary and all of its responses (`TypedResults`, `Produces`, `ProducesProblem`). The document describes the authentication scheme.
- For an interactive UI, use Scalar (`Scalar.AspNetCore`), mapped in Development only.

## Logging and Telemetry

- Configure Serilog with `Serilog.AspNetCore`, read settings from configuration, and add `UseSerilogRequestLogging()`.
- Use W3C trace context (`Activity`) for correlation. Don't invent custom correlation ID headers. Include `TraceId` and `SpanId` in logs.
- Send telemetry to Application Insights through the Azure Monitor OpenTelemetry distro (`Azure.Monitor.OpenTelemetry.AspNetCore`).

## Performance

- Cache with `HybridCache`. Add output caching only to endpoints whose responses are safe to share between callers.

## Hosting

- Expose health checks with `AddHealthChecks()`: `/health/live` (the process is up) and `/health/ready` (dependencies are reachable).

## Testing

- Integration tests cover every endpoint through `WebApplicationFactory<TProgram>`, including `401`/`403` responses for protected endpoints.
