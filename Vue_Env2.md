# Vue 에서 환경 변수 사용.

## vue cli는 기본적으로 3개의 모드를 제공함.
  - development
  - production
  - test
 
## 그 외 사용자 모드를 정의할 수 있음.

/package.json
```json
{
  "scripts": {
    "custom": "vue-cli-service serve --mode custom"
  }
}
```

## 모드가 변경된다는 것은 환경변수 또한 바뀌었다는 의미
- 기본적으로 2개의 환경변수 NODE_ENV, BASE_URL이 있고 사용자가 정의한 변수를 추가할 수 있음.
  - NODE_ENV : 앱이 실행되는 모드. development, production, test, 사용자 정의 모드
  - BASE_URL : vue.config.js의 publicPath 옵션에 해당하고 앱이 배포되는 기본 경로.
  - 환경 변수들은 process.env 객체에 정의되어 있음.

## 사용자 변수 정의
- 환경 변수 파일을 생성하여 정의
  - 환경 변수 파일은 루트에 생성해야 하며 아래와 같은 규칙을 따른다.
  ```
  /.env.{모드명}
  
  ex)
  /.env.local
  /.env.development
  /.env.custom
  ```

## 사용자 변수 정의 유의사항
- **사용자 정의 변수는 반드시 "VUE_APP_" 접두어를 붙여야만 한다.**

> https://velog.io/@skyepodium/vue-%EC%8B%A4%ED%96%89-%EB%AA%A8%EB%93%9C%EC%99%80-%ED%99%98%EA%B2%BD-%EB%B3%80%EC%88%98-%EC%84%A4%EC%A0%95
