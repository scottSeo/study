# AspectJ
- [관점 지향 프로그래밍(Aspect Oriented Programming)](./AOP.md)의 원조격이며 거의 표준이다.
- Spring AOP와는 목적이 다르다. (Spring AOP는 간단한 AOP구현이 목적이라 완전한 AOP가 아니다. Beans에만 적용된다. 반면 AspectJ는 완전한 AOP를 위한 목적으로 모든 객체에 AOP가 가능하다.)
- AspectJ는 3가지 방식의 Weaving이 가능하다.
    - 컴파일 시점 : AspectJ 컴파일러가 AspectJ와 모든 소스를 입력받아 컴파일 함.
    - 컴파일 전 : 이미 존재하는 class와 jar파일을 처리
    - 로드 시점 : 위 컴파일 전 방식과 유사하나 클래스가 실제 JVM에 로드될 때까지 연기된다는 점이 다름.
- Spring AOP는 런타임 Weaving을 하고 그것만 가능함. 대상 객체의 Proxy(JDk Dynamic Proxy나 CGlib Proxy)를 사용하는 애플리케이션 실행시에 Weaving.
