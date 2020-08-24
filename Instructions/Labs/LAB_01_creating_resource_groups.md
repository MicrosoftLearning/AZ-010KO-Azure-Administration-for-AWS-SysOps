---
lab:
    title: '리소스 그룹 만들기'
    module: '모듈 1: Azure 관리'
---

# 랩 01: 리소스 그룹 만들기

Azure Cloud Shell CLI 소개

## 학생 랩 매뉴얼

## 시나리오

이 랩에서는 Azure Portal에서 과정에 사용할 구독을 설정하는 방법과, Azure CLI 명령을 통해 Azure Cloud Shell을 활용하는 방법을 소개합니다.  과정을 진행하는 강사는 특정 과정 환경에 따라 Azure 구독을 구성하려면 수행해야 하는 모든 단계를 제시해야 합니다. 여기서는 리소스 그룹 2개를 구성하여 Azure Cloud Shell에서 과정의 첫 번째 CLI 명령을 실행합니다.

## 목표

이 랩을 완료하면 다음 작업을 수행할 수 있습니다.

* Azure Portal을 사용하여 구독 설정(필요에 따라 과정을 진행하는 강사가 단계를 제시함)
* Azure CLI 명령을 사용하여 리소스 그룹 만들기

## 랩 설정

* **예상 소요 시간**: 20분

## 지침

### 시작하기 전에

#### 설정 태스크

1. 과정을 진행하는 강사가 제시하는 단계에 따라 Azure Portal을 사용하여 이 과정에 사용할 Azure 계정을 구성합니다.

## 연습 1: Cloud Shell을 사용하여 Azure CLI를 시작한 다음 리소스 그룹 2개를 만듭니다.

이 연습의 주요 태스크는 다음과 같습니다.

1. Azure Cloud Shell 시작
1. 랩에 사용할 기본 구독 설정
1. CLI 명령을 사용하여 WestRG 리소스 그룹 만들기
1. CLI 명령을 사용하여 EastRG 리소스 그룹 만들기

#### 연습 1 - 태스크 1: Cloud Shell 열기

**Azure Cloud Shell에서 구독 설정**

1. Portal 상단에서 **Cloud Shell** 아이콘을 클릭하여 Cloud Shell 창을 엽니다.

1. Cloud Shell 인터페이스에서 **Bash** 를 선택합니다.

1. **Cloud Shell** 명령 프롬프트에 다음 명령을 입력하고 **Enter** 키를 눌러 Portal 로그인에 사용하는 계정과 연결된 모든 구독의 목록을 표시합니다.

```sh
az account list --output table
```

1. 구독 목록을 검토하여 "default `true`" 레이블이 지정된 구독이 있는지 확인합니다.
1. 원하는 구독이 기본값으로 설정되어 있지 않으면 기본 구독을 다시 설정합니다.
1. **Cloud Shell** 명령 프롬프트에서 **원하는 구독 ID** 를 포함해 다음 명령을 입력하고 **Enter** 키를 눌러 기본 구독을 설정합니다.

```sh
# --subcrition 값은 자신의 구독 ID 또는 구독 이름으로 바꿉니다.
az account set --subscription [1111a1a1-22bb-3c33-d44d-e5e555ee5eee]
az account list --output table
```

4. 구독 목록을 검토하여 적절한 구독에 "default `true`" 레이블이 지정되어 있는지 확인합니다.

### 태스크 2: CLI를 사용하여 WestRG 리소스 그룹 만들기

1. **Cloud Shell** 명령 프롬프트에서 다음 명령을 입력하여 westus 지역에서 WestRG 리소스 그룹을 만듭니다.

```sh
az group create --location westus --name WestRG --output table
```

2. **Cloud Shell** 명령 프롬프트에서 다음 명령을 입력하여 westus 지역에서 사용 가능한 리소스 그룹의 목록을 표시합니다.

```sh
az group list --output table
```

3. 새로 만든 WestRG가 목록에 포함되어 있는지 확인합니다.

### 태스크 3: CLI를 사용하여 EastRG 리소스 그룹 만들기

1. **Cloud Shell** 명령 프롬프트에서 다음 명령을 입력하여 eastus 지역에서 EastRG 리소스 그룹을 만듭니다.

```sh
az group create --location eastus --name EastRG --output table
```

2. **Cloud Shell** 명령 프롬프트에서 다음 명령을 입력하여 eastus 지역에서 사용 가능한 리소스 그룹의 목록을 표시합니다.

```sh
az group list --output table
```

3. 새로 만든 EastRG가 목록에 포함되어 있는지 확인합니다.

> **결과**: 이 랩에서는 기본 Azure 구독을 구성하고 이후 랩에서 계속 사용할 리소스 그룹 2개를 만들었습니다.
