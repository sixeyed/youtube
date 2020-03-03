FROM mcr.microsoft.com/dotnet/core/sdk:3.1.101 AS sdk

# build
FROM sdk AS build
WORKDIR /src
COPY src/Demo.Common/Demo.Common.csproj ./Demo.Common/
COPY src/Demo.ConsoleApp/Demo.ConsoleApp.csproj ./Demo.ConsoleApp/

WORKDIR /src/Demo.ConsoleApp
RUN dotnet restore

COPY src/Demo.Common/ /src/Demo.Common/
COPY src/Demo.ConsoleApp/ /src/Demo.ConsoleApp/
RUN dotnet publish -c Release -o /out Demo.ConsoleApp.csproj

# test
FROM sdk AS test
WORKDIR /src
COPY src/Demo.Common/Demo.Common.csproj ./Demo.Common/
COPY src/Demo.ConsoleApp/Demo.ConsoleApp.csproj ./Demo.ConsoleApp/
COPY src/Demo.Tests/Demo.Tests.csproj ./Demo.Tests/

WORKDIR /src/Demo.Tests
RUN dotnet restore Demo.Tests.csproj

COPY src/Demo.Common/ /src/Demo.Common/
COPY src/Demo.ConsoleApp/ /src/Demo.ConsoleApp/
COPY src/Demo.Tests/ /src/Demo.Tests/
WORKDIR /src/Demo.Tests
RUN dotnet test Demo.Tests.csproj > /test-results.txt

# app 
FROM  mcr.microsoft.com/dotnet/core/runtime:3.1.1

WORKDIR /app
ENTRYPOINT ["dotnet", "Demo.ConsoleApp.dll"]

COPY --from=build /out/ .
COPY --from=test /test-results.txt .