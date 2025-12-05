# 코드레이 OpenAPI JAR 호출
>
> Docker 기반에서 실행이 불가한 경우 Jar를 통한 호출이 가능합니다. \
> 본 가이드는 Jar 호출 부분에 대하여 설명하며, 코드레이 관련 Key 발급 설명은 생략 합니다.

## 사전 준비 작업

- [x] Coderay Wrapper Jar 관련 파일 프로젝트 내 복사 [Wrapper Jar 관련 파일](https://github.com/skccmygit/skax-coderay/tree/main/coderay-open-api)
- [x] 파이프라인에서 해당 Jar를 호출\
      해당 Jar의 경우 코드레이 업체에서 제공합니다. (정보보호팀 연락필요)

## 예시
>
> GithubActions workflow 작성 예시입니다. \
> **variable 및 secret**을 사용하지 않고 .env 파일을 사용한 예시입니다. \
> 프로젝트 성향에 맞게 설정하면 됩니다.

- **env 파일 예시**

    ```properties
    ##코드레이 ACCESS_KEY
    ACCESS_KEY=9e2f3b5178c2f27b3c23e06aa17c41d0
    ##코드레이 SECRET_KEY
    SECRET_KEY=6baf3471984bfcca28283248c577a3cc
    ##코드레이 서비스 네임
    SERVICE_NAME=GithubActions
    ##코드레이 프로젝트코드
    PC_CODE=b9d3c029084040e8a490853fc3e44948
    ##코드레이 형상 인증 KEY  
    ACC_CODE=9f9cc939e5034270858917f6819a09f0
    ```

- **workflow 예시**

    ```yml

    jobs:
    java-run-script:
        runs-on:
        group: organization/Default 
        steps:
        ## 소스 체크 아웃
        - name: Checkout
            uses: actions/checkout@v4
        
        ## 코드레이 Open API 호출을 위한 Java 설정
        - name: Setup Java
            uses: actions/setup-java@v4
            with:
            distribution: temurin
            java-version: '17'  # 8~21 아무 버전 OK

        ## 코드레이 Open API 호출 (run.sh java)
        - name: Sast - CodeRay 호출
            working-directory: coderay-open-api
            run: |
            bash run.sh java
            env:
            REV: ${{ github.event.inputs.environment || github.ref_name }}
            SRC_URL: ${{ github.server_url }}/${{ github.repository }}

        - name: Deploy
            run: echo "Sast step succeeded! Continue pipeline."
    ```

## 참고사항

- 환경변수 설명

  | 환경변수명   | 설명                     |
  | --------- | ------------------------ |
  | ACCESS_KEY    | OpenAPI 생성시 취득한 Access Key                |
  | SECRET_KEY | OpenAPI 생성시 취득한 Secret Key            |
  | SERVICE_NAME     | OpenAPI 생성시 선택한 빌드 플랫폼명 |
  | PC_CODE      | 코드레이 프로젝트 코드      |
  | ACC_CODE    | 형상 인증 코드        |
  | SRC_URL    | 저장소 URL        |
  | REV    | 스캔 대상 브랜치명        |
