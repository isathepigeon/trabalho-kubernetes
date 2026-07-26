# Orquestração de Containers com Kubernetes — Nginx + Apache

Prova de conceito que coloca dois servidores web distintos — Nginx e Apache HTTPD — rodando de forma independente em um único cluster Kubernetes, cada um exposto em uma porta própria (**Nginx na 8080**, Apache na 8081).

Trabalho acadêmico baseado no desafio da empresa fictícia Web Solutions Ltda., que precisa modernizar sua infraestrutura (hoje em máquinas virtuais, com escala manual e conflitos de dependência) migrando para containers e orquestração.

## Arquitetura

```
                    ┌──────────────── Cluster Kubernetes ────────────────┐
   localhost:8080 → │  nginx-service (NodePort)  →  2 pods  webso-nginx   │
   localhost:8081 → │  apache-service (NodePort) →  2 pods  webso-apache  │
                    └─────────────────────────────────────────────────────┘
```

Cada servidor tem um Deployment (2 réplicas, para redundância e escala) e um Service do tipo NodePort (endereço estável + balanceamento de carga). As imagens partem de bases oficiais otimizadas (nginx:alpine, httpd:alpine) e servem uma página HTML personalizada.

## Estrutura do repositório

```
.
├── k8s/                        # Manifestos Kubernetes (a "Solução Técnica YAML")
│   ├── nginx-deployment.yaml
│   ├── nginx-service.yaml
│   ├── apache-deployment.yaml
│   └── apache-service.yaml
├── nginx/                      # Imagem do Nginx
│   ├── Dockerfile
│   └── index.html
├── apache/                     # Imagem do Apache HTTPD
│   ├── Dockerfile
│   └── index.html
└── docs/
    ├── trabalho-conceitual-yaml.pdf        # Texto conceitual sobre YAML (entrega)
    ├── livro-containers-kubernetes.pdf     # Material de estudo aprofundado
    └── livro-containers-kubernetes.epub
```

## Como executar

Pré-requisitos: Docker, kubectl e Minikube instalados.

```bash
# 1. Subir o cluster local
minikube start --driver=docker

# 2. Construir as imagens dentro do Docker do Minikube
eval $(minikube docker-env)
docker build -t webso-nginx:1.0 nginx/
docker build -t webso-apache:1.0 apache/

# 3. Aplicar os manifestos
kubectl apply -f k8s/

# 4. Conferir que subiu
kubectl get pods
kubectl get services

# 5. Expor nas portas do desafio
kubectl port-forward service/nginx-service 8080:8080 &
kubectl port-forward service/apache-service 8081:8081 &
```

Acesse http://localhost:8080 (Nginx) e http://localhost:8081 (Apache).

Para escalar um serviço de forma independente:

```bash
kubectl scale deployment nginx-deployment --replicas=4
```

## Decisões técnicas

- **Imagens Alpine** — bases minimalistas (~5 MB), reduzindo tamanho, tempo de deploy e superfície de ataque (requisito de "imagens otimizadas").
- **2 réplicas por serviço** — demonstram redundância, autocura e escala independente.
- **NodePort + port-forward** — expõem os serviços nas portas exatas do enunciado (8080/8081) em ambiente local; em produção, o caminho seria um Ingress com TLS.
- **imagePullPolicy: IfNotPresent** — usa a imagem construída localmente no Minikube, sem tentar baixá-la de um registry externo.
- **Modelo declarativo** — os manifestos descrevem o estado desejado; o Kubernetes o mantém (reconciliação contínua).

## Entregas do trabalho

1. **Vídeo Pitch** (YouTube, até 4 min) — demonstração da solução funcionando.
2. **Trabalho Conceitual sobre YAML** — docs/trabalho-conceitual-yaml.pdf.
3. **Solução Técnica YAML** — os arquivos em k8s/.

## Licença

MIT — veja [LICENSE](LICENSE).
