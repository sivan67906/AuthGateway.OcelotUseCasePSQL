# Build stage
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /src

# Copy project file and restore dependencies
COPY AuthGateway/AuthGateway.csproj AuthGateway/
COPY Directory.Packages.props ./
RUN dotnet restore "AuthGateway/AuthGateway.csproj"

# Copy source code and build
COPY . .
WORKDIR /src/AuthGateway
RUN dotnet build "AuthGateway.csproj" -c Release -o /app/build

# Publish stage
FROM build AS publish
RUN dotnet publish "AuthGateway.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS runtime
WORKDIR /app

# Install curl for health checks
RUN apt-get update && apt-get install -y --no-install-recommends curl && rm -rf /var/lib/apt/lists/*

# Create non-root user for security
RUN groupadd -r appgroup && useradd -r -g appgroup appuser

# Copy published application
COPY --from=publish /app/publish .

# Copy Docker-specific Ocelot configuration and rename it to override the default
COPY AuthGateway/ocelot.docker.json ./ocelot.json

# Create directory for HTTPS certificates
RUN mkdir -p /https && chown -R appuser:appgroup /https

# Set ownership of app directory
RUN chown -R appuser:appgroup /app

# Expose ports
EXPOSE 8080
EXPOSE 8081

# Switch to non-root user
USER appuser

# Set entry point
ENTRYPOINT ["dotnet", "AuthGateway.dll"]
