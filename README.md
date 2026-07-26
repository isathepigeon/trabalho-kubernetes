# Container Orchestration with Kubernetes — Nginx + Apache

Proof of concept running two different web servers — Nginx and Apache HTTPD — independently in a single Kubernetes cluster, each exposed on its own port (**Nginx on 8080**, Apache on 8081).

Academic project based on the challenge of the fictional company Web Solutions Ltda., which needs to modernize its infrastructure (currently running on virtual machines, with manual scaling and dependency conflicts) by migrating to containers and orchestration.

## Architecture

```
                    ┌──────────────── Kubernetes Cluster ────────────────┐
   localhost:8080 → │  nginx-service (NodePort)  →  2 pods  webso-nginx   │
   localhost:8081 → │  apache-service (NodePort) →  2 pods  webso-apache  │
                    └─────────────────────────────────────────────────────┘
```

Each server has its own Deployment (2 replicas, for redundancy and scalability*) and a Service of type NodePort (stable address + load balancing). The images are built from optimized official bases (nginx:alpine, httpd:alpine) and serve a custom HTML page.
<img width="600" height="467" alt="image" src="https://github.com/user-attachments/assets/1d99b3df-f758-417d-966f-15af40f19511" />

*Redundancy here essentially means having a fallback: if one pod goes down, the other keeps serving requests while Kubernetes spins up a replacement, so the website stays online.
*Scalability here is horizontal: if needed, we can increase the number of pods instead of adding computing power to a single machine.
*Alpine is a Linux distribution (distro) widely used in containers because it is very lightweight (~5 MB).

## Repository structure

```
.
├── k8s/                        # Kubernetes manifests (the YAML technical solution)
│   ├── nginx-deployment.yaml
│   ├── nginx-service.yaml
│   ├── apache-deployment.yaml
│   └── apache-service.yaml
├── nginx/                      # Nginx image
│   ├── Dockerfile
│   └── index.html
├── apache/                     # Apache HTTPD image
│   ├── Dockerfile
│   └── index.html
└── docs/
    └── conceptual-work-yaml.pdf        # Conceptual paper on YAML
```

## How to run it
Prerequisites: Docker, kubectl and Minikube installed.

```bash
# 1. Start the local cluster
minikube start --driver=docker

# 2. Build the images inside Minikube's Docker
eval $(minikube docker-env)
docker build -t webso-nginx:1.0 nginx/
docker build -t webso-apache:1.0 apache/

# 3. Apply the manifests
kubectl apply -f k8s/

# 4. Check that everything is up
kubectl get pods
kubectl get services

# 5. Expose the ports required by the challenge
kubectl port-forward service/nginx-service 8080:8080 &
kubectl port-forward service/apache-service 8081:8081 &
```

Open http://localhost:8080 (Nginx) and http://localhost:8081 (Apache).

To scale a service independently:

```bash
kubectl scale deployment nginx-deployment --replicas=4
```

## Technical decisions

- **Alpine** — minimal base images (~5 MB), as described above, reducing image size, deploy time and attack surface (the "optimized images" requirement).
- **2 replicas per service** — demonstrate redundancy, self-healing and independent scaling.
- **NodePort + port-forward** — expose the services on the exact ports required by the challenge (8080/8081), which is ideal for a local academic setup.
- **imagePullPolicy: IfNotPresent** — uses the image built locally in Minikube instead of trying to pull it from an external registry.
- **Declarative model** — the manifests describe the desired state and Kubernetes maintains it (continuous reconciliation).

## Deliverables

1. **Pitch video** (YouTube, up to 4 min) — a demo of the working solution, still in progress.
2. **Conceptual paper on YAML** — docs/conceptual-work-yaml.pdf.
3. **YAML technical solution** — the files in k8s/.

## License

MIT — see [LICENSE](LICENSE).
