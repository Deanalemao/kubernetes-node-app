# Node App – Kubernetes Deployment

A containerized MERN Stack Employee Management System deployed on a local Kubernetes cluster using **Minikube**.

This project demonstrates how to deploy and manage a full-stack application using Kubernetes resources such as Deployments, Services, StatefulSets, ConfigMaps, PersistentVolumeClaims and Ingress.

The application uses Docker Hub images for the React frontend and Node.js backend, with MongoDB providing persistent data storage.

---

## Project Architecture

The application follows a microservices-style architecture where the frontend, backend and database run as separate Kubernetes workloads.

```text
                       User / Browser
                             |
                             |
                    employee.local:8081
                             |
                             v
                     NGINX Ingress
                             |
                             v
                    Frontend Service
                             |
                             v
                    React Frontend Pod
                             |
                      NGINX Proxy
                         /api
                             |
                             v
                     Backend Service
                             |
                             v
                    Node.js Backend Pod
                             |
                             v
                      MongoDB Service
                             |
                             v
                    MongoDB StatefulSet
                             |
                             v
                     Persistent Volume
```

### Application Components

| Component           | Technology         | Kubernetes Resource |
| ------------------- | ------------------ | ------------------- |
| Frontend            | React, Vite, Nginx | Deployment          |
| Backend             | Node.js, Express   | Deployment          |
| Database            | MongoDB 7.0        | StatefulSet         |
| Frontend Networking | Kubernetes         | ClusterIP Service   |
| Backend Networking  | Kubernetes         | ClusterIP Service   |
| Database Networking | Kubernetes         | Headless Service    |
| Configuration       | Kubernetes         | ConfigMap           |
| Persistent Storage  | Kubernetes         | PVC                 |
| External Access     | NGINX Ingress      | Ingress             |

---

## Technologies Used

- React.js
- Node.js
- Express.js
- MongoDB
- Docker
- Docker Hub
- Kubernetes
- Minikube
- kubectl
- NGINX Ingress Controller
- YAML

---

## Docker Images

The application uses the following Docker Hub images:

**Frontend**

```text
deanalemao/mern-frontend-employee:1fa79a72e473605db47d2f60a2331e9866b62010
```

**Backend**

```text
deanalemao/mern-backend-employee:1fa79a72e473605db47d2f60a2331e9866b62010
```

**MongoDB**

```text
mongo:7.0
```

---

## Kubernetes Resources

### 1. Deployments

Deployments manage the frontend and backend Pods.

- Frontend Deployment manages React application Pods.
- Backend Deployment manages Node.js application Pods.
- Deployments maintain the desired number of replicas.
- Kubernetes replaces failed Pods automatically.

The backend and frontend Deployment is configured for two replicas to demonstrate application scaling.

### 2. Services

Kubernetes Services provide stable networking and service discovery.

Three Services are configured:

- `frontend` – Exposes the frontend Pods internally.
- `backend` – Exposes the backend Pods internally.
- `mongo` – Headless Service used for MongoDB StatefulSet networking.

Services allow application components to communicate using DNS names instead of relying on changing Pod IP addresses.

### 3. StatefulSet

MongoDB is deployed using a StatefulSet.

A StatefulSet provides:

- Stable Pod identities.
- Predictable Pod names.
- Persistent storage for database data.
- Ordered Pod management.

MongoDB uses a PersistentVolumeClaim to store data in `/data/db`.

**Note:** This project currently uses a single MongoDB replica. A MongoDB replica set has not been configured.

### 4. ConfigMap

A ConfigMap stores non-sensitive backend configuration.

The following environment variables are configured:

- `PORT`
- `NODE_ENV`
- `MONGO_URL`
- `CLIENT_URL`

The backend Deployment injects these values using `envFrom`.

### 5. Persistent Storage

MongoDB uses a PersistentVolumeClaim with:

- Storage: 1Gi
- Access Mode: ReadWriteOnce
- Storage Class: standard

This allows MongoDB data to persist beyond the lifecycle of an individual Pod.

### 6. Ingress

NGINX Ingress Controller manages external access to the application.

The Ingress resource defines:

- Hostname: `employee.local`
- Path: `/`
- Backend Service: `frontend`
- Service Port: `80`

The frontend's NGINX server also acts as a reverse proxy, forwarding `/api` requests to the backend Service at `backend:5001`.

---

## Project Structure

```text
mern-employee-kubernetes/
│
├── backend-configmap.yaml
├── backend-deployment.yaml
├── backend-service.yaml
│
├── frontend-deployment.yaml
├── frontend-service.yaml
│
├── mongo-service.yaml
├── mongo-statefulset.yaml
│
├── ingress.yaml
│
├── .gitignore
└── README.md
```

---

## Deployment Instructions

### Prerequisites

Make sure the following tools are installed:

- Docker
- Minikube
- kubectl
- WSL / Linux environment

### Step 1: Start Minikube

Start the Kubernetes cluster:

```bash
minikube start -p k8s-multi
```

Check the cluster nodes:

```bash
kubectl get nodes
```

### Step 2: Create the Namespace

Create a dedicated namespace for the application:

```bash
kubectl create namespace node-app
```

If the namespace already exists, you can continue without recreating it.

Set it as the current namespace:

```bash
kubectl config set-context --current --namespace=node-app
```

### Step 3: Enable NGINX Ingress

Enable the Minikube Ingress addon:

```bash
minikube addons enable ingress -p k8s-multi
```

Check the Ingress Controller:

```bash
kubectl get pods -n ingress-nginx
```

### Step 4: Deploy MongoDB

Apply the MongoDB Service:

```bash
kubectl apply -f mongo-service.yaml
```

Apply the MongoDB StatefulSet:

```bash
kubectl apply -f mongo-statefulset.yaml
```

### Step 5: Configure the Backend

Create the backend ConfigMap:

```bash
kubectl apply -f backend-configmap.yaml
```

Deploy the backend:

```bash
kubectl apply -f backend-deployment.yaml
```

Create the backend Service:

```bash
kubectl apply -f backend-service.yaml
```

### Step 6: Deploy the Frontend

Deploy the frontend:

```bash
kubectl apply -f frontend-deployment.yaml
```

Create the frontend Service:

```bash
kubectl apply -f frontend-service.yaml
```

### Step 7: Configure Ingress

Apply the Ingress configuration:

```bash
kubectl apply -f ingress.yaml
```

Check the Ingress:

```bash
kubectl get ingress
```

### Step 8: Verify Resources

Check all application Pods:

```bash
kubectl get pods -o wide
```

Check Services:

```bash
kubectl get services
```

Check StatefulSets:

```bash
kubectl get statefulsets
```

Check PersistentVolumeClaims:

```bash
kubectl get pvc
```

---

## Accessing the Application

Port forwarding is used to access the NGINX Ingress Controller from the local machine.

Run:

```bash
kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8081:80
```

Keep this terminal running.

On Windows, open the hosts file as Administrator:

```text
C:\Windows\System32\drivers\etc\hosts
```

Add the following entry:

```text
127.0.0.1 employee.local
```

Open the application in your browser:

```text
http://employee.local:8081
```

---

## Useful Kubernetes Commands

**View Pods**

```bash
kubectl get pods -o wide
```

**View Deployments**

```bash
kubectl get deployments
```

**View Services**

```bash
kubectl get services
```

**View Ingress**

```bash
kubectl get ingress
```

**View PersistentVolumeClaims**

```bash
kubectl get pvc
```

**Check Backend Logs**

```bash
kubectl logs deployment/backend-deployment
```

**Check MongoDB Logs**

```bash
kubectl logs mongo-0
```

**Scale Backend Deployment**

```bash
kubectl scale deployment backend-deployment --replicas=2
```

**Describe a Pod**

```bash
kubectl describe pod <pod-name>
```

---

## Key Learnings

Through this project, I gained practical experience with:

- Deploying a MERN Stack application on Kubernetes.
- Containerizing frontend and backend applications.
- Deploying applications using Kubernetes Deployments.
- Managing persistent database storage using StatefulSets and PVCs.
- Configuring application environment variables using ConfigMaps.
- Understanding Kubernetes Services and internal DNS-based communication.
- Configuring Headless Services for StatefulSet networking.
- Exposing applications through NGINX Ingress.
- Scaling backend application replicas.
- Working with a multi-node Minikube cluster.
- Troubleshooting Kubernetes networking and application connectivity.

---

## Future Improvements

- Configure MongoDB Replica Sets for database high availability.
- Implement Kubernetes Secrets for database authentication.
- Configure Liveness and Readiness Probes.
- Set CPU and memory requests and limits.
- Implement Horizontal Pod Autoscaling (HPA).
- Integrate Prometheus and Grafana for monitoring.
- Automate Kubernetes deployments using GitHub Actions.

---

## Author

**Dean Alemao**

MSc Information Technology Student | Interested in DevOps, Cloud Computing and Software Development.

GitHub: [Deanalemao](https://github.com/Deanalemao)

---

*This project was built as a hands-on learning exercise to understand Kubernetes application deployment, networking, storage, scaling and orchestration.*
