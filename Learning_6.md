# 연습 - Deploy an ingress for the front end
- NGINX를 실행하는 Kubernetes 수신 컨트롤러 배포
- ClusterIP를 사용하도록 평가 웹 서비스 다시 구성
- 평가 웹 서비스에 대한 수신 리소스 만들기
- 애플리케이션 테스트

## 1. Deploy a Kubernetes ingress controller running NGINX
- Namespace 생성
```bash
kubectl create namespace ingress
```
- Nginx ingress 설치
```bash
helm install nginx-ingress stable/nginx-ingress \
    --namespace ingress \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux
```
## 2. ClusterIP를 사용하도록 Rating Web 서비스 다시 구성
- `code ratings-web-service.yaml`
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
  type: ClusterIP
```
-적용
```bash
kubectl delete service \
    --namespace ratingsapp \
    ratings-web

kubectl apply \
    --namespace ratingsapp \
    -f ratings-web-service.yaml
```
## 3. Rating web 서비스에 대한 ingress 리소스 만들기
- 생성 `code ratings-web-ingress.yaml`
```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ratings-web-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: frontend.<ingress ip>.nip.io # IMPORTANT: update <ingress ip> with the dashed public IP of your ingress, for example frontend.13-68-177-68.nip.io
    http:
      paths:
      - backend:
          serviceName: ratings-web
          servicePort: 80
        path: /
```
- 적용
```bash
kubectl apply \
    --namespace ratingsapp \
    -f ratings-web-ingress.yaml
```
- 확인 url `frontend.52-170-173-31.nip.io`


