# Logging Framework

## 알아볼 것
- 로깅 프레임워크의 종류
- 로깅 프레임워크의 설정 로딩 방식 확인    
- slf4j와의 관계
- 각 로깅 프레임워크의 차이점 & 성능
- Additivity
- Logger
    - Hierarchy
- Log Level
- Appender
    - Appender 별 설정 방법 및 특징
- 환경별 로그 설정 변경 (spring?)
- Layout : 로그 포맷 지정
- 최신 스프링 부트의 기본 logging framework
- logback-spring.xml

## 참고 링크
- Log4j2 설정하기 : http://dveamer.github.io/java/Log4j2.html
- 설정 파일을 사용하는 방법 : https://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte3:fdl:%EC%84%A4%EC%A0%95_%ED%8C%8C%EC%9D%BC%EC%9D%84_%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94_%EB%B0%A9%EB%B2%95
- Spring Boot Logging log4j, logback, log4j2 : https://wildeveloperetrain.tistory.com/36
- Spring Boot - Logging, 20분 정리 : https://www.sangkon.com/hands-on-springboot-logging/
- 스프링부트 logback 설정하기 : https://dadadamarine.github.io/java/spring/2019/05/01/spring-logging-xml.html


## 쓸 내용
- 로깅 프레임워크를 왜 쓰는가?
- JCL
- 대표적인 로깅 프레임워크의 역사 및 성능 비교? Spring Boot 에서 채택하고 있는 기본 로깅 프레임워크, slf4j
- 로깅 프레임워크들의 설정 매커니즘, 스프링, 스프링 부트를 사용하게 되는 경우 스프링 프로퍼티의 이용, 환경에 따른 설정 등
- Logger, Appender, Layout, LogLevel 등 설정 파일 내의 기본 개념, additivity 등 설명
- 각 Appender 들의 특징
- slf4j
