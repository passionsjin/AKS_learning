# 연습 - Rating front end 배포

## 1. Rating front end - Deploy 생성
- yaml 생성 `code ratings-web-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ratings-web
spec:
  selector:
    matchLabels:
      app: ratings-web
  template:
    metadata:
      labels:
        app: ratings-web # the label for the pods and the deployments
    spec:
      containers:
      - name: ratings-web
        image: <acrname>.azurecr.io/ratings-web:v1 # IMPORTANT: update with your own repository
        imagePullPolicy: Always
        ports:
        - containerPort: 8080 # the application listens to this port
        env:
        - name: API # the application expects to connect to the API at this endpoint
          value: http://ratings-api.ratingsapp.svc.cluster.local
        resources:
          requests: # minimum resources required
            cpu: 250m
            memory: 64Mi
          limits: # maximum resources allocated
            cpu: 500m
            memory: 512Mi
```
- Container Image의 <acrname> 변경 필수
```bash
kubectl apply \
--namespace ratingsapp \
-f ratings-web-deployment.yaml
kubectl get pods --namespace ratingsapp -l app=ratings-web -w
kubectl get deployment ratings-web --namespace ratingsapp
```
## 2. Rating front end - Service 생성
- yaml 생성 `code ratings-web-service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: ratings-web
spec:
  selector:
    app: ratings-web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
```
- 적용
```bash
kubectl apply \
    --namespace ratingsapp \
    -f ratings-web-service.yaml
```
- 확인
```bash
kubectl get service ratings-web --namespace ratingsapp -w
```
- 외부 IP로 프론트에 접근가능.
