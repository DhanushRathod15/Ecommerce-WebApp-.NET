# Troubleshooting Guide

Common issues and solutions when building your e-commerce application.

## Table of Contents
- [Environment Setup Issues](#environment-setup-issues)
- [Build and Compilation Issues](#build-and-compilation-issues)
- [Docker Issues](#docker-issues)
- [Database Issues](#database-issues)
- [Runtime Issues](#runtime-issues)
- [Testing Issues](#testing-issues)
- [Deployment Issues](#deployment-issues)

---

## Environment Setup Issues

### .NET SDK Not Found

**Problem**: `dotnet: command not found` or `The specified SDK version could not be found`

**Solutions**:
```bash
# Verify .NET installation
dotnet --version

# List installed SDKs
dotnet --list-sdks

# Install .NET 9 SDK
# Windows: Download from https://dotnet.microsoft.com/download
# Linux: 
wget https://dot.net/v1/dotnet-install.sh
chmod +x dotnet-install.sh
./dotnet-install.sh --version latest

# macOS:
brew install dotnet
```

### Docker Desktop Not Running

**Problem**: `Cannot connect to the Docker daemon`

**Solutions**:
```bash
# Check Docker status
docker ps

# Windows/Mac: Start Docker Desktop from Start Menu/Applications
# Linux:
sudo systemctl start docker
sudo systemctl enable docker

# Verify Docker is running
docker run hello-world
```

### IDE Issues

**Problem**: Visual Studio or VS Code not recognizing .NET projects

**Solutions**:
- **Visual Studio**: Install ".NET desktop development" and "ASP.NET and web development" workloads
- **VS Code**: Install C# Dev Kit extension
- Restart IDE after installation
- Run `dotnet restore` in project directory

---

## Build and Compilation Issues

### Restore Failed

**Problem**: `Error NU1101: Unable to find package`

**Solutions**:
```bash
# Clear NuGet cache
dotnet nuget locals all --clear

# Restore with verbose output
dotnet restore --verbosity detailed

# Check NuGet sources
dotnet nuget list source

# Add official NuGet source if missing
dotnet nuget add source https://api.nuget.org/v3/index.json -n nuget.org
```

### Build Failed - Missing References

**Problem**: `The type or namespace name 'X' could not be found`

**Solutions**:
```bash
# Verify project references
dotnet list reference

# Add missing reference
dotnet add reference ../ProjectName/ProjectName.csproj

# Check for circular dependencies
# Remove and re-add references if needed

# Clean and rebuild
dotnet clean
dotnet build
```

### Version Conflicts

**Problem**: `Package X Y.Z is not compatible with netX.0`

**Solutions**:
```bash
# Check target framework in .csproj
<TargetFramework>net9.0</TargetFramework>

# Update package to compatible version
dotnet add package PackageName --version X.Y.Z

# List outdated packages
dotnet list package --outdated

# Update all packages
dotnet list package --outdated | ForEach-Object { dotnet add package $_ }
```

---

## Docker Issues

### Container Build Failed

**Problem**: `ERROR [internal] load metadata for mcr.microsoft.com/dotnet/sdk:9.0`

**Solutions**:
```bash
# Check internet connection and Docker Hub access
docker pull mcr.microsoft.com/dotnet/sdk:9.0

# Use Docker with proxy (if behind corporate firewall)
# Add to ~/.docker/config.json:
{
  "proxies": {
    "default": {
      "httpProxy": "http://proxy.example.com:8080",
      "httpsProxy": "http://proxy.example.com:8080"
    }
  }
}

# Restart Docker Desktop
```

### Port Already in Use

**Problem**: `Bind for 0.0.0.0:5101 failed: port is already allocated`

**Solutions**:
```bash
# Windows - Find process using port
netstat -ano | findstr :5101
taskkill /PID <process_id> /F

# Linux/Mac - Find and kill process
lsof -i :5101
kill -9 <PID>

# Or change port in docker-compose.yml
ports:
  - "5102:8080"  # Use different external port
```

### Container Won't Start

**Problem**: Container exits immediately or shows unhealthy status

**Solutions**:
```bash
# Check logs
docker-compose logs -f service-name

# Inspect container
docker inspect container-id

# Check health status
docker ps -a

# Common fixes:
# 1. Check environment variables
# 2. Verify connection strings
# 3. Ensure dependencies are ready (databases, etc.)
# 4. Check file permissions
```

### Out of Disk Space

**Problem**: `no space left on device`

**Solutions**:
```bash
# Clean up Docker
docker system prune -a --volumes

# Remove unused images
docker image prune -a

# Remove unused volumes
docker volume prune

# Check disk usage
docker system df
```

---

## Database Issues

### Cannot Connect to SQL Server

**Problem**: `A network-related or instance-specific error occurred`

**Solutions**:
```bash
# Wait for SQL Server to fully start (takes 30-60 seconds first time)
docker-compose logs sqlserver

# Test connection
docker exec -it <container-name> /opt/mssql-tools/bin/sqlcmd \
  -S localhost -U sa -P 'Your_password123'

# Check connection string
"Server=localhost;Database=CatalogDb;User Id=sa;Password=Your_password123;TrustServerCertificate=True"

# Common issues:
# 1. Password doesn't meet SQL Server requirements (must be complex)
# 2. Container not fully started
# 3. Firewall blocking port 1433
```

### Migration Failed

**Problem**: `Unable to create an object of type 'CatalogContext'`

**Solutions**:
```bash
# Ensure design-time tools are installed
dotnet add package Microsoft.EntityFrameworkCore.Design

# Set startup project
dotnet ef migrations add InitialCreate --project src/Infrastructure --startup-project src/API

# Specify context explicitly
dotnet ef migrations add InitialCreate --context CatalogContext

# Connection string for migrations
dotnet ef database update --connection "Server=localhost;Database=CatalogDb;User Id=sa;Password=Your_password123;TrustServerCertificate=True"
```

### Database Seeding Issues

**Problem**: Data not appearing in database

**Solutions**:
```csharp
// Ensure DbInitializer is called in Program.cs
using (var scope = app.Services.CreateScope())
{
    var context = scope.ServiceProvider.GetRequiredService<CatalogContext>();
    await context.Database.MigrateAsync();
    
    if (!context.Products.Any())
    {
        // Seed data
    }
}
```

---

## Runtime Issues

### 500 Internal Server Error

**Problem**: API returns 500 error with no details

**Solutions**:
```bash
# Enable developer exception page
# In Program.cs
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}

# Check logs
docker-compose logs -f api-service

# Enable detailed errors in appsettings.Development.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft.AspNetCore": "Warning"
    }
  }
}
```

### Swagger Not Loading

**Problem**: `/swagger` returns 404

**Solutions**:
```csharp
// Ensure Swagger is configured in Program.cs
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

app.UseSwagger();
app.UseSwaggerUI();

// Check if running in Development
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
```

### CORS Issues

**Problem**: `No 'Access-Control-Allow-Origin' header is present`

**Solutions**:
```csharp
// Add CORS policy in Program.cs
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAll",
        builder => builder
            .AllowAnyOrigin()
            .AllowAnyMethod()
            .AllowAnyHeader());
});

app.UseCors("AllowAll");

// For production, specify allowed origins
builder.Services.AddCors(options =>
{
    options.AddPolicy("Production",
        builder => builder
            .WithOrigins("https://yourdomain.com")
            .AllowAnyMethod()
            .AllowAnyHeader());
});
```

### Service Not Resolving Dependencies

**Problem**: `Unable to resolve service for type 'X'`

**Solutions**:
```csharp
// Register service in Program.cs
builder.Services.AddScoped<IProductService, ProductService>();

// For generic repositories
builder.Services.AddScoped(typeof(IRepository<>), typeof(Repository<>));

// Verify registration order (dependencies before dependents)
```

---

## Testing Issues

### Tests Not Discovering

**Problem**: Test Explorer shows no tests

**Solutions**:
```bash
# Rebuild test project
dotnet build tests/ProjectName.Tests

# Run tests from command line
dotnet test

# Check test framework is installed
# In .csproj:
<ItemGroup>
    <PackageReference Include="xunit" Version="2.6.1" />
    <PackageReference Include="xunit.runner.visualstudio" Version="2.5.3" />
</ItemGroup>
```

### Integration Tests Failing

**Problem**: Database or services not available during tests

**Solutions**:
```csharp
// Use WebApplicationFactory
public class TestFixture : WebApplicationFactory<Program>
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            // Replace DbContext with in-memory database
            services.RemoveAll<DbContextOptions<CatalogContext>>();
            services.AddDbContext<CatalogContext>(options =>
            {
                options.UseInMemoryDatabase("TestDb");
            });
        });
    }
}

// Or use Testcontainers for real databases
[CollectionDefinition("Database collection")]
public class DatabaseCollection : ICollectionFixture<DatabaseFixture>
{
}
```

### Mock Setup Issues

**Problem**: Mock not returning expected values

**Solutions**:
```csharp
// Ensure proper mock setup with Moq
var mockRepository = new Mock<IRepository<Product>>();
mockRepository
    .Setup(repo => repo.GetByIdAsync(It.IsAny<int>()))
    .ReturnsAsync(new Product { Id = 1, Name = "Test" });

// Verify mock was called
mockRepository.Verify(
    repo => repo.GetByIdAsync(It.IsAny<int>()), 
    Times.Once);

// Check for common issues:
// 1. Setup doesn't match actual call (parameter mismatch)
// 2. Missing .Object when passing mock to class
// 3. Forgetting to await async methods
```

---

## Deployment Issues

### Azure Deployment Failed

**Problem**: `Unable to deploy container to Azure Container Apps`

**Solutions**:
```bash
# Verify Azure CLI login
az login
az account show

# Check resource group exists
az group list --output table

# Verify container registry
az acr list --output table

# Check container image was pushed
az acr repository list --name <registry-name>

# View deployment logs
az containerapp logs show --name <app-name> --resource-group <rg-name>
```

### Kubernetes Pod Not Starting

**Problem**: Pod status is `CrashLoopBackOff` or `ImagePullBackOff`

**Solutions**:
```bash
# Check pod status
kubectl get pods
kubectl describe pod <pod-name>

# View logs
kubectl logs <pod-name>
kubectl logs <pod-name> --previous  # For crashed pods

# Common fixes:
# 1. ImagePullBackOff - Check image name and registry credentials
kubectl create secret docker-registry regcred \
  --docker-server=<your-registry> \
  --docker-username=<username> \
  --docker-password=<password>

# 2. CrashLoopBackOff - Check application logs and configuration
kubectl get events --sort-by='.lastTimestamp'
```

### Certificate/SSL Issues

**Problem**: HTTPS not working or certificate errors

**Solutions**:
```bash
# For local development, trust development certificate
dotnet dev-certs https --trust

# For production with Let's Encrypt
# Install cert-manager in Kubernetes
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml

# Or use Azure App Service managed certificates
az webapp config ssl bind \
  --certificate-thumbprint <thumbprint> \
  --ssl-type SNI \
  --name <app-name> \
  --resource-group <rg-name>
```

---

## Performance Issues

### Slow API Response

**Problem**: API taking too long to respond

**Diagnostic Steps**:
```csharp
// Add performance logging
using var operation = _telemetry.StartOperation<RequestTelemetry>("OperationName");
Stopwatch sw = Stopwatch.StartNew();

// Your code here

sw.Stop();
_logger.LogInformation("Operation took {ElapsedMs}ms", sw.ElapsedMilliseconds);
```

**Common Solutions**:
1. Add database indexes
2. Enable response caching
3. Use async/await properly
4. Implement pagination
5. Add Redis caching for frequently accessed data

### High Memory Usage

**Problem**: Container or application consuming too much memory

**Solutions**:
```bash
# Monitor memory usage
docker stats

# Set memory limits in docker-compose.yml
services:
  api:
    deploy:
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M

# Check for memory leaks
dotnet-dump collect --process-id <pid>
dotnet-dump analyze <dump-file>
```

---

## Getting More Help

If you can't find your issue here:

1. **Check the logs**: Most issues show up in application or container logs
2. **Search issues**: Look for similar problems on GitHub, Stack Overflow
3. **Enable verbose logging**: Set log level to Debug or Trace
4. **Isolate the problem**: Test components individually
5. **Ask for help**: 
   - [Stack Overflow - .NET tag](https://stackoverflow.com/questions/tagged/.net)
   - [GitHub Issues](https://github.com/dotnet/aspnetcore/issues)
   - [.NET Discord](https://aka.ms/dotnet-discord)

---

**Last Updated**: November 2024  
**For more information**, see the [complete guide](GUIDE.md).
