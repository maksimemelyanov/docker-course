# 🐳 Лабораторная работа №2 по Docker для C#

## Docker Compose: ASP.NET Core + PostgreSQL

**Время:** 40–50 минут  
**Цель:** научиться поднимать связку .NET-приложения с базой данных в Docker Compose.

* * *

## Ход работы

### Шаг 1. Создание проекта (5 мин)

```
mkdir DotNetCompose && cd DotNetCompose
dotnet new webapi -n MyApi
cd MyApi
```

### Шаг 2. Добавление зависимостей (2 мин)

```
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
dotnet add package Microsoft.EntityFrameworkCore.Design
```

### Шаг 3. Модель и контекст (10 мин)

Создайте `Models/Product.cs`:

```
namespace MyApi.Models;

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
}
```

Создайте `Data/AppDbContext.cs`:

```
using Microsoft.EntityFrameworkCore;
using MyApi.Models;

namespace MyApi.Data;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
    public DbSet<Product> Products => Set<Product>();
}
```

### Шаг 4. Настройка Program.cs (5 мин)

```
using Microsoft.EntityFrameworkCore;
using MyApi.Data;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(connectionString));

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.MapGet("/products", async (AppDbContext db) => await db.Products.ToListAsync());
app.MapPost("/products", async (Product product, AppDbContext db) =>
{
    db.Products.Add(product);
    await db.SaveChangesAsync();
    return Results.Created($"/products/{product.Id}", product);
});

app.Run();
```

### Шаг 5. Dockerfile (3 мин)

```
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY MyApi.csproj .
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "MyApi.dll"]
```

### Шаг 6. docker-compose.yml (10 мин)

```
version: '3.8'

services:
  api:
    build: .
    ports:
      - "5000:8080"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__DefaultConnection=Host=db;Database=mydb;Username=postgres;Password=postgres
    depends_on:
      - db

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  postgres_data:
```

### Шаг 7. Запуск (5 мин)

```
docker-compose up -d --build
curl http://localhost:5000/products
curl -X POST http://localhost:5000/products -H "Content-Type: application/json" -d '{"name":"Book","price":19.99}'
```

* * *

## Самостоятельное задание

1.  Добавьте эндпоинт `GET /products/{id}`.
2.  Настройте миграции Entity Framework через Docker (выполните `dotnet ef migrations add InitialCreate` внутри контейнера).
3.  Добавьте сервис `pgadmin` для управления БД.

* * *

## Контрольные вопросы

1.  Как в Compose передать строку подключения к БД?
2.  Зачем нужен volume для PostgreSQL?
3.  Как выполнить команду внутри контейнера (`docker exec`)?

* * *

[← Вернуться к списку лабораторных работ по Docker + C#](index_docker_csharp.html)