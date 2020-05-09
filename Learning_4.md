# 연습 - Rating API 배포
## 1. Rating API의 Kubernetes 배포 만들기
- yaml 파일을 통해 deployment 생성
```bash
code ratings-api-deployment.yaml
```
아래 yaml 사용
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ratings-api
spec:
  selector:
    matchLabels:
      app: ratings-api
  template:
    metadata:
      labels:
        app: ratings-api # the label for the pods and the deployments
    spec:
      containers:
      - name: ratings-api
        image: <acrname>.azurecr.io/ratings-api:v1 # IMPORTANT: update with your own repository
        imagePullPolicy: Always
        ports:
        - containerPort: 3000 # the application listens to this port
        env:
        - name: MONGODB_URI # the application expects to find the MongoDB connection details in this environment variable
          valueFrom:
            secretKeyRef:
              name: mongosecret # the name of the Kubernetes secret containing the data
              key: MONGOCONNECTION # the key inside the Kubernetes secret containing the data
        resources:
          requests: # minimum resources required
            cpu: 250m
            memory: 64Mi
          limits: # maximum resources allocated
            cpu: 500m
            memory: 256Mi
        readinessProbe: # is the container ready to receive traffic?
          httpGet:
            port: 3000
            path: /healthz
        livenessProbe: # is the container healthy?
          httpGet:
            port: 3000
            path: /healthz
```
- 수정필요 : container의 image 주소
-readinessProbe 및 livenessProbe: /healthz에서 상태 확인 엔드포인트 사용
- deployment 적용
```bash
kubectl apply \
    --namespace ratingsapp \
    -f ratings-api-deployment.yaml
```
- pod및service 상태확인
```bash
kubectl get pods \
    --namespace ratingsapp \
    -l app=ratings-api -w
kubectl get deployment ratings-api --namespace ratingsapp
```
## 2. Rating API의 Kubernetes 서비스 생성
- yaml 생성 `code ratings-api-service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: ratings-api
spec:
  selector:
    app: ratings-api
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: ClusterIP
```
- 적용
```bash
kubectl apply \
    --namespace ratingsapp \
    -f ratings-api-service.yaml
```
- 확인
```bash
kubectl get service ratings-api --namespace ratingsapp
- Endpoint 확인
```bash
kubectl get endpoints ratings-api --namespace ratingsapp
```

```
