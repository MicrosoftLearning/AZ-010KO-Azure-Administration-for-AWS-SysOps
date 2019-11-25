# 랩 02 답변 키

## 지침

1. Azure Portal에 로그인하여 Bash Cloud Shell을 엽니다.
1. 아래 CLI 스크립트를 복사합니다.
1. Bash Cloud Shell에 스크립트를 붙여넣습니다.
1. 실행 시간이 1분보다 오래 걸리는 태스크도 있으므로 스크립트가 완료될 때까지 기다렸다가 출력을 검토하세요.
1. 오류가 발생하는 경우 강사에게 문의하세요.

> 참고: 학생은 랩 01에 요약되어 있는 대로 기본 구독을 설정해야 합니다.

```sh
# AZ-010 LAB2 해답
# ---------------
# 의존 관계: LAB1이 정상적으로 완료됨(WestRG 및 EastRG가 생성됨)
# ---------------
# 경고: 1분보다 오래 걸리는 단계도 있습니다.
# 단계가 끝날 때까지 기다리세요. 오류 발생 시에는 강사에게 문의하세요.

# ----------시작----------

# ----WestVNet 및 WestSubNet1 만들기----
az network vnet create \
  --resource-group WestRG \
  --name WestVNet \
  --address-prefix 10.1.0.0/16 \
  --subnet-name WestSubnet1 \
  --subnet-prefix 10.1.0.0/24

# ----WestVNet 및 WestSubNet1이 생성되었는지 확인----
az network vnet subnet list --resource-group WestRG \
--vnet-name WestVNet --output table

# ----EastVNet 및 EastSubNet1 만들기----
az network vnet create \
  --resource-group EastRG \
  --name EastVNet \
  --address-prefix 10.2.0.0/16 \
  --subnet-name EastSubnet1 \
  --subnet-prefix 10.2.0.0/24

# ----EastVNet에서 EastSubNet2 만들기----
az network vnet subnet create \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --name EastSubnet2 \
  --address-prefix 10.2.1.0/24

# ----EastVNet이 생성되었는지 확인----
az network vnet list --output table

# ----EastVNet 서브넷이 생성되었는지 확인----
az network vnet subnet list --resource-group EastRG --vnet-name EastVNet --output table

# ------West에서 East로의 네트워크 피어링 시작------
# ----변수에 EastVNet ID 캡처----
EastVNetId=$(az network vnet show \
  --resource-group EastRG \
  --name EastVNet \
  --query id --out tsv)

  echo "EastVNetId = " $EastVNetId

# ----West에서 East로 피어링----
az network vnet peering create \
  --name WesttoEastPeering \
  --resource-group WestRG \
  --vnet-name WestVNet \
  --remote-vnet $EastVNetId \
  --allow-vnet-access

# ----피어링 상태 확인----
az network vnet peering list \
  --resource-group WestRG \
  --vnet-name WestVNet \
  --output table

# ------West에서 East로의 네트워크 피어링 시작------
# ----변수에 WestVNet ID 캡처----
WestVNetId=$(az network vnet show \
  --resource-group WestRG \
  --name WestVNet \
  --query id --out tsv)
  
echo "WestVNetId = " $WestVNetId

# ----East에서 West로 피어링----
az network vnet peering create \
  --name EasttoWestPeering \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --remote-vnet $WestVNetId \
  --allow-vnet-access

# ----피어링 상태 확인----
az network vnet peering list \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --output table
```
