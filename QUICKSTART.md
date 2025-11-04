# Quick Start Guide

This quick start guide helps you get up and running with building your e-commerce application in 30 minutes.

## Prerequisites Check

Before starting, ensure you have:

```bash
# Check .NET version (should be 9.0 or higher)
dotnet --version

# Check Docker
docker --version

# Check Git
git --version
```

If any are missing, see [Environment Setup](GUIDE.md#1-environment-setup) in the main guide.

## 30-Minute Quick Start

### Step 1: Create Solution (5 minutes)

```bash
# Create workspace
mkdir my-ecommerce-app
cd my-ecommerce-app

# Create solution
dotnet new sln -n EcommerceApp

# Create directory structure
mkdir -p src/Services/Catalog/{Catalog.API,Catalog.Domain,Catalog.Application,Catalog.Infrastructure}
mkdir -p tests
```

### Step 2: Create Catalog Service (10 minutes)

```bash
# Create projects
cd src/Services/Catalog/Catalog.Domain
dotnet new classlib -n Catalog.Domain
cd ../Catalog.Application
dotnet new classlib -n Catalog.Application
cd ../Catalog.Infrastructure
dotnet new classlib -n Catalog.Infrastructure
cd ../Catalog.API
dotnet new webapi -n Catalog.API
cd ../../../../..

# Add to solution
dotnet sln add src/Services/Catalog/Catalog.Domain/Catalog.Domain.csproj
dotnet sln add src/Services/Catalog/Catalog.Application/Catalog.Application.csproj
dotnet sln add src/Services/Catalog/Catalog.Infrastructure/Catalog.Infrastructure.csproj
dotnet sln add src/Services/Catalog/Catalog.API/Catalog.API.csproj

# Add project references
dotnet add src/Services/Catalog/Catalog.Application/Catalog.Application.csproj reference src/Services/Catalog/Catalog.Domain/Catalog.Domain.csproj
dotnet add src/Services/Catalog/Catalog.Infrastructure/Catalog.Infrastructure.csproj reference src/Services/Catalog/Catalog.Application/Catalog.Application.csproj
dotnet add src/Services/Catalog/Catalog.API/Catalog.API.csproj reference src/Services/Catalog/Catalog.Infrastructure/Catalog.Infrastructure.csproj
dotnet add src/Services/Catalog/Catalog.API/Catalog.API.csproj reference src/Services/Catalog/Catalog.Application/Catalog.Application.csproj

# Add NuGet packages
dotnet add src/Services/Catalog/Catalog.API/Catalog.API.csproj package Microsoft.EntityFrameworkCore.Design
dotnet add src/Services/Catalog/Catalog.Infrastructure/Catalog.Infrastructure.csproj package Microsoft.EntityFrameworkCore.SqlServer
dotnet add src/Services/Catalog/Catalog.Application/Catalog.Application.csproj package MediatR
```

### Step 3: Add Docker Support (5 minutes)

Create `docker-compose.yml` in the root directory:

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

  catalog-api:
    build:
      context: .
      dockerfile: src/Services/Catalog/Catalog.API/Dockerfile
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__CatalogDB=Server=sqlserver;Database=CatalogDb;User Id=sa;Password=Your_password123;TrustServerCertificate=True
    ports:
      - "5101:8080"
    depends_on:
      - sqlserver
```

### Step 4: Build and Run (5 minutes)

```bash
# Build the solution
dotnet build

# Run tests (if any exist)
dotnet test

# Start with Docker
docker-compose up -d

# View logs
docker-compose logs -f catalog-api
```

### Step 5: Test the API (5 minutes)

```bash
# Health check
curl http://localhost:5101/health

# Swagger UI
# Open in browser: http://localhost:5101/swagger
```

## Next Steps

Now that you have a basic setup:

1. **Add Domain Entities** - See [Core Features Implementation](GUIDE.md#3-core-features-implementation)
2. **Implement CQRS** - Add commands and queries with MediatR
3. **Add Tests** - Create unit and integration tests
4. **Add More Services** - Implement Basket, Ordering services
5. **Setup CI/CD** - Configure GitHub Actions

## Common Issues

### Docker not starting
```bash
# Check Docker is running
docker ps

# Restart Docker Desktop
# On Windows: Right-click Docker Desktop icon > Restart
# On Linux: sudo systemctl restart docker
```

### Port already in use
```bash
# Check what's using the port
# Windows: netstat -ano | findstr :5101
# Linux/Mac: lsof -i :5101

# Change port in docker-compose.yml
ports:
  - "5102:8080"  # Use different port
```

### Database connection issues
```bash
# Wait for SQL Server to be ready (takes ~30 seconds first time)
docker-compose logs sqlserver

# Test connection
docker exec -it <container-id> /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P Your_password123
```

## Learning Path

Follow this order for the best learning experience:

1. ✅ Quick Start (You are here!)
2. 📖 [Complete Guide](GUIDE.md) - Read sections 1-2
3. 💻 Implement Catalog Service - Section 3.1
4. 🧪 Add Tests - Section 4
5. 🐳 Containerize - Section 5
6. 🔄 Setup CI/CD - Section 6
7. ☁️ Deploy - Section 7
8. 🎯 Best Practices - Section 8

## Getting Help

- **Full Documentation**: [GUIDE.md](GUIDE.md)
- **Issues**: Check the common issues section above
- **Reference App**: [dotnet/eShop](https://github.com/dotnet/eShop)
- **Community**: [Stack Overflow - .NET](https://stackoverflow.com/questions/tagged/.net)

---

**Ready to dive deeper?** Continue with the [complete guide](GUIDE.md)! 🚀
