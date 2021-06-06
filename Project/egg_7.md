# egg 프로젝트

## MyBatis
- 우선 DB 접속을 위한 DataSource 세팅부터 해야 한다.
- jdbc와 DB 접속을 위한 드라이버가 필요하니 spring-boot-starter-jdbc, mysql-connector-java 의존성을 추가한다.
    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
        <version>2.5.0</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.22</version>
    </dependency>
    ```
- 이후 application.yml 에 DB 정보를 설정한다.
    ```yml
    spring:
        datasource:
            url: jdbc:mysql://123.456.789.0:1234?user=dev_db
            username: user
            password: 1234567890
            driver-class-name: com.mysql.cj.jdbc.Driver
    ```
- 설정한 정보로 DB 에 정상 접근이 되는지 확인하기 위해 TestRunner를 작성한다.
- ApplicationRunner 를 구현한 클래스는 스프링 부트 기동 이후 자동적으로 run 메서드가 수행된다.
    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.ApplicationArguments;
    import org.springframework.boot.ApplicationRunner;
    import org.springframework.stereotype.Component;

    import javax.sql.DataSource;
    import java.sql.Connection;

    @Component
    public class TestRunner implements ApplicationRunner {
        @Autowired
        private DataSource dataSource;

        @Override
        public void run(ApplicationArguments args) throws Exception {
            try (Connection connection = dataSource.getConnection()) {
                System.out.println("성공");
            } catch (Exception ex) {
                System.out.println("실패");
            }
        }
    }
    ```
- 서버를 구동시켜보고 로그에 "성공"이 나타나는지 확인해본다.
- 만약 "실패"라면 설정이 잘못되었거나 다른 이유로 DB에 접근 할 수 없는 것이다.
- 이제 MyBatis 설정을 진행하자.
- mybatis-spring-boot-starter 의존성을 추가한다. 이 의존성은 내부적으로 spring-boot-starter-jdbc 가지고 있으니 기존에 추가했던 spring-boot-starter-jdbc 의존성은 제거해도 무방하다.
    ```xml
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.2.0</version>
    </dependency>
    ```
- 이대로 다시 서버를 구동해보면 아래와 같은 로그를 확인할 수 있는데, 아직 MyBatis를 위한 설정이 추가되지 않았기 때문이다.
    ```
    No MyBatis mapper was found in '[my.package]' package. Please check your configuration.
    ```
- MyBatis 설정을 위해 사용되는 파일은 두가지. Mapper Interface와 Mapper Xml 파일이다. 
- Mapper Interface는 적절한 패키지에, Mapper Xml 파일은 resources 아래에 둔다.
    - TestMapper.java
    ```java
    import org.apache.ibatis.annotations.Mapper;

    @Mapper
    public interface TestMapper {
        int test();
    }
    ```

    - TestMapper.xml
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>

    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

    <mapper namespace="com.scott.TestMapper">
        <select id="test" resultType="int">
            SELECT 1
        </select>
    </mapper>
    ```
- TestMapper.java의 @Mapper애너테이션은 TestMapper.java가 MyBatis의 Mapper임을 알려주는 애너테이션이다.
- 서버를 다시 구동해보면 아까 확인했던 메세지가 사라진 것을 알 수 있다.
- TestRunner.java를 수정해서 실제 잘 동작하는지 테스트 해보자.
    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.ApplicationArguments;
    import org.springframework.boot.ApplicationRunner;
    import org.springframework.stereotype.Component;

    @Component
    public class TestRunner implements ApplicationRunner {
        @Autowired
        private TestMapper testMapper;

        @Override
        public void run(ApplicationArguments args) throws Exception {
            System.out.println("MyBatis test result : " + testMapper.test());
        }
    }
    ```

### Mapper Interface 와 Mapper Xml을 같은 위치에 두기
- 아무래도 Mapper Interface 와 Maper Xml는 서로 매우 밀접한 관계를 가지고 있기 때문에 같은 위치에 두면 여러모로 좋을 것이다.
- 그래서 TestMapper.java와 TestMapper.xml 을 같은 위치에 두고 서버를 구동하면 아래와 같은 메세지를 보게된다.
    ```
    Caused by: org.apache.ibatis.binding.BindingException: Invalid bound statement (not found)
    ```
- 쿼리 구문이 바인딩 되지 못했다는 이야기이다. 
- 프로젝트의 빌드 결과물이 위치하는 test 디렉토리를 찾아보면 TestMapper.xml 이 존재하지 않는 것을 확인할 수 있다.
- 일반적으로 자바코드는 .class로 빌드되어 결과에 포함되는데, 자바코드가 아닌 파일들이 결과에 필요한 경우는 리소스로 지정하여 빌드 결과에 포함시킬 수 있다.
- 그래서 TestMapper.xml 이 resources 아래에 있을 때는 리소스 파일로 지정되어 빌드 결과에 포함되며 정상 동작하지만, TestMapper.java와 같은 위치에 두면 리소스로 지정되지 않아 빌드 결과에 포함되지 않는 것이다.
- 이것을 가능하게 하기 위해서는 자바코드가 위치하는 디렉토리에도 리소스가 있음을 빌드 설정을 통해 알려줘야 한다.
- 아래와 같이 maven 설정(pom.xml)에 빌드 설정을 추가하면 Mapper Interface 와 Mapper Xml을 같은 위치에 보관하는 것이 가능하다.
    ```xml
    <project>
        ...

        <build>
            <resources>
                <resource>
                    <directory>src/main/java</directory>
                    <includes>
                        <include>**/*Mapper.xml</include>
                    </includes>
                </resource>
            </resources>
        </build>

    </project>
    ```

### 둘 이상의 DB를 위한 설정
- 하나의 DB를 위한 설정은 간단하지만 둘 이상의 DB를 위한 설정은 다소 처리해줘야 할 부분들이 있다.
- 우선 데이터소스 설정을 둘로 분리한다.
    ```yml
    spring.
        datasource:
            first:
                url: jdbc:mysql://123.456.789.0:1234?user=dev_db_first
                username: firstUser
                password: 1234567890
                driver-class-name: com.mysql.cj.jdbc.Driver
            second:
                url: jdbc:mysql://123.456.789.0:1234?user=dev_db_second
                username: secondUser
                password: 1234567890
                driver-class-name: com.mysql.cj.jdbc.Driver
    ```
- 데이터소스 빈을 둘 등록하기 위해 Configuration Class를 하나 만들어 설정한다.
    ```java
    @Configuration
    public class AppConfig {
        @ConfigurationProperties("spring.datasource.first")
        @Bean
        public DataSource firstDataSource() {
            return DataSourceBuilder.create().build();
        }

        @ConfigurationProperties("spring.datasource.second")
        @Bean
        public DataSource secondDataSource() {
            return DataSourceBuilder.create().build();
        }
    }
    ```
- 이후 MyBatis의 세션 생성을 위한 SqlSessionFactoryBean, Mapper 파일들의 스캔을 위한 설정을 담는 MapperScannerConfigurer 설정을 추가한다.
    ```java
    ...
        @Bean
        public SqlSessionFactoryBean firstSqlSessionFactory(@Qualifier("firstDataSource") DataSource dataSource) throws Exception {
            SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();

            sqlSessionFactoryBean.setDataSource(dataSource);

            return sqlSessionFactoryBean;
        }

        @Bean
        public SqlSessionFactoryBean secondSqlSessionFactory(@Qualifier("secondDataSource") DataSource dataSource) throws Exception {
            SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();

            sqlSessionFactoryBean.setDataSource(dataSource);

            return sqlSessionFactoryBean;
        }

        @Bean
        public MapperScannerConfigurer firstMapperScannerConfigurerCab() {
            MapperScannerConfigurer config = new MapperScannerConfigurer();

            config.setBasePackage("com.scott.mapper");
            config.setSqlSessionFactoryBeanName("firstSqlSessionFactory");

            return config;
        }

        @Bean
        public MapperScannerConfigurer secondMapperScannerConfigurerCab() {
            MapperScannerConfigurer config = new MapperScannerConfigurer();

            config.setBasePackage("com.scott.second_mapper");
            config.setSqlSessionFactoryBeanName("secondSqlSessionFactory");

            return config;
        }
    ...

    ```
- 각각의 매퍼파일은 별도 패키지를 생성하여 위치시킨다.

> 참고 : https://atoz-develop.tistory.com/entry/Spring-Boot-MyBatis-%EC%84%A4%EC%A0%95-%EB%B0%A9%EB%B2%95