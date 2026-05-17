# 🐳 Docker + C#: Лабораторные работы

3 работы по 40–50 минут каждая. Для студентов, изучающих C# и .NET.

*   [Лабораторная работа №1. Основы Docker: контейнеризация .NET-приложения](lab1.md)
*   [Лабораторная работа №2. Docker Compose: ASP.NET Core + PostgreSQL](lab2.md)
*   [Лабораторная работа №3. Микросервисы: два .NET-сервиса + Redis + RabbitMQ](lab3.md)

* * *

## Перед началом работы

Установите Docker и .NET SDK 8.0:

```
# Windows/macOS: скачать Docker Desktop и .NET SDK
# Linux (Ubuntu):
sudo apt update
sudo apt install docker.io docker-compose
sudo usermod -aG docker $USER

# Установка .NET SDK
wget https://dot.net/v1/dotnet-install.sh
chmod +x dotnet-install.sh
./dotnet-install.sh --channel 8.0
```
