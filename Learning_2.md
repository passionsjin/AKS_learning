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
