# 랩 05 답변 키

## 지침

1. 아래 CLI 스크립트를 메모장 등의 편집기에 복사합니다.
   1. 위치 섹션의 제목은 `# ----실행 전에 다음 값 편집----`입니다.
   1. 값이 사용자의 환경을 나타내도록 편집합니다. 그러지 않으면 **오류**가 발생합니다.
   1. 파일을 로컬에 저장합니다.
1. Azure Portal에 로그인하여 Bash Cloud Shell을 엽니다.
1. 종속성이 있는지 확인합니다(스크립트 맨 위의 주석 참조).
1. 로컬 파일에서 CLI 스크립트를 복사하여 Bash Cloud Shell에 붙여넣습니다.
1. 실행 시간이 1분보다 오래 걸리는 태스크도 있으므로 스크립트가 완료될 때까지 기다렸다가 출력을 검토하세요.
1. 오류가 발생하는 경우 강사에게 문의하세요.

> 학생은 랩 01에 요약되어 있는 대로 기본 구독(subscriptionID)을 설정해야 합니다.
>
> 사용자 도메인용 변수를 만듭니다.
> 계정 이메일 주소가 eric@contoso.com이면 명령이 `my_domain=ericcontoso.onmicrosoft.com`이 됩니다.
>
> * *필요한 경우 강사에게 문의하세요.*

```sh
# AZ-010 LAB5 해답
# ---------------
# 종속성: 이 정상적으로 완료됨(WestRG가 생성됨)
# ---------------
# 다음 값을 설정합니다.
#   subscriptionID
#   my_domain
# 이 값은 Azure CLI Bash에서 아래 명령을 실행하기 전에 설정해야 합니다.
# az account list -o table을 사용하여 구독을 확인합니다.

# ----------시작----------

# ----실행 전에 다음 값 편집----
subscriptionID=[**랩에 사용할 구독 ID**]
# 사용자 도메인용 변수 만들기
my_domain=[**UsernameEmaildomain.onmicrosoft.com**]
# 고유한 AD 사용자 암호 만들기
password_ad_user=[**sTR0ngP@ssWorD543%**]
# +++++++++++++++++++++++++++++++++++++++++++++++++++++

# ----주 스크립트----
# ----기본 구독 설정----
az account set --subscription $subscriptionID

#----사용자 계정 이름(사용자 주체 이름) 만들기----
my_user_account=AZ010@$my_domain

#----사용자 계정 만들기----
az ad user create \
    --display-name AZ010Tester \
    --password $password_ad_user \
    --user-principal-name $my_user_account

#----AD 사용자 목록 표시----
az ad user list --output json | jq '.[] | {"userPrincipalName":.userPrincipalName, "objectId":.objectId}'
# ====================================================
#----위의 출력에 나열된 새 AD 사용자 확인----
# ====================================================

#---- 새 사용자에게 역할("소유자") 추가----
az role assignment create --role "Owner" --assignee $my_user_account --resource-group WestRG

#---- 만든 사용자에 대한 역할 할당 목록 표시(변경 사항 확인)----
az role assignment list --assignee $my_user_account -g WestRG

#----소프트웨어 설치를 제한하는 정책 만들기----
#----정책 정의를 만듭니다.
az policy definition create --name 'require-sqlserver-version12' \
    --display-name 'Require SQL Server version 12.0' \
    --description '이 정책을 적용하는 경우 모든 SQL 서버가 버전 12.0을 사용해야 합니다.' \
    --rules 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-sqlserver-version12/azurepolicy.rules.json' \
    --params 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-sqlserver-version12/azurepolicy.parameters.json' \
    --mode All
#----구독 수준에서 범위 지정----
az policy assignment create --name SQL12AZ010 \
    --display-name 'Require SQL Server version 12.0 - subscription scope' \
    --scope '/subscriptions/'$subscriptionID \
    --policy 'require-sqlserver-version12'

#----새로 만든 정책 표시----
az policy assignment show --name 'SQL12AZ010'

# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
#----연습 2: 쿼리 탐색기를 통해 로그 및 경고 모니터링-----
# ******************************************************************
# ---------------------------수동 단계---------------------------
#----로그 분석 쿼리 데모로 이동----
#---------------https://portal.loganalytics.io/demo ----------------
#-----LAB05의 연습 2에 나온 지침에 따라 쿼리 탐색기 사용-----

```

> **연습 2 - 작업 1 – 모니터링 로그 및 경고 검토**
> 데모 환경 액세스**
>
> 1. 새 브라우저 탭에서 [로그 분석 쿼리 데모](https://portal.loganalytics.io/demo)로 이동합니다.
> 2. 쿼리 탐색기 사용
>     1. 오른쪽 위의 쿼리 탐색기를 선택합니다.
>     2. 즐겨찾기를 확장한 다음 *오류가 있는 모든 Syslog 레코드*를 선택합니다.
>     3. 쿼리가 편집 창에 추가됩니다. 쿼리의 구조를 확인합니다.
>     4. 쿼리를 실행합니다. 반환된 레코드를 살펴봅니다.
>     5. 강사의 추가 단계를 따릅니다.
>     6. 시간이 되면 다른 즐겨찾기와 저장된 쿼리도 사용해 봅니다.
