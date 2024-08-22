# Writing Secure Container Images

## Introduction
This guide provides best practices for writing secure container images for .NET Core web applications and Python applications/APIs. Following these practices will help ensure your containerized applications are secure and efficient.

## .NET Core Web Applications

### Use Official Base Images
Always start with official Microsoft base images for .NET Core. These images are regularly updated and maintained.

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build
WORKDIR /src
COPY ["MyApp/MyApp.csproj", "MyApp/"]
RUN dotnet restore "MyApp/MyApp.csproj"
COPY . .
WORKDIR "/src/MyApp"
RUN dotnet build "MyApp.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "MyApp.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

### Minimize Image Size
Use multi-stage builds to keep the final image size small. Only include necessary files and dependencies in the final stage.

### Run as Non-Root User
Avoid running your application as the root user. Create a non-root user and switch to it.

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

# Create a non-root user
RUN useradd -m appuser
USER appuser

COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

## Python Applications/APIs

### Use Official Base Images
Start with official Python base images. These images are maintained and updated regularly.

```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

### Minimize Image Size
Use slim or alpine versions of base images to reduce the image size. Remove unnecessary build tools and dependencies after installation.

```dockerfile
FROM python:3.9-alpine

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt && \
    apk --no-cache add build-base && \
    apk del build-base

COPY . .

CMD ["python", "app.py"]
```

### Run as Non-Root User
Create a non-root user and switch to it to avoid running the application as the root user.

```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Create a non-root user
RUN adduser -D appuser
USER appuser

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

## General Best Practices
### Regularly Update Base Images
Keep your base images up to date to ensure you have the latest security patches.

### Use Minimal Privileges
Grant the least amount of privileges necessary for your application to run.

### Scan Images for Vulnerabilities
Use tools like Trivy or Clair to scan your images for vulnerabilities before deploying them.

#### Example: Scanning with Trivy

Trivy is a popular open-source vulnerability scanner for container images.

1. Install Trivy:
```bash
brew install trivy  # macOS
sudo apt-get install trivy  # Ubuntu
```
2. Scan your Docker image:
```bash
trivy image your-image-name
```

#### Example: Scanning with Clair

Clair is an open-source project for the static analysis of vulnerabilities in application containers.

1. Install Clair: Follow the instructions on the Clair GitHub repository to set up Clair.
2. Scan your Docker image:
```bash
clairctl --config path/to/clair/config.yaml analyze your-image-name
clairctl --config path/to/clair/config.yaml report your-image-name
```
