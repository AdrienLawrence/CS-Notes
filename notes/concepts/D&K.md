# Docker Overview

- Docker builds, distributes, and runs applications as containers

- A container is an isolated process using the host operating-system kernel. It is lighter than a full virtual machine

- An image is an immutable packaged filesystem and configuration. A container is a running instance of an image

- A registry stores images. A repository groups versions/tags of one image

- Main flow:

    Dockerfile -> Build -> Image -> Push to registry -> Pull -> Run container

- Containers improve repeatability but do not automatically make an application secure, stateless, or portable across every CPU/runtime

# Dockerfile

    ```dockerfile
    FROM python:3.13-slim AS runtime

    ENV PYTHONDONTWRITEBYTECODE=1 \
        PYTHONUNBUFFERED=1

    WORKDIR /app

    RUN addgroup --system app && adduser --system --ingroup app app

    COPY requirements.txt .
    RUN pip install --no-cache-dir -r requirements.txt

    COPY . .
    USER app

    EXPOSE 8000
    CMD ["fastapi", "run", "app/main.py", "--port", "8000"]
    ```

- `FROM`: Base image and optional build stage

- `WORKDIR`: Default directory for following instructions and runtime

- `COPY`: Add files from the build context

- `RUN`: Execute a build-time command and create a layer

- `ENV`: Set environment variables

- `ARG`: Build-time variable, not a safe secret store

- `EXPOSE`: Documents the application port without publishing it

- `USER`: Select a non-root runtime user

- `ENTRYPOINT`: Fixed executable behavior

- `CMD`: Default command or arguments which may be overridden

- Exec form such as `CMD ["node", "server.js"]` handles signals more directly than shell form

- Each build instruction creates cacheable layers. Copy dependency manifests and install dependencies before copying frequently changed application code

- Use `.dockerignore` to exclude Git data, dependencies, builds, local secrets, tests when unnecessary, and other irrelevant files from the build context

# Building and Running

    ```bash
    docker build -t my-api:1.0 .
    docker run --rm -p 8000:8000 --env-file .env my-api:1.0
    ```

- `-t`: Name and tag the image

- `--rm`: Remove the stopped container

- `-p host:container`: Publish a container port on the host

- `-e` / `--env-file`: Pass runtime environment values

- `-v` / `--mount`: Attach a volume or bind mount

- `-d`: Detached/background mode

- Common inspection commands:

    ```bash
    docker ps
    docker logs -f container-name
    docker exec -it container-name sh
    docker inspect container-name
    docker stats
    docker image ls
    ```

- Containers exit when their main PID exits. Run one main application concern per container

- `docker stop` sends a termination signal and waits before forcing the process. The application should shut down gracefully

# Images and Multi-Stage Builds

- Image tags are mutable labels. Image digests identify exact immutable content

- Avoid relying on `latest` for repeatable production deployment

- Multi-stage builds separate compilation from a smaller runtime image

    ```dockerfile
    FROM node:22-alpine AS build
    WORKDIR /app
    COPY package*.json ./
    RUN npm ci
    COPY . .
    RUN npm run build

    FROM node:22-alpine AS runtime
    WORKDIR /app
    ENV NODE_ENV=production
    COPY --from=build /app/.next/standalone ./
    COPY --from=build /app/.next/static ./.next/static
    COPY --from=build /app/public ./public
    USER node
    CMD ["node", "server.js"]
    ```

- Smaller runtime images reduce transfer, startup, and attack surface but should remain debuggable enough for operations

- Pin base images deliberately and rebuild regularly for security updates

- BuildKit cache mounts and secret mounts improve builds without baking credentials into layers

- Never copy `.env` or private keys into an image. Deleting them in a later layer does not erase them from earlier layers

# Container Storage and Networking

- A container's writable layer is ephemeral. Removing the container removes that layer

- Named volumes persist Docker-managed data

- Bind mounts expose a specific host path, useful in development but less portable

- Store durable production database/files in managed storage or carefully managed volumes with backups

- Containers on a Docker network resolve one another by service/container name

- `localhost` inside a container refers to that container, not the host or another container

- Port publishing is for host/external access. Containers on the same network use the internal container port directly

# Docker Compose

- Compose defines a multi-container application in YAML

    ```yaml
    services:
      api:
        build: .
        ports:
          - "8000:8000"
        environment:
          DATABASE_URL: postgresql://app:app@db:5432/app
        depends_on:
          db:
            condition: service_healthy

      db:
        image: postgres:17
        environment:
          POSTGRES_USER: app
          POSTGRES_PASSWORD: app
          POSTGRES_DB: app
        volumes:
          - postgres-data:/var/lib/postgresql/data
        healthcheck:
          test: ["CMD-SHELL", "pg_isready -U app"]
          interval: 5s
          timeout: 3s
          retries: 10

    volumes:
      postgres-data:
    ```

- `docker compose up --build` builds and starts services

- `docker compose down` removes the Compose containers and networks. Named volumes remain unless explicitly removed

- `depends_on` controls startup order/health conditions but the application should still retry transient dependency unavailability

- Compose is excellent for local development, tests, and small deployments. Kubernetes solves a broader distributed orchestration problem

# Container Security

- Use trusted minimal base images and scan them for vulnerabilities

- Run as a non-root user and use read-only filesystems/capability restrictions where practical

- Do not expose the Docker socket inside untrusted containers; it effectively grants host control

- Set memory/CPU/process limits so one container cannot consume the entire host

- Keep secrets out of image layers, build arguments, source, and logs

- Sign and verify important images and maintain a software bill of materials

- Containers share a kernel, so isolation is not identical to separate virtual machines

# Kubernetes Overview

- Kubernetes orchestrates containerized workloads across a cluster

- It schedules containers, maintains desired replica counts, replaces failed instances, distributes network traffic, manages configuration, and coordinates rollouts

- Kubernetes uses declarative desired state. Controllers continuously compare desired resources to current resources and reconcile differences

- Cluster parts:

    Control plane: API server, scheduler, controllers, and cluster state

    Worker node: Kubelet, container runtime, networking, and running Pods

- `kubectl` communicates with the Kubernetes API using kubeconfig credentials and context

# Pods

- A Pod is the smallest schedulable unit and contains one or more tightly coupled containers

- Containers in one Pod share network identity and may share volumes

- Pods are replaceable. Their names, IPs, and local files should not be treated as durable

- Common multi-container pattern: one main application plus a tightly coupled sidecar or initialization container

- Init containers complete setup sequentially before normal containers start

- Avoid placing unrelated application services into one Pod because they can no longer scale and deploy independently

# Deployments and ReplicaSets

- A Deployment manages stateless Pods through ReplicaSets and rolling updates

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: api
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: api
      template:
        metadata:
          labels:
            app: api
        spec:
          containers:
            - name: api
              image: registry.example.com/api:1.0.0
              ports:
                - containerPort: 8000
              envFrom:
                - configMapRef:
                    name: api-config
                - secretRef:
                    name: api-secrets
              resources:
                requests:
                  cpu: 100m
                  memory: 128Mi
                limits:
                  memory: 512Mi
              readinessProbe:
                httpGet:
                  path: /health/ready
                  port: 8000
              livenessProbe:
                httpGet:
                  path: /health/live
                  port: 8000
    ```

- The selector must match template labels

- `replicas` is desired Pod count

- RollingUpdate gradually replaces the old ReplicaSet. Readiness prevents traffic reaching an unready new Pod

- `kubectl rollout status deployment/api` follows progress

- `kubectl rollout undo deployment/api` rolls back the workload manifest revision, not external database changes

# Services and Ingress

- A Service gives a stable virtual address and DNS name to a changing set of Pods selected by labels

- Service types:

    `ClusterIP`: Internal cluster access

    `NodePort`: Port on each node

    `LoadBalancer`: Provider load balancer

    `ExternalName`: DNS alias

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: api
    spec:
      selector:
        app: api
      ports:
        - port: 80
          targetPort: 8000
    ```

- Ingress describes HTTP routing to Services and requires an Ingress controller

- Gateway API provides a newer, more expressive networking model where supported

- NetworkPolicy restricts allowed Pod traffic when the cluster network plugin enforces it

# Configuration and Secrets

- ConfigMap stores non-secret configuration

- Secret stores sensitive values in the Kubernetes API. Base64 encoding is not encryption

- Protect Secrets with RBAC, encryption at rest, restricted namespaces/service accounts, and an external secret manager when appropriate

- Configuration can be provided as environment variables or mounted files

- Updating a ConfigMap or Secret does not guarantee every application automatically reloads it. Plan rollout/reload behavior

- Never place passwords directly in committed manifests

# Probes and Lifecycle

- Startup probe: Gives a slow-starting application time before liveness checks begin

- Readiness probe: Determines whether the Pod should receive Service traffic

- Liveness probe: Determines whether Kubernetes should restart the container

- A liveness probe should not fail merely because a temporary downstream database is unavailable, or every instance may restart together

- `preStop` hooks and termination grace periods support graceful shutdown

- The application should stop accepting new work, drain in-flight work, and close connections after `SIGTERM`

# Resources, Scheduling, and Autoscaling

- Resource requests guide scheduling and reserve expected capacity

- Resource limits cap use. CPU is throttled; exceeding memory commonly causes an OOM kill

- Missing requests make scheduling and cluster planning unreliable

- Horizontal Pod Autoscaler changes replica count from metrics

- Vertical Pod Autoscaler recommends or changes Pod resources depending on mode

- Cluster autoscaling adds/removes nodes when Pods cannot be scheduled or capacity is unused

- Autoscaling is delayed feedback. Keep spare capacity and protect dependencies from sudden concurrency

- Scheduling controls include node selectors/affinity, Pod affinity/anti-affinity, topology spread constraints, taints, and tolerations

- PodDisruptionBudget limits voluntary simultaneous disruptions but does not guarantee availability during involuntary failure

# Stateful Workloads and Storage

- PersistentVolume represents storage. PersistentVolumeClaim requests storage

- StorageClass defines dynamic provisioning behavior

- StatefulSet gives Pods stable identities, ordered lifecycle, and stable volume claims

- StatefulSet does not automatically provide database replication, backup, leader election, or correct distributed operations

- Prefer a managed database unless the team can operate the chosen database safely in Kubernetes

- DaemonSet runs a Pod on selected/all nodes, commonly for logging, monitoring, or networking agents

- Job runs finite work. CronJob creates Jobs on a schedule

# Namespaces, RBAC, and Security

- Namespaces group resources and provide naming/authorization boundaries, but are not complete security isolation by themselves

- RBAC grants verbs on resource types through Roles/ClusterRoles and bindings

- ServiceAccounts provide workload identities. Do not give every workload the default broad identity

- Security contexts configure user IDs, groups, capabilities, privilege escalation, seccomp, and read-only filesystems

- Admission policies can reject images, privileges, missing limits, or other unsafe resources

- Use least privilege for users, controllers, CI systems, and workloads

# Helm, Kustomize, and Operators

- Helm packages parameterized Kubernetes resources into charts

- `values.yaml` supplies environment-specific inputs to templates

- Kustomize overlays patches on plain YAML without templates

- An Operator is a custom controller encoding operational knowledge for a specialized resource

- CustomResourceDefinitions add new API resource types

- More abstraction is not always better. Keep deployment configuration understandable and validate rendered manifests

# Common kubectl Commands

    ```bash
    kubectl config get-contexts
    kubectl get pods -A
    kubectl describe pod pod-name
    kubectl logs -f pod-name -c container-name
    kubectl exec -it pod-name -- sh
    kubectl get events --sort-by=.metadata.creationTimestamp
    kubectl apply -f deployment.yml
    kubectl diff -f deployment.yml
    kubectl rollout status deployment/api
    kubectl port-forward service/api 8000:80
    kubectl top pods
    ```

- Start debugging with resource status, events, container logs, probes, endpoints, DNS/network policy, and resource limits

- `CrashLoopBackOff` means repeated process failure, not one specific cause

- `Pending` commonly indicates scheduling, resource, volume, or image constraints

- A healthy Service with no endpoints usually has label selector or readiness problems

# Docker vs Kubernetes

- Docker builds and runs individual containers and local multi-container systems

- Kubernetes schedules and manages replicated containers across a cluster

- Docker/OCI images are deployment artifacts Kubernetes can run through a container runtime

- A small application may only need Docker Compose or a managed platform. Kubernetes adds controllers, networking, security, upgrades, observability, and cluster operations

- Use Kubernetes when its orchestration benefits justify its operational cost

# Production Checklist

- Immutable versioned image and trusted registry

- Non-root process and minimal permissions

- No secrets in image or source

- Resource requests and memory limits

- Startup, readiness, and sensible liveness probes

- Graceful shutdown and bounded timeouts

- Multiple replicas and topology distribution where availability requires it

- Network policy and least-privilege service account

- Observability with request correlation

- Tested rollout and rollback

- Database migrations coordinated separately

- Persistent data backed up and restoration tested

- Vulnerability scanning, image provenance, and patch process

# Quick Reference

- Package application -> Docker image

- Run image -> Container

- Persist local Docker data -> Named volume

- Local multi-service stack -> Docker Compose

- Smallest Kubernetes unit -> Pod

- Stateless replicated workload -> Deployment

- Stable Pod access -> Service

- External HTTP routing -> Ingress/Gateway

- Non-secret settings -> ConfigMap

- Sensitive setting -> Protected Secret/external manager

- One-time work -> Job

- Scheduled work -> CronJob

- Stable identity/volume workload -> StatefulSet

- Every-node agent -> DaemonSet

- Pod receives traffic -> Readiness probe

- Pod should restart -> Liveness probe

- Scheduling capacity -> Resource requests

- Maximum memory -> Resource limit
