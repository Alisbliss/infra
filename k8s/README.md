# Minikube deployment

Manifests to run everything on minikube. Services are ClusterIP and named the same as
the docker-compose ones (postgres-auth, postgres-user, postgres-order, mongodb, redis,
kafka, auth-service, user-service, order-service, payment-service, api-gateway), so the
docker spring profile resolves hosts via cluster DNS.

Every container has liveness/readiness/startup probes hitting
/actuator/health/liveness and /actuator/health/readiness. The gateway is exposed
through an nginx ingress.

## Prereqs

```bash
brew install minikube
minikube start
minikube addons enable ingress
```

## Build images

Repos are siblings of infra:

```
microservices/
  auth-service/ user-service/ order-service/ payment-service/ api-gateway/ infra/
```

```bash
eval "$(minikube docker-env)"

docker build -t auth-service:latest    ../auth-service
docker build -t user-service:latest    ../user-service
docker build -t order-service:latest   ../order-service
docker build -t payment-service:latest ../payment-service
docker build -t api-gateway:latest     ../api-gateway

# or: minikube image build -t auth-service:latest ../auth-service
```

rebuild after any code change, the probes will fail otherwise (actuator was added to
all services for this).

## Secrets

01-secrets.yaml has placeholders for the JWT keys, swap them for the base64 ones
from .env. Or just create the secrets directly:

```bash
kubectl -n microservices create namespace microservices 2>/dev/null || true
kubectl -n microservices create secret generic db-credentials \
  --from-literal=DB_USERNAME=postgres --from-literal=DB_PASSWORD=postgres
kubectl -n microservices create secret generic jwt-keys \
  --from-literal=JWT_PRIVATE_KEY="$(grep '^JWT_PRIVATE_KEY=' ../.env | cut -d= -f2-)" \
  --from-literal=JWT_PUBLIC_KEY="$(grep '^JWT_PUBLIC_KEY=' ../.env | cut -d= -f2-)"
```

## Deploy

```bash
kubectl apply -k .
kubectl -n microservices get pods -w
```

## Reaching it

Ingress routes on the Host header, so every request has to carry microservices.local.
Three ways to do that, pick one.

**1. /etc/hosts** (add once, -a appends so re-running leaves a duplicate):

```bash
# macOS, with minikube tunnel running
echo "127.0.0.1 microservices.local" | sudo tee -a /etc/hosts
# linux: echo "$(minikube ip) microservices.local" | sudo tee -a /etc/hosts
```

```bash
curl -X POST http://microservices.local/api/auth/register \
  -H 'Content-Type: application/json' \
  -d '{"login":"e2euser","password":"secret123","name":"E2E","surname":"User","birthDate":"1990-01-01","email":"e2e@example.com"}'
```

**2. Host header by hand** (no /etc/hosts changes):

```bash
curl -H 'Host: microservices.local' http://127.0.0.1/api/auth/register \
  -H 'Content-Type: application/json' \
  -d '{"login":"e2euser", ...}'
```

On linux replace 127.0.0.1 with $(minikube ip).

**3. --resolve** (also skips /etc/hosts, useful in scripts):

```bash
curl --resolve microservices.local:80:127.0.0.1 \
  http://microservices.local/api/auth/register ...
```

On macOS the node IP isn't routable from the host, so run minikube tunnel in another
terminal and use 127.0.0.1. On linux $(minikube ip) works directly.

## Cleanup

```bash
kubectl delete -k .
minikube delete
```