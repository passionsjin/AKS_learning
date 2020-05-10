# 연습 - Fontend ingress에 SSL/TLS 활성화
- Helm을 사용하여 인증서 관리자 배포
- Let's Encrypt용 ClusterIssuer 리소스 배포
- 수신 시 Rating Web 서비스에 대해 SSL/TLS 사용
- 애플리케이션 테스트

## 1. 인증서 관리자 배포
- 인증서 관리자는 클라우드 네이티브 환경에서 인증서 관리를 자동화하도록 해주는 Kubernetes 인증서 관리 컨트롤러
- namespace 생성
```bash
kubectl create namespace cert-manager
```
- jetstack Helm repo를 이용해 인증서 관리자를 추가
```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
```
- 설치
```bash
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.14/deploy/manifests/00-crds.yaml

helm install cert-manager \
    --namespace cert-manager \
    --version v0.14.0 \
    jetstack/cert-manager
```
- 확인
```bash
kubectl get pods --namespace cert-manager
```
## 2. Let's Encrypt용 ClusterIssuer 리소스 배포
- 인증서 관리자를 사용하면 웹 사이트의 인증서가 유효하고 최신 상태인지 확인하고, 심지어 인증서가 만료되기 전에 구성된 시간에 인증서를 갱신
- `code cluster-issuer.yaml`
```yaml
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: <your email> # IMPORTANT: Replace with a valid email from your organization
    privateKeySecretRef:
      name: letsencrypt
    solvers:
    - http01:
        ingress:
          class: nginx
```
- 적용
```bash
kubectl apply \
    --namespace ratingsapp \
    -f cluster-issuer.yaml
```
## 3. Ingress Rating web 서비스에 대해 SSL/TLS 사용
- `code ratings-web-ingress.yaml`
```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ratings-web-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  tls:
    - hosts:
      - frontend.<ingress ip>.nip.io # IMPORTANT: update <ingress ip> with the dashed public IP of your ingress, for example frontend.13-68-177-68.nip.io
      secretName: ratings-web-cert
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
- 확인 url https://frontend.52-170-173-31.nip.io/
