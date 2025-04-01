# **Argo Rollout Strategies Playground**

This repository provides example implementations of different **Argo Rollout strategies**, including **Blue-Green** and **Canary** deployments. It serves as a hands-on learning and testing environment for Kubernetes progressive delivery strategies.

## 📂 **Repository Structure**

```
argo-rollout-strategies-playground/
│-- argo-rollout-bluegreen/               # Blue-Green Deployment example
│-- argo-rollout-canary/                  # Canary Deployment example
│-- README.md                             # Documentation
```

## **Tools:**

- **Minikube**: A tool to run a local Kubernetes cluster for testing.
- **ArgoCD**: A GitOps continuous delivery tool to manage and deploy applications in Kubernetes.
- **Argo Rollouts**: A tool to manage advanced deployment strategies like Blue-Green and Canary in Kubernetes.

## **Deployment Strategies:**

- **Blue-Green**: Switch between two identical environments (Blue and Green) to ensure zero-downtime updates.
- **Canary**: Gradually roll out updates by directing a small percentage of traffic to the new version before full deployment.

---

## **Let’s Get Our Hands Dirty!**

### **Install Minikube:**

To install Minikube on your system, run the following command:

```bash
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

### **Start Minikube and Enable Ingress:**

Start Minikube and enable ingress for routing traffic:

```bash
minikube start
minikube addons enable ingress
```

### **Define Local Addresses for Testing:**

Add two local addresses to `/etc/hosts` for the test:

```bash
echo "$(minikube ip)    bluegreen.local" | sudo tee -a /etc/hosts > /dev/null
echo "$(minikube ip)    canary.local" | sudo tee -a /etc/hosts > /dev/null
```

### **Install ArgoCD in Your Minikube Cluster:**

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

#### Verify ArgoCD Installation:

Check the ArgoCD components are running:

```bash
kubectl get pods -n argocd
```

### **Install Argo Rollouts:**

```bash
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```

#### Verify Argo Rollouts Installation:

Check the Argo Rollouts controller pod is running:

```bash
kubectl get pods -n argo-rollouts
```

---

## **Blue-Green Strategy:**

### **Clone the Project:**

```bash
git clone https://github.com/raeis-i/argo-rollout-strategies-playground
cd ./argo-rollout-strategies-playground/argo-rollout-bluegreen
```

### **Apply Blue-Green Resources:**

Deploy the Blue-Green rollout example:

```bash
kubectl apply -f blue-green-rollout.yaml -f blue-green-svc.yaml -f blue-green-ingress.yml
```

### **Verify in Your Browser:**

- **Blue Version**: [http://bluegreen.local/active](http://bluegreen.local/active)
- **Green Version**: [http://bluegreen.local/preview](http://bluegreen.local/preview)

You can see why active and preview point to the same version of app.

```bash
raeisi@HP-ProBook:/devops$ kubectl get svc -o wide
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE     SELECTOR
kubernetes                  ClusterIP   10.96.0.1        <none>        443/TCP    24m     <none>
rollout-bluegreen-active    ClusterIP   10.110.141.176   <none>        8080/TCP   2m10s   app=blue-green-deployment,rollouts-pod-template-hash=67c79665f7
rollout-bluegreen-preview   ClusterIP   10.98.89.39      <none>        8080/TCP   2m10s   app=blue-green-deployment,rollouts-pod-template-hash=67c79665f7
```

##

```bash
raeisi@HP-ProBook:/devops$ kubectl describe ingress blue-green-ingress
Name:             blue-green-ingress
Labels:           <none>
Namespace:        default
Address:          192.168.49.2
Ingress Class:    nginx
Default backend:  <default>
Rules:
  Host             Path  Backends
  ----             ----  --------
  bluegreen.local
                   /active    rollout-bluegreen-active:8080 (10.244.0.35:8080,10.244.0.36:8080,10.244.0.37:8080 + 1 more...)
                   /preview   rollout-bluegreen-preview:8080 (10.244.0.35:8080,10.244.0.36:8080,10.244.0.37:8080 + 1 more...)
Annotations:       <none>
Events:
  Type    Reason  Age                   From                      Message
  ----    ------  ----                  ----                      -------
  Normal  Sync    105s (x2 over 2m26s)  nginx-ingress-controller  Scheduled for sync
raeisi@HP-ProBook:/devops$
```

### **Apply the Green Version:**

We will change the color of the application to green by updating the deployment.

```bash
kubectl patch rollout blue-green-deployment --type=merge -p '{"spec":{"template":{"spec":{"containers":[{"name":"blue-green","image":"samraeisi/simple-blue-green-app:v1","env":[{"name":"PAGE_COLOR","value":"green"}]}]}}}}'
```

### **Verify in Your Browser:**

- **Blue Version**: [http://bluegreen.local/active](http://bluegreen.local/active)
- **Green Version**: [http://bluegreen.local/preview](http://bluegreen.local/preview)

Promote

```bash
kubectl argo rollouts promote blue-green-deployment
```

##### Check URLs again, active and preview point to the green version.

---

## **Canary Strategy:**

### **Navigate to Canary Directory:**

```bash
cd ../argo-rollout-canary
```

### **Apply Canary Resources:**

```bash
kubectl apply -f canary-rollout.yml -f canary-svc.yml -f canary-ingress.yml
```

### **Verify in Your Browser:**

- Current Version: [http://canary.local](http://canary.local)
- If you try 4 times, you will see the blue version.

### **Apply the Green Version:**

Running the following kubectl patch command will update 1 pod with the new version:

```bash
kubectl patch rollout canary-rolling-update --type=merge -p '{"spec":{"template":{"spec":{"containers":[{"name":"blue-green","image":"samraeisi/simple-blue-green-app:v1","env":[{"name":"PAGE_COLOR","value":"green"}]}]}}}}'
```

### **Verify in Your Browser:**

- Canary Version: [http://canary.local](http://canary.local)
- If you try 4 times, you will see 3 times blue and 1 time green.

You can run the promote command, and each time, 25% of the pods will be updated.

```bash
kubectl argo rollouts promote canary-rolling-update
```

After running it 3 times, all pods will be updated to the green version.

### **Verify in Your Browser:**

- Now all traffic should be directed to the green version.

---

## 📜 **License**

This project is licensed under the MIT License.
