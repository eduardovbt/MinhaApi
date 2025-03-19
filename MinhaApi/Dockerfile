# Etapa 1: Construção
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build

# Defina o diretório de trabalho como /src dentro do container
WORKDIR /src

# Copie o arquivo .csproj para o diretório correto dentro do container
COPY ["MinhaApi/MinhaApi.csproj", "MinhaApi/"]

# Restaure as dependências do projeto
RUN dotnet restore "MinhaApi/MinhaApi.csproj"

# Copie o restante dos arquivos do projeto
COPY MinhaApi/. MinhaApi/

# Defina o diretório de trabalho como o diretório do projeto
WORKDIR "/src/MinhaApi"

# Compile o projeto
RUN dotnet build "MinhaApi.csproj" -c Release -o /app/build

# Etapa 2: Publicação
FROM build AS publish
RUN dotnet publish "MinhaApi.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Etapa 3: Execução
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base

# Defina o diretório de trabalho para o diretório de execução
WORKDIR /app

# Exponha a porta 80 para o container
EXPOSE 80

# Copie o aplicativo publicado para o diretório de execução
COPY --from=publish /app/publish .

# Defina o ponto de entrada para o aplicativo
ENTRYPOINT ["dotnet", "MinhaApi.dll"]
