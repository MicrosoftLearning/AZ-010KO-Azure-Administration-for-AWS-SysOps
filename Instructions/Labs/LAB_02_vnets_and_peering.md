---
lab:
    title: 'Azure 가상 네트워크 및 피어링'
    module: '모듈 2: Azure 네트워크'
---
    
# 랩 02: Azure 가상 네트워크 및 피어링

## 학생 랩 매뉴얼

## 시나리오

이 랩에서는 Bash 인터페이스를 통해 Cloud Shell에서 CLI 명령을 사용하여 VNet(가상 네트워크)을 관리합니다. 구체적으로는 여러 리소스 그룹과 지역에서 사용되는 VNet을 만들고 VNet 피어링을 구성합니다.

## 목표

이 랩을 완료하면 Azure CLI를 사용하여 다음 작업을 수행할 수 있습니다.

* 원하는 리소스 그룹에서 서브넷이 포함된 VNet 생성 및 구성
* VNet 간에 VNet 피어링 구성

## 랩 설정

* **예상 소요 시간**: 30분

## 지침

### 시작하기 전에

#### 설정 태스크

1. 에서 구성한 EastRG 및 WestRG 리소스 그룹 **모듈 1: Azure 관리, 랩: 리소스 그룹 만들기**.

## 연습 1: 서브넷이 포함된 가상 네트워크 만들기

이 연습의 주요 태스크는 다음과 같습니다.

1. 서브넷이 포함된 West VNet 만들기
1. West 네트워크 및 서브넷이 생성되었는지 확인
1. 서브넷이 포함된 East VNet 만들기
1. East 네트워크 및 서브넷이 생성되었는지 확인

### 연습 1 - 태스크 1: 서브넷이 포함된 West VNet 만들기

1. **Cloud Shell** 명령 프롬프트에서 다음 명령을 입력하여 WestSubNet1 서브넷이 포함된 WestVNet 가상 네트워크를 만듭니다.

```sh
az network vnet create \
  --resource-group WestRG \
  --name WestVNet \
  --address-prefix 10.1.0.0/16 \
  --subnet-name WestSubnet1 \
  --subnet-prefix 10.1.0.0/24
```

2. West 네트워크 및 서브넷이 생성되었는지 확인합니다.

```sh
az network vnet list --output table
```

3. West 네트워크 및 서브넷이 생성되었는지 확인합니다.

```sh
az network vnet subnet list --resource-group WestRG --vnet-name WestVNet --output table
```

### 태스크 2: 서브넷이 포함된 East VNet 만들기

1. **Cloud Shell** 명령 프롬프트에서 다음 명령을 입력하여 EastSubNet1 서브넷이 포함된 EastVNet 가상 네트워크를 만듭니다.

```sh
az network vnet create \
  --resource-group EastRG \
  --name EastVNet \
  --address-prefix 10.2.0.0/16 \
  --subnet-name EastSubnet1 \
  --subnet-prefix 10.2.0.0/24
```

2. 기존 VNet에 서브넷을 더 추가할 수 있습니다.
3. **Cloud Shell** 명령 프롬프트에서 다음 명령을 입력하여 EastVNet EastSubNet2 서브넷을 만듭니다.

```sh
az network vnet subnet create \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --name EastSubnet2 \
  --address-prefix 10.2.1.0/24
```

4. East 네트워크(EastVNet)가 생성되었는지 확인합니다.

```sh
az network vnet list --output table
```

5. East 네트워크 서브넷이 생성되었는지 확인합니다.

```sh
az network vnet subnet list --resource-group EastRG --vnet-name EastVNet --output table
```

### 태스크 3: West에서 East로의 간 피어링 네트워크 만들기

1. `remote-vnet-id` CLI 명령을 사용하여 West VNet과 East VNet 간에 피어핑을 만듭니다.
1. **Cloud Shell** 명령 프롬프트에서 다음 명령을 입력하여 EastVNet ID를 변수에 캡처합니다.

```sh
EastVNetId=$(az network vnet show \
  --resource-group EastRG \
  --name EastVNet \
  --query id --out tsv)

  echo "EastVNetId = " $EastVNetId
```

3. 다음 명령을 입력하여 WestVNet을 EastVNet에 피어링합니다.

```sh
az network vnet peering create \
  --name WesttoEastPeering \
  --resource-group WestRG \
  --vnet-name WestVNet \
  --remote-vnet $EastVNetId \
  --allow-vnet-access
```

> *참고: 여기서는 Cloud Shell `--remote-vnet-id $EastVNetId` 명령을 사용하지 않으며, 이 명령을 사용하면 경고가 발생합니다. 하지만 다른 CLI 셸에서는 이 명령을 사용해야 할 수도 있습니다.*

4. 다음 명령을 입력하여 피어링 상태를 확인합니다.

```sh
az network vnet peering list \
  --resource-group WestRG \
  --vnet-name WestVNet \
  --output table
  ```

### 태스크 4: East에서 West로의 간 피어링 네트워크 만들기

1. WestVNet ID를 변수에 캡처합니다.

```sh
WestVNetId=$(az network vnet show \
  --resource-group WestRG \
  --name WestVNet \
  --query id --out tsv)
  
echo "WestVNetId = " $WestVNetId
```

2. 다음 명령을 입력하여 EastVNet을 WestVNet에 피어링합니다.

```sh
az network vnet peering create \
  --name EasttoWestPeering \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --remote-vnet $WestVNetId \
  --allow-vnet-access
```

3. 다음 명령을 입력하여 피어링 상태를 확인합니다.

```sh
az network vnet peering list \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --output table
  ```

> **결과**: 이 랩에서는 East 및 West 가상 네트워크와 서브넷을 구성했으며 네트워크 간에 피어링(East에서 West로/West에서 East로)을 생성했습니다. 후속 랩에서 이러한 리소스를 추가로 사용할 것입니다.
