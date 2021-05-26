# JPA 에서 Java의 enum 타입을 엔티티 클래스의 속성으로 사용하기

- 엔티티 클래스의 속성으로 사용시 @Enumerated 어노테이션을 붙인다.
- 기본적으로 EnumType.ORDINAL로 정의되어 있는데 enum이 가지는 순번을 의미한다. 결국 숫자형 데이터 취급된다.
- EnumType.STRING은 enum이 가지는 이름을 의미한다. 문자열 데이터 취급된다.

> https://lng1982.tistory.com/280
