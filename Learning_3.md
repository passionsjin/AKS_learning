# 연습 - MongoDB 배포
- 이 연습에서는 Helm을 사용하여 AKS(Azure Kubernetes Service) 클러스터로 MongoDB를 배포한다.
  - Helm 안정화 리포지토리 구성
  - MongoDB 차트 설치
  - 데이터베이스 자격 증명을 보관할 Kubernetes 비밀 만들기

## 1. Helm 안정화 리포지토리 추가
- Helm은 Kubernetes용 애플리케이션 패키지 관리자이며 차트를 사용해 애플리케이션과 서비스를 쉽게 배포할 수 있는 방법을 제공한다.
- Azure shell에서는 기본제공
- stable 을 추가하고, list를 확인
```bash
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm search repo stable
```
## 2. Helm 차트 설치
- Helm chart를 이용해 MongoDB 설치
```bash
helm install ratings stable/mongodb \
    --namespace ratingsapp \
    --set mongodbUsername=passionsjin,mongodbPassword=1q2w3e4r,mongodbDatabase=ratingsdb
```

## 3. secret를 이용해 MongoDB 정보값 저장.
```bash
kubectl create secret generic mongosecret \
    --namespace ratingsapp \
    --from-literal=MONGOCONNECTION="mongodb://passionsjin:1q2w3e4r@ratings-mongodb.ratingsapp.svc.cluster.local:27017/ratingsdb"
    
kubectl describe secret mongosecret --namespace ratingsapp
```
