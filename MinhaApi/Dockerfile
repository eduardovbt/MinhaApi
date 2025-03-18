# Etapa 1: Imagem base para a construção do projeto
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS base
WORKDIR /app
EXPOSE 8080
EXPOSE 8081

# Etapa 2: Imagem para o build do código
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["MinhaApi/MinhaApi.csproj", "MinhaApi/"]
RUN dotnet restore "./MinhaApi/MinhaApi.csproj"
COPY . .
WORKDIR "/src/MinhaApi"
RUN dotnet build "./MinhaApi.csproj" -c %BUILD_CONFIGURATION% -o /app/build

# Etapa 3: Publish para produção
FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "./MinhaApi.csproj" -c %BUILD_CONFIGURATION% -o /app/publish /p:UseAppHost=false

# Etapa 4: Definindo a imagem de execução
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MinhaApi.dll"]
