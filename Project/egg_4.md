# egg 프로젝트

## 프로파일
- 진행중인 프로젝트가 개발환경과 운영환경에서 설정이 달라야 할 필요가 있다.
- 이럴 때 프로파일을 설정하고, 환경에 따라 다른 설정으로 서버가 동작해야 한다.
- 서버 실행을 위해 클릭했던 아이콘을 다시 클릭 해보면 [Modify Run Configuration...]이 있다. 
- 서버 실행 시 옵션을 주고 프로파일을 지정할 수 있다.
- 중간 쯤 `Active profiles` 란 항목에 `dev` 라고 입력 후 [OK] 를 클릭해서 창을 닫고 서버를 구동해보자.
- `Console` 로그에서 달라진 메세지를 확인할 수 있다.
    ```
    The following profiles are active: dev
    ```
- dev 프로파일이 활성화 되어 있다는 뜻이다. 하지만 현재는 아무런 설정이 없어서 변함이 없다.
- 프로파일에 따라 설정을 변화시키기 위해 `application.yml`파일을 생성한다.
    - src/main/resources 경로에 생성한다.
    - `application.yml`은 Spring Boot가 각 프로파일 정보를 설정해두고 적용하기 위해 지정된 파일명이다.
    - properties 파일과 같은 역할이지만 사용법에서 차이가 있다.
    - yml은 들여쓰기가 중요하다.
    
    ```yml
    spring:
        config:
            activate:
                on-profile: dev
    server:
        port: 80

    ---

    spring:
        config:
            activate:
                on-profile: real
    server:
        port: 10080
    ```
- dev와 real 프로파일을 설정하고 각각 포트를 80, 10080 으로 설정하였다.
- 이제 dev 프로파일로 서버가 구동 될 때는 80포트로, real 프로파일로 서버가 구동 될 때는 10080포트로 구동된다.
