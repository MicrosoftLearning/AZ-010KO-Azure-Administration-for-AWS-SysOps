# 랩 04 답변 키

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
