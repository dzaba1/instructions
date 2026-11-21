# C# Development

## Language version
- Always use the latest version C#.
- Never set `<LangVersion>latest</LangVersion>`. Always use explicit version.

## Naming Conventions

- Follow PascalCase for component names, method names, and public members.
- Use camelCase for private fields and local variables.
- Prefix interface names with "I" (e.g., IUserService).

## Formatting

- Apply code-formatting style defined in `.editorconfig`.
- Prefer file-scoped namespace declarations and single-line using directives.
- Insert a newline before the opening curly brace of any code block (e.g., after `if`, `for`, `while`, `foreach`, `using`, `try`, etc.).
- Ensure that the final return statement of a method is on its own line.
- Use pattern matching and switch expressions wherever possible.
- Use `nameof` instead of string literals when referring to member names.
- Ensure that XML doc comments are created for any public APIs. When applicable, include `<example>` and `<code>` documentation in the comments.

## Project Setup and Structure

- Guide users through creating a new .NET project with the appropriate templates.
- Explain the Program.cs and configuration system in ASP.NET Core 10 including environment-specific settings.
- `InternalsVisibleTo` only for test projects.
- Always use Central Package Management (CPM).
- Use best practices for creating nuget packages. Check if properties are correctly configured. Example:
```xml
<Authors>Bob</Authors>
<RepositoryType>git</RepositoryType>
<RepositoryUrl>https://github.com/Bob/lib</RepositoryUrl>
<PublishRepositoryUrl>true</PublishRepositoryUrl>
<PackageLicenseExpression>MIT</PackageLicenseExpression>
<EmbedUntrackedSources>true</EmbedUntrackedSources>
<IncludeSymbols>true</IncludeSymbols>
<SymbolPackageFormat>snupkg</SymbolPackageFormat>
```

## Nullable Reference Types

- Declare variables non-nullable, and check for `null` at entry points.
- Always use `is null` or `is not null` instead of `== null` or `!= null`.
- Trust the C# null annotations and don't add null checks when the type system says a value cannot be null.

## Data Access Patterns

- Guide the implementation of a data access layer using Entity Framework Core.
- Explain different options (SQL Server, SQLite, In-Memory) for development and production.
- Show how to implement database migrations and data seeding.
- Explain efficient query patterns to avoid common performance issues.

## Authentication and Authorization

- Guide users through implementing authentication using JWT Bearer tokens.
- Explain OAuth 2.0 and OpenID Connect concepts as they relate to ASP.NET Core.
- Show how to implement role-based and policy-based authorization.
- Demonstrate integration with Microsoft Entra ID (formerly Azure AD).
- Explain how to secure both controller-based and Minimal APIs consistently.

## Validation and Error Handling

- Guide the implementation of model validation using data annotations and FluentValidation.
- Explain the validation pipeline and how to customize validation responses.
- Demonstrate a global exception handling strategy using middleware.
- Show how to create consistent error responses across the API.
- Explain problem details (RFC 9457) implementation for standardized error responses.

## API Versioning and Documentation

- Guide users through implementing and explaining API versioning strategies.
- Demonstrate Swagger/OpenAPI implementation with proper documentation.
- Show how to document endpoints, parameters, responses, and authentication.
- Explain versioning in both controller-based and Minimal APIs.
- Guide users on creating meaningful API documentation that helps consumers.

## Logging and Monitoring

- Guide the implementation of structured logging using Serilog.
- Explain the logging levels and when to use each.
- Demonstrate integration with Application Insights for telemetry collection.
- Show how to implement custom telemetry and correlation IDs for request tracking.
- Explain how to monitor API performance, errors, and usage patterns.
- Every DI-registered service injects `ILogger<T>` (its own type) and logs what helps debugging and operations.
- Use message templates with named placeholders (`"Loaded {FileCount} files from {Folder}"`), never string interpolation.

## Testing

- Integration tests use Serilog Console and Debug logging. Never use null logger for integration tests.
- Use [NUnit](https://www.nuget.org/packages/NUnit), [NUnit.Analyzers](https://www.nuget.org/packages/NUnit.Analyzers), [Moq](https://www.nuget.org/packages/Moq), [AutoFixture.AutoMoq](https://www.nuget.org/packages/AutoFixture.AutoMoq), [FluentAssertions](https://www.nuget.org/packages/FluentAssertions) (always lower than 8.0) nuget packages.
- For unit tests always reference and use [Dzaba.TestUtils](https://www.nuget.org/packages/Dzaba.TestUtils) nuget package.
- Use MethodName_When_Then naming convention.
- For unit tests always at least use `AutoFixtureTestFixture`. Create `CreateSut()` method with `Fixture.Create<T>()` property usage. Example:
```csharp
[TestFixture]
public class MyServiceTests : AutoFixtureTestFixture
{
    private MyService CreateSut()
    {
        return Fixture.Create<MyService>();
    }

    [Test]
    public void When_Then()
    {
        var dep = Fixture.FreezeMock<IMyDependency>();
        dep.Setup(x => x.Hello()).Returns(42);

        var sut = CreateSut();

        sut.Hello().Should().Be(42);
    }
}
```
- For unit tests mocking use `Fixture.FreezeMock<T>();` extension.
- For unit tests initialize mocks in `[SetUp]` method.
- For integration tests reference and use [Dzaba.TestUtils.Integration](https://www.nuget.org/packages/Dzaba.TestUtils.Integration) nuget package.
- For integration tests use `IocTestFixture`

## Deployment and DevOps

- Guide users through containerizing their API using .NET's built-in container support (`dotnet publish --os linux --arch x64 -p:PublishProfile=DefaultContainer`).
- Explain the differences between manual Dockerfile creation and .NET's container publishing features.
- Explain CI/CD pipelines for NET applications.

## Collections

- Use `IEnumerable<T>` (not IReadOnlyList<T>, T[], etc), for method parameters and return types.
- Don't call `.ToArray()` or `.ToList()` just because the return type is `IEnumerable<T>`. A pure projection with no side effects is returned lazily. Materialise only when enumerating twice would repeat side effects (random draws, I/O) or when the source may change.
- Never return `Task<IEnumerable<T>>`. Async methods that return a collection use `IAsyncEnumerable<T> XxxAsync(..., CancellationToken)`. Implementations that await something are `async IAsyncEnumerable<T>` with `[EnumeratorCancellation]`. Synchronous implementations return a built list with .ToAsyncEnumerable() which avoids CS1998.

## Multithreading

- `.ConfigureAwait(false)` on every `await` in library code. This includes `await foreach (... in x.ConfigureAwait(false))` and `await using`.
- Never use `volatile` keyword for synchronization

## Inversion of Control

- One static `Bootstrapper` class per project holds the DI registration extension in `Bootstrapper.cs` at the project root namespace.
- Interfaces are public, service implementations are `internal sealed` (repositories, unit of work, scopes, hosted services…). Consumers depend on the interface and get the implementation from the project's `Bootstrapper`.
- Code in other projects that needs an implementation builds it through DI, not with `new`.
- Register services as Transient by default (`AddTransient`/`TryAddTransient`). Use Singleton or Scoped only when the service has to keep state.
- Inject the options class, not `IOptions<TOptions>`.
