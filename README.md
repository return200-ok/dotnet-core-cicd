# dotnet-core-cicd

## Dockerize dotnet core

The .NET Docker images have been updated for this release. The .NET Docker samples show various ways to use .NET and Docker together.

The following repos have been updated.

    dotnet/sdk: .NET Core SDK
    dotnet/aspnet: ASP.NET Core Runtime
    dotnet/runtime: .NET Core Runtime
    dotnet/runtime-deps: .NET Core Runtime Dependencies

Different between `runtime` and `SDK`
> Runtime: to run apps

> SDK (Runtime + Tooling): to build and run apps

## Build & Publish 
- dotnet build - Builds a project and all of its dependencies.

- dotnet publish - Packs the application and its dependencies into a folder for deployment to a hosting system. (PS - this also builds the application before packing)

## Dockerfile
```
# Learn about building .NET container images:
# https://github.com/dotnet/dotnet-docker/blob/main/samples/README.md
FROM mcr.microsoft.com/dotnet/sdk:7.0-alpine AS build
WORKDIR /source

# copy csproj and restore as distinct layers
COPY aspnetapp/*.csproj .
RUN dotnet restore --use-current-runtime /p:PublishReadyToRun=true

# copy everything else and build app
COPY aspnetapp/. .
RUN dotnet publish --use-current-runtime --no-restore -o /app /p:PublishTrimmed=true /p:PublishReadyToRun=true


# Enable globalization and time zones:
# https://github.com/dotnet/dotnet-docker/blob/main/samples/enable-globalization.md
# final stage/image
FROM mcr.microsoft.com/dotnet/runtime-deps:7.0-alpine
WORKDIR /app
COPY --from=build /app .

# This port needs to match the port being used
HEALTHCHECK CMD wget -qO- -t1 http://localhost:80/healthz || exit 1
ENTRYPOINT ["./aspnetapp"]
```
How to determine if .NET Core is installed
- netcore sdk
```
dotnet --version
```
- netcore runtime
```
dotnet --info
```