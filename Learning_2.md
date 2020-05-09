# 연습 - 고가용성 프라이빗 컨테이너 레지스트리 만들기
- docker container를 빌드하고, node에 배포해보자.

## 1. Container Registy 생성
```bash
ACR_NAME=acr$RANDOM
az acr create \
    --resource-group $RESOURCE_GROUP \
    --location $REGION_NAME \
    --name $ACR_NAME \
    --sku Standard
```

## 2. rating-api 이미지 빌드
```bash
git clone https://github.com/MicrosoftDocs/mslearn-aks-workshop-ratings-api.git

cd mslearn-aks-workshop-ratings-api/
```
- 여기서는 aks 명령어를 통해 build를 한다.
```bash
az acr build \
    --registry $ACR_NAME \
    --image ratings-api:v1 .
```
## 3. rating-web 이미지 빌드
- 2번과 같다
```bash
cd ~
git clone https://github.com/MicrosoftDocs/mslearn-aks-workshop-ratings-web.git
cd mslearn-aks-workshop-ratings-web
az acr build \
    --registry $ACR_NAME \
    --image ratings-web:v1 .
```
- 빌드중 뭔가 오류가 많이 뜨는듯하다.
- 아래 명령어를 통해 이미지 확인
```bash
az acr repository list \
    --name $ACR_NAME \
    --output table
```
## 4. AKS에서 ACR의 인증
- 서비스 간 통신을 허용하려면 컨테이너 레지스트리와 Kubernetes 클러스터 간에 인증을 설정해야 한다.
```bash
az aks update \
    --name $AKS_CLUSTER_NAME \
    --resource-group $RESOURCE_GROUP \
    --attach-acr $ACR_NAME
```
