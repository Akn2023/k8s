# Kubernetes Crash Course: Complete Notes
## Learn the Basics and Build a Microservice Application

---

## Course Overview
- **Duration:** 2 hours video + 2 hours hands-on labs (4 hours total)
- **Format:** Video lectures + interactive labs via code.wiki/kubernetes-labs
- **Target Audience:** Beginners to DevOps & Cloud
- **Key Advantage:** Free labs with no local setup needed; learn by doing

---

## Table of Contents
1. [Introduction & Setup](#introduction--setup)
2. [Containers & Docker](#containers--docker)
3. [Kubernetes Fundamentals](#kubernetes-fundamentals)
4. [Kubernetes Architecture](#kubernetes-architecture)
5. [Kubectl CLI](#kubectl-cli)
6. [Pods](#pods)
7. [ReplicaSets](#replicasets)
8. [Deployments](#deployments)
9. [Services & Networking](#services--networking)
10. [Hands-On Project: Voting App](#hands-on-project-voting-app)
11. [Key Commands Reference](#key-commands-reference)
12. [Tips & Best Practices](#tips--best-practices)

---

## 1. Introduction & Setup

### Course Structure
- **Learning Model:** Seamless integration of video concepts followed by labs
- **Retention Focus:** 30% of what we see, 80%+ of what we experience → emphasis on hands-on labs
- **Resources Available:**
  - Free downloadable PDF slides and notes
  - Interactive labs with terminal access
  - Solution videos for each lab
  - Free community support via Slack (code.wiki/community)

### Getting Started
1. Go to **code.wiki/kubernetes-labs** (or scan QR code in video)
2. Enroll in the free course
3. Download resources (PDF slides, notes, transcripts)
4. Access hands-on labs section
5. Click "Start Lab" button to begin
6. Labs open in browser with terminal + guided questions

---

## 2. Containers & Docker

### The Problem Containers Solve
**Dependency Management Challenge:**
- Different applications need different versions of libraries
- Example: App A uses Flask 2.2, App B uses Flask 2.1
- Hard to manage on same OS
- Impossible to ensure identical environments across dev, test, and prod

### What Are Containers?
**Definition:** Isolated environments on systems that run applications with their own libraries and dependencies without affecting other containers.

**Key Benefits:**
- Application + system-level + app-level dependencies packaged together
- Same container runs anywhere (Linux machine)
- No dependency conflicts between applications
- Can run applications based on different OS flavors on same host

### Docker Basics
**Docker Dockerfile Example:**
```dockerfile
FROM python:3.9
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

**Key Commands:**
- `docker build` — Build an image from Dockerfile
- `docker run` — Run a container from an image
- **Result:** One image can run on any system with Docker installed

---

## 3. Kubernetes Fundamentals

### What Is Kubernetes?
**Definition:** Container orchestration platform that automates deployment, scaling, and management of containerized applications.

### Problems Kubernetes Solves
| Without Kubernetes | With Kubernetes |
|---|---|
| Run 1 app instance | Run 1,000+ instances with single command |
| Manual scaling | Automatic scaling based on load |
| Manual updates | Rolling updates with canary deployments |
| Manual failover | Auto-recovery of failed containers |
| No load balancing | Intelligent load distribution |
| Manual rollback | One-command rollback on failure |

### Declarative State Management
**Key Concept:** Define desired state in code; Kubernetes ensures it's maintained.

**Example:**
```
Define: 4 services total
- Web server: 3 instances
- Payment service: 2 instances
- Cache service: 3 instances
- Database: 1 instance

Kubernetes Guarantees: Maintains this state 24/7
```

---

## 4. Kubernetes Architecture

### Cluster Components

#### **Control Plane (formerly Master Node)**
Manages the entire cluster and orchestration.

**Components:**
1. **API Server**
   - Front-end for Kubernetes
   - All users, tools, and CLIs communicate with this
   - Entry point for all requests

2. **etcd (Distributed Key-Value Store)**
   - Stores all cluster data
   - Information about nodes, applications, configurations
   - Single source of truth for cluster state

3. **Controllers**
   - Brain behind orchestration
   - Monitor nodes, containers, endpoints
   - Take action when something fails
   - Bring up new containers when old ones fail

4. **Scheduler**
   - Distributes work (containers) across nodes
   - Assigns new containers to appropriate nodes
   - Considers resource requirements and constraints

#### **Worker Nodes**
Run the actual containers.

**Components on Each Node:**
1. **Kubelet**
   - Agent running on each node
   - Ensures containers run as expected
   - Reports node status to Control Plane

2. **Kube-Proxy**
   - Maintains networking rules on nodes
   - Enables communication between pods and services
   - Networking component; forwards traffic

3. **Container Runtime**
   - Runs containers (Docker, containerd, etc.)
   - Historically Docker, now using containerd
   - Kubernetes no longer supports Docker directly

### Architecture Visualization
```
┌─────────────────────────────────────────┐
│         KUBERNETES CLUSTER              │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │   CONTROL PLANE (Master)        │   │
│  │ ┌──────────────────────────┐    │   │
│  │ │ API Server               │    │   │
│  │ │ etcd (Key-Value Store)   │    │   │
│  │ │ Controllers & Scheduler  │    │   │
│  │ └──────────────────────────┘    │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌──────────────┐  ┌──────────────┐   │
│  │ Worker Node  │  │ Worker Node  │   │
│  │ ├─ Kubelet   │  │ ├─ Kubelet   │   │
│  │ ├─ Proxy     │  │ ├─ Proxy     │   │
│  │ ├─ Container │  │ ├─ Container │   │
│  │ │  Runtime   │  │ │  Runtime   │   │
│  │ └─ Containers│  │ └─ Containers│   │
│  └──────────────┘  └──────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

---

## 5. Kubectl CLI

### What Is Kubectl?
- **Pronunciation:** Kube-control (not kube-cuddle, though commonly used)
- **Function:** Command-line utility to operate Kubernetes clusters
- **Used For:** Deploy, manage, scale, delete, and monitor applications

### Basic Commands

**Check Version:**
```bash
kubectl version
# Lists client version and server version
```

**View Cluster Nodes:**
```bash
kubectl get nodes
# Shows all nodes in cluster

kubectl get nodes -o wide
# Shows detailed info: internal IP, OS image, kernel version, container runtime
```

**Help & Documentation:**
```bash
kubectl help
# Lists basic commands and options
```

---

## 6. Pods

### What Is a Pod?

**Definition:** Smallest deployable unit in Kubernetes; encapsulates a container (or containers).

**Key Characteristics:**
- Single instance of an application
- Typically one container per pod (1:1 relationship)
- Can have multiple containers (helper/sidecar containers)
- Containers in same pod share:
  - Network namespace (same IP address)
  - Storage volumes
  - Configuration

### Scaling Pods
**Rule:** Don't add more containers to one pod; create more pods instead.
- To scale up: Create more pods
- To scale down: Delete pods
- Each pod runs one app instance

### Creating Pods: Two Approaches

#### **Imperative Approach (Using Commands)**
```bash
kubectl run <pod-name> --image=<image-name>

# Example:
kubectl run nginx-pod --image=nginx
```

**Limitations:** 
- One pod at a time
- Hard to track/version
- Not scalable for production

#### **Declarative Approach (Using YAML) - RECOMMENDED**

**Pod YAML Structure:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    tier: frontend
spec:
  containers:
  - name: nginx
    image: nginx
```

**YAML Components Explained:**

| Field | Purpose | Example |
|-------|---------|---------|
| `apiVersion` | API version for object type | `v1` (for Pod) |
| `kind` | Object type | `Pod`, `ReplicaSet`, `Deployment` |
| `metadata` | Object info (name, labels) | `name: nginx-pod` |
| `metadata.labels` | Key-value pairs for filtering | `app: nginx` |
| `spec` | Object specifications | Container details |
| `spec.containers` | List of containers in pod | Container image, name |

**Important Notes on YAML:**
- **Indentation matters** (use 2 spaces, not tabs)
- **Case-sensitive** (`kind: Pod` NOT `kind: pod`)
- Four top-level fields are required: `apiVersion`, `kind`, `metadata`, `spec`
- Under `metadata`, only expected properties allowed (name, labels, etc.)
- Under `labels`, any custom key-value pairs allowed

### Creating & Managing Pods

**Create Pod from YAML:**
```bash
kubectl create -f pod-definition.yaml
# or
kubectl apply -f pod-definition.yaml
```

**List Pods:**
```bash
kubectl get pods
# Shows pod name, ready status, restarts, age

kubectl get pods -o wide
# Shows pod IP, node assignment, container status
```

**Get Pod Details:**
```bash
kubectl describe pod <pod-name>
# Shows creation time, labels, containers, events, logs
```

**View Pod Logs:**
```bash
kubectl logs <pod-name>
```

**Delete Pod:**
```bash
kubectl delete pod <pod-name>
```

### Pod Limitations
- Single point of failure (if pod crashes, no backup)
- No auto-recovery
- No automatic scaling
- No rolling updates
- **Solution:** Use ReplicaSets or Deployments

---

## 7. ReplicaSets

### What Is a ReplicaSet?

**Definition:** Kubernetes object that ensures a specified number of pod replicas are running at all times.

**Key Responsibilities:**
- Monitor pods continuously
- Replace failed pods automatically
- Ensure desired number maintained (e.g., always 3 replicas)
- Balance load across multiple nodes
- Enable horizontal scaling

### Why ReplicaSets Matter

**Scenario 1: High Availability**
```
Without ReplicaSet: 1 pod fails → Application down
With ReplicaSet: 1 pod fails → Auto-replaced immediately
```

**Scenario 2: Load Balancing**
```
Single pod can't handle 1000 users
ReplicaSet creates 10 pods across 10 nodes
Load distributed evenly
```

### Pod vs ReplicaSet Relationship
- **Pod:** Lives on exactly ONE node (1:1)
- **ReplicaSet:** Spans ENTIRE cluster, deploys pods on multiple nodes

### ReplicaSet YAML Structure

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-app-replica-set
  labels:
    app: my-app
    type: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      name: my-app-pod
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx
```

**Key Fields:**

| Field | Purpose | Example |
|-------|---------|---------|
| `apiVersion` | For ReplicaSet use `apps/v1` | `apps/v1` |
| `spec.replicas` | Number of pod copies to maintain | `3` |
| `spec.selector` | Label filter to identify pods | `matchLabels: {app: my-app}` |
| `spec.template` | Pod template to create replicas | Full pod definition |

### Understanding Labels & Selectors

**Why Needed?**
- ReplicaSet can manage pre-existing pods (not just newly created ones)
- Need way to identify which pods belong to which ReplicaSet
- Multiple ReplicaSets may exist in same cluster

**How It Works:**
```yaml
# Pod has label
labels:
  app: my-app

# ReplicaSet selector matches this label
selector:
  matchLabels:
    app: my-app
    # These must MATCH for ReplicaSet to manage the pod
```

**Practical Use:**
```
Scenario: 3 front-end pods exist before ReplicaSet creation
ReplicaSet with selector "app: frontend" found them
ReplicaSet now manages these 3 existing pods
If 1 fails → ReplicaSet creates new one to maintain count
```

### Scaling ReplicaSets

**Method 1: Edit YAML (Recommended)**
```bash
# Edit replicas field in YAML
replicas: 6

# Apply changes
kubectl replace -f replica-set-definition.yaml
```

**Method 2: Scale Command**
```bash
kubectl scale --replicas=6 -f replica-set-definition.yaml

# OR using resource name
kubectl scale --replicas=6 replicaset my-app-replica-set
```

**Important:** Using kubectl scale command doesn't update the YAML file. Best practice is to update YAML files and use `kubectl replace`.

### ReplicaSet Management Commands

| Command | Purpose |
|---------|---------|
| `kubectl create -f rs.yaml` | Create ReplicaSet |
| `kubectl get replicasets` | List ReplicaSets |
| `kubectl get replicasets -o wide` | Detailed info |
| `kubectl describe replicaset <name>` | Full details |
| `kubectl replace -f rs.yaml` | Update ReplicaSet |
| `kubectl scale --replicas=5 replicaset <name>` | Scale replicas |
| `kubectl delete replicaset <name>` | Delete ReplicaSet |
| `kubectl get pods` | View pods created by ReplicaSet |

### ReplicaSet Output Fields

```
NAME                    DESIRED   CURRENT   READY   AGE
my-app-replica-set      3         3         3       2m
```

| Field | Meaning |
|-------|---------|
| `DESIRED` | How many replicas should exist |
| `CURRENT` | How many replicas exist now |
| `READY` | How many are ready to receive traffic |
| `AGE` | How long the ReplicaSet has been running |

---

## 8. Deployments

### What Is a Deployment?

**Definition:** Kubernetes object that manages ReplicaSets and enables rolling updates, rollbacks, and declarative application management.

**Key Capabilities:**
- Manages ReplicaSets automatically
- Rolling updates (update pods one at a time)
- Canary deployments (update percentage of pods)
- Automatic rollback on failure
- Version control of application state
- Pause and resume updates

### Deployment vs ReplicaSet vs Pod

```
Deployment (Highest level)
    └── Manages ReplicaSets
            └── Manages Pods
                    └── Runs Container

Hierarchy: Deployment > ReplicaSet > Pod > Container
```

### Why Use Deployments?

**Scenario: Update Application**

Without Deployment:
1. Stop all old pods
2. Start all new pods
3. Downtime occurs

With Deployment (Rolling Update):
1. Create new ReplicaSet with new version
2. Gradually scale up new, scale down old
3. One pod at a time updated
4. Zero downtime

### Deployment YAML Structure

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
```

**Similarities to ReplicaSet:**
- Same `apiVersion: apps/v1`
- Same `selector`, `template`, `replicas` structure
- Key difference: `kind: Deployment` instead of `ReplicaSet`

### Deployment Management

**Create Deployment:**
```bash
kubectl create -f deployment.yaml
```

**View Deployments:**
```bash
kubectl get deployments
kubectl get deployment <name> -o wide
```

**Update Image (Rolling Update):**
```bash
kubectl set image deployment/my-app-deployment nginx=nginx:1.16.0
```

**View Rollout History:**
```bash
kubectl rollout history deployment/my-app-deployment
```

**Rollback to Previous Version:**
```bash
kubectl rollout undo deployment/my-app-deployment
```

**Check Rollout Status:**
```bash
kubectl rollout status deployment/my-app-deployment
```

---

## 9. Services & Networking

### What Are Services?

**Problem:** Pods are temporary; their IPs change. Users need stable way to access applications.

**Solution:** Services provide stable, persistent endpoint for accessing pods.

### Service Types

#### **1. ClusterIP (Default)**
- Internal-only access
- Pods communicate with each other
- Not accessible from outside cluster

#### **2. NodePort**
- Exposes service on node's IP + port
- Accessible from outside cluster
- Port range: 30000-32767

#### **3. LoadBalancer**
- Exposes service via external load balancer
- Works with cloud providers (AWS, GCP, Azure)
- Best for production external access

### Service Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  type: NodePort
  ports:
  - targetPort: 80        # Pod port
    port: 80              # Service port
    nodePort: 30008       # Node port (30000-32767)
  selector:
    app: my-app           # Route to pods with this label
```

**Flow:**
```
External User → Node:30008 → Service:80 → Pod:80 (Container)
```

---

## 10. Hands-On Project: Voting App

### Project Overview
Deploy a microservice application (voting app) to Kubernetes cluster using:
- Deployments (for scaling)
- Services (for networking)
- Multiple containers/services working together

### Services in Voting App
1. **Frontend (Voting Interface)** - What users see
2. **Backend (Vote Processor)** - Processes votes
3. **Database (Vote Storage)** - PostgreSQL or Redis
4. **Results Service** - Displays vote results

### Deployment Steps (Covered in Video)
1. Create Deployment YAML for each service
2. Create Service YAML for each component
3. Deploy using `kubectl create -f <file>`
4. Verify pods are running: `kubectl get pods`
5. Test by accessing through NodePort service

---

## 11. Key Commands Reference

### Cluster Information
```bash
kubectl cluster-info          # Cluster details
kubectl get nodes            # List all nodes
kubectl describe node <name> # Node details
```

### Pod Commands
```bash
kubectl run <pod-name> --image=<image>     # Create pod (imperative)
kubectl create -f pod.yaml                 # Create pod (declarative)
kubectl apply -f pod.yaml                  # Apply config (create or update)
kubectl get pods                           # List pods
kubectl get pods -o wide                   # Detailed pod info
kubectl describe pod <name>                # Pod details
kubectl logs <pod-name>                    # View logs
kubectl delete pod <name>                  # Delete pod
kubectl exec -it <pod-name> /bin/bash     # Execute command in pod
```

### ReplicaSet Commands
```bash
kubectl create -f replicaset.yaml          # Create ReplicaSet
kubectl get replicasets                    # List ReplicaSets
kubectl get rs                             # Shorthand for replicasets
kubectl describe replicaset <name>         # ReplicaSet details
kubectl replace -f replicaset.yaml         # Update ReplicaSet
kubectl scale --replicas=6 replicaset <name>  # Scale replicas
kubectl delete replicaset <name>           # Delete ReplicaSet
```

### Deployment Commands
```bash
kubectl create -f deployment.yaml          # Create Deployment
kubectl get deployments                    # List Deployments
kubectl describe deployment <name>         # Deployment details
kubectl set image deployment/<name> <container>=<image>  # Update image
kubectl rollout history deployment/<name>  # View update history
kubectl rollout undo deployment/<name>     # Rollback to previous
kubectl rollout status deployment/<name>   # Check rollout progress
kubectl delete deployment <name>           # Delete Deployment
```

### Service Commands
```bash
kubectl create -f service.yaml             # Create Service
kubectl get services                       # List Services
kubectl describe service <name>            # Service details
kubectl delete service <name>              # Delete Service
```

### File-Based Operations
```bash
kubectl create -f <file.yaml>              # Create resource from file
kubectl apply -f <file.yaml>               # Apply resource (idempotent)
kubectl replace -f <file.yaml>             # Replace/update resource
kubectl delete -f <file.yaml>              # Delete resource from file
```

---

## 12. Tips & Best Practices

### YAML Development Tips

**1. Use IDE with YAML Support**
- VS Code with Kubernetes extension
- JetBrains IDEs (IntelliJ, PyCharm)
- Syntax highlighting + validation

**2. Indentation Rules**
- Use 2 spaces (NOT tabs)
- Maintain consistent indentation
- Children must be indented more than parents
- Siblings must be at same indentation level

**3. Validation**
```bash
kubectl apply -f file.yaml --dry-run=client  # Test without applying
```

### Best Practices

**1. Always Use YAML (Declarative)**
- Commands like `kubectl run` are for demos only
- YAML enables version control
- Facilitates CI/CD pipelines
- Makes configs reproducible

**2. Use Labels Strategically**
- `app: <app-name>` - Identify application
- `version: v1.0` - Track versions
- `tier: frontend/backend` - Application layer
- `environment: dev/prod` - Environment

**3. Use Namespaces for Organization**
- Separate dev/test/prod environments
- Isolate teams or projects
- Prevent accidental deletions

**4. Set Resource Requests & Limits**
```yaml
spec:
  containers:
  - name: app
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "500m"
        memory: "512Mi"
```

**5. Use Deployments for Production**
- Never use Pods directly in production
- Use ReplicaSets via Deployments
- Enables rolling updates and rollbacks

**6. Health Checks**
```yaml
spec:
  containers:
  - name: app
    livenessProbe:      # Restart if unhealthy
      httpGet:
        path: /health
        port: 8080
    readinessProbe:     # Don't send traffic if not ready
      httpGet:
        path: /ready
        port: 8080
```

### Common Mistakes to Avoid

❌ **Wrong:** Using `kind: pod` (lowercase)
✅ **Correct:** Using `kind: Pod` (capitalized)

❌ **Wrong:** Using tabs for indentation
✅ **Correct:** Using spaces (2 or 4 consistently)

❌ **Wrong:** Not matching selector labels with pod labels
✅ **Correct:** Ensure `selector.matchLabels` matches `template.metadata.labels`

❌ **Wrong:** Wrong API version for object type
✅ **Correct:** Use `v1` for Pod, `apps/v1` for ReplicaSet/Deployment

❌ **Wrong:** Using `kubectl scale` and expecting file to update
✅ **Correct:** Always edit YAML file and use `kubectl replace`

---

## Learning Resources & Support

**Free Resources in Course:**
- PDF slides and notes (downloadable)
- Interactive hands-on labs (code.wiki/kubernetes-labs)
- Solution videos for each lab
- Community Slack channel (code.wiki/community)

**Recommended Learning Path:**
1. Watch video section
2. Immediately go to corresponding lab
3. Practice the concepts
4. Review solution video
5. Move to next topic

**Additional Kubernetes Topics (Beyond This Course):**
- Advanced networking (Ingress, Network Policies)
- Storage (Volumes, PersistentVolumes)
- ConfigMaps & Secrets
- StatefulSets
- DaemonSets
- Horizontal Pod Autoscaling
- RBAC (Role-Based Access Control)
- Helm (Package manager for Kubernetes)

---

## Quick Reference: YAML Templates

### Basic Pod Template
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
  - name: my-container
    image: my-image:latest
```

### ReplicaSet Template
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: my-image:latest
```

### Deployment Template
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: my-image:latest
        ports:
        - containerPort: 8080
```

### Service Template (NodePort)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
  - targetPort: 8080
    port: 8080
    nodePort: 30008
```

---

## Summary

**Kubernetes Hierarchy:**
```
Application Code
       ↓
Docker Container (via Dockerfile)
       ↓
Kubernetes Pod (smallest deployable unit)
       ↓
ReplicaSet or Deployment (multiple pods + auto-recovery)
       ↓
Service (networking + load balancing)
       ↓
Cluster (multiple nodes)
```

**Key Takeaway:** Kubernetes automates container deployment, scaling, and management—eliminating dependency problems and enabling zero-downtime updates.

---

*Notes compiled from Kubernetes Crash Course by KodeKloud*
*Video: https://www.youtube.com/watch?v=XuSQU5Grv1g*
*Labs: https://code.wiki/kubernetes-labs*
