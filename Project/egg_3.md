# egg 프로젝트

## Spring Boot
- 이제 Spring Boot 를 설정해보자.
- pom.xml 에 Spring Boot 설정을 위해 의존성을 추가한다.
- 우선 다양한 의존성 간의 버전 호환성 확보를 위해 `spring-boot-starter-parent`를 등록한다.
- mvnrepository.com 에 가서 `spring-boot-starter-parent`를 검색해서 최신 버전을 선택하고 정보를 확인한다.
- pom.xml에 등록해준다.
```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.0</version>
</parent>
```
- 그리고 웹 프로젝트로 만들거니까 `spring-boot-starter-web`도 검색해서 의존성 추가해준다.
- 현재 pom.xml엔 정말 아무것도 없으니까.. <dependencies></dependencies> 블럭도 추가해서 안에 넣어준다.
```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.5.0</version>
    </dependency>
</dependencies>
```
- 내 개발환경에 해당하는 의존성이 다운로드 된 적이 없다면 버전정보가 빨갛게 표시될 것이다. 
- IntelliJ 우측 `Maven` 탭을 선택해서 상단에 `Reload All Maven Proejcts` 버튼을 눌러준다. 그럼 하단에 프로그레스바가 차 오르면서 의존성을 다운로드 받고, 빨갛게 표시되었던 버전정보가 정상적으로 변한다.
- 이제 적절한 패키지를 생성하고 Spring Boot 가 제공해주는 설정으로 초 간단하게 서버프로그램을 작성한다.
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class);
    }
}
```
- 이제 이 main 프로그램을 구동하면 서버가 동작한다.
- `public static void main(String[] args) {`라인 좌측에 아이콘을 클릭 해서 `Run Application` 을 선택한다.
- 아래 `Console`에 로그가 올라오며 서버가 구동된다.
```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.5.0)

.........
.........
.........
```
- 로그 내용을 좀 살펴보면, 다음과 같은 내용을 알 수 있다.
    - No active profile set (현재 설정된 프로파일이 없음. 그래서 기본으로 구동.)
    - Tomcat initialized with port(s): 8080 (http) (Embedded Tomcat으로 구동되었으며 8080 서버 http 프로토콜로 제공.)

- 8080 포트로 구동되었다는 것을 알았으니 웹 브라우저를 통해 `http://localhost:8080`에 접근해보자.
- 당연하지만, 응답할 수 있는 무엇도 존재하지 않아서 기본 에러 페이지가 반환된다.
- 기본적인 응답을 확인해볼 수 있도록 간단하게 Controller 를 작성한다.
- 적절한 패키지에 TestController를 작성한다.
    - 앞서 작성한 Application 클래스와 최소한 같은 깊이 패키지에 작성해야 한다. 보다 상위에 있으면 ComponentScan이 안됨.
```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TestController {
	@GetMapping
	public String hello() {
		return "HELLO";
	}
}
```
- 그냥 루트 path (/) 요청에 "HELLO"라고 응답하는 Controller 코드를 작성해봤다.
- 지금 작성한 클래스가 적용되기 위해서 서버 재시작이 필요하다. 
- 서버 재시작 후 다시 `http://localhost:8080`에 접근하면 "HELLO"메시지를 볼 수 있다.

> 참고 : https://goddaehee.tistory.com/238