# 랩 04 답변 키

## 지침

1. 아래 CLI 스크립트를 메모장 등의 편집기에 복사합니다.
   1. 위치 섹션의 제목은 `# ----실행 전에 다음 값 편집----`입니다.
   1. **값을 편집하지 않으면** **오류**가 발생합니다.
   1. 파일을 로컬에 저장합니다.
1. Azure Portal에 로그인하여 Bash Cloud Shell을 엽니다.
1. 종속성이 있는지 확인합니다(스크립트 맨 위의 주석 참조).
1. 로컬 파일에서 CLI 스크립트를 복사하여 Bash Cloud Shell에 붙여넣습니다.
1. 실행 시간이 1분보다 오래 걸리는 태스크도 있으므로 스크립트가 완료될 때까지 기다렸다가 출력을 검토하세요.
1. 오류가 발생하는 경우 강사에게 문의하세요.

> `EDIT THESE VALUES` 섹션에 대한 참고 사항
>
>`myUploadPath` 및 `myDownloadPath`에 사용할 `az_user_name` 변수를 만듭니다.
>
> `az_user_name`을 설정하려면 아래 명령을 편집해야 합니다.
>
> Cloud Shell 프롬프트에서 <이름>@azure의 **이름**을 확인합니다.
> 예 - Cloud Shell의 프롬프트가 `eric@Azure:~$`인 경우 **`az_user_name='eric'`** 설정

```sh
## AZ-010 LAB4 해답
# ---------------
# 종속성:
#   LAB1이 정상적으로 완료됨(WestRG 및 EastRG가 생성됨)
#   LAB2가 정상적으로 완료됨(VNet/서브넷/피어링이 생성됨)
#   LAB3이 정상적으로 완료됨(EastDebianVM이 생성됨)
# ---------------
# 경고: 1분보다 오래 걸리는 단계도 있습니다.
# 단계가 끝날 때까지 기다리세요. 오류 발생 시에는 강사에게 문의하세요.

# ----------시작----------

# ----실행 전에 다음 값 편집----
subscriptionID=<**랩에 사용할 구독 ID**>

az_user_name=<name>

# ----주 스크립트----
# ----기본 구독 설정----
az account set --subscription $subscriptionID

# ----고유한 이름에 사용할 임의 문자열 생성----

myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "추가되는 임의 문자열:  "$myRand

# ----변수 설정----
my_resource_group=WestRG
location=westus
my_storage_account=weststore$myRand
my_storage_sku=Standard_RAGRS # default
my_storage_kind=StorageV2
my_access_tier=hot
my_storage_encryption=blob

# **************************************************************
echo "my_storage_account 이름: " $my_storage_account
# **************************************************************

# ----스토리지 계정 만들기----
az storage account create \
    -n $my_storage_account \
    -g $my_resource_group \
    -l $location \
    --kind $my_storage_kind \
    --access-tier $my_access_tier \
    --sku $my_storage_sku \
    --encryption $my_storage_encryption

# ----Bash CLI에서 환경 변수 설정----
export AZURE_STORAGE_ACCOUNT=$my_storage_account
export AZURE_STORAGE_CONNECTION_STRING="$(az storage account show-connection-string \
-n $AZURE_STORAGE_ACCOUNT -g $my_resource_group --query connectionString -o tsv)"

# ----스토리지 계정 키 환경 변수 만들기----
# 키 표시
az storage account keys list \
    --account-name $AZURE_STORAGE_ACCOUNT \
    --resource-group $my_resource_group \
    --output table
# ----AZURE_STORAGE_KEY=<storage_account_key1> 내보내기----
# ----key1을 환경 변수로 저장----
export AZURE_STORAGE_KEY="$(az storage account keys list --account-name \
$AZURE_STORAGE_ACCOUNT --resource-group $my_resource_group --query [0].value -o tsv)"

# ----Blob Storage에 업로드할 `helloAdmin.html` 파일 만들기
echo "<h1>Azure 관리자 여러분 안녕하세요</h1>">helloAdmin.html

# ----공용 액세스 권한이 있는 Blob 컨테이너 만들기----
container_name=westblobcontainerpublic

az storage container create --name $container_name --public-access blob

# ----공용 Blob 컨테이너에 파일 업로드----
blob_name=helloAdmin

az storage blob upload \
    --container-name $container_name \
    --file helloAdmin.html \
    --name $blob_name

# ----공개 파일 다운로드----
az storage blob download \
    --container-name $container_name \
    --name $blob_name \
    --file helloAdminDownload.html

# ---------upload-batch---------
mkdir uploadfiles

# ----`upload` 디렉터리에 업로드할 파일 만들기----
touch uploadfiles/myfile.html
touch uploadfiles/more.html
touch uploadfiles/hi.html
touch uploadfiles/myfile.txt

# ***********************************************************
# 위의 편집된 변수 섹션에서 <이름>이 대체된 것으로 가정
# az_user_name=<이름>
# ***********************************************************

# Cloud Shell에서 $home으로 경로 지정
myUploadPath=/home/$az_user_name/uploadfiles

# 경로에 html 파일 업로드
az storage blob upload-batch -d $container_name \
-s $myUploadPath -o table

#----업로드 경로 및 폴더 만들기----
mkdir downloadfiles

#----다운로드 경로 만들기----
myDownloadPath=/home/$az_user_name/downloadfiles

#----컨테이너에서 파일 일괄 다운로드----
az storage blob download-batch -d $myDownloadPath -s $container_name

#----Blob의 URL을 가져와 공개 파일 확인----
#----생성된 링크를 클릭하여 파일 확인
az storage blob url -c $container_name -n helloAdmin -o tsv

# ----작업 2: 스토리지 보안 액세스----
# ----west 스토리지 계정에 비공개 Blob 컨테이너 만들기----

# ----변수 새로 고침----
container_name=westblobcontainerprivate

# ----스토리지 키----
export AZURE_STORAGE_KEY="$(az storage account keys list \
    --account-name $AZURE_STORAGE_ACCOUNT \
    --resource-group $my_resource_group \
    --query [0].value -o tsv)"

# ----스토리지 연결 키----
export AZURE_STORAGE_CONNECTION_STRING="$(az storage account \
    show-connection-string -n $AZURE_STORAGE_ACCOUNT\
     -g $my_resource_group --query connectionString -o tsv)"

# ----비공개 Blob 컨테이너 새로 만들기
az storage container create --name $container_name --public-access blob
# ----파일 만들기 및 비공개 Blob 컨테이너에 업로드
echo "<h1>Azure 관리자 여러분 안녕하세요 - 비공개 메시지</h1>">helloAdmin.html
blob_name=helloAdmin

az storage blob upload \
    --container-name $container_name \
    --file helloAdmin.html --name $blob_name

# ----서비스 수준에서 SAS(공유 액세스 서명) 만들기----
end_date=`date -u -d "30분" '+%Y-%m-%dT%H:%MZ'`

CONTAINER_SAS_KEY="$(az storage container generate-sas \
    --name $container_name --https-only --auth-mode key \
    --expiry $end_date --permissions r -o tsv)"

BLOB_SAS_KEY="$(az storage blob generate-sas \
    --name $blob_name \
    --container-name $container_name \
    --permissions r --expiry $end_date \
    --auth-mode key -o tsv)"


# ----전체 Blob URI 생성/링크 테스트----
private_URI="$(az storage blob generate-sas \
    --name $blob_name \
    --container-name $container_name \
    --permissions r --auth-mode key \
    --expiry $end_date --https-only \
    --full-uri)"

echo $private_URI
# ***********************************************************************
# ----위에서 생성된 링크 테스트----
# ***********************************************************************


# ---------작업 3: 서브넷 서비스 엔드포인트 만들기----
# ----필요한 경우 네트워크 액세스를 기본적으로 거부하도록 기본 규칙 설정----
az storage account update --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT --default-action Deny

# ---- 서브넷 규칙을 업데이트하여 WestVNet - WestSubnet1에서 스토리지를 사용하도록 설정합니다.----
az network vnet subnet update --resource-group $my_resource_group \
    --vnet-name WestVNet --name WestSubnet1 \
    --service-endpoints "Microsoft.Storage"

# ----'subnet_id' 변수 만들기
subnet_id="$(az network vnet subnet show \
    --resource-group $my_resource_group \
    --vnet-name WestVNet --name WestSubnet1 \
    --query id --output tsv)"

# ----네트워크 서브넷 규칙을 추가합니다.
az storage account network-rule add --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT --subnet $subnet_id

# ----스토리지 계정 네트워크 규칙 업데이트 목록 표시----
az storage account network-rule list --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT

# **********************************************************
# ----이전 작업에서 액세스할 수 있던 페이지 테스트
# ----네트워크 규칙은 **액세스 거부**여야 함
echo $private_URI
# 위의 링크 테스트
# *********************************************************

# ----작업 4 – File Storage 만들기----
# ----Azure CLI를 사용하여 스토리지 계정 만들기----

# ----변수 만들기----
my_resource_group=EastRG
location=eastus
my_storage_account=eaststore$myRand

#----eastuse 스토리지 계정 만들기----
az storage account create --name $my_storage_account \
    --resource-group $my_resource_group --location $location \
    --sku Standard_LRS --kind StorageV2

#----https가 아닌 트래픽 허용(그러면 Linux 드라이브 탑재 시 "오류 13"이 발생하지 않음)----
az storage account update -n $my_storage_account --https-only false

#----파일 공유 만들기 및 파일 업로드----
#----환경 변수 설정----
export AZURE_STORAGE_ACCOUNT=$my_storage_account
export AZURE_STORAGE_KEY="$(az storage account keys list --account-name $AZURE_STORAGE_ACCOUNT --resource-group $my_resource_group --query [0].value -o tsv)"
export AZURE_STORAGE_CONNECTION_STRING="$(az storage account show-connection-string -n $AZURE_STORAGE_ACCOUNT -g $my_resource_group --query connectionString -o tsv)"
#----'eastfiles' 파일 공유 만들기----
file_share_name=eastfiles
#----스토리지 파일 공유 만들기----
az storage share create --name $file_share_name --quota 2048
#----업로드할 'myFileShareFile.html' 파일 만들기
echo "File Shares Share Files">myFileShareFile.html
#----공유할 파일 업로드----
az storage file upload --share-name $file_share_name --source ~/myFileShareFile.html

# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
# ******************************************************************
# ---------------------------수동 단계---------------------------
#----Linux 가상 머신에서 파일 공유 탑재----
#----Linux VM으로의 SSH 실행----
#----다음 명령을 Cloud Shell bash에 입력하여 SSH 세션 시작----
#> east_vm_ip="$(az vm show -d -g $my_resource_group -n eastdebianvm --query publicIps -o tsv)"
#> ssh azuser@$east_vm_ip
#
#---- 다음 명령을 SSH 세션에 입력(또는 lab04의 작업 4 참조)----
#----디렉터리를 만들고 Azure Storage에 연결----
#ssh> mkdir -p $my_storage_account/eastfiles
#ssh> sudo apt-get update
#ssh> sudo apt-get install cifs-utils
# ==================================================================
#----포털에서 코드를 가져와 Linux 컴퓨터에 스토리지 연결----
#
#    1. Portal에서 다음 단계를 수행합니다.
#        * eaststorage*123af4* 또는 비슷한 이름의 스토리지 계정을 엽니다.
#        * eastfiles 파일 공유로 이동합니다.
#    1. "eastfiles" 블레이드에서 연결을 클릭합니다.
#    1. **Linux** 탭으로 변경해 연결 문자열을 복사합니다.
#    1. EastDebianVM ssh 세션으로 돌아옵니다.
#    1. ssh 세션에서 연결 문자열을 붙여 넣습니다.
#
# ==================================================================
# ==== 새 Azure Portal 웹 페이지 탭을 열고 Cloud Shell 시작====
#----공유가 탑재되었으므로 https가 아닌 트래픽 허용 중지(CLI)
#---- SSH 세션이 아닌 **Azure CLI**에 명령 입력
#
#> az storage account update -n $my_storage_account --https-only true
#
# ==================================================================
# ==========================**리눅스 SSH 세션으로 돌아가기=====================
#----리눅스 SSH 세션에서 계속 다음 명령 실행----
#
#ssh> cd /mnt/$my_storage_account
#ssh> echo "I am from eastDebianVM">newFile.txt
#---- "ls" 명령을 사용하여 Linux 컴퓨터에서 공유 파일 보기
#----SSH 세션을 종료하려면 "exit" 입력----
#----Azure Portal에 파일(newFile.txt)이 있는지 확인

# =================================================================
# -----------준비가 되면 스토리지 랩 정리-----------
#----------CLI로 돌아가기(ssh 세션 종료)---------
#---다음 명령으로 File Storage 계정 제거----
#> az storage account delete -n eaststore$myRand -g EastRG
#> az storage account delete -n weststore$myRand -g WestRG
#
```

> 위의 명령 스크립트에서 다음 검토
> * 확인 단계
> * SSH에 대한 수동 단계
> * 수동 정리 단계
