---
title: Mitsubishi Controller Web Application
author: "Raymond Lau"
date: "2024-10-17"
---

# Mitsubishi Controller Web Application

## Table of Contents


## Steps to Set Up a Blazor Project with Clean Architecture

To set up a Blazor project using Clean Architecture with .NET, follow these steps:


### 1. Install .NET SDK

If you haven’t already installed the .NET SDK, download it from here. Verify the installation by running:

```properties
dotnet --version
```


### 2. Create Project Structure

A Clean Architecture typically separates concerns into different projects. You can have the following projects:

1. API or UI - Blazor project.
2. Application - Core business logic (Use Cases, DTOs, Interfaces).
3. Domain - Core domain entities and aggregates.
4. Infrastructure - Implementations (e.g., database access, external services).
5. Shared Kernel (optional) - Shared logic across multiple layers (if needed).

![Clean Architecture](./images/clean-architecture.png)


__Create the Projects:__

Start by creating a Blazor project for the UI layer:

```properties
dotnet new blazor -n CleanArchitectureWebApp.UI
```

Create Class Library Projects

1. Application layer:

```properties
dotnet new classlib -n CleanArchitectureWebApp.Application
```

2. Domain layer:

```properties
dotnet new classlib -n CleanArchitectureWebApp.Domain
```

3. Infrastructure layer:

```properties
dotnet new classlib -n CleanArchitectureWebApp.Infrastructure
```

4. (Optional) Create a shared kernel project for reusable logic:

```properties
dotnet new classlib -n CleanArchitectureWebApp.SharedKernel
```

5. Test projects (optional):

```properties
dotnet new xunit -o CleanArchitectureWebApp.Tests.Domain
dotnet new xunit -o CleanArchitectureWebApp.Tests.Application
dotnet new xunit -o CleanArchitectureWebApp.Tests.Infrastructure
dotnet new xunit -o CleanArchitectureWebApp.Tests.UI
```

6. Integration tests (optional):

```properties
dotnet new xunit -o CleanArchitectureWebApp.IntegrationTests
```

### 3. Add Project References

To ensure projects can communicate properly, add the necessary references:

1. From UI project to Application:

```properties
cd CleanArchitectureWebApp.UI
dotnet add reference ../CleanArchitectureWebApp.Application
```

2. From Application to Domain:

```properties
cd ../CleanArchitectureWebApp.Application
dotnet add reference ../CleanArchitectureWebApp.Domain
```

3. From Infrastructure to both Application and Domain:

```properties
cd ../CleanArchitectureWebApp.Infrastructure
dotnet add reference ../CleanArchitectureWebApp.Application
dotnet add reference ../CleanArchitectureWebApp.Domain
```

4. If using the Shared Kernel, reference it from Domain, Application, and Infrastructure if needed.

```properties
cd ../CleanArchitectureWebApp.Domain
dotnet add reference ../CleanArchitectureWebApp.SharedKernel

cd ../CleanArchitectureWebApp.Application
dotnet add reference ../CleanArchitectureWebApp.SharedKernel

cd ../CleanArchitectureWebApp.Infrastructure
dotnet add reference ../CleanArchitectureWebApp.SharedKernel
```

5. If you have test projects, reference the appropriate layers:

```properties
cd ../CleanArchitectureWebApp.Tests.Domain
dotnet add reference ../CleanArchitectureWebApp.Domain

cd ../CleanArchitectureWebApp.Tests.Application
dotnet add reference ../CleanArchitectureWebApp.Application

cd ../CleanArchitectureWebApp.Tests.Infrastructure
dotnet add reference ../CleanArchitectureWebApp.Infrastructure

cd ../CleanArchitectureWebApp.Tests.UI
dotnet add reference ../CleanArchitectureWebApp.UI
```

6. If you have integration tests, reference the necessary layers:

```properties
cd ../CleanArchitectureWebApp.IntegrationTests
dotnet add reference ../CleanArchitectureWebApp.Application
dotnet add reference ../CleanArchitectureWebApp.Infrastructure
dotnet add reference ../CleanArchitectureWebApp.Domain
```


### 4. Install Dependencies (Optional)

You may want to add dependencies like MediatR, FluentValidation, or Entity Framework Core for Clean Architecture patterns. Install them using NuGet:

```properties
# Domain Layer
dotnet add package FluentValidation


# CleanArchitectureWebApp.Application Layer
dotnet add package AutoMapper
dotnet add package FluentValidation


# Infrastructure Layer
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.Tools

dotnet add package Aspire.Npgsql.EntityFrameworkCore.PostgreSQL

dotnet add package Serilog
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File

dotnet add package Coravel
dotnet add package Newtonsoft.Json

dotnet add package Swashbuckle.AspNetCore
```

Install these packages in the appropriate projects depending on your design.


### 5. Implement the Layers

__Domain Layer__
- Define entities, value objects, and aggregates here.

```csharp
namespace CleanArchitectureWebApp.Domain.Entities
{
    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; }
    }
}
```


__Application Layer__
- Add interfaces, services, and use cases here.

```csharp
namespace CleanArchitectureWebApp.Application.Interfaces
{
    public interface IProductService
    {
        Task<IEnumerable<Product>> GetProductsAsync();
    }
}
```

__Infrastructure Layer__
- Implement interfaces defined in the Application layer, such as repositories or external services.

```csharp
namespace CleanArchitectureWebApp.Infrastructure.Services
{
    public class ProductService : IProductService
    {
        public Task<IEnumerable<Product>> GetProductsAsync()
        {
            // Implementation
        }
    }
}
```

__UI Layer (Blazor)__
- Inject services into the Blazor components using Dependency Injection (DI).

In Program.cs of the Blazor project:

```csharp
builder.Services.AddScoped<IProductService, ProductService>();
```

Then in a Blazor component:

```csharp
@page "/products"
@inject IProductService ProductService

<h3>Products</h3>

@code {
    private IEnumerable<Product> products;

    protected override async Task OnInitializedAsync()
    {
        products = await ProductService.GetProductsAsync();
    }
}
```


### 6. Configure Dependency Injection

Ensure Infrastructure services are registered in the DI container:

In Program.cs or Startup.cs of the UI project:

```csharp
builder.Services.AddScoped<IProductService, ProductService>();
```


### 7. Run the Application

Navigate to the Blazor UI project and run it:

```properties
cd CleanArchitectureWebApp.UI
dotnet run --launch-profile https
```

You should now have a working Blazor project set up with Clean Architecture principles!


### 8. Test the Application

In __Clean Architecture__, referencing the __Test Projects__ (both __Unit Tests__ and __Integration Tests__) to the core layers needs to be done thoughtfully to maintain the separation of concerns and follow the principles of Clean Architecture. The test projects should have access to the layers they are testing, but they should not introduce unnecessary dependencies or violate architectural rules.

#### 1. __Unit Tests Project__

- __Unit Tests__ are focused on testing individual components in isolation, typically within the __Domain__ and __Application__ layers. 
- These tests should __only reference__ the specific layers they are testing and any necessary __testing libraries__ (like xUnit, NUnit, or Moq).

##### References for Unit Tests:
- __Domain Unit Tests__: Reference the __Domain Layer__.
- __Application Unit Tests__: Reference the __Application Layer__.
- __Infrastructure Unit Tests__: Reference the __Infrastructure Layer__.

For example, the __DomainTests__ project should reference the `CleanArchitectureWebApp.Domain` project:

```properties
dotnet add CleanArchitectureWebApp.Tests.Domain reference ../CleanArchitectureWebApp.Domain
```

If you have a __Unit Test Project__ for the __Application Layer__:

```properties
dotnet add CleanArchitectureWebApp.Tests.Application reference ../CleanArchitectureWebApp.Application
```

#### 2. __Integration Tests Project__

- __Integration Tests__ are meant to test how different parts of the system interact, typically across layers. For example, you might want to test the interaction between the __Application Layer__ and the __Infrastructure Layer__ (such as making sure a repository correctly interacts with a database).
- These tests should reference __multiple layers__ as needed. For example, if you're testing a service from the __Application Layer__ that interacts with a database from the __Infrastructure Layer__, you'll need to reference both.

##### References for Integration Tests:
- __Integration Tests__ might reference:
  - __Application Layer__
  - __Infrastructure Layer__
  - __Domain Layer__ (if required by the application layer)

For example, if you are writing an integration test for an application service that depends on the infrastructure (like database access):

```properties
dotnet add CleanArchitectureWebApp.IntegrationTests reference ../CleanArchitectureWebApp.Application
dotnet add CleanArchitectureWebApp.IntegrationTests reference ../CleanArchitectureWebApp.Infrastructure
dotnet add CleanArchitectureWebApp.IntegrationTests reference ../CleanArchitectureWebApp.Domain
```

#### Steps for Referencing Test Projects to Other Layers

1. __Create the Test Projects__: First, ensure you have separate test projects for each of the layers you plan to test.
   - Example for __Unit Tests__:
     ```properties
     dotnet new xunit -o CleanArchitectureWebApp.Tests.Domain
     dotnet new xunit -o CleanArchitectureWebApp.Tests.Application
     ```
   - Example for __Integration Tests__:
     ```properties
     dotnet new xunit -o CleanArchitectureWebApp.IntegrationTests
     ```

2. __Add the Necessary Project References__: 
   Use the `dotnet add reference` command to reference the layers that the test project needs to access.
   
   - Example for a __Domain Unit Test__ project referencing the __Domain Layer__:
     ```properties
     dotnet add CleanArchitectureWebApp.Tests.Domain reference ../CleanArchitectureWebApp.Domain
     ```

   - Example for an __Application Unit Test__ project referencing the __Application Layer__:
     ```properties
     dotnet add CleanArchitectureWebApp.Tests.Application reference ../CleanArchitectureWebApp.Application
     ```

   - Example for an __Integration Test__ project referencing __multiple layers__:
     ```properties
     dotnet add CleanArchitectureWebApp.IntegrationTests reference ../CleanArchitectureWebApp.Application
     dotnet add CleanArchitectureWebApp.IntegrationTests reference ../CleanArchitectureWebApp.Infrastructure
     dotnet add CleanArchitectureWebApp.IntegrationTests reference ../CleanArchitectureWebApp.Domain
     ```

3. __Add Testing Libraries__: You may need to install additional packages for testing, mocking, and integration testing.
   - For __Unit Testing__ libraries like xUnit or NUnit:
     ```properties
     dotnet add CleanArchitectureWebApp.Tests package xunit
     dotnet add CleanArchitectureWebApp.Tests package Moq
     ```
   - For __Integration Testing__ libraries like `Microsoft.EntityFrameworkCore.InMemory` (if using EF Core):
     ```properties
     dotnet add CleanArchitectureWebApp.IntegrationTests package Microsoft.EntityFrameworkCore.InMemory
     ```

#### Best Practices for Test Project References

1. __Test Projects Should Depend on Other Projects, Not the Other Way Around__:
   - The core layers (Domain, Application, Infrastructure) __should not reference__ test projects.
   - Test projects should be self-contained and reference the __layers__ they need to test.

2. __Avoid Introducing Circular Dependencies__:
   - Keep each test project focused on the layer it's testing. For example, the __ApplicationTests__ project should only reference the __Application Layer__ and, if necessary, the __Domain Layer__.

3. __Mock Infrastructure Dependencies in Unit Tests__:
   - For unit tests, avoid direct references to the __Infrastructure Layer__ (like databases or external APIs). Instead, __mock__ these dependencies to keep the tests isolated and focused on the logic of the layer being tested.
   - Use libraries like __Moq__ or __NSubstitute__ for mocking.

4. __Use In-Memory Databases for Integration Tests__:
   - When testing interactions with the database in integration tests, consider using an __in-memory database__ like `Microsoft.EntityFrameworkCore.InMemory` to avoid dependencies on a real database.

#### Example of Integration Test with In-Memory Database

Here is a simple example of how an integration test might look, using an in-memory database for testing repository interaction:

```csharp
public class OrderServiceIntegrationTests
{
    private readonly DbContextOptions<AppDbContext> _dbContextOptions;

    public OrderServiceIntegrationTests()
    {
        // Setup in-memory database options
        _dbContextOptions = new DbContextOptionsBuilder<AppDbContext>()
            .UseInMemoryDatabase(databaseName: "TestDb")
            .Options;
    }

    [Fact]
    public async Task CreateOrder_ShouldAddOrderToDatabase()
    {
        // Arrange
        using var context = new AppDbContext(_dbContextOptions);
        var repository = new OrderRepository(context);
        var service = new OrderService(repository);

        var newOrder = new Order { OrderId = 1, ProductName = "Test Product" };

        // Act
        await service.CreateOrder(newOrder);

        // Assert
        var orderInDb = await context.Orders.FindAsync(1);
        Assert.NotNull(orderInDb);
        Assert.Equal("Test Product", orderInDb.ProductName);
    }
}
```

#### Summary

- __Unit Test Projects__ should reference only the specific layers they are testing.
  - __Domain Unit Tests__: Reference the __Domain Layer__.
  - __Application Unit Tests__: Reference the __Application Layer__.
  - __Infrastructure Unit Tests__: Reference the __Infrastructure Layer__.

- __Integration Test Projects__ should reference the layers they are testing together.
  - For example, an integration test might reference both the __Application Layer__ and __Infrastructure Layer__.

By organizing your references this way, you maintain the __separation of concerns__ and ensure that the __Clean Architecture__ principles are respected, while also ensuring testability across the different layers.


### Summary of the Folder Structure

Below is a comprehensive directory structure for a __Clean Architecture__ project. It adheres to Clean Architecture principles, separating concerns and ensuring that dependencies flow inward. This example assumes you are using a __.NET 7+__ Blazor WebAssembly (or MVC) application, but the structure can be adapted to any other web or desktop technology.

### Root Directory Structure:

```
/MitsubishiControllerWebApp
│
├── /CleanArchitectureWebApp.Domain            # Core business logic and domain models
│   ├── /Entities                   # Core entities of the domain
│   ├── /ValueObjects               # Value objects (immutable types)
│   ├── /Enums                      # Enums related to domain logic
│   ├── /Exceptions                 # Domain-specific exceptions
│   ├── /Interfaces                 # Domain interfaces for repositories, services, etc.
│   ├── /Specifications             # Specification pattern (optional)
│   └── /Events                     # Domain events (for event-driven architectures)
│
├── /CleanArchitectureWebApp.Application       # Application layer containing use cases
│   ├── /DTOs                       # Data Transfer Objects (input/output models)
│   ├── /Enums                      # Application-level enums
│   ├── /Interfaces                 # Interfaces for application services
│   ├── /Services                   # Application services (business logic or use cases)
│   ├── /Validators                 # Validation rules (e.g., FluentValidation)
│   ├── /Mappers                    # AutoMapper profiles and mapping logic
│   ├── /Features                   # Application features (e.g., CQRS commands/queries)
│       ├── /Commands               # Command handlers (CQRS write operations)
│       └── /Queries                # Query handlers (CQRS read operations)
│   ├── /Events                     # Application-level events
│   └── /Behaviors                  # Pipeline behaviors (e.g., validation, logging with MediatR)
│
├── /CleanArchitectureWebApp.Infrastructure    # Infrastructure layer (external dependencies)
│   ├── /Persistence                # Database context, migrations, repositories
│       ├── /EntitiesConfigurations # Entity Framework configurations
│       └── /Repositories           # Concrete repository implementations
│   ├── /Services                   # External services (e.g., API clients, email, caching)
│   ├── /Logging                    # Logging and observability tools
│   ├── /Files                      # File storage or cloud integrations (Azure, AWS)
│   ├── /Email                      # Email sending services (SMTP, MailKit)
│   ├── /HttpClients                # HTTP clients (for API calls)
│   ├── /Security                   # Security-related services (e.g., JWT, identity)
│   └── /Events                     # Infrastructure-level event handling (message brokers, etc.)
│
├── /CleanArchitectureWebApp.UI                # User Interface (e.g., Blazor WebAssembly or MVC)
│   ├── /Components                 # Blazor/MVC components (UI elements)
│   ├── /Pages                      # Blazor pages (for Razor components) or MVC views
│   ├── /Services                   # UI-specific services (state management, etc.)
│   ├── /wwwroot                    # Static files (CSS, JS, images)
│   ├── /Layouts                    # Layout components (header, sidebar, etc.)
│   ├── /Authentication             # UI-level authentication (e.g., OIDC, JWT)
│   ├── /Authorization              # UI-level authorization (policies, claims-based access)
│   └── /Resources                  # Localization resources
│
├── /CleanArchitectureWebApp.SharedKernel      # Shared kernel for cross-cutting concerns (optional)
│   ├── /Interfaces                 # Shared interfaces used across different layers
│   ├── /Services                   # Shared services (e.g., logging, caching, date/time)
│   ├── /Enums                      # Enums shared across multiple layers
│   ├── /Extensions                 # Extension methods shared by multiple layers
│   └── /Events                     # Cross-layer events or notifications
│
├── /CleanArchitectureWebApp.Tests             # Unit and integration tests
│   ├── /DomainTests                # Tests for the domain layer
│   ├── /ApplicationTests           # Tests for the application layer (e.g., use cases)
│   ├── /InfrastructureTests        # Tests for infrastructure services (e.g., repositories, APIs)
│   └── /UITests                    # Tests for the UI layer (e.g., component or page tests)
│
├── /CleanArchitectureWebApp.IntegrationTests  # Integration tests (optional)
│   ├── /ApiTests                   # Integration tests for API endpoints
│   ├── /DatabaseTests              # Database-related integration tests
│   └── /EndToEndTests              # End-to-end tests (optional)
│
├── CleanArchitectureWebApp.sln                # Solution file
├── /Configurations                 # Configuration files (e.g., appsettings.json, serilog.json)
└── /Docker                         # Docker-related files (e.g., Dockerfile, docker-compose.yml)
```

### Key Components:

1. __Domain Layer__:
   - __Entities__: Core business entities, like `Customer`, `Order`, or `Product`.
   - __ValueObjects__: Immutable types that represent concepts like `Money`, `Address`, or `DateRange`.
   - __Enums__: Enumerations used in the domain, like `OrderStatus`, `CustomerType`.
   - __Exceptions__: Custom exceptions tied to domain logic.
   - __Interfaces__: Domain-specific interfaces (e.g., repository interfaces).
   - __Events__: Domain events for event-driven architecture.
   - __Specifications__: For implementing query logic using the specification pattern (optional).

2. __Application Layer__:
   - __DTOs__: Data Transfer Objects to decouple the domain from the UI and infrastructure.
   - __Services__: Application services or use cases that implement business logic.
   - __Features__: Handlers for commands and queries (useful in CQRS).
   - __Validators__: Request validation logic, often using libraries like __FluentValidation__.
   - __Behaviors__: MediatR pipeline behaviors for logging, validation, etc.

3. __Infrastructure Layer__:
   - __Persistence__: Database context, entity configurations, and repository implementations.
   - __Services__: Implementation of external services, such as file storage, email, or APIs.
   - __Logging__: Infrastructure for logging and error tracking (e.g., Serilog).
   - __HttpClients__: HTTP clients for external API integration.
   - __Security__: JWT, OAuth, or other security-related services.

4. __UI Layer__:
   - __Components__: UI components such as buttons, forms, and other reusable elements.
   - __Pages__: Razor components (Blazor) or MVC views.
   - __Services__: Client-side services (state management, HTTP calls, etc.).
   - __Layouts__: Shared layouts (e.g., navigation menus, headers).
   - __Authentication/Authorization__: OIDC or JWT authentication handling, authorization policies.
   - __Static Files__: CSS, JS, images, and other static content under the `/wwwroot` folder.

5. __Shared Kernel__ (Optional):
   - A __Shared Kernel__ is useful for cross-cutting concerns, like logging, extension methods, or utility functions that need to be accessed by multiple layers.

6. __Testing__:
   - __Unit Tests__: Each layer has its corresponding test folder (e.g., DomainTests, ApplicationTests).
   - __Integration Tests__: For testing the entire flow, especially for services interacting with databases or external APIs.

7. __Configurations__:
   - Centralized configuration files (`appsettings.json`), environment variables, or Docker configurations.

### Benefits of This Structure:
- __Separation of Concerns__: Each layer is isolated, making it easier to manage changes and dependencies.
- __Testability__: By keeping the domain and application layers free from external dependencies, it's easier to unit test business logic.
- __Flexibility__: The infrastructure and UI layers can be swapped or changed without affecting the core business logic.

This structure provides clarity and flexibility, making it easier to maintain and extend the project over time. It follows __Clean Architecture__ principles by ensuring the core business logic is at the center, insulated from external concerns like UI or infrastructure.

