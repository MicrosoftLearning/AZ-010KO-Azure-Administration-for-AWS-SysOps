# 랩 03(2부) 답변 키

## 지침

1. 아래 CLI 스크립트를 메모장 등의 편집기에 복사합니다.
1. 위치 섹션의 제목은 `# ----실행 전에 다음 값 편집----`입니다.
1. 값이 사용자의 환경을 나타내도록 편집한 다음 파일을 로컬에 저장합니다.
1. Azure Portal에 로그인하여 Bash Cloud Shell을 엽니다.
1. 로컬 파일에서 CLI 스크립트를 복사하여 Bash Cloud Shell에 붙여넣습니다.
1. 실행 시간이 1분보다 오래 걸리는 태스크도 있으므로 스크립트가 완료될 때까지 기다렸다가 출력을 검토하세요.
1. 오류가 발생하는 경우 강사에게 문의하세요.

> 참고: 학생은 랩 01에 요약되어 있는 대로 기본 구독을 설정해야 합니다.

```sh
# AZ-010 LAB3 해답
# ---------------
# 종속성:
#   LAB1이 정상적으로 완료됨(WestRG 및 EastRG가 생성됨)
# ---------------
# 경고: 1분보다 오래 걸리는 단계도 있습니다.
# 단계가 끝날 때까지 기다리세요. 오류 발생 시에는 강사에게 문의하세요.

# ----------시작----------

# ----주 스크립트----

# ----연습 2: Ubuntu 확장 집합 만들기 및 테스트----
# ----Ubuntu 확장 집합 만들기----

# ----확장 집합 리소스 그룹 만들기----
az group create --name EastScaleRG --location eastus

# ----확장 집합 만들기----
az vmss create \
  --resource-group EastScaleRG \
  --name EastUbuntuServers \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --instance-count 2 \
  --admin-username azuser \
  --generate-ssh-keys

# ----자동 크기 조정 프로필 정의----
az monitor autoscale create \
  --resource-group EastScaleRG \
  --resource EastUbuntuServers \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale \
  --min-count 1 \
  --max-count 3 \
  --count 1

# ----자동 확장용 규칙 만들기----
az monitor autoscale rule create \
  --resource-group EastScaleRG \
  --autoscale-name autoscale \
  --condition "Percentage CPU > 50 avg 2m" \
  --scale out 1

# ----자동 규모 감축용 규칙 만들기----
az monitor autoscale rule create \
  --resource-group EastScaleRG \
  --autoscale-name autoscale \
  --condition "Percentage CPU < 30 avg 1m" \
  --scale in 1

# --------Ubuntu 확장 집합 테스트(선택 태스크)--------

# ----다음 출력을 사용하여 확장 집합 Ubuntu 서버 세션에 대한 SSH 연결----
az vmss list-instance-connection-info \
--resource-group EastScaleRG \
--name EastUbuntuServers

# *********************************************************************************
# ----위의 출력을 사용하여 SSH를 통해 Ubuntu 확장 집합 인스턴스 0에 연결----
# ----랩 03 연습 2 태스크 2로 이동하여 다음 단계 완료
# ----     * 스트레스 설치/실행
# ----     * 크기 조정 확인
# *********************************************************************************

# ----아래 CLI 명령을 실행하여 확장 집합 데모 정리----
# az group delete --name EastScaleRG --yes --no-wait
```

> Ununtu 확장 집합을 테스트하는 최종 태스크(선택 사항)를 실행하려면
> **위의 최종 연결 정보 출력을 캡처하여 SSH를 통해 Ubuntu 확장 집합 인스턴스 0에 연결**
> 그런 다음 랩 03 연습 2 태스크 2로 이동하여 다음 단계 완료
>
> * 스트레스 설치/실행
> * 크기 조정 확인
> * 확장 집합 정리
>
