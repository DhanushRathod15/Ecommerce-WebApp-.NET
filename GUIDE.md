# Building an E-Commerce Application with .NET - Complete Guide

This guide provides step-by-step instructions to build a production-ready e-commerce application similar to [dotnet/eShop](https://github.com/dotnet/eShop) from scratch using .NET 9.

## Table of Contents
1. [Environment Setup](#1-environment-setup)
2. [Project Structure and Skeleton](#2-project-structure-and-skeleton)
3. [Core Features Implementation](#3-core-features-implementation)
4. [Testing Strategy](#4-testing-strategy)
5. [Containerization](#5-containerization)
6. [CI/CD Pipeline](#6-cicd-pipeline)
7. [Deployment Strategies](#7-deployment-strategies)
8. [Best Practices and Patterns](#8-best-practices-and-patterns)

---

## 1. Environment Setup

### Prerequisites

Install the following tools on your development machine:

#### Required Software
- **.NET 9 SDK**: Download from [dotnet.microsoft.com](https://dotnet.microsoft.com/download)
  ```bash
  # Verify installation
  dotnet --version
  ```

- **Docker Desktop**: For containerization and local development
  - Windows/Mac: [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
  - Linux: Install Docker Engine
  ```bash
  # Verify installation
  docker --version
  docker-compose --version
  ```

- **Git**: Version control
  ```bash
  # Verify installation
  git --version
  ```

- **IDE/Editor** (choose one):
  - Visual Studio 2022 (v17.8+) with .NET 9 workload
  - Visual Studio Code with C# Dev Kit extension
  - JetBrains Rider

#### Optional but Recommended
- **Azure CLI**: For Azure deployments
  ```bash
  az --version
  ```

- **kubectl**: For Kubernetes deployments
  ```bash
  kubectl version --client
  ```

- **SQL Server**: Local database (or use Docker container)

### Setting Up Your Development Environment

1. **Create a workspace directory**
   ```bash
   mkdir ecommerce-app
   cd ecommerce-app
   ```

2. **Initialize Git repository**
   ```bash
   git init
   git config user.name "Your Name"
   git config user.email "your.email@example.com"
   ```

3. **Create .gitignore file**
   ```bash
   dotnet new gitignore
   ```

4. **Verify .NET templates**
   ```bash
   dotnet new list
   ```

---

## 2. Project Structure and Skeleton

### Architecture Overview

We'll use a **Microservices Architecture** with Clean Architecture principles, similar to eShop:

```
ecommerce-app/
├── src/
│   ├── ApiGateways/
│   │   └── Web.Gateway/
│   ├── Services/
│   │   ├── Catalog/
│   │   │   ├── Catalog.API/
│   │   │   ├── Catalog.Domain/
│   │   │   ├── Catalog.Application/
│   │   │   └── Catalog.Infrastructure/
│   │   ├── Basket/
│   │   │   ├── Basket.API/
│   │   │   └── Basket.Infrastructure/
│   │   ├── Ordering/
│   │   │   ├── Ordering.API/
│   │   │   ├── Ordering.Domain/
│   │   │   ├── Ordering.Application/
│   │   │   └── Ordering.Infrastructure/
│   │   ├── Identity/
│   │   │   └── Identity.API/
│   │   └── Payment/
│   │       └── Payment.API/
│   └── WebApps/
│       └── WebApp/
├── tests/
│   ├── Catalog.UnitTests/
│   ├── Catalog.IntegrationTests/
│   ├── Ordering.UnitTests/
│   └── Ordering.IntegrationTests/
├── docker-compose.yml
├── docker-compose.override.yml
├── .github/
│   └── workflows/
│       ├── build.yml
│       └── deploy.yml
└── README.md
```

### Step 1: Create Solution Structure

```bash
# Create solution file
dotnet new sln -n EcommerceApp

# Create directory structure
mkdir -p src/{ApiGateways,Services,WebApps}
mkdir -p tests
mkdir -p docker
```

### Step 2: Create Catalog Service (Example)

```bash
# Navigate to services directory
cd src/Services

# Create Catalog service projects
mkdir -p Catalog/{Catalog.API,Catalog.Domain,Catalog.Application,Catalog.Infrastructure}

# Create Domain project (Class Library)
cd Catalog/Catalog.Domain
dotnet new classlib -n Catalog.Domain
cd ../..

# Create Application project (Class Library)
cd Catalog/Catalog.Application
dotnet new classlib -n Catalog.Application
cd ../..

# Create Infrastructure project (Class Library)
cd Catalog/Catalog.Infrastructure
dotnet new classlib -n Catalog.Infrastructure
cd ../..

# Create API project (Web API)
cd Catalog/Catalog.API
dotnet new webapi -n Catalog.API
cd ../..

# Go back to solution root
cd ../../..

# Add projects to solution
dotnet sln add src/Services/Catalog/Catalog.Domain/Catalog.Domain.csproj
dotnet sln add src/Services/Catalog/Catalog.Application/Catalog.Application.csproj
dotnet sln add src/Services/Catalog/Catalog.Infrastructure/Catalog.Infrastructure.csproj
dotnet sln add src/Services/Catalog/Catalog.API/Catalog.API.csproj
```

### Step 3: Set Up Project References

```bash
# Add references between layers
dotnet add src/Services/Catalog/Catalog.Application/Catalog.Application.csproj reference src/Services/Catalog/Catalog.Domain/Catalog.Domain.csproj

dotnet add src/Services/Catalog/Catalog.Infrastructure/Catalog.Infrastructure.csproj reference src/Services/Catalog/Catalog.Application/Catalog.Application.csproj

dotnet add src/Services/Catalog/Catalog.API/Catalog.API.csproj reference src/Services/Catalog/Catalog.Infrastructure/Catalog.Infrastructure.csproj
dotnet add src/Services/Catalog/Catalog.API/Catalog.API.csproj reference src/Services/Catalog/Catalog.Application/Catalog.Application.csproj
```

### Step 4: Install Essential NuGet Packages

```bash
# For API projects
dotnet add src/Services/Catalog/Catalog.API/Catalog.API.csproj package Swashbuckle.AspNetCore
dotnet add src/Services/Catalog/Catalog.API/Catalog.API.csproj package Microsoft.EntityFrameworkCore.Design

# For Infrastructure projects
dotnet add src/Services/Catalog/Catalog.Infrastructure/Catalog.Infrastructure.csproj package Microsoft.EntityFrameworkCore.SqlServer
dotnet add src/Services/Catalog/Catalog.Infrastructure/Catalog.Infrastructure.csproj package Microsoft.EntityFrameworkCore.Tools

# For Application projects
dotnet add src/Services/Catalog/Catalog.Application/Catalog.Application.csproj package FluentValidation.DependencyInjectionExtensions
dotnet add src/Services/Catalog/Catalog.Application/Catalog.Application.csproj package MediatR
```


---

## 3. Core Features Implementation

### Feature 1: Product Catalog Service

#### Domain Layer (Catalog.Domain)

Create domain entities in `Catalog.Domain/Entities/Product.cs`:

```csharp
namespace Catalog.Domain.Entities;

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public string PictureFileName { get; set; } = string.Empty;
    public int CatalogTypeId { get; set; }
    public CatalogType? CatalogType { get; set; }
    public int CatalogBrandId { get; set; }
    public CatalogBrand? CatalogBrand { get; set; }
    public int AvailableStock { get; set; }
}

public class CatalogType
{
    public int Id { get; set; }
    public string Type { get; set; } = string.Empty;
}

public class CatalogBrand
{
    public int Id { get; set; }
    public string Brand { get; set; } = string.Empty;
}
```

#### Infrastructure Layer (Catalog.Infrastructure)

Create `Catalog.Infrastructure/Data/CatalogContext.cs`:

```csharp
using Microsoft.EntityFrameworkCore;
using Catalog.Domain.Entities;

namespace Catalog.Infrastructure.Data;

public class CatalogContext : DbContext
{
    public CatalogContext(DbContextOptions<CatalogContext> options) : base(options)
    {
    }

    public DbSet<Product> Products => Set<Product>();
    public DbSet<CatalogBrand> CatalogBrands => Set<CatalogBrand>();
    public DbSet<CatalogType> CatalogTypes => Set<CatalogType>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        modelBuilder.Entity<Product>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Name).IsRequired().HasMaxLength(100);
            entity.Property(e => e.Price).HasColumnType("decimal(18,2)");
        });

        modelBuilder.Entity<CatalogBrand>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Brand).IsRequired().HasMaxLength(100);
        });

        modelBuilder.Entity<CatalogType>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Type).IsRequired().HasMaxLength(100);
        });
    }
}
```

#### Application Layer (Catalog.Application)

Create `Catalog.Application/Queries/GetProductsQuery.cs`:

```csharp
using MediatR;
using Catalog.Domain.Entities;

namespace Catalog.Application.Queries;

public record GetProductsQuery(int PageSize, int PageIndex) : IRequest<PaginatedItems<Product>>;

public record PaginatedItems<T>(int PageIndex, int PageSize, long Count, List<T> Data);
```

Create `Catalog.Application/Queries/GetProductsQueryHandler.cs`:

```csharp
using MediatR;
using Microsoft.EntityFrameworkCore;
using Catalog.Infrastructure.Data;
using Catalog.Domain.Entities;

namespace Catalog.Application.Queries;

public class GetProductsQueryHandler : IRequestHandler<GetProductsQuery, PaginatedItems<Product>>
{
    private readonly CatalogContext _context;

    public GetProductsQueryHandler(CatalogContext context)
    {
        _context = context;
    }

    public async Task<PaginatedItems<Product>> Handle(GetProductsQuery request, CancellationToken cancellationToken)
    {
        var totalItems = await _context.Products.LongCountAsync(cancellationToken);
        
        var itemsOnPage = await _context.Products
            .Include(p => p.CatalogBrand)
            .Include(p => p.CatalogType)
            .OrderBy(p => p.Name)
            .Skip(request.PageSize * request.PageIndex)
            .Take(request.PageSize)
            .ToListAsync(cancellationToken);

        return new PaginatedItems<Product>(request.PageIndex, request.PageSize, totalItems, itemsOnPage);
    }
}
```

#### API Layer (Catalog.API)

Update `Catalog.API/Program.cs`:

```csharp
using Microsoft.EntityFrameworkCore;
using Catalog.Infrastructure.Data;
using Catalog.Application.Queries;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Add DbContext
builder.Services.AddDbContext<CatalogContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("CatalogDB")));

// Add MediatR
builder.Services.AddMediatR(cfg => 
    cfg.RegisterServicesFromAssembly(typeof(GetProductsQuery).Assembly));

var app = builder.Build();

// Configure the HTTP request pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

Create `Catalog.API/Controllers/CatalogController.cs`:

```csharp
using Microsoft.AspNetCore.Mvc;
using MediatR;
using Catalog.Application.Queries;

namespace Catalog.API.Controllers;

[ApiController]
[Route("api/[controller]")]
public class CatalogController : ControllerBase
{
    private readonly IMediator _mediator;

    public CatalogController(IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpGet("items")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    public async Task<IActionResult> GetItems(
        [FromQuery] int pageSize = 10,
        [FromQuery] int pageIndex = 0)
    {
        var result = await _mediator.Send(new GetProductsQuery(pageSize, pageIndex));
        return Ok(result);
    }
}
```

Update `Catalog.API/appsettings.json`:

```json
{
  "ConnectionStrings": {
    "CatalogDB": "Server=localhost;Database=CatalogDb;User Id=sa;Password=Your_password123;TrustServerCertificate=True"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

### Feature 2: Shopping Basket Service (Redis-based)

Create Basket service structure:

```bash
# Create Basket service
mkdir -p src/Services/Basket/Basket.API
cd src/Services/Basket/Basket.API
dotnet new webapi -n Basket.API
cd ../../../..

# Add to solution
dotnet sln add src/Services/Basket/Basket.API/Basket.API.csproj

# Add Redis support
dotnet add src/Services/Basket/Basket.API/Basket.API.csproj package StackExchange.Redis
```

Create `Basket.API/Services/BasketService.cs`:

```csharp
using StackExchange.Redis;
using System.Text.Json;

namespace Basket.API.Services;

public interface IBasketService
{
    Task<CustomerBasket?> GetBasketAsync(string customerId);
    Task<CustomerBasket?> UpdateBasketAsync(CustomerBasket basket);
    Task<bool> DeleteBasketAsync(string customerId);
}

public class BasketService : IBasketService
{
    private readonly IDatabase _database;

    public BasketService(IConnectionMultiplexer redis)
    {
        _database = redis.GetDatabase();
    }

    public async Task<CustomerBasket?> GetBasketAsync(string customerId)
    {
        var data = await _database.StringGetAsync(customerId);
        return data.IsNullOrEmpty ? null : JsonSerializer.Deserialize<CustomerBasket>(data!);
    }

    public async Task<CustomerBasket?> UpdateBasketAsync(CustomerBasket basket)
    {
        var created = await _database.StringSetAsync(
            basket.BuyerId, 
            JsonSerializer.Serialize(basket),
            TimeSpan.FromDays(30));
        
        return created ? await GetBasketAsync(basket.BuyerId) : null;
    }

    public async Task<bool> DeleteBasketAsync(string customerId)
    {
        return await _database.KeyDeleteAsync(customerId);
    }
}

public class CustomerBasket
{
    public string BuyerId { get; set; } = string.Empty;
    public List<BasketItem> Items { get; set; } = new();
}

public class BasketItem
{
    public int ProductId { get; set; }
    public string ProductName { get; set; } = string.Empty;
    public decimal UnitPrice { get; set; }
    public int Quantity { get; set; }
}
```

### Feature 3: Ordering Service (DDD Pattern)

The Ordering service follows Domain-Driven Design with aggregates.

Create `Ordering.Domain/AggregatesModel/OrderAggregate/Order.cs`:

```csharp
namespace Ordering.Domain.AggregatesModel.OrderAggregate;

public class Order
{
    public int Id { get; private set; }
    public DateTime OrderDate { get; private set; }
    public string BuyerId { get; private set; }
    public Address Address { get; private set; }
    public OrderStatus OrderStatus { get; private set; }
    
    private readonly List<OrderItem> _orderItems = new();
    public IReadOnlyCollection<OrderItem> OrderItems => _orderItems.AsReadOnly();

    public Order(string buyerId, Address address)
    {
        BuyerId = buyerId;
        Address = address;
        OrderDate = DateTime.UtcNow;
        OrderStatus = OrderStatus.Submitted;
    }

    public void AddOrderItem(int productId, string productName, decimal unitPrice, int units)
    {
        var existingOrderItem = _orderItems.FirstOrDefault(o => o.ProductId == productId);

        if (existingOrderItem != null)
        {
            existingOrderItem.AddUnits(units);
        }
        else
        {
            _orderItems.Add(new OrderItem(productId, productName, unitPrice, units));
        }
    }

    public decimal GetTotal() => _orderItems.Sum(o => o.GetTotal());
}

public class OrderItem
{
    public int Id { get; private set; }
    public int ProductId { get; private set; }
    public string ProductName { get; private set; }
    public decimal UnitPrice { get; private set; }
    public int Units { get; private set; }

    public OrderItem(int productId, string productName, decimal unitPrice, int units)
    {
        ProductId = productId;
        ProductName = productName;
        UnitPrice = unitPrice;
        Units = units;
    }

    public void AddUnits(int units) => Units += units;
    public decimal GetTotal() => Units * UnitPrice;
}

public class Address
{
    public string Street { get; private set; }
    public string City { get; private set; }
    public string State { get; private set; }
    public string Country { get; private set; }
    public string ZipCode { get; private set; }

    public Address(string street, string city, string state, string country, string zipCode)
    {
        Street = street;
        City = city;
        State = state;
        Country = country;
        ZipCode = zipCode;
    }
}

public enum OrderStatus
{
    Submitted = 1,
    AwaitingValidation = 2,
    StockConfirmed = 3,
    Paid = 4,
    Shipped = 5,
    Cancelled = 6
}
```

### Feature 4: API Gateway (YARP)

```bash
# Create API Gateway
mkdir -p src/ApiGateways/Web.Gateway
cd src/ApiGateways/Web.Gateway
dotnet new web -n Web.Gateway
cd ../../..

dotnet sln add src/ApiGateways/Web.Gateway/Web.Gateway.csproj

# Add YARP
dotnet add src/ApiGateways/Web.Gateway/Web.Gateway.csproj package Yarp.ReverseProxy
```

Update `Web.Gateway/appsettings.json`:

```json
{
  "ReverseProxy": {
    "Routes": {
      "catalog-route": {
        "ClusterId": "catalog-cluster",
        "Match": {
          "Path": "/api/catalog/{**catch-all}"
        }
      },
      "basket-route": {
        "ClusterId": "basket-cluster",
        "Match": {
          "Path": "/api/basket/{**catch-all}"
        }
      },
      "ordering-route": {
        "ClusterId": "ordering-cluster",
        "Match": {
          "Path": "/api/ordering/{**catch-all}"
        }
      }
    },
    "Clusters": {
      "catalog-cluster": {
        "Destinations": {
          "destination1": {
            "Address": "http://catalog-api:8080"
          }
        }
      },
      "basket-cluster": {
        "Destinations": {
          "destination1": {
            "Address": "http://basket-api:8080"
          }
        }
      },
      "ordering-cluster": {
        "Destinations": {
          "destination1": {
            "Address": "http://ordering-api:8080"
          }
        }
      }
    }
  }
}
```

Update `Web.Gateway/Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddReverseProxy()
    .LoadFromConfig(builder.Configuration.GetSection("ReverseProxy"));

var app = builder.Build();

app.MapReverseProxy();

app.Run();
```

### Feature 5: Web Application (Blazor)

```bash
# Create Web App
mkdir -p src/WebApps/WebApp
cd src/WebApps/WebApp
dotnet new blazor -n WebApp
cd ../../..

dotnet sln add src/WebApps/WebApp/WebApp.csproj
```


---

## 4. Testing Strategy

### Unit Tests

Create unit test projects for each service:

```bash
# Create test projects
mkdir -p tests
cd tests

# Catalog unit tests
dotnet new xunit -n Catalog.UnitTests
cd ..
dotnet sln add tests/Catalog.UnitTests/Catalog.UnitTests.csproj

# Add references
dotnet add tests/Catalog.UnitTests/Catalog.UnitTests.csproj reference src/Services/Catalog/Catalog.Domain/Catalog.Domain.csproj
dotnet add tests/Catalog.UnitTests/Catalog.UnitTests.csproj reference src/Services/Catalog/Catalog.Application/Catalog.Application.csproj

# Add testing packages
dotnet add tests/Catalog.UnitTests/Catalog.UnitTests.csproj package Moq
dotnet add tests/Catalog.UnitTests/Catalog.UnitTests.csproj package FluentAssertions
dotnet add tests/Catalog.UnitTests/Catalog.UnitTests.csproj package Microsoft.EntityFrameworkCore.InMemory
```

Create `tests/Catalog.UnitTests/Domain/ProductTests.cs`:

```csharp
using Xunit;
using FluentAssertions;
using Catalog.Domain.Entities;

namespace Catalog.UnitTests.Domain;

public class ProductTests
{
    [Fact]
    public void Product_Should_Have_Valid_Properties()
    {
        // Arrange & Act
        var product = new Product
        {
            Id = 1,
            Name = "Test Product",
            Description = "Test Description",
            Price = 99.99m,
            AvailableStock = 10
        };

        // Assert
        product.Id.Should().Be(1);
        product.Name.Should().Be("Test Product");
        product.Price.Should().Be(99.99m);
        product.AvailableStock.Should().BeGreaterThan(0);
    }

    [Theory]
    [InlineData(10.00)]
    [InlineData(99.99)]
    [InlineData(1000.00)]
    public void Product_Price_Should_Be_Positive(decimal price)
    {
        // Arrange
        var product = new Product { Price = price };

        // Assert
        product.Price.Should().BeGreaterOrEqualTo(0);
    }
}
```

Create `tests/Catalog.UnitTests/Application/GetProductsQueryHandlerTests.cs`:

```csharp
using Xunit;
using FluentAssertions;
using Microsoft.EntityFrameworkCore;
using Catalog.Application.Queries;
using Catalog.Infrastructure.Data;
using Catalog.Domain.Entities;

namespace Catalog.UnitTests.Application;

public class GetProductsQueryHandlerTests
{
    [Fact]
    public async Task Handle_Should_Return_Paginated_Products()
    {
        // Arrange
        var options = new DbContextOptionsBuilder<CatalogContext>()
            .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString())
            .Options;

        await using var context = new CatalogContext(options);
        
        // Seed data
        context.Products.AddRange(
            new Product { Id = 1, Name = "Product 1", Price = 10.00m, AvailableStock = 5 },
            new Product { Id = 2, Name = "Product 2", Price = 20.00m, AvailableStock = 10 },
            new Product { Id = 3, Name = "Product 3", Price = 30.00m, AvailableStock = 15 }
        );
        await context.SaveChangesAsync();

        var handler = new GetProductsQueryHandler(context);
        var query = new GetProductsQuery(PageSize: 10, PageIndex: 0);

        // Act
        var result = await handler.Handle(query, CancellationToken.None);

        // Assert
        result.Should().NotBeNull();
        result.Count.Should().Be(3);
        result.Data.Should().HaveCount(3);
    }

    [Fact]
    public async Task Handle_Should_Return_Paginated_Results_With_Correct_Page()
    {
        // Arrange
        var options = new DbContextOptionsBuilder<CatalogContext>()
            .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString())
            .Options;

        await using var context = new CatalogContext(options);
        
        // Seed 15 products
        for (int i = 1; i <= 15; i++)
        {
            context.Products.Add(new Product 
            { 
                Id = i, 
                Name = $"Product {i}", 
                Price = i * 10.00m,
                AvailableStock = i 
            });
        }
        await context.SaveChangesAsync();

        var handler = new GetProductsQueryHandler(context);
        var query = new GetProductsQuery(PageSize: 5, PageIndex: 1);

        // Act
        var result = await handler.Handle(query, CancellationToken.None);

        // Assert
        result.Should().NotBeNull();
        result.Count.Should().Be(15);
        result.Data.Should().HaveCount(5);
        result.PageIndex.Should().Be(1);
        result.PageSize.Should().Be(5);
    }
}
```

### Integration Tests

```bash
# Create integration tests
cd tests
dotnet new xunit -n Catalog.IntegrationTests
cd ..
dotnet sln add tests/Catalog.IntegrationTests/Catalog.IntegrationTests.csproj

# Add packages
dotnet add tests/Catalog.IntegrationTests/Catalog.IntegrationTests.csproj package Microsoft.AspNetCore.Mvc.Testing
dotnet add tests/Catalog.IntegrationTests/Catalog.IntegrationTests.csproj package Testcontainers
dotnet add tests/Catalog.IntegrationTests/Catalog.IntegrationTests.csproj package Testcontainers.MsSql
```

Create `tests/Catalog.IntegrationTests/CatalogApiTests.cs`:

```csharp
using System.Net;
using System.Net.Http.Json;
using Microsoft.AspNetCore.Mvc.Testing;
using Xunit;
using FluentAssertions;

namespace Catalog.IntegrationTests;

public class CatalogApiTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public CatalogApiTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task GetItems_Should_Return_Success()
    {
        // Act
        var response = await _client.GetAsync("/api/catalog/items?pageSize=10&pageIndex=0");

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.OK);
    }

    [Fact]
    public async Task GetItems_Should_Return_Valid_Json()
    {
        // Act
        var response = await _client.GetAsync("/api/catalog/items?pageSize=10&pageIndex=0");
        var content = await response.Content.ReadAsStringAsync();

        // Assert
        response.IsSuccessStatusCode.Should().BeTrue();
        content.Should().NotBeNullOrEmpty();
    }
}
```

### Running Tests

```bash
# Run all tests
dotnet test

# Run with coverage
dotnet test /p:CollectCoverage=true /p:CoverageReportFormat=opencover

# Run specific test project
dotnet test tests/Catalog.UnitTests/Catalog.UnitTests.csproj

# Run tests with verbose output
dotnet test --logger "console;verbosity=detailed"

# Run tests and generate coverage report
dotnet test /p:CollectCoverage=true /p:CoverageReportFormat=cobertura
```

---

## 5. Containerization

### Docker Support

Create Dockerfiles for each service:

Create `src/Services/Catalog/Catalog.API/Dockerfile`:

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS base
WORKDIR /app
EXPOSE 8080
EXPOSE 8081

FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# Copy project files
COPY ["src/Services/Catalog/Catalog.API/Catalog.API.csproj", "Services/Catalog/Catalog.API/"]
COPY ["src/Services/Catalog/Catalog.Application/Catalog.Application.csproj", "Services/Catalog/Catalog.Application/"]
COPY ["src/Services/Catalog/Catalog.Domain/Catalog.Domain.csproj", "Services/Catalog/Catalog.Domain/"]
COPY ["src/Services/Catalog/Catalog.Infrastructure/Catalog.Infrastructure.csproj", "Services/Catalog/Catalog.Infrastructure/"]

# Restore dependencies
RUN dotnet restore "Services/Catalog/Catalog.API/Catalog.API.csproj"

# Copy all source files
COPY src/ .

# Build
WORKDIR "/src/Services/Catalog/Catalog.API"
RUN dotnet build "Catalog.API.csproj" -c $BUILD_CONFIGURATION -o /app/build

# Publish
FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "Catalog.API.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

# Final stage
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Catalog.API.dll"]
```

### Docker Compose

Create `docker-compose.yml` in the root:

```yaml
version: '3.8'

services:
  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=Your_password123
    ports:
      - "1433:1433"
    volumes:
      - sqlserver-data:/var/opt/mssql
    healthcheck:
      test: /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P "Your_password123" -Q "SELECT 1" || exit 1
      interval: 10s
      timeout: 3s
      retries: 10
      start_period: 10s

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

  catalog-api:
    build:
      context: .
      dockerfile: src/Services/Catalog/Catalog.API/Dockerfile
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=http://+:8080
      - ConnectionStrings__CatalogDB=Server=sqlserver;Database=CatalogDb;User Id=sa;Password=Your_password123;TrustServerCertificate=True
    ports:
      - "5101:8080"
    depends_on:
      sqlserver:
        condition: service_healthy

  basket-api:
    build:
      context: .
      dockerfile: src/Services/Basket/Basket.API/Dockerfile
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=http://+:8080
      - ConnectionStrings__Redis=redis:6379
    ports:
      - "5102:8080"
    depends_on:
      redis:
        condition: service_healthy

  ordering-api:
    build:
      context: .
      dockerfile: src/Services/Ordering/Ordering.API/Dockerfile
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=http://+:8080
      - ConnectionStrings__OrderingDB=Server=sqlserver;Database=OrderingDb;User Id=sa;Password=Your_password123;TrustServerCertificate=True
    ports:
      - "5103:8080"
    depends_on:
      sqlserver:
        condition: service_healthy

  gateway:
    build:
      context: .
      dockerfile: src/ApiGateways/Web.Gateway/Dockerfile
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=http://+:8080
    ports:
      - "5100:8080"
    depends_on:
      - catalog-api
      - basket-api
      - ordering-api

  webapp:
    build:
      context: .
      dockerfile: src/WebApps/WebApp/Dockerfile
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=http://+:8080
      - GatewayUrl=http://gateway:8080
    ports:
      - "5104:8080"
    depends_on:
      - gateway

volumes:
  sqlserver-data:
  redis-data:
```

Create `docker-compose.override.yml` for development:

```yaml
version: '3.8'

services:
  catalog-api:
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
    ports:
      - "5101:8080"

  basket-api:
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
    ports:
      - "5102:8080"
```

### Building and Running with Docker

```bash
# Build all services
docker-compose build

# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# View logs for specific service
docker-compose logs -f catalog-api

# Stop all services
docker-compose down

# Remove volumes (clean slate)
docker-compose down -v

# Scale a service
docker-compose up -d --scale catalog-api=3

# Rebuild specific service
docker-compose up -d --build catalog-api
```

### .dockerignore

Create `.dockerignore` in the root:

```
**/bin/
**/obj/
**/.vs/
**/.vscode/
**/node_modules/
**/.git/
**/.gitignore
**/.dockerignore
**/docker-compose*.yml
**/Dockerfile*
**/*.md
**/tests/
**/*.trx
**/.env
**/charts/
```

### Docker Best Practices

1. **Use multi-stage builds** to reduce image size
2. **Layer caching**: Order Dockerfile commands from least to most frequently changing
3. **Health checks**: Add health check endpoints

Add health checks to your APIs:

```csharp
// In Program.cs
builder.Services.AddHealthChecks()
    .AddDbContextCheck<CatalogContext>();

app.MapHealthChecks("/health");
app.MapHealthChecks("/ready");
```


---

## 6. CI/CD Pipeline

### GitHub Actions

Create `.github/workflows/build.yml`:

```yaml
name: Build and Test

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '9.0.x'
    
    - name: Restore dependencies
      run: dotnet restore
    
    - name: Build
      run: dotnet build --no-restore --configuration Release
    
    - name: Test
      run: dotnet test --no-build --configuration Release --verbosity normal --collect:"XPlat Code Coverage"
    
    - name: Upload coverage reports
      uses: codecov/codecov-action@v3
      with:
        files: '**/coverage.cobertura.xml'
        fail_ci_if_error: false

  docker-build:
    runs-on: ubuntu-latest
    needs: build

    steps:
    - uses: actions/checkout@v4
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
    
    - name: Build Catalog API
      run: |
        docker build -f src/Services/Catalog/Catalog.API/Dockerfile -t catalog-api:${{ github.sha }} .
    
    - name: Build Basket API
      run: |
        docker build -f src/Services/Basket/Basket.API/Dockerfile -t basket-api:${{ github.sha }} .
```

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to Production

on:
  push:
    branches: [ main ]
    tags:
      - 'v*'

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production

    steps:
    - uses: actions/checkout@v4
    
    - name: Login to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    
    - name: Build and push Docker images
      run: |
        docker-compose -f docker-compose.yml build
        docker-compose -f docker-compose.yml push
    
    - name: Deploy to Azure (Optional)
      uses: azure/webapps-deploy@v2
      with:
        app-name: 'ecommerce-app'
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
```

### Azure DevOps Pipeline

Create `azure-pipelines.yml`:

```yaml
trigger:
  branches:
    include:
    - main
    - develop

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  dotnetVersion: '9.0.x'

stages:
- stage: Build
  displayName: 'Build and Test'
  jobs:
  - job: Build
    steps:
    - task: UseDotNet@2
      displayName: 'Install .NET SDK'
      inputs:
        version: $(dotnetVersion)
    
    - task: DotNetCoreCLI@2
      displayName: 'Restore packages'
      inputs:
        command: 'restore'
        projects: '**/*.csproj'
    
    - task: DotNetCoreCLI@2
      displayName: 'Build solution'
      inputs:
        command: 'build'
        arguments: '--configuration $(buildConfiguration) --no-restore'
    
    - task: DotNetCoreCLI@2
      displayName: 'Run tests'
      inputs:
        command: 'test'
        arguments: '--configuration $(buildConfiguration) --no-build --collect:"XPlat Code Coverage"'
        publishTestResults: true
    
    - task: PublishCodeCoverageResults@1
      displayName: 'Publish code coverage'
      inputs:
        codeCoverageTool: 'Cobertura'
        summaryFileLocation: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'

- stage: Docker
  displayName: 'Build Docker Images'
  dependsOn: Build
  jobs:
  - job: BuildImages
    steps:
    - task: Docker@2
      displayName: 'Build Catalog API'
      inputs:
        command: 'build'
        Dockerfile: 'src/Services/Catalog/Catalog.API/Dockerfile'
        tags: '$(Build.BuildId)'
    
    - task: Docker@2
      displayName: 'Build Basket API'
      inputs:
        command: 'build'
        Dockerfile: 'src/Services/Basket/Basket.API/Dockerfile'
        tags: '$(Build.BuildId)'

- stage: Deploy
  displayName: 'Deploy to Azure'
  dependsOn: Docker
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployToProduction
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebAppContainer@1
            displayName: 'Deploy to Azure App Service'
            inputs:
              azureSubscription: 'Azure-Connection'
              appName: 'ecommerce-app'
              containers: 'myregistry.azurecr.io/catalog-api:$(Build.BuildId)'
```

---

## 7. Deployment Strategies

### Option 1: Azure Container Apps

```bash
# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Login
az login

# Create resource group
az group create --name ecommerce-rg --location eastus

# Create container app environment
az containerapp env create \
  --name ecommerce-env \
  --resource-group ecommerce-rg \
  --location eastus

# Create SQL Database
az sql server create \
  --name ecommerce-sql \
  --resource-group ecommerce-rg \
  --location eastus \
  --admin-user sqladmin \
  --admin-password 'YourPassword123!'

az sql db create \
  --resource-group ecommerce-rg \
  --server ecommerce-sql \
  --name CatalogDb \
  --service-objective S0

# Create Azure Container Registry
az acr create \
  --resource-group ecommerce-rg \
  --name ecommerceacr \
  --sku Basic

# Build and push images
az acr build \
  --registry ecommerceacr \
  --image catalog-api:latest \
  --file src/Services/Catalog/Catalog.API/Dockerfile .

# Deploy Catalog API
az containerapp create \
  --name catalog-api \
  --resource-group ecommerce-rg \
  --environment ecommerce-env \
  --image ecommerceacr.azurecr.io/catalog-api:latest \
  --target-port 8080 \
  --ingress external \
  --registry-server ecommerceacr.azurecr.io \
  --env-vars \
    "ConnectionStrings__CatalogDB=Server=ecommerce-sql.database.windows.net;Database=CatalogDb;User Id=sqladmin;Password=YourPassword123!;"
```

### Option 2: Azure Kubernetes Service (AKS)

Create `kubernetes/catalog-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: catalog-api
  labels:
    app: catalog-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: catalog-api
  template:
    metadata:
      labels:
        app: catalog-api
    spec:
      containers:
      - name: catalog-api
        image: myregistry.azurecr.io/catalog-api:latest
        ports:
        - containerPort: 8080
        env:
        - name: ASPNETCORE_ENVIRONMENT
          value: "Production"
        - name: ConnectionStrings__CatalogDB
          valueFrom:
            secretKeyRef:
              name: catalog-secrets
              key: connection-string
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: catalog-api
spec:
  selector:
    app: catalog-api
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
```

Deploy to AKS:

```bash
# Create AKS cluster
az aks create \
  --resource-group ecommerce-rg \
  --name ecommerce-aks \
  --node-count 3 \
  --enable-addons monitoring \
  --generate-ssh-keys

# Get credentials
az aks get-credentials --resource-group ecommerce-rg --name ecommerce-aks

# Create secrets
kubectl create secret generic catalog-secrets \
  --from-literal=connection-string='Server=...'

# Apply deployments
kubectl apply -f kubernetes/catalog-deployment.yaml

# Check status
kubectl get pods
kubectl get services

# Scale deployment
kubectl scale deployment catalog-api --replicas=5
```

### Option 3: Docker Swarm

```bash
# Initialize swarm
docker swarm init

# Deploy stack
docker stack deploy -c docker-compose.yml ecommerce

# List services
docker service ls

# Scale service
docker service scale ecommerce_catalog-api=3

# View logs
docker service logs ecommerce_catalog-api

# Remove stack
docker stack rm ecommerce
```

### Deployment Checklist

Before deploying to production:

- [ ] Configure environment variables for production
- [ ] Set up database backups
- [ ] Configure SSL/TLS certificates
- [ ] Set up monitoring (Application Insights, Prometheus)
- [ ] Configure logging (ELK Stack, Azure Monitor)
- [ ] Set up alerts and notifications
- [ ] Implement rate limiting
- [ ] Configure CORS policies
- [ ] Set up CDN for static assets
- [ ] Enable auto-scaling
- [ ] Configure health checks
- [ ] Set up disaster recovery plan
- [ ] Perform security audit
- [ ] Load testing
- [ ] Documentation review


---

## 8. Best Practices and Patterns

### Architecture Patterns

#### 1. Clean Architecture

Organize your code in layers with clear dependencies:

- **Domain Layer**: Contains business logic and entities
  - No dependencies on other layers
  - Contains domain events, aggregates, value objects
  
- **Application Layer**: Contains use cases and business rules
  - Depends only on Domain layer
  - Contains commands, queries, validators
  
- **Infrastructure Layer**: Contains data access and external services
  - Depends on Application and Domain layers
  - Contains DbContext, repositories, external API clients
  
- **Presentation Layer**: Contains UI and API controllers
  - Depends on Application layer
  - Contains controllers, views, DTOs

#### 2. CQRS (Command Query Responsibility Segregation)

Separate read and write operations:

```csharp
// Command (Write)
public record CreateProductCommand(string Name, decimal Price) : IRequest<int>;

// Query (Read)
public record GetProductQuery(int Id) : IRequest<ProductDto>;

// Command Handler
public class CreateProductCommandHandler : IRequestHandler<CreateProductCommand, int>
{
    private readonly CatalogContext _context;

    public CreateProductCommandHandler(CatalogContext context)
    {
        _context = context;
    }

    public async Task<int> Handle(CreateProductCommand request, CancellationToken cancellationToken)
    {
        var product = new Product
        {
            Name = request.Name,
            Price = request.Price
        };

        _context.Products.Add(product);
        await _context.SaveChangesAsync(cancellationToken);

        return product.Id;
    }
}
```

#### 3. Event-Driven Architecture (Optional)

Use domain events for loose coupling:

```csharp
public abstract class DomainEvent
{
    public Guid Id { get; } = Guid.NewGuid();
    public DateTime OccurredOn { get; } = DateTime.UtcNow;
}

public class OrderCreatedEvent : DomainEvent
{
    public int OrderId { get; set; }
    public string BuyerId { get; set; }
    public decimal TotalAmount { get; set; }
}

public class OrderCreatedEventHandler : INotificationHandler<OrderCreatedEvent>
{
    public async Task Handle(OrderCreatedEvent notification, CancellationToken cancellationToken)
    {
        // Send email notification
        // Update inventory
        // Publish to message bus
    }
}
```

### Design Patterns

#### Repository Pattern

```csharp
public interface IRepository<T> where T : class
{
    Task<T?> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<T> AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}

public class Repository<T> : IRepository<T> where T : class
{
    protected readonly DbContext _context;
    protected readonly DbSet<T> _dbSet;

    public Repository(DbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }

    public virtual async Task<T?> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }

    public virtual async Task<IEnumerable<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }

    public virtual async Task<T> AddAsync(T entity)
    {
        await _dbSet.AddAsync(entity);
        await _context.SaveChangesAsync();
        return entity;
    }

    public virtual async Task UpdateAsync(T entity)
    {
        _dbSet.Update(entity);
        await _context.SaveChangesAsync();
    }

    public virtual async Task DeleteAsync(int id)
    {
        var entity = await GetByIdAsync(id);
        if (entity != null)
        {
            _dbSet.Remove(entity);
            await _context.SaveChangesAsync();
        }
    }
}
```

#### Unit of Work Pattern

```csharp
public interface IUnitOfWork : IDisposable
{
    IRepository<Product> Products { get; }
    IRepository<Order> Orders { get; }
    Task<int> CommitAsync();
    Task RollbackAsync();
}

public class UnitOfWork : IUnitOfWork
{
    private readonly DbContext _context;
    
    public IRepository<Product> Products { get; }
    public IRepository<Order> Orders { get; }

    public UnitOfWork(DbContext context)
    {
        _context = context;
        Products = new Repository<Product>(context);
        Orders = new Repository<Order>(context);
    }

    public async Task<int> CommitAsync()
    {
        return await _context.SaveChangesAsync();
    }

    public async Task RollbackAsync()
    {
        await Task.Run(() =>
        {
            foreach (var entry in _context.ChangeTracker.Entries())
            {
                entry.State = EntityState.Detached;
            }
        });
    }

    public void Dispose()
    {
        _context.Dispose();
    }
}
```

### Security Best Practices

#### 1. Authentication with JWT

```csharp
// Add to Program.cs
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]!))
        };
    });

app.UseAuthentication();
app.UseAuthorization();
```

#### 2. API Rate Limiting

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(context =>
        RateLimitPartition.GetFixedWindowLimiter(
            partitionKey: context.User.Identity?.Name ?? context.Request.Headers.Host.ToString(),
            factory: partition => new FixedWindowRateLimiterOptions
            {
                AutoReplenishment = true,
                PermitLimit = 100,
                QueueLimit = 0,
                Window = TimeSpan.FromMinutes(1)
            }));
});

app.UseRateLimiter();
```

#### 3. Input Validation with FluentValidation

```csharp
public class CreateProductValidator : AbstractValidator<CreateProductCommand>
{
    public CreateProductValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Product name is required")
            .MaximumLength(100).WithMessage("Product name must not exceed 100 characters");
        
        RuleFor(x => x.Price)
            .GreaterThan(0).WithMessage("Price must be greater than 0")
            .LessThan(1000000).WithMessage("Price must be less than 1,000,000");
    }
}

// Register validators
builder.Services.AddValidatorsFromAssemblyContaining<CreateProductValidator>();
```

### Performance Optimization

#### 1. Response Caching

```csharp
builder.Services.AddResponseCaching();

app.UseResponseCaching();

// In controller
[HttpGet]
[ResponseCache(Duration = 60, Location = ResponseCacheLocation.Any)]
public async Task<IActionResult> GetProducts()
{
    var products = await _mediator.Send(new GetProductsQuery(10, 0));
    return Ok(products);
}
```

#### 2. Redis Distributed Cache

```csharp
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("Redis");
    options.InstanceName = "EcommerceCache:";
});

// Usage in service
public class ProductService
{
    private readonly IDistributedCache _cache;
    private readonly IRepository<Product> _repository;

    public ProductService(IDistributedCache cache, IRepository<Product> repository)
    {
        _cache = cache;
        _repository = repository;
    }

    public async Task<Product?> GetProductAsync(int id)
    {
        var cacheKey = $"product_{id}";
        var cached = await _cache.GetStringAsync(cacheKey);
        
        if (cached != null)
            return JsonSerializer.Deserialize<Product>(cached);
        
        var product = await _repository.GetByIdAsync(id);
        
        if (product != null)
        {
            await _cache.SetStringAsync(
                cacheKey, 
                JsonSerializer.Serialize(product),
                new DistributedCacheEntryOptions 
                { 
                    AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5) 
                });
        }
        
        return product;
    }
}
```

#### 3. Database Indexing

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Single column index
    modelBuilder.Entity<Product>()
        .HasIndex(p => p.Name)
        .HasDatabaseName("IX_Product_Name");
    
    // Composite index
    modelBuilder.Entity<Product>()
        .HasIndex(p => new { p.CatalogTypeId, p.CatalogBrandId })
        .HasDatabaseName("IX_Product_Type_Brand");
    
    // Unique index
    modelBuilder.Entity<Product>()
        .HasIndex(p => p.Sku)
        .IsUnique()
        .HasDatabaseName("IX_Product_Sku");
}
```

#### 4. Async/Await Best Practices

```csharp
// Good - Use ConfigureAwait(false) in library code
public async Task<Product> GetProductAsync(int id)
{
    return await _repository.GetByIdAsync(id).ConfigureAwait(false);
}

// Good - Avoid async void (except for event handlers)
public async Task ProcessOrderAsync(Order order)
{
    await _orderService.ProcessAsync(order);
}

// Good - Use Task.WhenAll for parallel operations
public async Task<IEnumerable<Product>> GetMultipleProductsAsync(List<int> ids)
{
    var tasks = ids.Select(id => _repository.GetByIdAsync(id));
    return await Task.WhenAll(tasks);
}
```

### Monitoring and Observability

#### Application Insights

```csharp
builder.Services.AddApplicationInsightsTelemetry(options =>
{
    options.ConnectionString = builder.Configuration["ApplicationInsights:ConnectionString"];
});

// Custom telemetry
public class OrderService
{
    private readonly TelemetryClient _telemetry;

    public OrderService(TelemetryClient telemetry)
    {
        _telemetry = telemetry;
    }

    public async Task CreateOrderAsync(Order order)
    {
        using var operation = _telemetry.StartOperation<RequestTelemetry>("CreateOrder");
        try
        {
            await _repository.AddAsync(order);
            _telemetry.TrackEvent("OrderCreated", new Dictionary<string, string>
            {
                { "OrderId", order.Id.ToString() },
                { "TotalAmount", order.GetTotal().ToString() }
            });
        }
        catch (Exception ex)
        {
            _telemetry.TrackException(ex);
            throw;
        }
    }
}
```

#### Health Checks

```csharp
builder.Services.AddHealthChecks()
    .AddDbContextCheck<CatalogContext>("Database")
    .AddRedis(builder.Configuration.GetConnectionString("Redis")!, "Redis")
    .AddUrlGroup(new Uri("https://external-api.com/health"), "External API")
    .AddCheck<CustomHealthCheck>("Custom Health Check");

app.MapHealthChecks("/health", new HealthCheckOptions
{
    ResponseWriter = UIResponseWriter.WriteHealthCheckUIResponse
});

app.MapHealthChecks("/health/ready", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("ready"),
});

app.MapHealthChecks("/health/live", new HealthCheckOptions
{
    Predicate = _ => false
});
```

#### Structured Logging with Serilog

```csharp
builder.Host.UseSerilog((context, configuration) =>
{
    configuration
        .ReadFrom.Configuration(context.Configuration)
        .Enrich.FromLogContext()
        .Enrich.WithProperty("ApplicationName", "EcommerceApp")
        .WriteTo.Console(outputTemplate: "[{Timestamp:HH:mm:ss} {Level:u3}] {Message:lj}{NewLine}{Exception}")
        .WriteTo.File("logs/log-.txt", rollingInterval: RollingInterval.Day)
        .WriteTo.ApplicationInsights(
            context.Configuration["ApplicationInsights:ConnectionString"]!, 
            TelemetryConverter.Traces);
});

// Usage in code
public class ProductService
{
    private readonly ILogger<ProductService> _logger;

    public ProductService(ILogger<ProductService> logger)
    {
        _logger = logger;
    }

    public async Task<Product> CreateProductAsync(CreateProductCommand command)
    {
        _logger.LogInformation("Creating product {ProductName} with price {Price}", 
            command.Name, command.Price);
        
        try
        {
            var product = await _repository.AddAsync(new Product 
            { 
                Name = command.Name, 
                Price = command.Price 
            });
            
            _logger.LogInformation("Product {ProductId} created successfully", product.Id);
            return product;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error creating product {ProductName}", command.Name);
            throw;
        }
    }
}
```

### Resilience Patterns

#### Circuit Breaker with Polly

```csharp
builder.Services.AddHttpClient<ICatalogService, CatalogService>(client =>
{
    client.BaseAddress = new Uri(builder.Configuration["Services:Catalog"]!);
})
.AddTransientHttpErrorPolicy(policy => 
    policy.WaitAndRetryAsync(3, retryAttempt => 
        TimeSpan.FromSeconds(Math.Pow(2, retryAttempt))))
.AddTransientHttpErrorPolicy(policy => 
    policy.CircuitBreakerAsync(5, TimeSpan.FromSeconds(30)));

// Or use the newer resilience pipelines
builder.Services.AddHttpClient<ICatalogService, CatalogService>()
    .AddStandardResilienceHandler(options =>
    {
        options.Retry.MaxRetryAttempts = 3;
        options.CircuitBreaker.SamplingDuration = TimeSpan.FromSeconds(10);
        options.Timeout.Timeout = TimeSpan.FromSeconds(30);
    });
```

---

## Next Steps and Learning Path

### Phase 1: Foundation (Week 1-2)
- [ ] Set up development environment
- [ ] Create solution structure
- [ ] Implement Catalog service with basic CRUD
- [ ] Add unit tests for domain entities
- [ ] Set up Docker Compose for local development

### Phase 2: Core Services (Week 3-4)
- [ ] Implement Basket service with Redis
- [ ] Implement Ordering service with DDD patterns
- [ ] Add API Gateway with YARP
- [ ] Implement authentication with Identity
- [ ] Add comprehensive unit and integration tests

### Phase 3: Advanced Features (Week 5-6)
- [ ] Implement event-driven communication (RabbitMQ/Azure Service Bus)
- [ ] Add payment processing integration
- [ ] Build web frontend (Blazor/React)
- [ ] Implement background jobs (Hangfire/Quartz)
- [ ] Add real-time notifications (SignalR)

### Phase 4: Production Ready (Week 7-8)
- [ ] Set up CI/CD pipeline
- [ ] Configure monitoring and logging
- [ ] Implement distributed tracing
- [ ] Performance optimization and caching
- [ ] Security hardening and penetration testing
- [ ] Complete documentation

### Phase 5: Deployment (Week 9-10)
- [ ] Containerize all services
- [ ] Deploy to cloud platform (Azure/AWS)
- [ ] Configure auto-scaling
- [ ] Set up disaster recovery
- [ ] Load testing and optimization
- [ ] Go-live checklist

---

## Additional Resources

### Official Documentation
- [.NET Documentation](https://docs.microsoft.com/dotnet/)
- [ASP.NET Core Documentation](https://docs.microsoft.com/aspnet/core/)
- [Entity Framework Core](https://docs.microsoft.com/ef/core/)
- [Docker Documentation](https://docs.docker.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Azure Documentation](https://docs.microsoft.com/azure/)

### Reference Applications
- [eShop Reference Application](https://github.com/dotnet/eShop) - Official .NET 9 microservices reference app
- [Clean Architecture Solution Template](https://github.com/jasontaylordev/CleanArchitecture) - Clean Architecture with ASP.NET Core
- [Practical Microservices](https://github.com/dotnet-architecture/eShopOnContainers) - Comprehensive microservices example

### Learning Resources
- [Microsoft Learn - ASP.NET Core](https://learn.microsoft.com/training/paths/aspnet-core-web-app/)
- [Microservices Architecture](https://learn.microsoft.com/azure/architecture/guide/architecture-styles/microservices)
- [Domain-Driven Design](https://www.domainlanguage.com/ddd/)
- [Cloud Design Patterns](https://learn.microsoft.com/azure/architecture/patterns/)

### Books
- "Domain-Driven Design" by Eric Evans
- "Clean Architecture" by Robert C. Martin
- "Microservices Patterns" by Chris Richardson
- "Building Microservices" by Sam Newman
- ".NET Microservices: Architecture for Containerized .NET Applications" (Free eBook from Microsoft)

### Community and Support
- [.NET Foundation](https://dotnetfoundation.org/)
- [ASP.NET Community Standup](https://dotnet.microsoft.com/live/community-standup)
- [Stack Overflow - .NET Tag](https://stackoverflow.com/questions/tagged/.net)
- [Reddit - r/dotnet](https://reddit.com/r/dotnet)
- [.NET Discord Server](https://aka.ms/dotnet-discord)

### Tools and Extensions
- **Visual Studio Extensions**
  - ReSharper
  - CodeMaid
  - Productivity Power Tools
  
- **VS Code Extensions**
  - C# Dev Kit
  - .NET Core Test Explorer
  - Docker
  - REST Client

- **Development Tools**
  - Postman / Insomnia (API Testing)
  - Azure Data Studio (Database Management)
  - Redis Commander (Redis Management)
  - k9s (Kubernetes Management)

---

## Conclusion

This guide provides a comprehensive roadmap for building a production-ready e-commerce application using .NET 9 and modern software engineering practices. The architecture and patterns demonstrated here are based on real-world, battle-tested solutions used in enterprise applications.

### Key Takeaways

1. **Start Simple, Scale Gradually**: Begin with a monolith if appropriate, then extract microservices as needed
2. **Test-Driven Development**: Write tests early and maintain high coverage
3. **Clean Architecture**: Maintain clear separation of concerns and dependency rules
4. **Security First**: Implement security measures from day one, not as an afterthought
5. **Monitor Everything**: Use comprehensive logging, metrics, and tracing
6. **Automate**: CI/CD pipelines and infrastructure as code save time and reduce errors
7. **Document**: Keep documentation up-to-date and accessible
8. **Learn Continuously**: Technology evolves, stay current with best practices

### Getting Help

If you encounter issues or need clarification:

1. **Check the official documentation** - Most questions are answered there
2. **Review the eShop reference app** - See how Microsoft implements these patterns
3. **Ask the community** - Stack Overflow, Reddit, Discord are great resources
4. **GitHub Issues** - Many projects have active issue trackers with helpful maintainers

### Contributing Back

As you learn and build:

- Share your experiences through blog posts or talks
- Contribute to open-source projects
- Help others in the community
- Report bugs and suggest improvements

---

**Good luck with your e-commerce project!** 🚀

Remember: Every expert was once a beginner. Take it one step at a time, learn from mistakes, and keep building!

---

*Last Updated: November 2024*  
*Version: 1.0*  
*Compatible with: .NET 9.0*
