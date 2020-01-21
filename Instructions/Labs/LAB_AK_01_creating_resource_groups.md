# 랩 01 답변 키

## 지침

1. 아래 CLI 스크립트를 메모장 등의 편집기에 복사합니다.
   1. 위치 섹션의 제목은 `# ----실행 전에 다음 값 편집----`입니다.
   1. subscriptionID를 편집합니다. 그러지 않으면 오류가 발생합니다.
   1. 파일을 로컬에 저장합니다.
1. Azure Portal에 로그인하여 Bash Cloud Shell을 엽니다.
1. 로컬 파일에서 CLI 스크립트를 복사하여 Bash Cloud Shell에 붙여넣습니다.
1. 실행 시간이 1분보다 오래 걸리는 태스크도 있으므로 스크립트가 완료될 때까지 기다렸다가 출력을 검토하세요.
1. 오류가 발생하는 경우 강사에게 문의하세요.

> 참고: 학생은 랩 01에 요약되어 있는 대로 기본 구독을 설정해야 합니다.

```sh
# AZ-010 LAB1 해답
# 다음 값을 설정합니다.
#   subscriptionID
# 이 값은 Azure CLI Bash에서 아래 명령을 실행하기 전에 설정해야 합니다.
# az account list -o table을 사용하여 구독을 확인합니다.

# ----------시작----------

# ----실행 전에 다음 값 편집----
subscriptionID=[**랩에 사용할 구독 ID**]

# ----주 스크립트----
# ----기본 구독 설정----
az account set --subscription $subscriptionID
# ---기본 구독 확인---
az account list --output table

# ----WestRG 만들기----
az group create --location westus --name WestRG --output table

# ----EastRG 만들기----
az group create --location eastus --name EastRG --output table

# ----리소스 그룹 목록 표시----
az group list -o table
```
