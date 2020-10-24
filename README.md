![](https://raw.githubusercontent.com/dejanstojanovic/sql-server-script-seeding/master/src/EntityFrameworkCore.SqlServer.Seeding/icon.png)

# SQL Server script seeding for Entity Framework Core

## Overview

Allows you to seed the data to SQL Server database using pure SQL scripts but in a managed fashion similar to Entity Framework Core migrations.
More details on the logic how seeding works you can read in [Seeding data in EF Core using SQL scripts](https://dejanstojanovic.net/aspnet/2020/september/seeding-data-in-ef-core-using-sql-scripts/) article.
Details about the global tool logic can be found in article [Building and using advanced .NET Core CLI global tools](https://dejanstojanovic.net/aspnet/2020/september/building-and-using-advanced-net-core-cli-global-tools/).

## Usage
### Invoking the seeding
To use SQL script seeding with EF Core and targeting SQL Server database you need to bring in the NuGet package [EntityFrameworkCore.SqlServer.Seeding](https://www.nuget.org/packages/EntityFrameworkCore.SqlServer.Seeding/) which contains extension methods for dependency injection and ASP.NET Core pipeline.

[![NuGet](https://img.shields.io/nuget/v/EntityFrameworkCore.SqlServer.Seeding.svg)](https://www.nuget.org/packages/EntityFrameworkCore.SqlServer.Seeding)
```
dotnet add package EntityFrameworkCore.SqlServer.Seeding
```

Add script seeding to services collection and supply the connection string to the database

```csharp
public void ConfigureServices(IServiceCollection services)
{
	...
	services.AddScriptSeeding(Configuration.GetConnectionString("EmployeesDatabase"));
	...
}
```

Invoke the seeding in the pipeline by supplying the assembly where your scripts are sitting along with a folder under which scripts sit under project tree. The default value is "Seedings"

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
	...
    app.SeedFromScripts(typeof(EmployeesDatabaseContext).Assembly, "Seedings");
	...
}
```

### Adding new seeding script

Seeding scripts can be added either **manually from IDE** or **using CLI tool**.

#### Adding seeding script manually

To add script manualy you need to create new .sql file in your porject and mark it as embeded resource. Make sure that your file name starts with timestamp in format **yyyyMMddHHss**. Here is a sample of a seedign SQL script:

```xml
  <ItemGroup Label="seeding">
    <None Remove="Seedings\20201017235255_Add_Initial_Employees.sql" />
    <EmbeddedResource Include="Seedings\20201017235255_Add_Initial_Employees.sql" />
  </ItemGroup>
```

#### Adding seedign script via CLI

As a first step you need to install global tool [EntityFrameworkCore.SqlServer.Seeding.Tool](https://www.nuget.org/packages/EntityFrameworkCore.SqlServer.Seeding.Tool/) which will help you add the seeding script to the project where you want to have your seeding scripts.
I usually keep them together with migrations as a part of infrastructure.

[![NuGet](https://img.shields.io/nuget/v/EntityFrameworkCore.SqlServer.Seeding.Tool.svg)](https://www.nuget.org/packages/EntityFrameworkCore.SqlServer.Seeding.Tool)
```
dotnet tool install --global EntityFrameworkCore.SqlServer.Seeding.Tool 
```

From a command line (cmd, PowerShell, bash…) navigate to project folder where you want your scripts to sit.
Simply add the script by invoking the previously installed global tool

```
seeding add "Add_Initial_Employees" -o Seedings
```

This will create new file under Seedings folder in your project tree.
Once you run the main project, seeding files will be executed and recorded in __SeedingHistory table

