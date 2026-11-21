# C# Development

This file builds on the general coding rules. Read them first: [general_coding.md](general_coding.md) (raw: https://raw.githubusercontent.com/dzaba1/instructions/main/general_coding.md). Where they conflict, this file wins.

For ASP.NET Core projects, also read [aspnetcore.md](aspnetcore.md) (raw: https://raw.githubusercontent.com/dzaba1/instructions/main/aspnetcore.md).

## Language version

- Target the newest .NET version and set `LangVersion` explicitly to the C# version that ships with it (for example `net10.0` with `<LangVersion>14</LangVersion>`).
- Never set `LangVersion` to `latest`, `latestMajor`, `preview` or `default`.

## Naming Conventions

- Use PascalCase for type names, method names, properties, constants and other public members.
- Use camelCase for private fields and local variables.
- Prefix interface names with "I" (e.g., `IUserService`).

## Formatting

- Apply code-formatting style defined in `.editorconfig`.
- Prefer file-scoped namespace declarations and single-line using directives.
- Insert a newline before the opening curly brace of any code block (e.g., after `if`, `for`, `while`, `foreach`, `using`, `try`, etc.).
- Ensure that the final return statement of a method is on its own line.
- Use pattern matching and switch expressions where they make the code clearer.
- Use `nameof` instead of string literals when referring to member names.
- Write XML doc comments for all public APIs. When applicable, include `<example>` and `<code>` in them.

## Project Setup and Structure

- Create projects with `dotnet new` templates, not hand-written project files.
- Always use Central Package Management (CPM).
- Use `InternalsVisibleTo` only for test projects, plus `DynamicProxyGenAssembly2` when Moq has to mock an internal type.
- Configure NuGet packages with complete metadata, Source Link and symbols. Example:
```xml
<PropertyGroup>
  <Authors>Bob</Authors>
  <Description>One sentence on what the package does.</Description>
  <PackageTags>tag1;tag2</PackageTags>
  <PackageReadmeFile>README.md</PackageReadmeFile>
  <PackageLicenseExpression>MIT</PackageLicenseExpression>
  <RepositoryType>git</RepositoryType>
  <RepositoryUrl>https://github.com/Bob/lib</RepositoryUrl>
  <PublishRepositoryUrl>true</PublishRepositoryUrl>
  <EmbedUntrackedSources>true</EmbedUntrackedSources>
  <IncludeSymbols>true</IncludeSymbols>
  <SymbolPackageFormat>snupkg</SymbolPackageFormat>
</PropertyGroup>
<!-- Deterministic paths in CI builds only; local builds keep real paths for debugging. -->
<PropertyGroup Condition="'$(GITHUB_ACTIONS)' == 'true'">
  <ContinuousIntegrationBuild>true</ContinuousIntegrationBuild>
</PropertyGroup>
<ItemGroup>
  <None Include="README.md" Pack="true" PackagePath="\" />
</ItemGroup>
```

## Nullable Reference Types

- Enable nullable reference types (`<Nullable>enable</Nullable>`).
- Declare variables non-nullable. Use `?` only when `null` is a valid value.
- Public API entry points validate arguments with `ArgumentNullException.ThrowIfNull` (and `ArgumentException.ThrowIfNullOrEmpty` for strings), because callers may not use nullable annotations.
- Inside the project, trust the C# null annotations and don't add null checks when the type system says a value cannot be null.
- Always use `is null` or `is not null` instead of `== null` or `!= null`.

## Data Access

- When a project needs a database, use Entity Framework Core behind repository interfaces.
- Schema changes go through EF Core migrations. Seed static reference data with `HasData`, and data that needs logic with `UseSeeding`/`UseAsyncSeeding`.
- Read-only queries use `AsNoTracking()` and project to DTOs with `Select`. Avoid N+1 queries (use projections or `Include`). Filter, sort and paginate in the database, not in memory.
- Integration tests that involve EF Core, including ASP.NET Core tests, use the EF Core in-memory provider (`UseInMemoryDatabase`) with a unique database name per test.

## Logging

- Applications use Serilog for structured logging. Libraries depend only on `Microsoft.Extensions.Logging.Abstractions`.
- Every DI-registered service injects `ILogger<T>` (its own type) and logs what helps debugging and operations.
- Use message templates with named placeholders (`"Loaded {FileCount} files from {Folder}"`), never string interpolation.
- Log levels: `Debug` for diagnostic detail, `Information` for significant events, `Warning` for recoverable problems, `Error` for failed operations, `Critical` for failures that take the application down.

## Testing

- Use [NUnit](https://www.nuget.org/packages/NUnit), [NUnit.Analyzers](https://www.nuget.org/packages/NUnit.Analyzers), [Moq](https://www.nuget.org/packages/Moq), [AutoFixture.AutoMoq](https://www.nuget.org/packages/AutoFixture.AutoMoq) and [FluentAssertions](https://www.nuget.org/packages/FluentAssertions) nuget packages.
- Keep FluentAssertions on 7.x. Version 8.0 and later require a paid commercial license, so never upgrade it.
- Name tests `MethodName_When_Then`. In an existing test file that uses a different convention, match that file.

### Unit tests

- Always reference and use the [Dzaba.TestUtils](https://www.nuget.org/packages/Dzaba.TestUtils) nuget package.
- Test classes derive from `AutoFixtureTestFixture` (at least).
- Each test class has a `CreateSut()` method that returns `Fixture.Create<T>()`.
- Create mocks with the `Fixture.FreezeMock<T>()` extension. Mocks that every test in the class needs are frozen in a `[SetUp]` method. Mocks and `Setup()` calls used by only one test go in that test.

Example:
```csharp
[TestFixture]
public class MyServiceTests : AutoFixtureTestFixture
{
    private Mock<IMyDependency> dependency;

    [SetUp]
    public void SetupMocks()
    {
        dependency = Fixture.FreezeMock<IMyDependency>();
    }

    private MyService CreateSut()
    {
        return Fixture.Create<MyService>();
    }

    [Test]
    public void Hello_WhenDependencyReturnsValue_ThenReturnsIt()
    {
        dependency.Setup(x => x.Hello()).Returns(42);

        var sut = CreateSut();

        sut.Hello().Should().Be(42);
    }
}
```

### Integration tests

- Reference and use the [Dzaba.TestUtils.Integration](https://www.nuget.org/packages/Dzaba.TestUtils.Integration) nuget package.
- Test classes derive from `IocTestFixture`.
- Use Serilog Console and Debug logging. Never use a null logger.

## Collections

- Use `IEnumerable<T>` (not `IReadOnlyList<T>`, `T[]`, etc.) for method parameters and return types.
- Don't call `.ToArray()` or `.ToList()` just because the return type is `IEnumerable<T>`. A pure projection with no side effects is returned lazily. Materialize only when enumerating twice would repeat side effects (random draws, I/O) or when the source may change.
- Never return `Task<IEnumerable<T>>`. Async methods that return a collection use `IAsyncEnumerable<T> XxxAsync(..., CancellationToken)`. Implementations that await something are `async IAsyncEnumerable<T>` with `[EnumeratorCancellation]`. Synchronous implementations return a built list with `.ToAsyncEnumerable()`, which avoids CS1998. (`ToAsyncEnumerable` is built into .NET 10; on older targets it comes from `System.Linq.Async`.)
- An `IAsyncEnumerable<T>` backed by an EF Core query keeps the `DbContext` busy until the caller finishes enumerating. Don't let the `DbContext` be disposed or reused before that.

## Async and Multithreading

- I/O is async end to end. Never block on async code with `.Result`, `.Wait()` or `.GetAwaiter().GetResult()`.
- Async methods take a `CancellationToken` as the last parameter and pass it to every call that accepts one.
- `.ConfigureAwait(false)` on every `await` in library code (every project except application hosts and tests). This includes `await foreach (... in x.ConfigureAwait(false))` and `await using`. For `await using`, declare the resource first; declaring it inline changes the variable's type:
```csharp
var stream = File.OpenRead(path);
await using var _ = stream.ConfigureAwait(false);
```
- Never use the `volatile` keyword for synchronization. Use `lock` on a `System.Threading.Lock` field, `Interlocked`, or concurrent collections.

## Performance

- Benchmark with [BenchmarkDotNet](https://www.nuget.org/packages/BenchmarkDotNet) before and after an optimization.

## Inversion of Control

- One static `Bootstrapper` class per project holds the DI registration extension in `Bootstrapper.cs` at the project root namespace.
- Interfaces are public, service implementations are `internal sealed` (repositories, unit of work, scopes, hosted services…). Consumers depend on the interface and get the implementation from the project's `Bootstrapper`.
- Code in other projects that needs an implementation builds it through DI, not with `new`.
- Split abstractions from technology-specific implementations. `Xxx.Core` holds the interfaces and technology-neutral code. Each technology gets its own `Xxx.<Technology>` project that references `Xxx.Core` and registers its implementations in its `Bootstrapper`. Only the application host references a technology project. Example: `DataAccess.Core` references only `Microsoft.EntityFrameworkCore`, while `DataAccess.Postgres` and `DataAccess.SqlServer` add their providers and their own migrations.
- Lifetimes follow the general rule: register with `AddTransient`/`TryAddTransient` by default.
- Inject the options class, not `IOptions<TOptions>`. Register it from the options pipeline (this gives up reload-on-change):
```csharp
services.AddOptions<MyOptions>().BindConfiguration(MyOptions.SectionName);
services.TryAddSingleton(sp => sp.GetRequiredService<IOptions<MyOptions>>().Value);
```

## Deployment

- Containerize with the SDK's built-in container support (`dotnet publish -t:PublishContainer`), not a hand-written Dockerfile, unless the image needs something the SDK can't produce.
