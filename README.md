# 🚀 Spring Boot Microservices: Comunicação Assíncrona com RabbitMQ

Um exemplo prático de arquitetura de Microserviços implementando o fluxo de cadastro de usuário e envio de e-mail de boas-vindas com comunicação **assíncrona não-bloqueante** via Mensageria (RabbitMQ).

-----

## 💡 Contexto e Arquitetura do Projeto

O projeto demonstra a implementação de dois microserviços independentes que se comunicam por meio de um **Message Broker** (RabbitMQ).

### Padrões Aplicados

  * **Padrão de Comunicação:** Assíncrona via Mensageria (Comandos). O `user-microservice` publica um comando (`SEND_EMAIL`) para que o `email-microservice` execute a ação.
  * **Padrão de Dados:** Database per Service (cada microsserviço possui seu próprio banco de dados isolado).
  * **Tecnologia Base:** Java 17 e Spring Boot 3.

### Fluxo de Comunicação

1.  Um cliente envia uma requisição `POST` para o `user-microservice` para cadastrar um novo usuário.
2.  O `user-microservice` **salva** o usuário em seu banco de dados (PostgreSQL `ms_user`).
3.  Imediatamente, o `user-microservice` atua como **Producer**, publicando uma mensagem (comando) no RabbitMQ.
4.  O RabbitMQ roteia a mensagem para a fila específica.
5.  O `email-microservice` atua como **Consumer**, ouvindo a fila.
6.  Ao receber a mensagem, o `email-microservice` **envia** o e-mail de boas-vindas (via SMTP do Gmail).
7.  O `email-microservice` **salva** o histórico do e-mail em seu próprio banco de dados (PostgreSQL `ms_email`).

-----

## 🛠️ Stack Tecnológica

As seguintes tecnologias foram utilizadas para construir e orquestrar os serviços:

| Categoria | Tecnologia | Versão | Uso |
| :--- | :--- | :--- | :--- |
| **Linguagem** | Java | 17 (LTS) | Linguagem principal de desenvolvimento. |
| **Framework** | Spring Boot | 3.x | Base para o desenvolvimento rápido dos microserviços. |
| **Mensageria** | RabbitMQ | — | Message Broker para comunicação assíncrona. |
| **Banco de Dados** | PostgreSQL | — | Banco de dados relacional (um para cada serviço). |
| **Persistência** | Spring Data JPA/Hibernate | — | Camada de abstração e mapeamento objeto-relacional (ORM). |
| **SMTP** | Spring Mail / Gmail | — | Configuração para envio dos e-mails. |

-----

## 📂 Estrutura do Projeto

O projeto está dividido em dois módulos (microsserviços):

| Módulo | Descrição | Endpoints Principais |
| :--- | :--- | :--- |
| **`user-microservice`** | Gerencia a entidade `User`. Responsável por receber o cadastro e iniciar o processo assíncrono. | `POST /users` (Cadastrar Usuário) |
| **`email-microservice`** | Gerencia o envio e o registro de e-mails. Atua como consumidor da fila do RabbitMQ. | Não possui API REST pública. |

-----

## ⚙️ Configuração e Instalação

Siga os passos para configurar e rodar a aplicação localmente.

### Pré-requisitos

Você precisará ter instalado:

1.  **Java JDK 17+**
2.  **Maven 3+**
3.  **Docker e Docker Compose** (ou instâncias locais de PostgreSQL e RabbitMQ)
4.  **Conta no CloudAMQP** (para a instância gratuita de RabbitMQ) ou instale o RabbitMQ localmente.
5.  **Senha de App do Google** para o SMTP do Gmail.

### 1\. Configuração do Ambiente

Crie dois bancos de dados no PostgreSQL (local ou em um container):

  * `ms_user`
  * `ms_email`

### 2\. Configuração do RabbitMQ (CloudAMQP)

  * Crie uma instância gratuita no CloudAMQP e obtenha a **RabbitMQ URL**.

### 3\. Configuração do SMTP (Gmail)

  * No seu Gmail, habilite a verificação de duas etapas.
  * Crie uma **Senha de App** para o aplicativo (que será usada no lugar da senha normal no arquivo de configuração).

### 4\. Configuração dos Arquivos `application.properties`

Em cada módulo (`user-microservice/src/main/resources/application.properties` e `email-microservice/src/main/resources/application.properties`), substitua as variáveis de ambiente com seus dados:

**`application.properties` (Geral para ambos os serviços)**

```properties
# RabbitMQ (Substitua a URL do CloudAMQP)
spring.rabbitmq.addresses=${RABBITMQ_URL:amqp://user:password@host:port/vhost}

# Configurações do PostgreSQL
spring.datasource.url=jdbc:postgresql://localhost:5432/<nome_do_banco> 
spring.datasource.username=<seu_usuario_postgres>
spring.datasource.password=<sua_senha_postgres>
spring.jpa.hibernate.ddl-auto=update
```

**`email-microservice/application.properties` (Configuração de E-mail)**

```properties
# Configurações SMTP (Porta 8082)
server.port=8082
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=<seu_email_remetente>
spring.mail.password=<sua_senha_app_google>
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
```

### 5\. Execução dos Serviços

Execute os dois microsserviços separadamente.

```bash
# Navegue até o diretório do User Microservice e compile
cd user-microservice
mvn clean install

# Execute o User Microservice na porta 8081
mvn spring-boot:run
```

```bash
# Navegue até o diretório do Email Microservice
cd ../email-microservice
mvn clean install

# Execute o Email Microservice na porta 8082
mvn spring-boot:run
```

-----

## 🧪 Teste de Fluxo Completo

Com ambos os serviços em execução, utilize uma ferramenta como **Postman** ou **Insomnia** para testar o fluxo de ponta a ponta.

**Requisição:** `POST http://localhost:8081/users`

**Corpo (JSON):**

```json
{
    "name": "Nome do Usuário",
    "email": "teste.microservices@email.com"
}
```

### Comportamento Esperado

1.  O `user-microservice` deve retornar um status `201 Created`.
2.  No console do `email-microservice`, você deve ver logs indicando que a mensagem foi consumida (`ListenEmailQueue`).
3.  O e-mail de boas-vindas será enviado para o endereço fornecido.
4.  O registro de envio será salvo no banco de dados `ms_email`.

-----

## 🗺️ Próximos Passos (Roadmap)

Sugestões para evoluir o projeto:

  * **Padrão SAGA:** Implementar o SAGA Pattern (Orquestração ou Coreografia) para garantir a consistência de dados em transações distribuídas.
  * **Service Discovery:** Adicionar um Service Discovery (como Netflix Eureka ou Consul) para gerenciar a localização dos serviços.
  * **Testes de Integração:** Criar testes de integração utilizando **Testcontainers** para simular o ambiente de produção (PostgreSQL e RabbitMQ) de forma isolada.
  * **Gateway API:** Adicionar um Gateway API (como Spring Cloud Gateway) para roteamento, segurança e balanceamento de carga.

-----

## 📄 Licença

Este projeto está sob a **Licença MIT**. Sinta-se à vontade para utilizar, modificar e distribuir o código.

[//][//]: https://www.google.com/search?q=%23 ([Ver Arquivo de Licença](https://www.google.com/search?q=LICENSE))

Feito com ☕ e ❤️.

[//]: https://www.google.com/search?q=%23 "Link para o arquivo LICENSE"
