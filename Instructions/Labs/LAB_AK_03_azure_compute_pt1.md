# 랩 03(1부) 답변 키

## 지침

1. 아래 CLI 스크립트를 메모장 등의 편집기에 복사합니다.
1. 위치 섹션의 제목은 `# ----실행 전에 다음 값 편집----`입니다.
1. 값이 사용자의 환경을 나타내도록 편집한 다음 파일을 로컬에 저장합니다.
1. Azure Portal에 로그인하여 Bash Cloud Shell을 엽니다.
1. 종속성이 있는지 확인합니다(스크립트 맨 위의 주석 참조).
1. 로컬 파일에서 CLI 스크립트를 복사하여 Bash Cloud Shell에 붙여넣습니다.
1. 실행 시간이 1분보다 오래 걸리는 태스크도 있으므로 스크립트가 완료될 때까지 기다렸다가 출력을 검토하세요.
1. 오류가 발생하는 경우 강사에게 문의하세요.

> 참고: 학생은 랩 01에 요약되어 있는 대로 기본 구독을 설정해야 합니다.

```sh
# AZ-010 LAB3 해답
# ---------------
# 종속성:
#   LAB1이 정상적으로 완료됨(WestRG 및 EastRG가 생성됨)
#   LAB2가 정상적으로 완료됨(VNet/서브넷/피어링이 생성됨)
# ---------------
# 경고: 1분보다 오래 걸리는 단계도 있습니다.
# 단계가 끝날 때까지 기다리세요. 오류 발생 시에는 강사에게 문의하세요.

# ----------시작----------

# ----실행 전에 다음 값을 고유하게 편집----
adminUserName='azuser'
adminPassword='UniqueP@$$w0rd-Here'

# ----변수 설정----
resourceGroupName='WestRG'
location='westus'
vmName='WestWinVM'
vmSize='Standard_D1'
availabilitySet='WestAS'

# ----주 스크립트----

# ----가용성 집합 만들기----
az vm availability-set create \
  --name $availabilitySet \
  --resource-group $resourceGroupName \
  --location $location

# ----WinVM VM 만들기----
az vm create --name $vmName --resource-group $resourceGroupName \
  --image win2016datacenter \
  --admin-username $adminUserName \
  --admin-password $adminPassword \
  --location $location \
  --size $vmSize \
  --availability-set $availabilitySet

# ----포트 열기----
az vm open-port -g WestRG -n $vmName --port 80 --priority 1500
az vm open-port -g WestRG -n $vmName --port 3389 --priority 2000

# ----ICMPv4-In 허용(Bash CLI에서 PowerShell 실행)----
az vm extension set --publisher Microsoft.Compute \
--version 1.8 --name CustomScriptExtension \
--vm-name $vmName --resource-group $resourceGroupName \
--settings '{"commandToExecute":"powershell.exe New-NetFirewallRule –DisplayName “ICMPv4-In 허용” –Protocol ICMPv4"}'

# ----IIS 설치(Bash CLI에서 PowerShell 실행)----
az vm extension set --publisher Microsoft.Compute \
--version 1.8 \
--name CustomScriptExtension \
--vm-name $vmName \
--resource-group $resourceGroupName \
--settings '{"commandToExecute":"powershell.exe Install-WindowsFeature -name Web-Server -IncludeManagementTools"}'

# ----VM 공용 IP 주소에서 IIS 서버가 실행되고 있는지 확인----
az vm show -d -g $resourceGroupName -n $vmName --query publicIps -o tsv

# ***********************************************************
# 위의 IP 주소를 브라우저에 붙여넣어 IIS가 실행 중인지 확인
# ***********************************************************

# ----DNS로 구성된 Debian 가상 머신 만들기----
# ----WestDebianVM 만들기----
az vm create \
--image credativ:Debian:8:latest \
--admin-username azuser \
--resource-group WestRG \
--vnet-name WestVNet \
--subnet WestSubnet1 \
--availability-set WestAS \
--size 'Standard_D1' \
--location westus \
--name WestDebianVM \
--generate-ssh-keys

# ----EastAS 가용성 집합 만들기----
az vm availability-set create --name EastAS --resource-group EastRG

# ----EastDebianVM 만들기----
az vm create \
--image credativ:Debian:8:latest \
--admin-username azuser \
--resource-group EastRG \
--vnet-name EastVNet \
--subnet EastSubnet2 \
--availability-set EastAS \
--size 'Standard_D1' \
--location eastus \
--name EastDebianVM \
--generate-ssh-keys

# ----Debian 머신의 DNS 구성----
myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "추가되는 임의 문자열:  "$myRand

# ----DNS 구성----
az network public-ip update --resource-group WestRG --name WestDebianVMPublicIP --dns-name westdebianvm$myRand

az network public-ip update --resource-group EastRG --name EastDebianVMPublicIP --dns-name eastdebianvm$myRand

# ----두 데비안 VM이 모두 실행 중인지 확인----
az vm get-instance-view --name WestDebianVM --resource-group WestRG --query instanceView.statuses[1] --output table

az vm get-instance-view --name EastDebianVM --resource-group EastRG --query instanceView.statuses[1] --output table

# ----SSH를 사용해 WestDebianVM에 연결한 다음 WestWinVM의 ping 테스트 실행----
# ----WestRG의 Debian 가상 머신에 연결----

# ----VM의 공용/프라이빗 IP 주소 가져오기----
WestDebianIP=$(az vm list-ip-addresses -n WestDebianVM --query "[*].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv)

EastDebianIP=$(az vm list-ip-addresses -n EastDebianVM --query "[*].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv)

WestWinIP=$(az vm list-ip-addresses -n WestWinVM --query "[*].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv)

#----WestDebianVM 가상 머신으로의 SSH 실행----

# *******************************************
# SSH 세션을 시작하려면 아래 명령을 사용합니다.
echo "WestDebianVM으로의 SSH를 실행하려면 다음 명령 사용: " 'ssh' $adminUserName'@'$WestDebianIP
echo "EastDebianVM IP: " $EastDebianIP
echo "WestWinVM IP: " $WestWinIP
# *******************************************

# *******************************************
# --------SSH 세션 명령 시작--------
# --랩 03 연습 1 태스크 4: VM ping 테스트 참조--
# --위의 SSH 연결 및 IP 값 참조--
# *******************************************
```

**전체 연습 1 태스크 4: SSH 및 ping**

**Lab03 답변 키 해답 2부(LAB_AK_03_azure_compute_pt2.md) 계속 진행**
