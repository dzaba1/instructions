# General Coding

## General

- Write clear and concise comments for each function.
- Make only high confidence suggestions when reviewing code changes.
- Write code with good maintainability practices, including comments on why certain design decisions were made.
- Handle edge cases and write clear exception handling.
- For libraries or external dependencies, mention their usage and purpose in comments.
- Don't copy-paste code. When a second place needs the same logic, move it into a common class or common project and call it from both, instead of duplicating it. This applies to tests too.

## Project Setup and Structure

- Explain the purpose of each generated file and folder to build understanding of the project structure.
- Demonstrate how to organize code using feature folders or domain-driven design principles.
- Show proper separation of concerns with models, services, and data access layers.

## Data Access Patterns

- Demonstrate repository pattern implementation and when it's beneficial.

## Testing

- Always include test cases for critical paths of the application.
- Guide users through creating unit tests.
- Do not emit "Act", "Arrange" or "Assert" comments.
- Copy existing style in nearby files for test method names and capitalization.
- Explain integration testing approaches for API endpoints.
- Demonstrate how to mock dependencies for effective testing.
- Show how to test authentication and authorization logic.
- Explain test-driven development principles as applied to API development.

## Performance Optimization

- Guide users on implementing caching strategies (in-memory, distributed, response caching).
- Explain asynchronous programming patterns and why they matter for API performance.
- Demonstrate pagination, filtering, and sorting for large data sets.
- Show how to implement compression and other performance optimizations.
- Explain how to measure and benchmark API performance.

## Deployment and DevOps

- Demonstrate deployment to Azure App Service, Azure Container Apps, or other hosting options.
- Show how to implement health checks and readiness probes.
- Explain environment-specific configurations for different deployment stages.

## Inversion of Control

- Register services as Transient by default. Use Singleton or Scoped only when the service has to keep state.