# 스프링에서 Resource 가지고 오기

## Resource란?
- java 프로젝트를 빌드하고 .class 이외에 필요한 파일들을 resource 라고 한다.
- 빌드 결과 classes 디렉토리로 이동하는데 테스트인 경우는 test-classes로 이동한다.
- maven에서 리소스를 빌드결과에 포함하기 위해서 빌드 설정에 리소스를 지정해줘야 한다.

```java
@Bean
public Resource myResource(ApplicationContext applicationContext) {
    return applicationContext.getResource("classpath:/myResource.file");
}
```