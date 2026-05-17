# 🐳 Лабораторная работа №1 по Docker для C#

## Основы Docker: контейнеризация .NET-приложения

**Время:** 40–50 минут  
**Цель:** научиться создавать Docker-образы для .NET-приложений, запускать контейнеры, работать с volume и портами.

* * *

## Теоретическая справка

*   **.NET SDK** – для сборки приложения.
*   **.NET Runtime** – только для запуска (образ меньше).
*   **Multi-stage build** – сначала сборка в SDK, потом копирование результата в runtime.

* * *

## Ход работы

### Шаг 1. Создание .NET-приложения (5 мин)

```
mkdir DockerDotNet && cd DockerDotNet
dotnet new console -n HelloDocker
cd HelloDocker
```

### Шаг 2. Проверка работы (2 мин)

```
dotnet run
# Вывод: Hello, World!
```

### Шаг 3. Создание Dockerfile (10 мин)

Создайте файл `Dockerfile` в папке `HelloDocker`:

```
# Этап 1: сборка
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY HelloDocker.csproj .
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app/publish

# Этап 2: запуск
FROM mcr.microsoft.com/dotnet/runtime:8.0
WORKDIR /app
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "HelloDocker.dll"]
```

### Шаг 4. Сборка образа (3 мин)

```
docker build -t hello-dotnet .
```

### Шаг 5. Запуск контейнера (3 мин)

```
docker run hello-dotnet
```

### Шаг 6. Web-приложение (ASP.NET Core) (12 мин)

```
cd ..
dotnet new web -n WebDocker
cd WebDocker
```

Измените `Program.cs`:

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello from Docker + C#!");

app.Run();
```

Dockerfile для web:

```
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY WebDocker.csproj .
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "WebDocker.dll"]
```

### Шаг 7. Запуск web-контейнера (5 мин)

```
docker build -t web-dotnet .
docker run -d -p 8080:8080 --name myweb web-dotnet
curl http://localhost:8080
```

* * *

## Самостоятельное задание

1.  Измените приложение так, чтобы оно возвращало текущее время.
2.  Добавьте переменную окружения `ASPNETCORE_URLS=http://+:5000` и запустите на порту 5000.
3.  Подключите volume для логирования (папка `logs`).

* * *

## Контрольные вопросы

1.  Почему в Dockerfile для консоли используется `runtime`, а для web — `aspnet`?
2.  Зачем нужен multi-stage build?
3.  Как передать переменную окружения через `docker run`?

* * *

[← Вернуться к списку лабораторных работ по Docker + C#](index_docker_csharp.html)