# My Bank - Spring Boot + PostgreSQL

A minimal banking REST API with Customers, Accounts (SAVINGS/CURRENT), and Transactions.

## Quick start (no DB setup: H2 profile)
- Run with in-memory H2 DB for local testing:
```
./mvnw spring-boot:run -Dspring-boot.run.profiles=h2
```
- H2 console (optional): http://localhost:8080/h2-console (JDBC URL: jdbc:h2:mem:demobank, user: sa, no password)

## Prerequisites (PostgreSQL mode)
- Java 21
- PostgreSQL running and a database you can connect to

Configure `src/main/resources/application.properties`:
- `spring.datasource.url=jdbc:postgresql://localhost:5432/bank_db`
- `spring.datasource.username=bank_app`
- `spring.datasource.password=my_password`
- `spring.jpa.hibernate.ddl-auto=update`

Then run:
```
./mvnw spring-boot:run
```

## API Overview
Base path: `/api`

Customers
- POST `/api/customers`
  Body: {"firstName":"Alice","lastName":"Doe","email":"a@x.com","phone":"123","dob":"1990-01-01"}
- GET `/api/customers/{id}`

Accounts
- POST `/api/accounts`
  Body examples:
  - Savings: {"customerId":1, "accountType":"SAVINGS", "openingBalance":100.00, "interestRate":0.025}
  - Current: {"customerId":1, "accountType":"CURRENT", "openingBalance":0, "overdraftLimit":200.00}
  Optional: accountNumber
- GET `/api/accounts/{id}`
- POST `/api/accounts/{id}/deposit`
  Body: {"amount":50.00, "note":"cash"}
- POST `/api/accounts/{id}/withdraw`
  Body: {"amount":20.00, "note":"atm"}
- POST `/api/accounts/transfer`
  Body: {"fromAccountId":1, "toAccountId":2, "amount":10.00, "note":"move"}
- GET `/api/accounts/{id}/transactions`

## Database Schema (Current H2 / PostgreSQL Compatible)
```
TABLE customers (
  id BIGSERIAL PRIMARY KEY,
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100),
  email VARCHAR(255) UNIQUE,
  phone VARCHAR(20),
  dob DATE,
  created_at TIMESTAMP WITH TIME ZONE
);

TABLE accounts (
  id BIGSERIAL PRIMARY KEY,
  account_number VARCHAR(32) NOT NULL UNIQUE,
  customer_id BIGINT NOT NULL REFERENCES customers(id),
  account_type VARCHAR(16) NOT NULL,         -- SAVINGS | CURRENT
  balance NUMERIC(18,2) NOT NULL DEFAULT 0,
  opened_at TIMESTAMP WITH TIME ZONE,
  interest_rate NUMERIC(5,4),                -- for SAVINGS
  overdraft_limit NUMERIC(18,2),             -- for CURRENT
  status VARCHAR(20) NOT NULL DEFAULT 'ACTIVE'
);

TABLE transactions (
  id BIGSERIAL PRIMARY KEY,
  account_id BIGINT NOT NULL REFERENCES accounts(id),
  txn_type VARCHAR(20) NOT NULL,             -- DEPOSIT | WITHDRAWAL | TRANSFER_IN | TRANSFER_OUT
  amount NUMERIC(18,2) NOT NULL,
  txn_date TIMESTAMP WITH TIME ZONE,
  note VARCHAR(255)
);
```

## Swagger / API Docs
- OpenAPI JSON: `http://localhost:8080/v3/api-docs`
- Swagger UI: `http://localhost:8080/swagger-ui/index.html`

## Backend Testing
Run backend unit tests:
```
./mvnw test
```

## Frontend (Angular) – Development
```
cd frontend
npm install
npm start
# open http://localhost:4201
```

### Frontend Unit Tests (Karma/Jasmine)
```
cd frontend
npm test
```

## Seeded Test User
A default user is auto-created on startup (controlled by `demobank.seed.test-user=true`):
```
Email: test@example.com
Password: (any – password not enforced in demo)
```
