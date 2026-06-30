# OrderFlow — API REST de E-commerce com Spring Boot e JPA

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Java](https://img.shields.io/badge/Java-25-orange.svg)](https://openjdk.org/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.0.3-brightgreen.svg)](https://spring.io/projects/spring-boot)

> API REST para gestão de pedidos de e-commerce, construída com Spring Boot, JPA/Hibernate e PostgreSQL — com banco H2 em memória para testes.

---

## Sobre o Projeto

O **OrderFlow** é uma API REST completa para um domínio de e-commerce, cobrindo o ciclo de vida de um pedido: clientes fazem pedidos com múltiplos itens de produtos categorizados, e cada pedido passa por um fluxo de status (aguardando pagamento → pago → em transporte → entregue / cancelado). O projeto demonstra na prática:

- Modelagem de domínio rico com relacionamentos JPA complexos (`@ManyToMany`, `@OneToMany`, `@OneToOne`, chave primária composta com `@EmbeddedId`)
- Arquitetura em três camadas: **Resource → Service → Repository**
- Tratamento centralizado de exceções com `@ControllerAdvice` e respostas de erro padronizadas
- Perfis Spring (`test` / `prod`) separando banco em memória de banco de produção

---

## Endpoints da API

### Users `/users`

| Método   | Rota          | Descrição               |
| -------- | ------------- | ----------------------- |
| `GET`    | `/users`      | Lista todos os usuários |
| `GET`    | `/users/{id}` | Busca usuário por ID    |
| `POST`   | `/users`      | Cria novo usuário       |
| `PUT`    | `/users/{id}` | Atualiza usuário        |
| `DELETE` | `/users/{id}` | Remove usuário          |

### Orders `/orders`

| Método | Rota           | Descrição                                             |
| ------ | -------------- | ----------------------------------------------------- |
| `GET`  | `/orders`      | Lista todos os pedidos                                |
| `GET`  | `/orders/{id}` | Busca pedido por ID (com itens, subtotal e pagamento) |

### Products `/products`

| Método | Rota             | Descrição               |
| ------ | ---------------- | ----------------------- |
| `GET`  | `/products`      | Lista todos os produtos |
| `GET`  | `/products/{id}` | Busca produto por ID    |

### Categories `/categories`

| Método | Rota               | Descrição                 |
| ------ | ------------------ | ------------------------- |
| `GET`  | `/categories`      | Lista todas as categorias |
| `GET`  | `/categories/{id}` | Busca categoria por ID    |

---

## Modelo de Domínio

```
User ──────────────────────── Order
(1)                           (N)
                               │
                               │ OrderItem (chave composta: order_id + product_id)
                               │
                            Product ──── ManyToMany ──── Category
                               │
                            Payment (OneToOne com Order, compartilha ID)
```

### Entidades

| Entidade    | Campos principais                                        |
| ----------- | -------------------------------------------------------- |
| `User`      | id, name, email, phone, password                         |
| `Order`     | id, moment, orderStatus, client (User), items, payment   |
| `Product`   | id, name, description, price, imgUrl, categories         |
| `Category`  | id, name                                                 |
| `OrderItem` | order + product (PK composta), quantity, price, subTotal |
| `Payment`   | id (= order.id), moment, order                           |

### Status do Pedido (`OrderStatus`)

```
1 → WAITING_PAYMENT
2 → PAID
3 → SHIPPMENT
4 → DELIVERED
5 → CANCELED
```

---

## Arquitetura

```
resources/              ← Controllers REST (@RestController)
  └── exceptions/       ← Tratamento global de erros (@ControllerAdvice)
services/               ← Regras de negócio (@Service)
  └── exceptions/       ← Exceções customizadas do domínio
repositories/           ← Acesso ao banco via Spring Data JPA (@Repository)
entities/               ← Entidades JPA (@Entity)
  ├── enums/            ← OrderStatus
  └── pk/               ← Chave primária composta (OrderItemPK)
config/                 ← Seed de dados para perfil de teste
```

### Tratamento de Erros

Todas as exceções retornam um corpo de erro padronizado:

```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "status": 404,
  "error": "Resource not found",
  "message": "Resource not found! Id 99",
  "path": "/users/99"
}
```

| Exceção                     | HTTP Status       |
| --------------------------- | ----------------- |
| `ResourceNotFoundException` | `404 Not Found`   |
| `DatabaseException`         | `400 Bad Request` |

---

## Tecnologias

- **Java 25**
- **Spring Boot 4.0.3** (Spring MVC, Spring Data JPA)
- **Hibernate** — ORM e geração de DDL
- **H2 Database** — banco em memória para o perfil `test`
- **PostgreSQL** — banco de dados para o perfil de produção
- **Maven** — gerenciamento de dependências e build

---

## Como Executar

### Pré-requisitos

- Java 25+
- Maven 3.9+
- PostgreSQL (apenas para perfil `prod`)

### Perfil de teste (H2 — sem instalação extra)

```bash
# Clonar o repositório
git clone https://github.com/jordao-asato/orderflow.git
cd orderflow

# Rodar com Maven Wrapper
./mvnw spring-boot:run
```

O perfil `test` é ativado por padrão (`spring.profiles.active=test`). O banco H2 é criado em memória e populado automaticamente com dados de seed ao iniciar.

### Console do H2

Com a aplicação rodando, acesse:

```
URL:      http://localhost:8080/h2-console
JDBC URL: jdbc:h2:mem:testdb
User:     sa
Password: (vazio)
```

### Perfil de produção (PostgreSQL)

Crie um arquivo `application-prod.properties` com as credenciais do banco:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/orderflow
spring.datasource.username=seu_usuario
spring.datasource.password=sua_senha
spring.jpa.hibernate.ddl-auto=update
```

E ative o perfil:

```bash
./mvnw spring-boot:run -Dspring-boot.run.profiles=prod
```

---

## Estrutura do Projeto

```
src/
└── main/
    ├── java/com/educandoweb/course/
    │   ├── CourseApplication.java
    │   ├── config/
    │   │   └── TestConfig.java             # Seed de dados (perfil test)
    │   ├── entities/
    │   │   ├── User.java
    │   │   ├── Order.java
    │   │   ├── OrderItem.java
    │   │   ├── Product.java
    │   │   ├── Category.java
    │   │   ├── Payment.java
    │   │   ├── enums/OrderStatus.java
    │   │   └── pk/OrderItemPK.java
    │   ├── repositories/
    │   │   ├── UserRepository.java
    │   │   ├── OrderRepository.java
    │   │   ├── ProductRepository.java
    │   │   ├── CategoryRepository.java
    │   │   └── OrderItemRepository.java
    │   ├── services/
    │   │   ├── UserService.java
    │   │   ├── OrderService.java
    │   │   ├── ProductService.java
    │   │   ├── CategoryService.java
    │   │   └── exceptions/
    │   │       ├── ResourceNotFoundException.java
    │   │       └── DatabaseException.java
    │   └── resources/
    │       ├── UserResource.java
    │       ├── OrderResource.java
    │       ├── ProductResource.java
    │       ├── CategoryResource.java
    │       └── exceptions/
    │           ├── ResourceExceptionHandler.java
    │           └── StandardError.java
    └── resources/
        ├── application.properties          # Perfil ativo: test
        └── application-test.properties     # Config H2
```

---

## Autor

**Jordão Asato**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Jordão%20Asato-blue?logo=linkedin)](https://www.linkedin.com/in/jordao-asato-327063385)
