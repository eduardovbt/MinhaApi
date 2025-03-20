
# Microsserviço para Processamento de CNPJ com Kafka, PostgreSQL e Kubernetes

## Visão Geral

Este projeto foi criado para atender aos requisitos de um teste técnico, cujo objetivo é criar um microsserviço que processa registros de CNPJ utilizando Kafka, PostgreSQL com conteinerização possibilitando a utilização do kubernetes .

O serviço realiza as seguintes operações:
- Consome mensagens de CNPJ do Kafka.
- Valida os CNPJs.
- Persiste os CNPJs válidos no banco de dados PostgreSQL.
- Publica os resultados de volta no Kafka.
  - Se o CNPJ for válido, ele é enviado para o tópico `success` do Kafka.
  - Se inválido, uma mensagem de erro é publicada no tópico `error` do Kafka.

## Visão Geral da Arquitetura

O microserviço foi desenvolvido com **.NET 8**, utilizando o **Entity Framework Core** (ORM) para a comunicação com o banco de dados PostgreSQL. O serviço também usa o **Kafka** como um barramento de mensagens (ESB - Enterprise Service Bus) para rotear as mensagens entre os produtores e consumidores.

O projeto segue a **Onion Architecture**, que é baseada no princípio de inversão de controle, onde as camadas são dispostas de forma concêntrica, com o núcleo representando o domínio central da aplicação. Diferente das arquiteturas tradicionais de camadas, a arquitetura em cebola não depende da camada de dados, mas sim dos modelos de domínio reais.

![Imagem Ilustrativa](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRxC67hvjpPQ6D_xe_HnvKfy9n-bUw1eDZlag&s)

## Como Instalar

### Dependências

- **Kafka**
- **PostgreSQL**

Para facilitar o desenvolvimento e a configuração do ambiente, foi adicionado um arquivo `docker-compose.yml`. Ele irá preparar o ambiente de desenvolvimento, subindo as imagens necessárias para o Kafka e o PostgreSQL, além de executar ações como a criação de tópicos no Kafka e a execução de scripts SQL para a criação das tabelas no PostgreSQL.

### Passo a Passo para Subir o Ambiente

1. Abra o prompt de comando.
2. Navegue até o diretório `src/UI/Nstech.Mdm.Api`.
3. Execute o seguinte comando para subir o ambiente:

   ```bash
   docker-compose up -d
   ```

### Configurações do Docker Compose

```yaml

services:
  postgres:
    image: postgres
    volumes:
      - ./01-create-cnpj-records.sql:/docker-entrypoint-initdb.d/01-create-cnpj-records.sql
    environment:
      POSTGRES_PASSWORD: "nstech123"
    ports:
      - "5432:5432"

  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    networks:
      - net
    ports:
      - 2181:2181
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka:
    image: confluentinc/cp-kafka:latest
    networks:
      - net
    depends_on:
      - zookeeper
    ports:
      - 9092:9092
      - 29092:29092
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      TOPIC_AUTO_CREATE: "true"
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"

  create-topic:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - kafka
    entrypoint: ["sh", "-c", "until kafka-topics --bootstrap-server kafka:29092 --list >/dev/null 2>&1; do sleep 10; done; kafka-topics --bootstrap-server kafka:29092 --create --topic cnpj-validate-topic --partitions 1 --replication-factor 1"]
    networks:
      - net

networks:
  net:
    driver: bridge

```

## Como Usar

Para facilitar os testes da aplicação, foi adicionado um **controller** chamado `CnpjIntegrationController`. Este controller tem como objetivo postar uma mensagem na fila do Kafka, permitindo que o consumidor de integração execute a operação de validação e persistência dos CNPJs.

### Endpoints Expostos

- **POST /api/publish/message**: Envia uma mensagem de CNPJ para o Kafka.
  - Body: JSON com o CNPJ a ser processado.
  
Exemplo de request:

```json
{
  "cnpj": "12345678000195"
}
```

## CI/CD

O projeto foi configurado com uma **pipeline de CI/CD** simplificada que executa os seguintes passos:

1. **Execução de Testes**: A pipeline executa testes automatizados sempre que um *pull request* é enviado.
2. **Publicação de Imagem Docker**: A publicação da imagem Docker é realizada apenas quando há um *push* no branch `main`.

### Passos na Pipeline

1. **Execução de Testes**: Testes unitários e de integração são executados em todas as PRs para garantir que o código esteja funcionando corretamente.
2. **Publicação da Imagem Docker**: Após a aprovação de um PR e o merge no branch `main`, a pipeline constrói e publica a imagem Docker no repositório de contêineres.

## Estrutura de Diretórios

A estrutura do projeto segue uma abordagem modular com base na arquitetura Onion. Aqui está uma visão geral das pastas principais:

```
/src
  /UI
    /Nstech.Mdm.Api      # Contém a API e controllers do microsserviço
  /Core
    /Domain              # Entidades e modelos de domínio
    /Services            # Lógica de negócios e regras de processamento
    /Abstract            # Contratos 
  /Infrastructure
    /Data                # Implementação do acesso a dados (EF Core,postgres) 
    /Messaging           # Implementação do Kafka para comunicação (kafka,Masstransit)
    /DI                  # Implementação de controle de dependencia
  /Tests
    /Unity                # Implementação de testes unitarios(Xunity)
```

## Tecnologias Usadas

- **.NET 8**: Framework utilizado para o desenvolvimento do microsserviço.
- **Entity Framework Core**: ORM para acesso ao banco de dados PostgreSQL.
- **Kafka**: Broker de mensagens utilizado para comunicação entre os microsserviços.
- **Docker**: Usado para containerizar o ambiente de desenvolvimento e produção.
- **PostgreSQL**: Banco de dados utilizado para persistir os CNPJs válidos.
- **Kubernetes**: Usado para orquestrar os containers em produção.
