# 🐳 Лабораторная работа №3 по Docker для C#

## Микросервисы: два .NET-сервиса + Redis + RabbitMQ

**Время:** 40–50 минут  
**Цель:** собрать два взаимодействующих микросервиса с брокером сообщений и кэшем.

* * *

## Ход работы

### Шаг 1. Структура проекта (5 мин)

```
mkdir Microservices && cd Microservices
dotnet new webapi -n OrderService
dotnet new webapi -n NotificationService
```

### Шаг 2. OrderService (API для заказов) (10 мин)

В `OrderService/Program.cs`:

```
using StackExchange.Redis;

var builder = WebApplication.CreateBuilder(args);

// Redis
var redis = ConnectionMultiplexer.Connect("redis:6379");
builder.Services.AddSingleton(redis.GetDatabase());

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

app.UseSwagger();
app.UseSwaggerUI();

app.MapPost("/orders", async (Order order, IDatabase redis) =>
{
    var orderId = Guid.NewGuid().ToString();
    await redis.StringSetAsync($"order:{orderId}", System.Text.Json.JsonSerializer.Serialize(order));
    
    // Отправка в RabbitMQ (упрощённо)
    return Results.Ok(new { OrderId = orderId, Status = "Created" });
});

app.Run();

record Order(string ProductName, int Quantity);
```

### Шаг 3. NotificationService (5 мин)

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/notify", async (HttpContext ctx) =>
{
    // В реальном проекте подписываемся на RabbitMQ
    await Task.Delay(100);
    return Results.Ok(new { Message = "Notification sent" });
});

app.Run();
```

### Шаг 4. Dockerfile для каждого сервиса (5 мин)

`OrderService/Dockerfile`:

```
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY OrderService.csproj .
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "OrderService.dll"]
```

Аналогично для NotificationService.

### Шаг 5. docker-compose.yml (10 мин)

```
version: '3.8'

services:
  order-api:
    build: ./OrderService
    ports:
      - "5001:8080"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
    depends_on:
      - redis
      - rabbitmq

  notification-api:
    build: ./NotificationService
    ports:
      - "5002:8080"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"

  rabbitmq:
    image: rabbitmq:3-management-alpine
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      - RABBITMQ_DEFAULT_USER=guest
      - RABBITMQ_DEFAULT_PASS=guest
```

### Шаг 6. Запуск (5 мин)

```
docker-compose up -d --build
docker-compose ps
# Создать заказ
curl -X POST http://localhost:5001/orders -H "Content-Type: application/json" -d '{"productName":"Laptop","quantity":1}'
```

### Шаг 7. Проверка RabbitMQ (3 мин)

Откройте браузер: `http://localhost:15672` (guest/guest)

* * *

## Самостоятельное задание

1.  Реализуйте реальную отправку сообщений через RabbitMQ (используйте `RabbitMQ.Client`).
2.  Добавьте сервис `gateway` на YARP (реверс-прокси для обоих API).
3.  Настройте healthchecks для всех сервисов.

* * *

## Контрольные вопросы

1.  Зачем в микросервисной архитектуре нужен брокер сообщений?
2.  Чем Redis отличается от PostgreSQL?
3.  Как в Docker Compose настроить зависимость запуска (сначала БД, потом приложение)?

* * *

[← Вернуться к списку лабораторных работ по Docker + C#](index_docker_csharp.html)