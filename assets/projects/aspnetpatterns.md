# ASP.NET Core data-access patterns

This repository implements the same task-management use case with two data-access
architectures as a practical example for exploring ASP.NET Core data-access
patterns. It is an educational example designed to demonstrate and compare
different approaches to working with SQL Server.

*[View on GitHub](https://github.com/XarisA/AspNetCore-DataAccess-Patterns)*

## Architecture comparison

Entity Framework Core is generally the better choice for applications where
development speed, maintainable domain models, composable LINQ queries, change
tracking, and versioned schema migrations are priorities. It reduces repetitive
data-access code, although its abstraction can make highly specialized SQL and
performance tuning less direct.

ADO.NET is a strong fit when an application needs precise control over
SQL, must work with an existing or stored-procedure-heavy database, or has
performance-critical queries that benefit from explicit optimization. Raw
ADO.NET provides maximum control with more boilerplate, while Dapper keeps the
SQL explicit and handles object mapping with minimal overhead.

Both projects contain HTTP and data-access layers without a view layer.

| Project | Application architecture | Data access | Design intent |
| --- | --- | --- | --- |
| `Backend_Entity` | Pragmatic controller-to-data-access architecture | Entity Framework Core with SQL Server | Controllers use `AppDbContext` directly. EF Core supplies repository-like collections, change tracking, unit-of-work behavior, LINQ queries, and versioned migrations. |
| `Backend_ADO.Net` | Layered architecture with the Strategy pattern | Dapper or raw ADO.NET with SQL Server | Controllers depend on the `IDataService` abstraction. Dependency injection selects either the Dapper or raw ADO.NET strategy through `DataAccess:Provider`. |

Both APIs use asynchronous I/O, cancellation tokens, model validation,
parameterized SQL, correct `404` semantics for missing records, and a CORS policy
for the Angular development origin.

## Requirements

- .NET 8 SDK
- SQL Server or SQL Server Express
- A database connection available as `ConnectionStrings:DefaultConnection`

The committed development settings use Windows authentication against
`Server=localhost;Database=XarDB`. If your SQL Server uses a different instance,
open `appsettings.Development.json` in each backend project through Solution
Explorer and update `ConnectionStrings:DefaultConnection`.

Do not commit credentials to `appsettings.json`.

## Entity Framework implementation

To create or update the database, open the solution in Visual Studio, select
`BackEndEntity` as the default project in Package Manager Console, and run
`Update-Database`. Entity Framework will apply the migrations stored in the
`Backend_Entity/Migrations` folder.

To run the API, right-click `BackEndEntity` in Solution Explorer, select
**Set as Startup Project**, and start it using the Visual Studio Run button.

The main task endpoint is `api/tasks`. The Entity project also includes a small
`api/products` example.

## ADO.NET implementation

To create the `Tasks` table, open `Backend_ADO.Net/Database/schema.sql` in SQL
Server Management Studio or a SQL Server Object Explorer query window, connect
to the target database, and select **Execute**.

Choose the SQL implementation in `Backend_ADO.Net/appsettings.json`:

```json
{
  "DataAccess": {
    "Provider": "Dapper"
  }
}
```

Use `Raw` to exercise `SqlConnection`, `SqlCommand`, parameters, and manual data
reader mapping without Dapper.

To run the API, right-click `BackEndADO.Net` in Solution Explorer, select
**Set as Startup Project**, and start it using the Visual Studio Run button.

The task endpoint is `api/tasks`, matching the Entity Framework implementation
so both architectures expose the same task contract.

## Build and test

Open `Solution/BackendPatterns.sln` in Visual Studio. Select **Build > Build Solution**
to compile all projects. Open **Test > Test Explorer** and select
**Run All** to execute both test projects.

The test projects cover resource-location responses, missing-row semantics, and
basic EF persistence behavior.

## Deliberate scope

Authentication and authorization are not included because no identity or role
requirements are defined for this exercise. A production API would also normally
add authentication, pagination, structured observability, health checks, and
container/deployment configuration.
