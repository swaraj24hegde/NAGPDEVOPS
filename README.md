# NAGPDEVOPS
# DevOps & Kubernetes Assignment - README

## **Project Overview**
This project demonstrates a complete **CI/CD pipeline using Docker & Kubernetes**, where we:
- Built a **Python Flask app**
- Containerized it using **Docker**
- Deployed it to **Kubernetes** with **ConfigMaps & Secrets**
- Used **Minikube** as the Kubernetes cluster
- Exposed the app via a Kubernetes **Service**

---

## **Project Structure**
📂 devops-k8s-app
 ├── app.py              # Python Flask Application
 ├── Dockerfile          # Dockerfile for containerization
 ├── .dockerignore       # Ignore unnecessary files in Docker build
 ├── deployment.yaml     # Kubernetes Deployment (3 replicas)
 ├── service.yaml        # Kubernetes Service (LoadBalancer)
 ├── configmap.yaml      # Kubernetes ConfigMap
 ├── secret.yaml         # Kubernetes Secret

---

## **Step 1: Set Up the Project**
```sh
mkdir devops-k8s-app && cd devops-k8s-app
python3 -m venv venv
source venv/bin/activate
pip install flask
```

Create `app.py`:
```python
from flask import Flask
app = Flask(__name__)
@app.route('/')
def home():
    return "Hello, DevOps and Kubernetes!"
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```
Run the app locally:
```sh
python app.py
```

---

## **Step 2: Containerize with Docker**
Create `Dockerfile`:
```dockerfile
FROM python:3.9
WORKDIR /app
COPY . /app
RUN pip install flask
EXPOSE 5000
CMD ["python", "app.py"]
```

Build & run the Docker image:
```sh
docker build -t devops-k8s-app .
docker run -p 5000:5000 devops-k8s-app
```

---

## **Step 3: Push to Docker Hub**
```sh
docker login
docker tag devops-k8s-app <your-docker-username>/devops-k8s-app:latest
docker push <your-docker-username>/devops-k8s-app:latest
```

---

## **Step 4: Deploy to Kubernetes**
Start Minikube:
```sh
minikube start --driver=docker
```
Create Kubernetes namespaces:
```sh
kubectl create namespace public
kubectl create namespace private
```

Create `deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devops-k8s-app
  namespace: public
spec:
  replicas: 3
  selector:
    matchLabels:
      app: devops-k8s-app
  template:
    metadata:
      labels:
        app: devops-k8s-app
    spec:
      containers:
      - name: devops-k8s-app
        image: swarajhegde/devops-k8s-app:latest
        ports:
        - containerPort: 5000
```
Apply the deployment:
```sh
kubectl apply -f deployment.yaml
```
Check running pods:
```sh
kubectl get pods -n public
```

---

## **Step 5: Expose the Application**
Create `service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: devops-k8s-service
  namespace: public
spec:
  selector:
    app: devops-k8s-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: LoadBalancer
```
Apply the service:
```sh
kubectl apply -f service.yaml
```
Get the service URL:
```sh
minikube service devops-k8s-service -n public --url
```

---

## **Step 6: ConfigMap & Secrets**
Create `configmap.yaml`:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: devops-config
  namespace: public
data:
  APP_ENV: "production"
  LOG_LEVEL: "debug"
```
Apply ConfigMap:
```sh
kubectl apply -f configmap.yaml
```

Create `secret.yaml`:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: devops-secret
  namespace: public
type: Opaque
data:
  DB_USER: c3dhcmFq  # Base64 encoded "swaraj"
  DB_PASSWORD: c2VjcmV0  # Base64 encoded "secret"
```
Apply Secret:
```sh
kubectl apply -f secret.yaml
```

---

## **Step 7: Commit Code to Azure Repo**
```sh
git init
git add .
git commit -m "Initial commit"
git remote add origin https://dev.azure.com/<your-org>/<your-repo>
git push -u origin main
```
---

## **Step 8: Cleanup (After Submission)**
```sh
kubectl delete deployment devops-k8s-app -n public
kubectl delete svc devops-k8s-service -n public
kubectl delete namespace public private
minikube stop
docker rmi <your-docker-username>/devops-k8s-app:latest
```


