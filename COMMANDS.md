# ShopEasy DevSecOps – Run & Screenshot Steps (PowerShell)

## 1. Push to GitHub
git init; git add .; git commit -m "Initial ShopEasy DevSecOps project"
git branch -M main
git remote add origin https://github.com/<you>/ecommerce-devsecops.git
git push -u origin main

## 2. Build BLUE (v1)
docker build -t shopeasy-app:v1 .
minikube image load shopeasy-app:v1        # if using minikube

## 3. Build GREEN (v2)
# In app/index.html change: orange (#e65100) -> green (#2e7d32), footer "Version 2.0 (GREEN)"
docker build -t shopeasy-app:v2 .
minikube image load shopeasy-app:v2

## 4. Trivy scan  (screenshot)
trivy image shopeasy-app:v1
# or: docker run --rm -v //var/run/docker.sock:/var/run/docker.sock aquasec/trivy image shopeasy-app:v1

## 5. Deploy
kubectl apply -f k8s/
kubectl get pods,svc                        # screenshot

## 6. Open BLUE  (screenshot)
kubectl port-forward service/shopeasy-service 8084:80
# browser -> http://localhost:8084
# or: minikube service shopeasy-service

## 7. Switch to GREEN (zero downtime)  (screenshot)
kubectl patch service shopeasy-service -p '{\"spec\":{\"selector\":{\"app\":\"shopeasy\",\"version\":\"green\"}}}'
kubectl port-forward service/shopeasy-service 8085:80
# browser -> http://localhost:8085

## 8. Rollback to BLUE
kubectl patch service shopeasy-service -p '{\"spec\":{\"selector\":{\"app\":\"shopeasy\",\"version\":\"blue\"}}}'

## 9. Jenkins (http://localhost:8081)
New Item -> "ShopEasy-DevSecOps" -> Pipeline -> Pipeline script from SCM -> Git -> repo URL -> branch main -> Save -> Build Now
Screenshot: build status, Console Output, Pipeline Overview
