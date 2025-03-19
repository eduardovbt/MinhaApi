# CNPJ Processing Microservice

## Overview

This microservice is designed to process CNPJ records using Kafka, PostgreSQL, and Kubernetes. The service will consume CNPJ messages from Kafka, validate them, persist valid CNPJs in a PostgreSQL database, and republish results to Kafka. If the CNPJ is valid, it will be sent to a "success" Kafka topic. If invalid, an error message will be published to an "error" Kafka topic.

## Architecture Overview

The architecture consists of the following components:

1. **Kafka**:
   - Kafka is used as a message queue to publish and consume CNPJ records.
   - The microservice will subscribe to an input topic (`cnpj-input`), validate the CNPJ, and republish the results to either the `cnpj-validated` (valid) or `cnpj-error` (invalid) topic.

2. **PostgreSQL**:
   - A PostgreSQL database will store valid CNPJ records with their statuses.
   - The `CnpjRecords` table will have fields like `Id`, `Cnpj`, `ValidatedAt`, and `Status`.

3. **Kubernetes**:
   - The service will be deployed on a Kubernetes cluster (e.g., Minikube or k3s).
   - A custom Helm chart will be used to deploy the service, including the necessary configurations for Kafka, PostgreSQL, and environment variables.

4. **Docker & Helm**:
   - The service will be containerized using Docker.
   - Helm will be used to manage the Kubernetes deployment, including the creation of ConfigMaps, Secrets, and Kubernetes Resources.

## Prerequisites

Before getting started, ensure the following are available:

- .NET SDK (version compatible with the project)
- Docker
- Helm (for Kubernetes deployment)
- Kafka and PostgreSQL running (locally or in a cloud environment)
- A local Kubernetes cluster (e.g., Minikube, k3s)

## Setup Instructions

### 1. Clone the Repository

```bash
git clone <repository-url>
cd <repository-name>
```

### 2. Build the Application

Use the following command to build the microservice:

```bash
dotnet build
```

### 3. Test the Application

To run the unit tests for CNPJ validation and Kafka message handling, use:

```bash
dotnet test
```

### 4. Run the Application Locally

To run the service locally, use the following command:

```bash
dotnet run
```

Ensure the necessary environment variables (e.g., Kafka and PostgreSQL details) are set for local execution.

## Dockerization

To build a Docker image for the microservice:

1. Create a `Dockerfile` to containerize the service.

Example `Dockerfile`:

```Dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["CnpjProcessingMicroservice/CnpjProcessingMicroservice.csproj", "CnpjProcessingMicroservice/"]
RUN dotnet restore "CnpjProcessingMicroservice/CnpjProcessingMicroservice.csproj"
COPY . .
WORKDIR "/src/CnpjProcessingMicroservice"
RUN dotnet build "CnpjProcessingMicroservice.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "CnpjProcessingMicroservice.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "CnpjProcessingMicroservice.dll"]
```

Then, build and run the Docker container:

```bash
docker build -t cnpj-processing-service .
docker run -d -p 8080:80 cnpj-processing-service
```

## Kubernetes & Helm Chart

### 1. Create a Helm Chart

A custom Helm chart will be used to deploy the service. The Helm chart will include Kubernetes resources like:

- Deployment
- Service
- ConfigMap/Secret for environment variables (Kafka broker and PostgreSQL details)

To create a Helm chart:

```bash
helm create cnpj-processing
```

Modify the chart to include the necessary values, such as Kafka and PostgreSQL details. Then, deploy the service to a Kubernetes cluster:

```bash
helm install cnpj-processing ./cnpj-processing
```

### 2. Kubernetes Deployment

Ensure that the Kubernetes cluster is running. For Minikube, start it with:

```bash
minikube start
```

Then, deploy the service:

```bash
kubectl apply -f deployment.yaml
```

## Makefile

A `Makefile` is included to streamline common tasks like build, test, and deploy.

Example `Makefile`:

```makefile
build:
    dotnet build

test:
    dotnet test

docker-build:
    docker build -t cnpj-processing-service .

docker-run:
    docker run -d -p 8080:80 cnpj-processing-service

deploy:
    helm install cnpj-processing ./cnpj-processing
```

You can run these tasks using the following commands:

```bash
make build
make test
make docker-build
make deploy
```


Good luck!
