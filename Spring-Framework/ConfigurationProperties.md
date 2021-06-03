> 참고 : https://www.baeldung.com/configuration-properties-in-spring-boot
 
# @ConfigurationProperties

## 소개
- Since 1.0.0 시절 부터 있었으니 Spring Boot면 그냥 다 된다고 봐야겠다.
- 부트는 외부 설정(직접 클래스에 설정하지 않으니 외부 설정이라고 하는듯?)과 properties files에 정의된 프로퍼티에 쉽게 액세스 할 수 있는 다양한 방법을 제공한다 함.
  - 다양한 방법들이 어떤게 있는지 좀 더 찾아볼만할 것 같다.
- 설정 파일을 읽어다 bean에 바인딩 해주는 편리한 애너테이션.

## 설정
- 심플하게 의존성 추가해서 사용 가능.
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
    <relativePath/>
</parent>
```

## 프로퍼티 바인딩
- 앞서 이야기 적어두었지만 `설정 파일을 읽어다 bean에 바인딩 해주는 편리한 애너테이션`이다.
- 어떤 설정을 읽을 것인지 @ConfigurationProperties를 선언, prefix로 지정해준다.
- 모두 동일한 접두사를 가지는 계층구조의 프로퍼티에서 가장 잘 동작한다. 따라서 프로퍼티 파일은 그 다음과 같다.
- 그리고 Spring은 표준 getter, setter를 사용하므로 설정을 바인딩 할 bean은 적절한 setter를 가져야 한다.
- bean에 대해서 바인딩 되므로 bean으로 등록될 수 있도록 @Component 애너테이션, @Configuration 애너테이션을 붙여줘야 한다.
  ```java
  @Configuration
  @ConfigurationProperties(prefix = "mail")
  public class ConfigProperties {    
      private String hostName;
      private int port;
      private String from;

      void setHostName(String hostName) {
        ...
      }

      void setPort(int port) {
        ...
      }

      void setFrom(String from){
        ...
      }
  }
  ```
  ```yml
  mail:
    hostName: testHostName
    port: testPort
    from: testFrom
  ```
- 바인딩 될 클래스에 @Component, @Configuration 애너테이션을 붙이고 싶지 않다면 @Configuration 애너테이션이 붙은 설정 파일에 @EnableComfigrationProperties 애너테이션을 붙이고 클래스를 지정해줘야 한다.
  ```java
  @Configuration
  @EnableConfigProperties(ConfigProperties.class)
  public class BootApplication {
  ....
  ```


- 스프링은 프로퍼티를 바인딩 하는데 느슨한 규칙을 적용해서 아래와 같은 다양한 형태의 프로퍼티가 실제로 바인딩이 가능하다.
  ```
  mail.hostName
  mail.hostname
  mail.host_name
  mail.host-name
  mail.HOST_NAME
  ```

### Spring Boot 2.2
- 2.2 부터는 classpah 스캔을 통해 @ConfigurationProperties클래스가 탐색되고 등록된다. 
- 따라서 굳이 @Component 애너테이션을 붙이거나, @Configuration, @EnableConfigurationProperties 로 지정해주지 않아도 된다.
- @ConfigurationPropertiesScan 애너테이션을 통해 ConfigurationProperty 클래스의 위치를 지정해줄 수도 있는데 이 경우 해당 패키지 이하에서만 찾게 된다.
  ```java
  @SpringBootApplication
  @ConfigurationPropertiesScan("com.baeldung.configurationproperties")
  public class EnableConfigurationDemoApplication { 

      public static void main(String[] args) {   
          SpringApplication.run(EnableConfigurationDemoApplication.class, args); 
      } 
  }
  ```

## 중첩된 프로퍼티
- List, Maps, 클래스와 같은 중첩된 프로퍼티도 가능하다.
  ```java
  public class Credentials {
    private String authMethod;
    private String username;
    private String password;
    // standard getters and setters
  }
  ```
  ```java 
  public class ConfigProperties {

      private String host;
      private int port;
      private String from;
      private List<String> defaultRecipients;
      private Map<String, String> additionalHeaders;
      private Credentials credentials;

      // standard getters and setters
  }
  ```
  ```yml
  #Simple properties
  mail.hostname=mailer@mail.com
  mail.port=9000
  mail.from=mailer@mail.com

  #List properties
  mail.defaultRecipients[0]=admin@mail.com
  mail.defaultRecipients[1]=owner@mail.com

  #Map Properties
  mail.additionalHeaders.redelivery=true
  mail.additionalHeaders.secure=true

  #Object properties
  mail.credentials.username=john
  mail.credentials.password=password
  mail.credentials.authMethod=SHA1
  ```
  
## @Bean 메서드 위에 @ConfigurationProperties 사용하기
- 설정파일 등에서 bean 등록을 위해 @Bean 애너테이션을 붙인 메서드를 사용하곤 한다.
- 이때도, @ConfigurationProperties 를 사용할 수 있는데 메서드에 의해 반환되는 객체에 바인딩 된다.
  ```java
  @Configuration
  public class ConfigProperties {

      @Bean
      @ConfigurationProperties(prefix = "item")
      public Item item() {
          return new Item();
      }
  }
  ```
  ```java
  public class Item {
      private String name;
      private int size;

      // standard getters and setters
  }
  ```

## 프로퍼티 유효성 체크
- @ConfigurationProperties는 JSR-303 포맷을 이용한 프로퍼티 유효성 체크를 지원한다.
- @NotBlank, @Length, @Min, @Max, @Pattern 등의 애너테이션으로 다양하게 프로퍼티 값에 대한 유효성 체크가 가능하다.

## 프로퍼티 컨버전
- 프로퍼티 파일에 설정된 내용을 Java Class 로 컨버전이 가능하다.
- 기본적으로 컨버전이 제공되는 Duration, DataSize 등의 타입들이 있지만 커스텀도 가능하다.
  ```java
  public class Employee {
    private String name;
    private double salary;
  }
  ```
  ```yml
  conversion.employee=john,2000
  ```
  ```java
  private Employee employee
  ```
- @ConfigurationPropertiesBinding 애너테이션을 붙인 bean을 등록하고, Converter 인터페이스를 구현한다.
  ```java
  @Component
  @ConfigurationPropertiesBinding
  public class EmployeeConverter implements Converter<String, Employee> {
    @Override
    public Employee convert(String from) {
      String[] data = from.split(",");
      return new Employee(data[0], Double.parseDouble(data[1]));
    }
  }
  ```

## 불변 @ConfigurationProperties 바인딩
- Spring Boot 2.2 부터 @ConstructorBinding 애너테이션을 통해 프로퍼티를 바인딩 할 수 있다.
- 이것은 setter를 통해서 바인딩하는 것이 아니고, 생성자를 통해 바인딩하며 프로퍼티가 바인딩 된 객체의 불변을 보장할 수 있게 된다.


## 더 확인해볼 것
- JSR-303
- 부트가 제공하는 외부 설정과 properties files에 정의된 프로퍼티에 쉽게 액세스 할 수 있는 다양한 방법
