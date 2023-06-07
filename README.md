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
dotnet --version        Display .NET SDK version in use.
dotnet --list-sdks       Display the installed SDKs.
```
- netcore runtime
```
dotnet --info       Display .NET information.
dotnet --list-runtimes   Display the installed runtimes.
```
# Target frameworks in SDK-style projects
Target framework | Latest stable version | Target framework moniker (TFM) | Implemented .NET Standard version 
--- | --- | --- | --- 
--- | --- | --- | --- 
.NET |7 |7 	|net7.0 |2.1
.NET |6 |6 |net6.0 |2.1
.NET |5 |5 |net5.0 |2.1
.NET Standard |2.1 |netstandard2.1 |N/A
.NET Core |3.1 |netcoreapp3.1 |2.1
.NET Framework |4.8 |net48 |2.0

# Switch a .net framework project to a .netcore
open the `csproj` file and change the TargetFramework from something like this
```
<TargetFramework>net462</TargetFramework>
```
to something like this
```
<TargetFramework>netcoreapp3.1</TargetFramework>
```
You could also change it to .net standard, in case you want compatibility between .net core and .net framework consumer projects, by changing it to this:
```
<TargetFramework>netstandard2.0</TargetFramework>
```
You could target multiple frameworks like so:
```
<TargetFrameworks>net462;netstandard2.0</TargetFrameworks> 
```
Ensure you use the correct version number and obviously depending on what this project already targets, things are going to break and will need fixing. For example, you can't use a .net framework class library with a .net core project.

A more detailed process is provided here: https://learn.microsoft.com/en-us/dotnet/core/porting/