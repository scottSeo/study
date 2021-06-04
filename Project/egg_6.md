# egg 프로젝트

## Swagger-UI
- Swagger 는 서비스가 제공하는 REST API에 대한 스펙을 시각화 해주고, 직접 사용해볼 수 있게 해주는 툴이다.
- 설정 해두면 서비스에 추가되는 REST API에 대한 별도 문서 작성이 불필요해지고, 간편히 테스트 용도로도 사용할 수 있다.
- 우선 의존성을 추가한다.
    - 3.0.0 버전부터는 이전 버전과 설정 방법에서 차이가 난다.
    ```
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-boot-starter</artifactId>
        <version>3.0.0</version>
    </dependency>
    
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>3.0.0</version>
    </dependency>
    ```

- Swagger 설정 파일을 생성한다.
    ```java
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
    import springfox.documentation.builders.PathSelectors;
    import springfox.documentation.builders.RequestHandlerSelectors;
    import springfox.documentation.spi.DocumentationType;
    import springfox.documentation.spring.web.plugins.Docket;

    @Configuration
    public class SwaggerConfig implements WebMvcConfigurer {
        @Bean
        public Docket api() {
            return new Docket(DocumentationType.OAS_30)
                .select()
                .apis(RequestHandlerSelectors.any())
                .paths(PathSelectors.any())
                .build();
        }
    }
    ```
- /swagger-ui/index.html 로 접근해서 확인해본다.

> https://medium.com/@hala3k/setting-up-swagger-3-with-spring-boot-2-a7c1c3151545