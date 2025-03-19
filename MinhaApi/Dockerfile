#See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443
ENV SimpleProperty="hello-from-BASE-dockerfile"

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS base
WORKDIR /src
COPY ["MinhaApi/MinhaApi.csproj", "MinhaApi/"]
RUN dotnet restore "MinhaApi/MinhaApi.csproj"
COPY . .
WORKDIR "/src/MinhaApi"
RUN dotnet build "MinhaApi.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "MinhaApi.csproj" -c Release -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENV SimpleProperty="hello-from-FINAL-dockerfile"
ENTRYPOINT ["dotnet", "MinhaApi.dll"]

# Etapa 4: Definindo a imagem de execução
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MinhaApi.dll"]
