---
lab:
    title: 'Blob, 보안 액세스 스토리지, 서비스 엔드포인트 및 File Storage'
    module: '모듈 4: Azure Storage'
---

# 랩 04: Azure Storage

스토리지 계정, Blob, 보안 액세스 스토리지, 서비스 엔드포인트 및 File Storage

## 학생 랩 매뉴얼

## 시나리오

이 랩에서는 Bash 인터페이스를 통해 Cloud Shell에서 CLI 명령을 사용하여 Azure Storage를 관리합니다. 구체적으로는 스토리지 계정, Azure Blob Storage, SAS(보안 액세스 스토리지), 서브넷 서비스 엔드포인트 및 File Storage를 구성합니다.

## 목표

이 랩을 완료하면 Azure CLI를 사용하여 다음 항목을 만들고 구성할 수 있습니다.

* 스토리지 계정
* Blob Storage
* SAS(보안 액세스 스토리지)
* 서브넷 서비스 엔드포인트
* File Storage

## 랩 설정

* **예상 소요 시간**: 45분

## 지침

### 시작하기 전에

#### 설정 태스크

1. **이전 랩과의 의존 관계:**
    1. 모듈 1: Azure 관리 - **랩: 리소스 그룹 만들기**. EastRG 및 WestRG 리소스 그룹을 구성한 상태여야 합니다.
    1. 모듈 2: Azure 네트워킹 - **랩: 가상 네트워크 및 피어링**. 서브넷이 포함된 VNet과 피어링을 구성한 상태여야 합니다.
    1. 모듈 3: Azure 컴퓨팅 - **랩: Azure VM**. EastDebianVM을 만든 상태여야 합니다.

### 연습 1: Blob, 보안 액세스 스토리지, 서비스 엔드포인트 및 File Storage

이 연습의 주요 태스크는 다음과 같습니다.

1. Blob Storage 만들기 및 구성
1. SAS(보안 액세스 스토리지)를 사용하여 Blob 구성
1. 서브넷 서비스 엔드포인트 구성
1. File Storage 만들기 및 구성

#### 태스크 1: Blob Storage를 사용하여 스토리지 계정 만들기

**스토리지 계정 변수 만들기**

1. 고유한 이름에 사용할 임의 문자열 생성

```sh
myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "추가되는 임의 문자열:  "$myRand
```

2. 변수 설정


```sh
# 변수 설정
my_resource_group=WestRG
location=westus
my_storage_account=weststore$myRand
my_storage_sku=Standard_RAGRS # 기본값
my_storage_kind=StorageV2myRand
my_access_tier=hot
my_storage_encryption=blob

echo "my_storage_account 이름: " $my_storage_account
```

>위 명령에서 스토리지 계정의 이름을 확인합니다.

**스토리지 계정 만들기**

```sh
# 계정 만들기
az storage account create \
    -n $my_storage_account \
    -g $my_resource_group \
    -l $location \
    --kind $my_storage_kind \
    --access-tier $my_access_tier \
    --sku $my_storage_sku \
    --encryption $my_storage_encryption
```

**Bash CLI에서 환경 변수 설정**

1. 스토리지 계정 및 스토리지 계정 연결 문자열 환경 변수 만들기

```sh
# 환경 변수 설정
export AZURE_STORAGE_ACCOUNT=$my_storage_account

export AZURE_STORAGE_CONNECTION_STRING="$(az storage account show-connection-string \
-n $AZURE_STORAGE_ACCOUNT -g $my_resource_group --query connectionString -o tsv)"
```

2. 스토리지 계정 키 환경 변수 만들기

```sh
# 키 표시
az storage account keys list \
    --account-name $AZURE_STORAGE_ACCOUNT \
    --resource-group $my_resource_group \
    --output table

# AZURE_STORAGE_KEY=<storage_account_key1> 내보내기
# key1을 환경 변수로 저장
export AZURE_STORAGE_KEY="$(az storage account keys list --account-name \
$AZURE_STORAGE_ACCOUNT --resource-group $my_resource_group --query [0].value -o tsv)"
```

> 팁: 모든 변수와 환경 변수 명령을 텍스트 파일에 캡처해 두세요. 그러면 20분이 지나 Cloud Shell 시간이 초과되어 임시 메모리에서 변수가 손실되더라도 쉽게 명령을 다시 입력할 수 있습니다.

**Blob Storage에 추가할 파일 준비**

1. Blob Storage에 업로드할 `helloAdmin.html` 파일 만들기

```sh
# 컨테이너에 추가할 파일을 Blob로 만들기
echo "<h1>Azure 관리자 여러분 안녕하세요</h1>">helloAdmin.html
```

2. `helloAdmin.html` 파일이 작성되었는지 확인

```sh
ls
```

**west 스토리지 계정에 공용 Blob 컨테이너 만들기**

1. 공용 액세스 권한이 있는 Blob 컨테이너 만들기

```sh
container_name=westblobcontainerpublic

az storage container create --name $container_name --public-access blob
# --connection-string $AZURE_STORAGE_CONNECTION_STRING
```

**공용 Blob 컨테이너에 파일 업로드**

```sh
blob_name=helloAdmin

az storage blob upload \
    --container-name $container_name \
    --file helloAdmin.html \
    --name $blob_name
    #--connection-string $AZURE_STORAGE_CONNECTION_STRING
```

#### 공개 파일 다운로드

1. `helloAdmin.html` 이 새 이름(`helloAdminDownload.html`)으로 다운로드됩니다.

```sh
az storage blob download \
    --container-name $container_name \
    --name $blob_name \
    --file helloAdminDownload.html
    #--account-name $AZURE_STORAGE_ACCOUNT \
```

#### `ls` 를 실행하여 `helloAdminDownload.html` 파일이 다운로드되었는지 확인

```bash
ls
```

**upload-batch**

1. `upload` 디렉터리 만들기

```sh
# uploadfiles 디렉터리에 파일 만들기
mkdir uploadfiles
```

2. `upload` 디렉터리에 업로드할 파일 만들기

```sh
# 파일 만들기...
touch uploadfiles/myfile.html
touch uploadfiles/more.html
touch uploadfiles/hi.html
touch uploadfiles/myfile.txt
```

3. `myUploadPath` 및 `myDownloadPath`에 사용할 `az_user_name` 변수 만들기

> 참고: `az_user_name`을 설정하려면 아래 명령을 편집해야 합니다.
>
> Cloud Shell 프롬프트에서 이름 `<이름>@azure` 캡처
> 예 - Cloud Shell의 프롬프트가 `eric@Azure:~$`인 경우 `az_user_name='eric'` 설정

```sh
# <이름> 바꾸기
az_user_name=<이름>
```

4. 업로드 경로 및 폴더 만들기

```sh
# Cloud Shell에서 $home으로 경로 지정
myUploadPath=/home/$az_user_name/uploadfiles
```

5. 컨테이너에 파일 일괄 업로드

```sh
# 경로에 html 파일 업로드
az storage blob upload-batch -d $container_name \
-s $myUploadPath -o table
```

6. Blob의 파일 목록 표시/업로드한 파일 확인

```sh
az storage blob list --container-name $container_name -o table
```

**download-batch**

1. `download` 디렉터리 만들기

> 참고: 위 지침에 따라 `az_user_name` 변수를 설정했는지 확인하세요.

2. 업로드 경로 및 폴더 만들기

```sh
mkdir downloadfiles

myDownloadPath=/home/$az_user_name/downloadfiles
```

3. 컨테이너에서 파일 일괄 다운로드

```sh
az storage blob download-batch -d $myDownloadPath -s $container_name
```

4. download 디렉터리의 파일 목록 표시

```sh
# downloadfiles 나열
ls downloadfiles
```

**Blob의 URL을 가져와 공개 파일 확인**

1. 명령을 실행한 다음 생성된 링크를 클릭합니다.

```sh
az storage blob url -c $container_name -n helloAdmin -o tsv
```

**공용 Blob 컨테이너 제거**

1. 검토 완료 후 Blob 컨테이너 정리

```bash
az storage container delete -n $container_name
```

#### 태스크 2: 스토리지 보안 액세스

**west 스토리지 계정에 프라이빗 Blob 컨테이너 만들기**

1. 필요한 경우 변수 새로 고침
1. 생성된 계정 이름을 반영하도록 `AZURE_STORAGE_ACCOUNT` **편집**

> 참고: `container_name`의 새 값은 `westblobcontainerprivate` 입니다.

```sh
my_resource_group=WestRG
location=westus
container_name=westblobcontainerprivate

export AZURE_STORAGE_ACCOUNT=<*스토리지 계정 입력*> # $my_storage_account

export AZURE_STORAGE_KEY="$(az storage account keys list \
    --account-name $AZURE_STORAGE_ACCOUNT \
    --resource-group $my_resource_group \
    --query [0].value -o tsv)"

export AZURE_STORAGE_CONNECTION_STRING="$(az storage account \
    show-connection-string -n $AZURE_STORAGE_ACCOUNT\
     -g $my_resource_group --query connectionString -o tsv)"
```

**프라이빗 액세스 권한이 있는 Blob 컨테이너 만들기**

1. 비공개 Blob 컨테이너 새로 만들기

```sh
az storage container create --name $container_name --public-access blob
```

**프라이빗 Blob 컨테이너에 파일 업로드**

```bash
echo "<h1>Azure 관리자 여러분 안녕하세요 - 비공개 메시지</h1>">helloAdmin.html
blob_name=helloAdmin

az storage blob upload \
    --container-name $container_name \
    --file helloAdmin.html --name $blob_name
```

**서비스 수준에서 SAS(공유 액세스 서명) 만들기**

1. `end_date` 설정

> *참고: 30분 동안만 액세스 가능하도록 구성해야 합니다.*

```sh
# 컨테이너에 30분 동안 사용 가능한 읽기 전용 SAS 토큰을 만들고 CONTAINER_SAS_KEY로 저장
end_date=`date -u -d "30분" '+%Y-%m-%dT%H:%MZ'`
```

2. 컨테이너 및 Blob 키 변수 생성

```sh
CONTAINER_SAS_KEY="$(az storage container generate-sas \
    --name $container_name --https-only --auth-mode key \
    --expiry $end_date --permissions r -o tsv)"

BLOB_SAS_KEY="$(az storage blob generate-sas \
    --name $blob_name \
    --container-name $container_name \
    --permissions r --expiry $end_date \
    --auth-mode key -o tsv)"
```

3. 전체 Blob URI 생성/링크 테스트

```sh
# 전체 Blob URI 생성/링크 테스트
private_URI="$(az storage blob generate-sas \
    --name $blob_name \
    --container-name $container_name \
    --permissions r --auth-mode key \
    --expiry $end_date --https-only \
    --full-uri)"

echo $private_URI
```




