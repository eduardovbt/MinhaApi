#See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.
# Etapa 1: Construção
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["MinhaApi/MinhaApi.csproj", "MinhaApi/"]
RUN dotnet restore "MinhaApi/MinhaApi/MinhaApi.csproj"
COPY . .
WORKDIR "/src/MinhaApi"
RUN dotnet build "MinhaApi.csproj" -c Release -o /app/build

# Etapa 2: Publicação
FROM build AS publish
RUN dotnet publish "MinhaApi.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Etapa 3: Execução
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MinhaApi.dll"]
