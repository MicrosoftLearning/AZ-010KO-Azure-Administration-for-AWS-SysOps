---
lab:
    title: 'Azure VM 및 확장 집합'
    module: '모듈 3: Azure 컴퓨팅'
---
    
# 랩 03: Azure 컴퓨팅

## 학생 랩 매뉴얼

## 시나리오

이 랩에서는 Bash 인터페이스를 통해 Cloud Shell에서 CLI 명령을 사용하여 가용성 집합 내의 Azure Windows 및 Debian VM을 관리합니다. 또한 테스트(선택 사항)가 포함된 Ubuntu 확장 집합을 만들어 규모 감축/확장 과정을 시연합니다.

## 목표

이 랩을 완료하면 Azure CLI를 사용하여 다음 작업을 수행할 수 있습니다.

* 확장 집합 만들기
* Windows 및 Linux VM 만들기/구성
* Ubuntu 확장 집합 구성

## 랩 설정

* **예상 소요 시간**: 60분

## 지침

### 시작하기 전에

#### 설정 태스크

1. **이전 랩과의 의존 관계:**
    1. 모듈 1: Azure 관리 - **랩: 리소스 그룹 만들기**. EastRG 및 WestRG 리소스 그룹을 구성한 상태여야 합니다.
    1. 모듈 2: Azure 네트워킹 - **랩: 가상 네트워크 및 피어링**. 서브넷이 포함된 VNet과 피어링을 구성한 상태여야 합니다.

## 연습 1: 가용성 집합 내에서 구성된 VM 만들기

이 연습의 주요 태스크는 다음과 같습니다.

1. 가용성 집합 만들기
1. Windows 가상 머신 만들기
1. Linux Debian 가상 머신 만들기

### 연습 1 - 태스크 1: Windows 가상 머신 만들기

* 가용성 집합 만들기
* Windows Server 2016 DataCenter VM 만들기
* 웹 서버로 구성

**변수 준비**

1. **Cloud Shell** 명령 프롬프트에 다음 명령을 입력하고 **Enter** 키를 눌러 다음 스크립트에서 사용할 변수를 준비합니다.

> **참고**: 고유한 암호를 만들고 기록해 둡니다. 이때 암호의 선행 "!"를 제거하지 않으면 오류가 발생합니다.

```sh
resourceGroupName='WestRG'
location='westus'
adminUserName='azuser'
adminPassword=!'UniqueP@$$w0rd-Here' # "!"를 제거하지 않으면 오류 발생 + 암호는 고유해야 함
vmName='WestWinVM'
vmSize='Standard_D1'
availabilitySet='WestAS'
```

**가용성 집합 만들기**

1. 다음 명령을 입력하여 **WestAS** 가용성 집합을 만듭니다.

```sh
az vm availability-set create \
  --name $availabilitySet \
  --resource-group $resourceGroupName \
  --location $location
```

**WestWinVM VM 만들기**

1. 다음 명령을 입력하여 **WestWinVM** Windows Server VM을 만듭니다.

```sh
az vm create --name $vmName --resource-group $resourceGroupName \
  --image win2016datacenter \
  --admin-username $adminUserName \
  --admin-password $adminPassword \
  --size $vmSize \
  --location $location \
  --availability-set $availabilitySet
  ```

**포트 열기**

1. 다음 명령을 입력하여 WestWinVM에서 포트를 엽니다.

```sh
az vm open-port -g WestRG -n $vmName --port 80 --priority 1500
az vm open-port -g WestRG -n $vmName --port 3389 --priority 2000
```

**작업 확인**

1. Azure Portal을 시작하여 WestRG로 이동한 후에
2. WestWinVM을 포함한 리소스를 확인합니다.
3. Azure Advisor를 시작하여 권장 사항을 살펴봅니다.

### 태스크 2: WestWinVM을 웹 서버로 구성하고 Ping 허용

**ICMPv4-In 허용(Bash CLI에서 PowerShell 실행)**

1. **Cloud Shell** 명령 프롬프트에 다음 명령을 입력하고 **Enter** 키를 눌러 WestWinVM Windows Server로 PowerShell 명령을 전송해 ICMPv4-In을 허용합니다.

> **참고**: Cloud Shell 시간이 초과된 경우에는 태스크 1에서 변수를 새로 고쳐야 할 수 있습니다.

```sh
az vm extension set --publisher Microsoft.Compute \
--version 1.8 --name CustomScriptExtension \
--vm-name $vmName --resource-group $resourceGroupName \
--settings '{"commandToExecute":"powershell.exe New-NetFirewallRule –DisplayName “ICMPv4-In 허용” –Protocol ICMPv4"}'
```

**IIS 설치(Bash CLI에서 PowerShell 실행)**

1. 다음 명령을 입력하여 WestWinVM Windows Server로 PowerShell 명령을 전송해 IIS를 설치합니다.

```sh
az vm extension set --publisher Microsoft.Compute \
--version 1.8 \
--name CustomScriptExtension \
--vm-name $vmName \
--resource-group $resourceGroupName \
--settings '{"commandToExecute":"powershell.exe Install-WindowsFeature -name Web-Server -IncludeManagementTools"}'
```

**VM 공용 IP 주소에서 IIS 서버가 실행되고 있는지 확인**

1. 다음 명령을 입력하여 WestWSinVM 공용 IP 주소를 캡처합니다.

```sh
az vm show -d -g $resourceGroupName -n $vmName --query publicIps -o tsv

# ----위의 IP 주소를 브라우저에 붙여넣어 IIS가 실행 중인지 확인----
```

2. 결과로 반환된 IP 주소를 웹 브라우저에 붙여넣어 IIS 기본 페이지가 있는지 확인합니다.

### 태스크 3: DNS로 구성된 Debian 가상 머신 만들기

Cloud Shell에서 Debian 가상 서버 2개를 만들고 SSH를 사용하여 서버에 연결

머신 1: **WestDebianVM**

> - WestRG 리소스 그룹
>   - WestVNet
>      - WestSubnet1

및

머신 2: **EastDebianVM**

> - East 리소스 그룹
>   - EastVNet
>     - EastSubnet2 서브넷

**WestDebianVM 만들기**

1. 다음 명령을 입력하여 WestDebianVM을 만듭니다.

```sh
az vm create \
--image credativ:Debian:8:latest \
--size 'Standard_D1' \
--admin-username azuser \
--resource-group WestRG \
--vnet-name WestVNet \
--subnet WestSubnet1 \
--availability-set WestAS \
--location westus \
--name WestDebianVM \
--generate-ssh-keys
```

2. 참고: shh 키는 없는 경우 생성되어 `~/.ssh` 디렉터리에 저장됩니다. Cloud Shell에 다음 명령을 입력하여 '~/.ssh' 디렉터리의 내용을 확인합니다.

```sh
ls .ssh
```

**EastAS 가용성 집합 만들기**

1. 다음 명령을 입력하여 EastAS를 만듭니다.

```sh
az vm availability-set create --name EastAS --resource-group EastRG
```

**EastDebianVM 만들기**

1. 다음 명령을 입력하여 EastDebianVM을 만듭니다.

```sh
az vm create \
--image credativ:Debian:8:latest \
--admin-username azuser \
--resource-group EastRG \
--vnet-name EastVNet \
--subnet EastSubnet2 \
--availability-set EastAS \
--location eastus \
--name EastDebianVM \
--generate-ssh-keys
```

**Debian 머신의 DNS 구성**

1. 다음 명령을 입력하여 고유한 DNS 이름을 할당하는 데 사용할 임의의 문자열을 Bash를 사용해 만듭니다.

```sh
myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "추가되는 임의 문자열:  "$myRand
```

**DNS 구성**

1. 다음 명령을 입력하여 Linux Debian VM에 고유한 DNS 이름을 할당합니다.

```sh
az network public-ip update --resource-group WestRG --name WestDebianVMPublicIP --dns-name westdebianvm$myRand

az network public-ip update --resource-group EastRG --name EastDebianVMPublicIP --dns-name eastdebianvm$myRand
```

2. Linux Debian VM의 DNS 이름을 확인합니다.

**두 Debian VM이 모두 실행 중인지 확인**

1. 다음 명령을 입력하여 Linux Debian VM의 상태를 확인합니다. 두 VM이 모두 실행 중이어야 합니다.

```sh
az vm get-instance-view --name WestDebianVM --resource-group WestRG --query instanceView.statuses[1] --output table

az vm get-instance-view --name EastDebianVM --resource-group EastRG --query instanceView.statuses[1] --output table
```

### 태스크 4: SSH를 사용해 WestDebianVM에 연결한 다음 WestWinVM의 ping 테스트 실행

**West 리소스 그룹의 Debian 가상 머신에 연결**

1. 다음 명령을 입력하여 VM의 공용 및 프라이빗 IP 주소를 가져옵니다.

```sh
az vm list-ip-addresses --resource-group WestRG

az vm list-ip-addresses --resource-group EastRG
```

2. EastDebianVM, WestDebianVM and WestWinVM IP 주소의 값을 확인한 다음 **IP를 기록** 해 두거나 Portal에서 각 VM의 개요 페이지를 확인합니다.

**WestDebianVM 가상 머신으로의 SSH 실행**

1. WestDebianVM IP 주소를 사용하여 다음 명령을 편집해 SSH 세션을 시작합니다.

```sh
ssh azuser@<West Debian VM의 공용 IP 주소>
```

**SSH 세션에서 Windows VM 프라이빗 주소에 대해 ping 실행**

> *두 VM은 같은 **프라이빗** VNet에 있으므로 ping은 정상적으로 실행됩니다.*

1. WestWinVM IP 주소를 사용하여 다음 명령을 편집한 다음 Cloud Shell SSH 세션을 입력하여 WestWinVM에 대해 ping을 실행합니다.
1. **`ctrl+c`** 를 통해 Bash에서 ping 종료(ssh 세션)

```sh
ping <Windows 서버의 프라이빗 IP 주소>
```

**Ping EastDebianVM**

1. EastDebianVM IP 주소를 사용하여 다음 명령을 편집한 다음 Cloud Shell SSH 세션을 입력하여 WestWinVM에 대해 ping을 실행합니다.

> *이전에 VNet 피어링을 구성했으므로 ping은 정상적으로 실행됩니다.*

1. **`ctrl+c`** 를 통해 Bash에서 ping 종료(ssh 세션)

```sh
ping <eastdebianvm의 프라이빗 IP 주소>
```

> **참고**: SSH를 끝내려면 `exit` 을 입력합니다.

### 연습 2: Ubuntu 확장 집합 만들기 및 테스트

이 연습의 주요 태스크는 다음과 같습니다.

1. Ubuntu 확장 집합 및 자동 크기 조정 프로필 만들기
1. 자동 확장 및 규모 감축 규칙 만들기
1. Ubuntu 확장 집합 테스트(선택 사항)

#### 연습 2 - 태스크 1: Ubuntu 확장 집합 만들기

**확장 집합 리소스 그룹 만들기**

1. 다음 명령을 입력하여 확장 집합용 리소스 그룹을 만듭니다.

```sh
az group create --name EastScaleRG --location eastus
```

**확장 집합 만들기**

1. 다음 명령을 입력하여 확장 집합을 만듭니다.

```sh
az vmss create \
  --resource-group EastScaleRG \
  --name EastUbuntuServers \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --instance-count 2 \
  --admin-username azuser \
  --generate-ssh-keys
```

Portal 대시보드에서 크기 조정 규칙 앞의 서버 2개 확인

> - 리소스 그룹
>   - EastScaleRG
>     - EastUbuntuServers > 인스턴스

**자동 크기 조정 프로필 정의**

1. 다음 명령을 입력하여 자동 크기 조정 프로필을 정의합니다.

```sh
az monitor autoscale create \
  --resource-group EastScaleRG \
  --resource EastUbuntuServers \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale \
  --min-count 1 \
  --max-count 3 \
  --count 1
  ```

> *위의 코드를 실행하면* "follow up with `az monitor autoscale rule create` to add scaling rules."(크기 조정 규칙을 만들려면 `az monitor autoscale rule create`를 추가로 실행하세요.) 메시지가 표시되어야 합니다.

**자동 확장용 규칙 만들기**

1. 다음 명령을 입력하여 **자동 확장** 규칙을 만듭니다.

```sh
az monitor autoscale rule create \
  --resource-group EastScaleRG \
  --autoscale-name autoscale \
  --condition "Percentage CPU > 50 avg 2m" \
  --scale out 1
```

**자동 규모 감축용 규칙 만들기**

1. 다음 명령을 입력하여 **자동 규모 감축** 규칙을 만듭니다.

```sh
az monitor autoscale rule create \
  --resource-group EastScaleRG \
  --autoscale-name autoscale \
  --condition "Percentage CPU < 30 avg 1m" \
  --scale in 1
```

> *참고: "규모 감축"은 짧으면 2분 이내에도 수행될 수 있으므로 처음 몇 분 동안에는 아래 결과가 달라질 수 있습니다.*

### 태스크 2: Ubuntu 확장 집합 테스트(선택 태스크)

> 이 태스크에서는 확장 집합 동작을 시연하기 위한 테스트로 서버에서 CPU 로드를 생성합니다.

**확장 집합에서 실행 중인 서버 목록 표시**

* 첫 1분 동안에는 스크립트를 실행하면 서버 인스턴스 2개(예: 0과 1)가 표시됩니다.
* 1분이 지나면 서버 인스턴스 1개(예: 0)만 표시될 수도 있습니다.

1. 다음 명령을 입력하여 서버 목록을 표시합니다.
1. 30초 후에 목록에 서버 인스턴스 1개만 표시될 때까지 명령을 **반복** 합니다.

```sh
az vmss list-instance-connection-info \
--resource-group EastScaleRG \
--name EastUbuntuServers
```

**SSH를 사용하여 인스턴스 0에 연결**

1. Portal을 사용하여 확장 집합의 인스턴스 0 IP 주소를 가져옵니다.
1. 다음의 편집된 명령을 입력하여 SSH를 통해 연결합니다.

```sh
# 예: ssh azuser@13.92.224.66 -p 50000  
ssh azuser@<인스턴스 0 IP> -p 50000
```

**4분(240초) 동안 스트레스 실행**

1. SSH 세션에서 다음 명령을 입력하여 스트레스 애플리케이션을 설치합니다. 이 세션은 240초 후에 시간 초과 처리됩니다.

```sh
sudo apt-get -y install stress
sudo stress --cpu 10 --timeout 240 &
```

**SSH 세션에서 스트레스 확인**

1. SSH에서 다음 명령을 입력하여 스트레스를 모니터링합니다.

```sh
top
```

**세션과 SSH 끝내기**

```sh
# ctrl-c
exit
```

**자동 확장 및 자동 규모 감축 감시**

1. 다음 명령을 입력하여 자동 크기 조정 모니터링을 실행합니다.

```sh
watch az vmss list-instances \
  --resource-group EastScaleRG \
  --name EastUbuntuServers \
  --output table
```

> **참고**: 스트레스가 로드를 50% 이상 등록하기 시작할 때까지 몇 분 정도 걸릴 수 있습니다.
>
> **참고**: "watch"를 닫으려면 Ctrl+C를 누릅니다.

**확장 집합 데모 정리**

```sh
az group delete --name EastScaleRG --yes --no-wait
```
