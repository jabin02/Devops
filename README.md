# Game Application Deployment on Amazon EKS

## 1. **Deployment**
Created a deployment for the **2048 Game App** with 5 replicas:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: app-2024
  name: deployment-2048
spec:
  replicas: 5
  selector:
    matchLabels:
      app.kubernetes.io/name: app-2048
  template:
    metadata:
      labels:
        app.kubernetes.io/name: app-2048
    spec:
      containers:
      - name: app-2048
        image: public.ecr.aws/l6m2t8p7/docker-2048:latest
        ports:
        - containerPort: 80
```

## 2. **Service**
Created a **NodePort** service to expose the application:

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: app-2024
  name: service-2048
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
  selector:
    app.kubernetes.io/name: app-2048
```

## 3. **Ingress**
Configured an **ALB Ingress** to expose the app publicly:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: app-2024
  name: ingress-2048
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: service-2048
            port:
              number: 80
```

## 4. **IAM Permissions**
To resolve issues with **ALB health checks**, the IAM role `AmazonEKSLoadBalancerControllerRole` was updated with the following permissions:

```json
"Action": [
  "ec2:AuthorizeSecurityGroupIngress",
  "elasticloadbalancing:DescribeTargetHealth"
],
"Resource": "*"
```

## 5. **Verification**

Confirmed **pods**, **services**, and **ingress**:

```bash
kubectl get all -n app-2024
kubectl describe ingress ingress-2048 -n app-2024
```

Accessed the application via the **ALB DNS URL**:

```
http://k8s-app2024-ingress2-<unique-id>.us-east-1.elb.amazonaws.com
```

## 6. **Troubleshooting**

### **Issue 1: 503 Service Temporarily Unavailable**
- **Cause**: Missing permissions for `DescribeTargetHealth`.
- **Solution**: Added the required IAM permissions.

### **Issue 2: Target Group Health Check Failure**
- **Cause**: Misconfigured paths or missing security group ingress rules.
- **Solution**: Ensured health check path `/` and port `80` were correctly configured.

---

## 7. **Future Plan: Stock Prediction App**

The next step involves replacing the **2048 Game Application** with a **Stock Prediction App**. Below is the planned deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: stockprediction
  name: stock-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app.kubernetes.io/name: stock-app
  template:
    metadata:
      labels:
        app.kubernetes.io/name: stock-app
    spec:
      containers:
      - name: stock-app
        image: <ECR-URL>/stock-management-app:latest
        ports:
        - containerPort: 8501
```

---

## 8. **Tools Used**
- **Kubernetes**: Deployment, Service, Ingress
- **AWS EKS**: Managed Kubernetes cluster
- **IAM**: Role permissions for ALB
- **Docker**: Containerized applications
- **kubectl**: CLI for managing Kubernetes resources

---

## 9. **Results**
- Successfully deployed the **2048 Game Application** on Amazon EKS.
- Configured and troubleshooted ALB to expose the application publicly.
- Application is now accessible via the **ALB DNS URL**.

---

**Next Steps:** Transition to the **Stock Prediction App** with enhanced capabilities and updated deployment configurations.
