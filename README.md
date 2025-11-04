# E-Commerce Web Application with .NET

A comprehensive guide and reference for building a production-ready e-commerce application using .NET 9, inspired by the [dotnet/eShop](https://github.com/dotnet/eShop) reference architecture.

## 📚 Complete Guide

See **[GUIDE.md](GUIDE.md)** for the complete step-by-step tutorial covering:

1. **Environment Setup** - Prerequisites and development environment configuration
2. **Project Structure** - Microservices architecture with Clean Architecture principles
3. **Core Features** - Implementation of Catalog, Basket, Ordering, Gateway, and Web services
4. **Testing Strategy** - Unit tests, integration tests, and test automation
5. **Containerization** - Docker and Docker Compose setup
6. **CI/CD Pipeline** - GitHub Actions and Azure DevOps configurations
7. **Deployment** - Azure Container Apps, AKS, and Docker Swarm strategies
8. **Best Practices** - Architecture patterns, security, performance, and monitoring

## 🚀 Quick Start

### Prerequisites

- [.NET 9 SDK](https://dotnet.microsoft.com/download)
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Git](https://git-scm.com/)
- IDE: Visual Studio 2022, VS Code, or JetBrains Rider

### Getting Started

Follow the detailed instructions in [GUIDE.md](GUIDE.md) to:

1. Set up your development environment
2. Create the solution structure with microservices
3. Implement core e-commerce features
4. Add comprehensive testing
5. Containerize and deploy your application

## 🏗️ Architecture

This project demonstrates a **microservices architecture** with:

- **API Gateway** (YARP) - Single entry point for all clients
- **Catalog Service** - Product catalog management
- **Basket Service** - Shopping cart with Redis
- **Ordering Service** - Order processing with DDD patterns
- **Identity Service** - Authentication and authorization
- **Web Application** - Frontend (Blazor/React)

## 📖 What You'll Learn

- Microservices architecture and design patterns
- Clean Architecture and Domain-Driven Design (DDD)
- CQRS and Event-Driven Architecture
- Entity Framework Core and database management
- Docker containerization and orchestration
- CI/CD with GitHub Actions and Azure DevOps
- Cloud deployment (Azure, Kubernetes)
- Testing strategies (unit, integration, load testing)
- Monitoring, logging, and observability
- Security best practices

## 🛠️ Technology Stack

- **.NET 9** - Latest .NET framework
- **ASP.NET Core** - Web API and MVC
- **Entity Framework Core** - ORM for data access
- **MediatR** - CQRS implementation
- **Redis** - Distributed caching
- **SQL Server** - Relational database
- **Docker** - Containerization
- **Kubernetes** - Container orchestration (optional)
- **YARP** - API Gateway
- **SignalR** - Real-time communication
- **Serilog** - Structured logging
- **xUnit** - Unit testing framework

## 📋 Project Status

This repository provides a complete learning guide and reference implementation based on industry best practices and the official .NET eShop reference application.

## 🤝 Contributing

Contributions are welcome! Feel free to:

- Report bugs or issues
- Suggest improvements
- Submit pull requests
- Share your implementations

## 📚 Additional Resources

- [Official eShop Reference App](https://github.com/dotnet/eShop)
- [.NET Documentation](https://docs.microsoft.com/dotnet/)
- [Microservices Architecture Guide](https://learn.microsoft.com/azure/architecture/guide/architecture-styles/microservices)
- [Domain-Driven Design](https://www.domainlanguage.com/ddd/)

## 📄 License

This project is licensed under the terms specified in the [LICENSE](LICENSE) file.

---

**Start building your e-commerce application today!** Follow the [complete guide](GUIDE.md) for step-by-step instructions. 🚀
