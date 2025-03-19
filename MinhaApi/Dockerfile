# Etapa 1: Imagem base para a construção do projeto
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS base
WORKDIR /app
EXPOSE 8080
EXPOSE 8081

# Etapa 2: Imagem para o build do código
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /app

# Copiar o arquivo .csproj para o contêiner (ajustado para o diretório correto)
COPY "MinhaApi/MinhaApi.csproj" "./MinhaApi/"

# Restaurar pacotes NuGet
RUN dotnet restore "./MinhaApi/MinhaApi.csproj"

# Copiar o restante dos arquivos para dentro do contêiner
COPY . . 

# Definir o diretório correto para o build
WORKDIR "/app/MinhaApi"

# Construir o projeto
RUN dotnet build "./MinhaApi.csproj" -c ${BUILD_CONFIGURATION} -o /app/build

# Etapa 3: Publicação para produção
FROM build AS publish
RUN dotnet publish "./MinhaApi.csproj" -c ${BUILD_CONFIGURATION} -o /app/publish /p:UseAppHost=false

# Etapa 4: Definindo a imagem de execução
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MinhaApi.dll"]
