# 연습 - Azure Kubernetes Service를 사용한 Kubernetes 배포
- 새 리소스 그룹 만들기
- 클러스터 네트워킹 구성
- Azure Kubernetes Service 클러스터 만들기
- kubectl을 사용하여 Kubernetes 클러스터에 연결
- Kubernetes 네임스페이스 만들기

## 1. Create resource group
- Azure Cloud Shell 에서 진행.
```bash
REGION_NAME=eastus
RESOURCE_GROUP=aksworkshop
SUBNET_NAME=aks-subnet
VNET_NAME=aks-vnet
```
- 위 변수내용으로 Resource group 생성
```bash
az group create \
    --name $RESOURCE_GROUP \
    --location $REGION_NAME
```

## 2. Network 구성
- AKS는 두가지 모델의 네트워크가 있다고한다.
  - Kubernete의 CNI.
  - Azure CNI 모델.
- * Azure CNI를 사용했을때 어떤점이 좋은지 직접 해보면서 알아보자.

## 3. Azure CNI 생성
```bash
az network vnet create \
    --resource-group $RESOURCE_GROUP \
    --location $REGION_NAME \
    --name $VNET_NAME \
    --address-prefixes 10.0.0.0/8 \
    --subnet-name $SUBNET_NAME \
    --subnet-prefix 10.240.0.0/16

SUBNET_ID=$(az network vnet subnet show \
    --resource-group $RESOURCE_GROUP \
    --vnet-name $VNET_NAME \
    --name $SUBNET_NAME \
    --query id -o tsv)
```

## 4. AKS 클러스터 생성
- 최신 AKS 버전 가져오기 (preview 제외)
```bash
VERSION=$(az aks get-versions \
    --location $REGION_NAME \
    --query 'orchestrators[?!isPreview] | [-1].orchestratorVersion' \
    --output tsv)
```
- Cluster Nameing
```bash
AKS_CLUSTER_NAME=aksworkshop-$RANDOM
```
- Create
```bash
az aks create \
--resource-group $RESOURCE_GROUP \
--name $AKS_CLUSTER_NAME \
--vm-set-type VirtualMachineScaleSets \
--load-balancer-sku standard \
--location $REGION_NAME \
--kubernetes-version $VERSION \
--network-plugin azure \
--vnet-subnet-id $SUBNET_ID \
--service-cidr 10.2.0.0/24 \
--dns-service-ip 10.2.0.10 \
--docker-bridge-address 172.17.0.1/16 \
--generate-ssh-keys
```
- 위 명령어를 입력하면, Quota 문제로 생성이 되지않는다.
- 옵션에는 `standardDSv2Family`의 VM size를 지정해주지않았는데, 지정해서 다시 시도해보자.
- ~(않히.. 4core 제한은 너무 심하자나,... 뭐하란거야)~
- B2s size의 node를 2개 설정.
```bash
az aks create \
--resource-group $RESOURCE_GROUP \
--name $AKS_CLUSTER_NAME \
--vm-set-type VirtualMachineScaleSets \
--load-balancer-sku standard \
--location $REGION_NAME \
--kubernetes-version $VERSION \
--network-plugin azure \
--vnet-subnet-id $SUBNET_ID \
--service-cidr 10.2.0.0/24 \
--dns-service-ip 10.2.0.10 \
--docker-bridge-address 172.17.0.1/16 \
--generate-ssh-keys \
--node-vm-size Standard_B2s \
--node-count 2
```
## 5. 클러스터와 연결
- 명령어를 통해 kubectl을 통해 클러스터와 연결.
```bash
az aks get-credentials \
    --resource-group $RESOURCE_GROUP \
    --name $AKS_CLUSTER_NAME
```
## 6. App 배포를 위한 namespace 생성
```bash
kubectl create namespace ratingsapp
```


# 생각해볼것
## node 개수는 홀수로 만드는걸 권장한다고한다.
- 예제에서는 3개의 App을 올린다. Node 갯수는 2개이다.
- service는 3개인데, pod에 어떻게 올라갈까?
