```
az extension add --name connectedk8s
az extension add --name k8sconfiguration
az group create --name AzureArcTest -l EastUS -o table

az aks get-credentials --resource-group $RESOURCE_GROUP --name $AKS_CLUSTER_NAME --admin

az connectedk8s connect --name AzureArcTest1 --resource-group AzureArcTest
az connectedk8s list -g AzureArcTest

```
# Azure arc를 붙여보자
- Azure Cloud Shell 사용

1. azure cli - extension등록  
```
az extension add --name connectedk8s
az extension add --name k8sconfiguration
```
2. arc에 대한 리소스 그룹 생성  
`az group create --name AzureArcTest -l EastUS -o table`  
3. AKS credential 등록  
- Admin 권한이 필요하다는 docs에 적혀있다. --admin 옵션을 붙임.  
`az aks get-credentials --resource-group $RESOURCE_GROUP --name $AKS_CLUSTER_NAME --admin`  
4. arc 연결  
`az connectedk8s connect --name AzureArcTest1 --resource-group AzureArcTest`  
5. arc connect list보기  
`az connectedk8s list -g AzureArcTest`
