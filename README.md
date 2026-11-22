# Infrastructure

Master Docker Compose for micro-service project(auth-service, user-service, order-service, payment-service, api-gateway), incl. PostgreSQL, MongoDB, Redis and Kafka (KRaft).

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
cp .env.example .env

docker compose up -d --build