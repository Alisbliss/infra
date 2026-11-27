# infra

Docker Compose for the whole microservice stack.

## Tech Stack

- Docker, Docker Compose
- PostgreSQL (auth, user, order)
- MongoDB (payment)
- Redis, Kafka
- GitHub Actions

## Features

- One compose file for all five services + infra
- Config through .env
- Shared network, services find each other by name
- Healthchecks on every container

## Endpoints

All external traffic goes through the API Gateway (http://localhost:8080).

- Auth (auth-service): POST /api/auth/register, POST /api/auth/login, POST /api/auth/refresh, GET /.well-known/jwks.json (all public)
- Users (user-service): GET /api/users/{id}, GET /api/users/email/{email}, GET /api/users (ADMIN), PUT /api/users/{id}, PATCH /api/users/{id}/status (ADMIN), DELETE /api/users/{id} (ADMIN)
- Cards (user-service): POST /api/users/{userId}/cards, GET /api/users/{userId}/cards, GET /api/cards/{id}, PUT /api/cards/{id}, PATCH /api/cards/{id}/status, DELETE /api/cards/{id}
- Orders (order-service): POST /api/orders, GET /api/orders/{id}, GET /api/orders/user/{userId}, PUT /api/orders/{id}, DELETE /api/orders/{id}, GET /api/orders (ADMIN)
- Payments (payment-service): POST /api/payments, GET /api/payments/{id}, GET /api/payments/my, GET /api/payments/total, GET /api/payments/total/all (ADMIN)
- Health (all services): /actuator/health, /actuator/health/liveness, /actuator/health/readiness

## Launch

```bash
mkdir microservices && cd microservices

git clone https://github.com/Alisbliss/auth-service.git     auth-service
git clone https://github.com/Alisbliss/user-service.git     user-service
git clone https://github.com/Alisbliss/order-service.git    order-service
git clone https://github.com/Alisbliss/payment-service.git  payment-service
git clone https://github.com/Alisbliss/api-gateway.git      api-gateway
git clone https://github.com/Alisbliss/infra.git            infra

cd infra
cp .env.example .env   # fill in JWT keys + db password
docker compose up --build
```

Everything goes through the gateway on http://localhost:8080.