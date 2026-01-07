# Blog (microservices demo)

This repository contains a small microservices example built with Node (Express) and React, packaged with Docker and deployable to Kubernetes (Minikube). It's intended for local development and demonstration of a multi-service setup: client, server, database, cache, and ingress.

## Architecture

- client: React single-page app served by nginx (development build served by webpack-dev-server in dev mode)
- server: Node/Express API serving JSON endpoints on port 5000
- postgres: Postgres database (container)
- redis: Redis cache (container)
- ingress: nginx ingress routes `/` to client and `/api` to server

Services run in Docker containers for development and can be deployed to Kubernetes manifests located in `k8s/`.

## Prerequisites

- Docker (Desktop) or Docker Engine
- kubectl
- minikube (for local Kubernetes)
- Node.js and npm (for building client/server locally)

## Local development (without Kubernetes)

### Start with Docker Compose (dev)

1. From the project root (where `docker-compose-dev.yml` lives):

```
docker compose -f docker-compose-dev.yml up --build
```

This starts postgres, redis, api (server), client, worker, and nginx (ingress-like front) for local development. The client and server use volume mounts so code changes are reflected live.

Access the app at http://localhost:3050 (or the port mapped in your compose file).

### Run services individually (npm)

- Server:

```
cd server
npm install
npm run dev    # or npm start
```

- Client:

```
cd client
npm install
npm start     # runs webpack-dev-server
```

## Docker

- Build a single service image (example: server):

```
docker build -f server/Dockerfile -t youruser/multi-server:dev ./server
```

- Push images to a registry if you want to deploy them to a remote Kubernetes cluster.

## Kubernetes (Minikube) — quickstart

1. Start minikube (docker driver recommended for local dev):

```
minikube start --driver=docker
```

2. Apply manifests (assuming `k8s/` contains the deployment/service/ingress YAMLs):

```
kubectl apply -f k8s/
```

3. Verify pods and services:

```
kubectl get pods -A
kubectl get svc -A
kubectl get ingress -A
```

4. Access the app

- Via ingress controller NodePort (may vary by driver): check the ingress controller Service NodePort (example 30696) and curl the Minikube IP and port:

```
minikube ip
curl http://$(minikube ip):<nodePort>/api/
```

- Or port-forward locally (recommended for development):

```
# port-forward server
kubectl port-forward svc/server-cluster-ip-service 5000:5000 -n default
curl http://localhost:5000/api/

# port-forward ingress controller
kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8080:80
curl http://localhost:8080/api/
```

Notes: depending on your minikube driver, NodePorts may not be reachable directly from the host. Use `minikube tunnel` or port-forward as needed.

## Troubleshooting

- Ingress routes but host:port doesn't respond: try port-forwarding to confirm the app responds inside the cluster. If port-forward works, the issue is networking/driver exposure (see Minikube docs).
- Service shows `ClusterIP` and endpoints exist but not reachable externally: ClusterIP is internal only. Use NodePort, LoadBalancer + `minikube tunnel`, or ingress.
- Check logs:

```
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller
kubectl logs deploy/server-deployment
```

## File layout (important paths)

- `client/` — React app
- `server/` — Node/Express API
- `k8s/` — Kubernetes manifests (deployments, services, ingress)
- `docker-compose.yml` / `docker-compose-dev.yml` — multi-container dev setups

## Next steps / Improvements

- Add health endpoints for liveness/readiness on the server Deployment.
- Add Kubernetes readiness/liveness probes and resource requests/limits.
- Add CI pipeline to build and push Docker images.

If you want, I can add a short section with exact commands to convert the server Service to a NodePort or generate a `server-nodeport.yaml` manifest and apply it for you.

---

License: MIT
