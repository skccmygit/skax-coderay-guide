# 코드레이 연계 가이드
>
> 기존에 구성된 DevOps 파이프라인에 `소스정적분석 단계 추가`하는 것을 안내합니다.\
> 참고로, `코드레이`는 SK AX에서 사용하고 있는 **소스정적분석도구 솔루션**입니다.\
> 본 가이드는 DevOps 파이프라인에 **단계 추가**에 관한 부분만 안내하며\
> 실제 코드레이 사용법은 정보보호팀에게 연락 하시기 바랍니다. \
> 도커이미지경로: ghcr.io/skccmygit/coderay-openapi:latest

## 사전 준비 작업

- [X] **코드레이 접속 계정** (<https://coderay.skax.co.kr:28443>)
- [X] **코드레이 관리 프로젝트 코드 정보**
- [X] **코드레이 연동관리 인증 KEY**
- [X] **코드레이 Open API ACCESS 및 SECRET KEY**
- [X] **빌드서버 → 코드레이 서버 방화벽해제**
- [X] **코드레이서버 → 소스저장소 방화벽해제**

※ 코드레이 계정 및 프로젝트 존재하지 않을 경우 신청 필요

## 코드레이 연동 프로세스

### 1. 형상관리 인증 KEY 발급 및 접속 테스트

- **코드레이 접속로그인 [https://coderay.skcc.com:28443](https://coderay.skcc.com:28443)**\
  ![코드레이 로그인](./images/coderay_0_login.png)
- **우측상단 - 관리 - 연동관리 클릭**\
  ![코드레이 연동관리](./images/coderay_1_set_1.png)
- **연동관리 - 형상관리 인증 정보 클릭 → 우측 `+` 클릭**\
  ![코드레이 연동관리 추가](./images/coderay_1_set_2.png)
- **인증이름, 아이디, 토큰 입력**\
  ![인증 정보 입력](./images/coderay_1_set_3.png)
- **생성된 인증정보의 `새로고침` 클릭**\
  ![새로고침 클릭](./images/coderay_2_con_test_1.png)
- **GIT 접속 URL 입력 후 `접속 테스트` 클릭**\
  ![접속테스트 클릭](./images/coderay_2_con_test_2.png)
- **접속 성공 시, 아래와 같은 메시지 안내**\
  ![접속 성공](./images/coderay_2_con_test_3.png)\
- **형상관리 인증 정보의 문자열은 `연동관리 인증 KEY`로 사용됩니다.**\
  ![새로고침 클릭](./images/coderay_2_con_test_4.png)

### 2. Open API 접속 KEY 발급

- **우측상단 - 관리 - 연동관리 클릭**\
  ![코드레이 연동관리](./images/coderay_1_set_1.png)
- **연동관리 - Open API 클릭 → 우측 `+` 클릭**\
  ![코드레이 연동관리 API 추가](./images/openapi_set_1.png)
- **Open API 기능(형상관리분석, 분석상태, 분석결과) 선택, 서비스(빌드플랫폼) 선택, 코드레이 로그인 비밀번호 입력후 확인 버튼 클릭**\
  ![OPEN API 생성](./images/openapi_set_2.png)
- **생성된 ACCESS Key 및 SECRET Key를 저장**\
  ![ACCESS, SECRET KEY 저장](./images/openapi_set_3.png)

### 3. 코드레이 프로젝트 코드 확인

- **코드레이 프로젝트 코드 확인**\
  ![프로젝트 코드 확인](./images/openapi_set_4.png)

### 4. CICD 파이프라인에 코드레이 연계 단계 추가
>
> 기존 빌드 스텝에서 `코드레이 연계 단계만 추가`합니다.\
> 코드레이 연계는 Docker 기반으로 호출 합니다.\
> Docker 호출 시, `변수 또는 시크릿`은 빌드 플랫폼 내의 기능을 활용하여도 무방합니다.\
> 도커이미지 정보: **ghcr.io/skccmygit/coderay-openapi:latest**
  
#### 2-1. GithubActions workflow
>
> 기존의 workflow.yml 파일 내용에서 `Coderay 연동 부분`만 추가하면 됩니다. \
> Github Actions variable 및 secret을 이용한 예제입니다. \
> workflow 구성은 각 프로젝트에 맞게 만들면 됩니다. (Job 또는 Step 구성)

- 아래는 Step 구성으로 호출한 샘플 입니다. (체크아웃-빌드-코드레이스캔-배포)

  ```yml
  jobs:
    build:
      runs-on:
        group: organization/Default
      steps:
        - name: Checkout
          uses: actions/checkout@v3

        - name: Build
          run: |
            echo "===== Build Stage ====="
            echo "빌드 수행"
        
        - name: Coderay Docker Image Pull
          run: docker pull ${{ vars.CODERAY_IMAGE_NAME }}
        - name: Coderay Scan
          run: |
            docker run --rm \
              --name coderay-open-api \
              -e SERVICE_NAME=${{ vars.SERVICE_NAME }} \
              -e ACCESS_KEY=${{ secrets.ACCESS_KEY }} \
              -e SECRET_KEY=${{ secrets.SECRET_KEY }} \
              -e PC_CODE=${{ vars.PC_CODE }} \
              -e ACC_CODE=${{ vars.ACC_CODE }} \
              -e SRC_URL=${{ github.server_url }}/${{ github.repository }} \
              -e REV=${{ github.event.inputs.environment || github.ref_name }} \
              ${{ vars.CODERAY_IMAGE_NAME }}

        - name: Deploy
          run: |
            echo "Deploy Stage"
            echo "배포 수행"
  ```

#### 2-2. Jenkins
>
> Jenkins 파이프라인에서 `코드레이 연계 단계(stage)만 추가` 추가 하여 사용하면 됩니다.
>

```groovy
pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        sh "echo ===== Checkout Stage ====="
        sh "echo 각 저장소에서 체크아웃을 받음"
      }
    }

    stage('Build') {
      steps {
        sh "echo ===== Build Stage ====="
        sh "echo 각저장소에서 소스 빌드"
      }
    }

    stage('SAST') {
      steps {
        script {
          int exitCode = sh(
            script: '''
              echo "===== SAST Scan (SUCCESS) ====="
              docker run --rm \
                --name coderay-open-api-success \
                -e SERVICE_NAME=<서비스명> \
                -e ACCESS_KEY=<코드레이 접근키> \
                -e SECRET_KEY=<코드레이 비밀키> \
                -e PC_CODE=<코드레이 프로젝트코드> \
                -e ACC_CODE=<코드레이 형상관리 키> \
                -e SRC_URL=<저장소 URL> \
                -e REV=<분석대상 브랜치> \
                ghcr.io/skccmygit/coderay-openapi:latest
            ''',
            returnStatus: true
          )

          if (exitCode == 0) {
            echo "✅ SAST PASS (Success Flow)"
            env.SAST_RESULT = "PASS"
          } else {
            echo "⚠️ SAST FAIL (exitCode=${exitCode}) - 배포 막을 예정"
            env.SAST_RESULT = "FAIL"
          }
        }
      }
    }

    stage('Deploy') {
      when {
        expression { env.SAST_RESULT == "PASS" }
      }
      steps {
        sh "echo ===== Deploy Stage Success Flow ====="
        sh "echo 배포 수행"
      }
    }
  }

  post {
    success { echo "🎉 Pipeline completed successfully!" }
    failure { echo "❌ Pipeline failed" }
  }
}

```

### 5. 코드레이 사이트에서 결과 확인

- **코드레이 결과확인 (좌측 분석회차별 목록 클릭하여 확인)**
  ![코드레이 결과확인](./images/coderay_result.png)

## 참고사항

- 환경변수 설명

  | 환경변수명   | 설명                     |
  | --------- | ------------------------ |
  | ACCESS_KEY    | OpenAPI 생성시 취득한 Access Key                |
  | SECRET_KEY | OpenAPI 생성시 취득한 Secret Key            |
  | SERVICE_NAME     | OpenAPI 생성시 선택한 빌드 플랫폼명 |
  | PC_CODE      | 코드레이 프로젝트 코드      |
  | ACC_CODE    | 형상 인증 키        |
  | SRC_URL    | 저장소 URL        |
  | REV    | 스캔 대상 브랜치명        |

- 코드레이 최초 연동시에는 분석시간이 오래 걸릴 수 있습니다.\
  최초 이후 분석의 경우 `증분대상`만 스캔합니다.
- **운영 파이프라인**에서는 `SAST 분석`단계는 반드시 포함되어야 합니다. \
  그 외 단계에서는 프로젝트 내부적으로 관리하시길 바랍니다.
- Docker 기반 호출이 불가능한 상황에서는 `Jar 파일` 수행으로 진행하면 됩니다.\
  [코드레이 JAR 연계가이드](./coderay-openapi-guide-jar/README.md)
