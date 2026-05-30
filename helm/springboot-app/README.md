# Spring Boot + PostgreSQL + Redis + Kafka with Helm

This project deploys the entire application stack using Helm:

* PostgreSQL
* Redis
* Kafka
* Spring Boot

The deployment target is Minikube.

---

# Prerequisites

Required tools:

* Docker
* kubectl
* Helm 3
* Minikube

Verify installation:

```bash
docker --version
kubectl version --client
helm version
minikube version
```

---

# Start Minikube

Start Minikube:

```bash
minikube start \
  --cpus=4 \
  --memory=8192
```

Verify cluster:

```bash
kubectl get nodes
```

Expected:

```text
NAME       STATUS   ROLES
minikube   Ready    control-plane
```

---

# Build Application Image

Configure Docker to use Minikube Docker daemon:

```bash
eval $(minikube docker-env)
```

Build image:

```bash
docker build -t springboot-app:latest .
```

Verify image:

```bash
docker images | grep springboot-app
```

Expected:

```text
springboot-app   latest
```

---

# Validate Helm Chart

Move to chart directory:

```bash
cd helm
```

Validate chart:

```bash
helm lint .
```

Expected:

```text
1 chart(s) linted, 0 chart(s) failed
```

Render manifests:

```bash
helm template springboot-template .
```

This generates all Kubernetes manifests without installing anything.

---

# Install Application Stack

Install the release:

```bash
helm install springboot-template . \
  --namespace springboot-template \
  --create-namespace
```

Verify release:

```bash
helm list -n springboot-template
```

Expected:

```text
NAME   NAMESPACE STATUS
springboot-template   springboot-template      deployed
```

---

# Verify Deployments

Check deployments:

```bash
kubectl get deployments -n springboot-template
```

Expected:

```text
postgres
redis
kafka
springboot-app
```

Check pods:

```bash
kubectl get pods -n springboot-template
```

Expected:

```text
postgres-xxxxx
redis-xxxxx
kafka-xxxxx
springboot-app-xxxxx
```

All pods should eventually become:

```text
Running
```

---

# Verify Services

Check services:

```bash
kubectl get svc -n springboot-template
```

Expected:

```text
NAME             TYPE        PORT
postgres         ClusterIP   5432
redis            ClusterIP   6379
kafka            ClusterIP   9092
springboot-app   ClusterIP   80
```

These service names become internal DNS names:

```text
postgres
redis
kafka
```

Which match the application configuration:

```properties
spring.datasource.url=jdbc:postgresql://postgres:5432/app
spring.data.redis.host=redis
spring.kafka.bootstrap-servers=kafka:9092
```

---

# Verify Application Logs

Check Spring Boot logs:

```bash
kubectl logs -f deployment/springboot-app -n springboot-template
```

Expected startup messages:

```text
Started Application
Tomcat started on port(s): 8080
Flyway migration successful
```

---

# Access Application

Forward service port:

```bash
kubectl port-forward svc/springboot-app 8080:80 -n springboot-template
```

Application:

```text
http://localhost:8080
```

Health endpoint:

```text
http://localhost:8080/actuator/health
```

Expected:

```json
{
  "status": "UP"
}
```

# Validate Ingress
```text
kubectl get ingress -n springboot-template
```
Update /etc/hosts
```text
192.168.49.2 spring.local
```



---

# Verify Database Connectivity

Open PostgreSQL shell:

```bash
kubectl exec -it deployment/postgres -n springboot-template -- psql -U user -d app
```

List tables:

```sql
\dt
```

Exit:

```sql
\q
```

---

# Verify Redis Connectivity

Open Redis CLI:

```bash
kubectl exec -it deployment/redis -n springboot-template -- redis-cli
```

Test:

```bash
PING
```

Expected:

```text
PONG
```

Exit:

```bash
exit
```

---

# Verify Kafka

Open Kafka container:

```bash
kubectl exec -it deployment/kafka -n springboot-template -- bash
```

List topics:

```bash
/opt/kafka/bin/kafka-topics.sh \
  --bootstrap-server localhost:9092 \
  --list
```

Exit:

```bash
exit
```

---

# Upgrade Release

After changing templates or values:

```bash
helm upgrade springboot-template . \
  --namespace springboot-template
```

Verify rollout:

```bash
kubectl rollout status deployment/springboot-app -n springboot-template
```

---

# Rebuild Application

After code changes:

```bash
eval $(minikube docker-env)

docker build -t springboot-app:latest .
```

Restart deployment:

```bash
kubectl rollout restart deployment/springboot-app -n springboot-template
```

Verify:

```bash
kubectl get pods -n springboot-template
```

---

# View Release Information

Release list:

```bash
helm list -n springboot-template
```

Release history:

```bash
helm history springboot-template -n springboot-template
```

Generated manifests:

```bash
helm get manifest springboot-template -n springboot-template
```

Values used:

```bash
helm get values springboot-template -n springboot-template
```

---

# Rollback

View revisions:

```bash
helm history springboot-template -n springboot-template
```

Rollback:

```bash
helm rollback springboot-template 1 -n springboot-template
```

Verify:

```bash
helm history springboot-template -n springboot-template
```

---

# Uninstall

Remove release:

```bash
helm uninstall springboot-template -n springboot-template
```

Verify:

```bash
kubectl get all -n springboot-template
```

Expected:

```text
No resources found
```

Delete namespace:

```bash
kubectl delete namespace springboot-template
```

---

# Cleanup

Stop Minikube:

```bash
minikube stop
```

Delete cluster:

```bash
minikube delete
```

---

# Deployment Flow

```text
Minikube
    ↓
Docker Build
    ↓
Helm Install
    ↓
PostgreSQL
    ↓
Redis
    ↓
Kafka
    ↓
Spring Boot
```

The entire stack is managed by a single Helm release:

```bash
helm install springboot-template .
```

which is roughly the Kubernetes equivalent of:

```bash
docker compose up
```

except with significantly more YAML, because apparently humanity looked at Docker Compose and decided it was not sufficiently verbose.
