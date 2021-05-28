# 1장. JPA 소개
## 1.1 SQL을 직접 다룰 때 발생하는 문제점
- 관계형 데이터베이스는 가장 대중적이고 신뢰할 만한 안전한 데이터 저장소.
    - 대부분 자바 어플리케이션은 거의 이걸 사용
- 데이터베이스에 데이터를 관리하려면 SQL을 사용해야함.
    - JDBC API 사용

### 1.1.1 반복, 반복 그리고 반복
- SQL을 직접 다룰 때의 문제점은 무엇인가?
    - 회원 관리 기능을 개발한다고 가정하고 살펴보자.
    - CRUD 개발 필요

***
- Member.class
```java
public class Member {
    private String memberId;
    private String name;
}
```

- 회원용 DAO
```java
public class MemberDAO {
    public Member find(String memberId) {...}
}
```

***
- 위와 같이 주어졌을때, find() 메소드를 완성해서 회원을 조회하는 기능을 개발해보자.

1. 회원 조회용 SQL을 작성한다.
```sql
SELECT MEMBER_ID, NAME FROM MEMBER M WHERE MEMBER_ID = ?
```

2. JDBC API를 사용해서 SQL을 실행한다.
```sql
ResultSet rs = stmt.executeQuery(sql);
```

3. 조회 결과를 Member 객체로 매핑한다.
```
String memberId = rs.getString("MEMBER_ID");
String name = rs.getString("NAME");

Member member = new Member();
member.setMemberId(memberId);
member.setName(name);
...
```

- 여기까지 회원 조회 기능을 완성

***
- 이후 회원 등록 기능을 생성해보자.

```java
public class MemberDAO {
    public Member find(String memberId) {...}
    public void save(Member member) {...}
}
```

1. 회원 등록용 SQL을 작성한다.
```sql
String sql = "INSERT INTO MEMBER (MEMBER_ID, NAME) VALUES (?, ?)"
```

2. 회원 객체의 값을 꺼내서 등록 SQL에 저장한다.
```sql
pstmt.setString(1, member.getMemberId());
pstmt.setString(2, member.getName());
```

3. JDBC API를 사용해서 SQL을 실행한다.
```
pstmt.executeQuery(sql);
```

***

- 여기까지 회원 조회, 등록 기능을 만듬.
- 이후 수정 및 삭제를 개발해보자.
    - 아마도 거의 비슷한 작업을 반복할 것이다.
- 만약, 회원 객체를 데이터베이스가 아닌 자바 컬렉션에 보관한다면 어떨까? 
    - list.add(member);
- 하지만, 데이터베이스는 객체 구조와는 다른 데이터 중심의 구조를 가지므로 객체를 데이터베이스에 직접 저장하거나 조회할 수 없다.
- 따라서 **개발자가 객체지향 애플리케이션과 데이터베이스 중간에서 SQL과 JDBC API를 사용해서 변환 작업을 직접 해주어야 한다.**
- 문제는 객체를 데이터베이스에 CRUD하려면 너무 많은 SQL과 JDBC API를 코드로 작성해야 한다는 점이다.
- 그리고 테이블마다 이런 비슷한 일을 반복해야 하는데, 테이블이 1000개라면..?
- 지루, 반복의 연속

### 1.1.2 SQL에 의존적인 개발
- 추가로 회원의 연락처도 함께 저장해달라는 요구사항이 추가된다면?
***
#### 등록 코드 변경

- Member.class
```
public class Member {
    private String memberId;
    private String name;
    private String tel; // 추가
}
```

- INSERT SQL 수정
```sql
String sql = "INSERT INTO MEMBER(MEMBER_ID, NAME, TEL) VALUES (?, ?, ?)";
```

- 회원 객체의 연락처 값을 꺼내서 등록 SQL에 전달
```
pstmt.setString(3, member.getTel());
```

#### 조회 코드 변경
1. 회원 조회용 SQL을 수정한다.
```sql
SELECT MEMBER_ID, NAME, TEL FROM MEMBER M WHERE MEMBER_ID = ?
```

2. 조회 결과를 Member 객체로 매핑한다.
```
String tel = rs.getString("TEL");
...
member.setTel(tel);
...
```

***
- 요구사항에 맞춰 변경을 했는데,  연락처가 수정되지 않는 버그가 발생한다면?
- 자바 코드는 잘 수정이 되었는데, 무슨 문제일까?
    - 확인해보니, SQL이 문제였음.
    - UPDATE ... SET TEL = tel ...

- **결론은 만약 회원 객체를 데이터베이스가 아닌 자바 컬렉션에 보관했다면, 필드를 추가한다고 해서 이렇게 많은 코드를 수정할 필요는 없을 것이다.**

#### 연관된 객체
- 회원은 어떤 한 팀에 필수로 소속되어야 한다는 요구사항이 추가되었다.
- 팀 모듈을 담당하는 개발자가 팀을 관리하는 코드를 개발하고 커밋했다.

- 변경된 Member.class
```sql
public class Member {
    private String memberId;
    private String name;
    private String tel;
    private Team team;
    ...
}

//추가된 팀
class Team {
    ...
    private String teamName;
    ...
}
```

- 아래 코드를 통해 팀의 이름을 출력한다.
```
이름 : member.getName();
소속 팀 : member.getTeam().getTeamName(); // 추기
```

- 근데 member.getTeam() 의 값이 항상 null.
- 무엇이 문제인가?
    - DB를 확인해봐도 모든 회원이 어떤 팀에 항상 소속해있다.


```
public class MemberDAO {
    public Member find(String memberId) {...}
    public Member findWithTeam(String memberId) {...}
}
```

- MemberDAO를 확인해보니, 회원을 출력할 때 사용되는 find() 메소드는 회원만 조회하는 아래 SQL을 그대로 유지했다.
```
SELECT MEMBER_ID, NAME, TEL FROM MEMBER M
```

- 또한, findWithTeam()은 아래 SQL로 회원과 연관된 팀을 함께 조회했다.
```sql
SELECT M.MEMBER_ID, M.NAME, M.TEL, T.TEAM_ID, T.TEAM_NAME
FROM MEMBER M
JOIN TEAM T
    ON M.TEAM_ID = T.TEAM_ID
```

- 결국 회원 조회 코드를 memberDAO.find() -> memberDAO.findWithTeam()으로 변경해서 해결했다.

***
- 위 내용을 살펴본 것처럼 SQL과 문제점을 정리해보자.
- Member 객체가 연관된 Team 객체를 사용할 수 있을지 없을지는 전적으로 사용하는 SQL에 달려 있다.
- 이런 방식의 가장 큰 문제는 데이터 접근 계층을 사용해서 SQL을 숨겨도 어쩔 수 없이 DAO를 열어서 어떤 SQL이 실행되는지 확인해야 한다는 점이다.

- 지금처럼 SQL에 모든 것을 의존하는 상황에서는 개발자들이 엔티티를 신뢰하고 사용할 수 없다.
- 대신에 DAO를 열어서 어떤 SQL이 실행되고 어떤 객체들이 함께 조회되는지 일일이 확인해야 한다.
- **이것은 진정한 의미의 계층 분할이 아니다.**
    - 물리적으로는 분할이 되었다 하더라도
    - 논리적으로는 엔티티와 아주 강한 의존관계를 가지고 있다.
- 즉 이러한 의존관계 때문에 회원을 조회할 때는 물론이고 회원 객체에 필드를 하나 추가할 때도 DAO의 CRUD 코드와 SQL 대부분을 변경해야 하는 문제가 발생한다.

***
- 애플리케이션에서 SQL을 직접 다룰 때 발생하는 문제점
    - 진정한 의미의 계층 분할이 어렵다.
    - 엔티티를 신뢰할 수 없다.
    - SQL에 의존적인 개발을 피하기 어렵다.

### 1.1.3 JPA와 문제 해결
- JPA를 사용하면 객체를 데이터베이스에 저장하고 관리할 때, 개발자가 직접 SQL을 작성하는 것이 아니라 JPA가 제공하는 API를 사용하면 된다.

***
- JPA 가 제공하는 CRUD API 간단히 살펴보기

- 저장 기능
```java
jpa.persist(member);
```

- 조회 기능
```java
String memberId = "helloId";
Member member = jpa.find(Member.class, memberId);
```

- 수정 기능
```java
Member member = jpa.find(Member.class, memberId);
member.setName("이름변경"); //수정
```
- JPA는 별도의 수정 메소드를 제공하지 않는다.
- 대신에 객체를 조회해서 값을 변경만 하면 트랜잭션을 커밋할 때 데이터베이스에 적절한 UPDATE SQL이 전달된다.
- 이 마법같은 일에 대한 자세한 설명은 3장에..

- 연관된 객체 조회
```java
Member member = jpa.find(Member.class, memberId);
Team team = member.getTeam();
```
- JPA는 연관된 객체를 사용하는 시점에 적절한 SELECT SQL을 실행한다.
- 따라서 JPA를 사용하면 연관된 객체를 마음껏 조회할 수 있다.
- 이 마법같은 일에 대한 자세한 설명은 8장에..

## 1.2 패러다임의 불일치
- 복잡성을 제어하지 못하면 결국 유지보수하기 어려운 애플리케이션이 된다.
- 현대의 복잡한 애플리케이션은 대부분 객체지향 언어로 개발한다.
- 비즈니스 요구사항을 정의한 도메인 모델도 객체로 모델링하면 객체지향 언어가 가진 장점들을 활용할 수 있다.
    - 문제는 이렇게 정의한 도메인 모델을 저장할 때 발생한다.
    - 예를 들어 특정 유저가 시스템에 회원 가입하면 회원이라는 객체 인스턴스를 생성한 후에 이객체를 메모리가 아닌 어딘가에 영구 보관해야 한다.
- 객체는 속성(필드)과 기능(메소드)을 가진다.
    - 객체의 기능은 클래스에 정의되어 있으므로 객체 인스턴스의 상태인 속성만 저장했다가 필요할 때 불러와서 복구하면 된다.
    - 객체가 단순하면 객체의 모든 속성 값을 꺼내서 파일이나 데이터베이스에 저장하면 되지만, 부모 객체를 상속받았거나, 다른 객체를 참조하고 있다면 객체에 상태를 저장하기는 쉽지 않다.
    - ex) 회원 + 팀
- 자바는 이런 문제까지 고려해서 객체를 파일로 저장하는 직렬화 기능과 저장된 파일을 객체로 복구하는 역 직렬화 기능을 지원한다.
    - 하지만, 이 방법은 객체를 검색하기 어렵다는 문제가 있으므로 현실성이 없다.
    - 현실적인 방법은 관계형 데이터베이스에 객체를 저장하는 것인데, 관계형 데이터베이스는 데이터 중앙으로 구조화되어 있고, 집합적인 사고를 요구한다.
    - 그리고 객체지향에서 이야기하는 추상화, 상속, 다형성 같은 개념이 없다.
- 객체와 관계형 데이터베이스는 지향하는 목적이 서로 다르므로 둘의 기능과 표현 방법도 다르다.
    - **이것을 객체와 관계형 데이터베이스의 패러다임 불일치 문제라 한다.**
    - 즉 이것은 개발자가 중간에서 해결해야 하는데, 문제는 이 문제를 해결하는데 너무 많은 시간과 코드를 소비하는 것이다.
- 패러다임 불일치 문제를 구체적으로 살펴보고, JPA를 통한 해결책도 알아보자.

### 1.2.1 상속
- 객체는 상속이라는 기능을 가지고 있지만, 테이블은 상속이라는 기능이 없다.
    - 그림 1.2 참고 (p.41)

- 그나마 데이터베이스 모델링에서 이야기하는 슈퍼타입 서브타입 관계를 사용하면 객체 상속과 가장 유사한 형태로 테이블을 설계할 수 있다.
    - DTYPE이 MOVIE면, 영화 테이블과 관계
    - 그림 1.3 참고 (p.41)

- 객체 모델 코드
```java
abstract class Item {
    Long id;
    String name;
    int price;
}

class Album extends Item {
    String artist;
}

class Movie extends Item {
    String director;
    String actor;
}

class Book extends Item {
    String author;
    String isbn;
}
```

- Album 객체를 저장하려면 이 객체를 분해해서 다음 두 SQL을 만들어야 한다.
    - INSERT INTO ITEM ...
    - INSERT INTO ALBUM...
- 다른 객체도 마찬가지..

- JDBC API를 사용해서 이 코드를 완성하려면 부모 객체에서 부모 데이터만 꺼내서 ITEM용 INSERT SQL을 작성하고 자식 객체에서 자식 데이터만 꺼내서 ALBUM 용 INSERT SQL을 작성해야 하는데, 작성해야 할 코드량이 만만치 않다. 그리고 자식 타입에 따라서 DTYPE도 저장해야 한다.
- 조회하는 것도 힘들다..
    - Album을 조회한다면 ITEM과 ALBUM 테이블을 조인해서 조회한 다음 그 결과로 Album 객체를 생성해야 한다.
- 이런 과정이 모두 패러다임의 불일치를 해결하려고 소모하는 비용.
- 만약 자바 컬렉션에 보관한다면?
```java
list.add(album);
list.add(movie);

Album album = list.get(albumId);
```

### JPA와 상속
- JPA는 상속과 관련된 패러다임의 불일치 문제를 개발자 대신 해결해준다.
- 개발자는 마치 자바 컬렉션에 객체를 저장하듯이 JPA에게 객체를 저장하면 된다.

```
jpa.persist(album);
```

- JPA는 다음 SQL을 실행해서 객체를 ITEM, ALBUM 두 테이블에 나누어 저장한다.
```
INSERT INTO ITEM...
INSERT INTO ALBUM...
```

- 다음으로 Album 객체를 조회해보자.
```
String albumId = "id100";
Album album = jpa.find(Album.class, albumId);
```

- JPA는 ITEM과 ALBUM 두 테이블을 조인해서 필요한 데이터를 조회하고 그 결과를 반환한다.
```
SELECT I.*, A.*
FROM ITEM I
JOIN ALBUM A ON I.ITEM_ID = A.ITEM_ID;
```

### 1.2.2 연관관계
- `객체는 참조`를 사용해서 다른 객체와 연관관계를 가지고 `참조에 접근해서 연관된 객체를 조회`한다.
- 반면에 `테이블은 외래 키`를 사용해서 다른 테이블과 연관관계를 가지고 `조인을 사용해서 연관된 테이블을 조회`한다.
- 참조를 사용하는 객체와 외래 키를 사용하는 관계형 데이터베이스 사이의 패러다임 불일치는 객체지향 모델링을 거의 포기하게 만들 정도로 극복하기 어렵다.
- 그림 1.4 참고

***
- MEMBER 테이블은 MEMBER.TEAM_ID 외래 키 컬럼을 사용해서 TEAM 테이블과 관계를 맺는다.
- 이 외래 키를 사용해서 MEMBER 테이블과 TEAM 테이블을 조인하면 MEMBER 테이블과 연관된 TEAM 테이블을 조회할 수 있다.
- 객체는 참조가 있는 방향으로만 조회 가능.
- 반면에 테이블은 외래 키 하나로 MEMBER JOIN TEAM도 가능하지만, TEAM JOIN MEMBER도 가능.

### 객체를 테이블에 맞추어 모델링
- 객체와 테이블의 차이를 알아보자.

```java
class Member {
    String id;
    Long teamId;
    String username;
}

class Team {
    Long id;
    String name;
}
```

- Member 테이블의 컬럼을 그대로 가져와서 Member 클래스를 만들었다.
    - 이렇게 객체를 테이블에 맞추어 모델링하면 객체를 테이블에 저장하거나 조회할 때는 편리하다.
    - 그런데 여기서 TEAM_ID 외래 키의 값을 그대로 보관하는 teamId 필드에는 문제가 있다.
    - 관계형 데이터 베이스는 조인을 통해 Team을 가져올 수 있지만,
    - 객체는 Team 객체를 가져올 수 없음.
    - member.getTeam() // 불가..
    - 이런 방식을 따르면 좋은 객체 모델링은 기대하기 어렵고, 결국 객체지향의 특징을 잃어버리게 된다.

### 객체지향 모델링
- 객체는 참조를 통해서 관계를 맺는다. 따라서 아래와 같이 변경한다.

```java
class Member {
    String id;
    Team team;
    String username;
}

class Team {
    Long id;
    String name;
}
```

- 위 처럼 변경하면, 회원과 연관된 팀을 조회할 수 있다.
- 하지만, 객체를 테이블에 저장하거나 조회가기가 쉽지 않다.

***
#### 저장
member.getId(); //MEMBER_ID PK에 저장
member.getTeam().getId(); //TEAM_ID FK에 저장
member.getUserName(); //USERNAME 컬럼에 저장

#### 조회
```sql
SELECT M.*, T.*
FROM MEMBER M
JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
```

- SQL에 결과로 아래와 같이 객체를 생성하고 연관관계를 설정해서 반환하면 된다.
```java
pulbic Member find(String memberId) {
    //SQL 실행
    ...
    Member member = new Member();
    ...
    
    //데이터베이스에서 조회한 회원 관련 정보를 모두 입력
    Team team = new Team();
    ...
    //데이터베이스에서 조회한 팀 관련 정보를 모두 입력
    
    //회원과 팀 관계 설정
    member.setTeam(team);
    return member;
}
```

- 이런 과정은 패러다임 불일치를 해결하려고 소모하는 비용~

### JPA와 연관관계
- JPA는 연관관계와 관련된 패러다임의 불일치 문제를 해결해준다.
```
member.setTeam(team);
jpa.persist(member);
```
- 개발자는 회원과 팀의 관계를 설정하고 회원 객체를 저장하면 된다.
- JPA는 team의 참조를 외래 키로 변환해서 적절한 INSERT SQL을 데이터베이스에 전달한다.
- 조회할 떄도 마찬가지
```
Member member = jpa.find(Member.class, memberId);
Team team = member.getTeam();
```

### 1.2.3 객체 그래프 탐색
- 객체에서 회원이 소속된 팀을 조회할 때는 다음처럼 참조를 사용해서 연관된 팀을 찾으면 되는데, 이것을 객체 그래프 탐색이라 한다.
- 그림 1.5 참고 (p.48)
- member.getOrder().getOrderIteam()... 객체 그래프 탐색 코드
- 객체는 마음껏 객체 그래프를 탐색할 수 있어야한다.
    - 그런데???

```sql
SELECT M.*, T.*
FROM MEMBER M
JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
```

- 위와 같이 조회했다면 회원과 팀의 정보만 조회된다. 
    - Order 정보는 조회되지 않는다.
- **SQL을 직접 다루면 처음 실행하는 SQL에 따라 객체 그래프를 어디까지 탐색할 수 있는지 정해진다.**
    - 이것은 객체지향 개발자에겐 너무 큰 제약이다..

```java
class MemberService {
    ...
    public void process() {
        Member member = memberDAO.find(memberId);
        member.getTeam(); // 객체 그래프 탐색이 가능한가???
        member.getOrder();
    
    }
}
```

- MemberService에서는 memberDAO를 통해서 member 객체를 조회했지만, 이 객체와 연관된 Team, Order, Delivery 방향으로 객체 그래프를 탐색할 수 있을지 없을지느 이 코드만 보고는 전혀 예측할 수 없다.
- 즉, 직접 SQL을 봐야한다..

### JPA와 객체 그래프 탐색
- JPA를 사용하면 객체 그래프를 마음껏 탐색할 수 있다.
- member.getOrder().getOrderItem()...
- JPA를 사용하면 연관된 객체를 신뢰하고 마음껏 조회할 수 있다.
    - 이 기능은 실제 객체를 사용하는 시점까지 데이터베이스 조회를 미룬다고 해서 `지연 로딩` 이라 한다.

```java
//처음 조회 시점에 SELECT MEMBER SQL
Member member = jpa.find(Member.class, memberId);

Order order = member.getOrder();
order.getOrderDate(); // Order를 사용하는 시점에 SELECT ORDER SQL

```

- Member를 사용할 때마다 Order를 함께 사용하면, 이렇게 한 테이블씩 조회하는 것보다는 Member를 조회하는 시점에 SQL 조인을 사용해서 Member와 order를 함께 조회하는 것이 효과적이다.
- JPA는 연관된 객체를 즉시 함께 조회할지 아니면 실제 사용되는 시점에 지연해서 조회할지를 간단한 설정으로 정의할 수 있다.
    - 함께 조회하는 것으로 설정하면, 아래와 같이 SQL을 생성한다.

```sql
SELECT M.*, O.*
FROM MEMBER M
JOIN ORDER O ON M.MEMBER_ID = O.MEMBER_ID
```

### 1.2.4 비교
- 데이터베이스는 기본 키의 값으로 각 로우를 구분한다.
- 반면에 객체는 동일성(identity) 비교와 동등성(equiality) 비교라는 두 가지 비교 방법이 있다.
    - 동일성 : ==, 객체의 주소값 비교
    - 동등성 : equals, 객체의 내부 값 비교
- 즉, 테이블의 로우 구분 방법과 객체 구분 방법에는 차이가 있다.

***
```java
class MemberDAO {
    public Member getMember(String memberId) {
        String sql = "SELECT * FROM MEMBER WHERE MEMBER_ID = ?";
        ...
        //JDBC API, SQL 실행
        return new Member(...);
    }
}
```

```java
String memberId = "100";
Member member1 = memberDAO.getMember(memberId);
Member member2 = memberDAO.getMember(memberId);

member1 == member2; //다르다.
```

- 예제를 보면 기본 키 값이 같은 회원 객체를 두 번 조회했다.
- 그런데 둘을 동일성(==) 비교하면 false가 반환된다. 왜냐하면 member1과 member2는 같은 데이터베이스 로우에서 조회했지만, 객체 측면에서 볼 때 둘은 다른 인스턴스이다.
- 만약 객체를 컬렉션에 보관했다면 같았을 것이다.
```
Member member1 = list.get(0);
Member member2 = list.get(0);

member1 == member2 // 같다
```

- 이런 패러다임의 불일치 문제를 해결하기 위해 데이터베이스의 같은 로우를 조회할 때마다 같은 인스턴스를 반환하도록 구현하는 것은 쉽지 않다.

### JPA와 비교
- JPA는 같은 트랜잭션일 때 같은 객체가 조회되는 것을 보장한다.

```
String memberId = "100";
Member member1 = jpa.find(Member.class, memberId);
Member member2 = jpa.find(Member.class, memberId);

member1 == member2; //같다.
```

### 1.2.5 정리
- 객체 모델과 관계형 데이터베이스 모델은 지향하는 패러다임이 서로 다르다.
- 문제는 이 패러다임의 차이를 극복하려고 개발자가 너무 많은 시간과 코드를 소비한다는 점이다.
- 더 정교한 객체 모델링 -> 패러다임의 불일치 증가 -> 개발자가 소모해야되는 비용 증가
    - 결국, 객체 모델링은 힘을 잃고 점점 데이터 중심의 모델로 변해간다.
- 이러한 문제의 해결책의 결과물이 바로 JPA

## 1.3 JPA란 무엇인가?
- JPA는 자바 진영의 ORM 기술 표준이다.
    - Java Persistence API
    - 그림 1.6 참고 (p.54)
- ORM은 무엇인가?
    - Object-Relational Mapping
    - 객체와 관계형 데이터베이스를 매핑
    - ORM 프레임워크는 객체와 테이블을 매핑해서 패러다임의 불일치 문제를 개발자 대신 해결해준다.
    - 그림 1.7/1.8 참고 (p.54)

- ORM 프레임워크는 단순히 SQL을 개발자 대신 생성해서 데이터베이스에 전달해주는 것뿐만 아니라 앞서 이야기한 다양한 패러다임의 불일치 문제들도 해결해준다.
- 따라서, 객체 측면에서는 정교한 객체 모델링을 할 수 있고, 관계형 데이터베이스는 데이터베이스에 맞도록 모델링하면 된다.
- 그리고 둘을 어떻게 매핑해야 하는지 매핑 방법만 ORM 프레임워크에게 알려주면 된다.

### 1.3.1 JPA 소개
- 과거 자바 진영은 엔터프라이즈 자바 빈즈(EJB)라는 기술 표준을 만듬
- 이 안에는 엔티티 빈이라는 ORM 기술도 포함되어 있었음.
- 하지만, 너무 복잡하고 기술 성숙도도 떨어졌으며 자바 엔터프라이즈(J2EE) 애플리케이션 서버에서만 동작했음.
- 이때 하이버네이트라는 오픈소스 ORM 프레임워크 등장
- EJB의 ORM 기술과 비교해서 가볍고 실용적인 데다 기술 성숙도도 높았음.
- 결국 EJB 3.0에서 하이버네이트를 기반으로 새로운 자바 ORM 기술 표준이 만들어졌는데 이것이 바로 `JPA`

***
- 그림 1.9 참고 (p.56)
- **JPA는 자바 ORM 기술에 대한 API 표준 명세**
- 따라서, JPA를 사용하려면 JPA를 구현한 ORM 프레임워크를 선택해야 함.
    - 현재 JPA 2.1을 구현한 ORM 프레임워크는 `하이버네이트`, `EclipseLink`, `DataNucleus`가 있는데, 하이버네이트가 가장 대중적이다.
- JPA라는 표준 덕분에 특정 구현 기술에 대한 의존도를 줄일 수 있고 다른 구현 기술로 손쉽게 이동할 수 있는 장점이 있다.
    - JPA 표준은 일반적이고 공통적인 기능의 모음.
    - 표준을 먼저 이해하고 필요에 따라 JPA 구현체가 제공하는 고유의 기능을 알아가면 된다.

### 1.3.2 왜 JPA를 사용해야 하는가?
#### 생산성
    - JPA를 사용하면 자바 컬렌션에 객체를 저장하듯이 JPA에 저장할 객체를 전달하면 된다.

```java
jpa.persist(member); // 저장
Member member = jpa.find(memberId); // 조회
```
- 지루하고 반복적인 코드와 CRUD용 SQL을 개발자가 직접 작성하지 않아도 된다.
- 더 나아가서 JPA에는 CREATE TABLE과 같은 DDL 문을 자동으로 생성해주는 기능도 있다.
- 데이터베이스 설계 중심의 패러다임을 객체 설계 중심으로 역전시킬 수 있다.

#### 유지보수
- SQL을 직접 다루면 엔티티에 필드를 하나만 추가해도 관련된 등록, 수정, 조회 SQL과 결과를 매핑하기 위한 JDBC API 코드를 모두 변경해야 함.
- 반면에 JPA는 대신 처리해주므로 필드를 추가하거나 삭제해도 수정해야 할 코드가 줄어듬.
- 또한, JPA가 패러다임의 불일치 문제를 해결해주므로 객체지향 언어가 가진 장점들을 활용해서 유연하고 유지보수하기 좋은 도메인 모델을 편리하게 설계할 수 있다.

#### 패러다임의 불일치 해결
- 앞선 내용처럼 여러 가지의 패러다임의 불일치를 해결해줌.

#### 성능
- JPA는 애플리케이션과 데이터베이스 사이에서 다양한 성능 최적화 기회를 제공한다.
- 애플리케이션과 데이터베이스 사이에 계층이 하나 더 있으면 최적화 관점에서 시도해 볼 수 있는 것들이 많다.

```
String memberId = "helloId";
Member member1 = jpa.find(memberId);
Member member2 = jpa.find(memberId);
```

- 같은 트랜잭션 안에서 같은 회원을 두 번 조회하는 코드의 일부분이다.
- JDBC API를 사용했더라면 데이터베이스와 두 번 통신했을 텐데,
- JPA를 사용하면 회원을 조회하는 SELECT SQL을 한 번만 데이터베이스에 전달하고 두 번째는 조회한 회원 객체를 재사용한다.

#### 데이터 접근 추상화와 벤더 독립성
- 관계형 데이터베이스는 같은 기능도 벤더마다 사용법이 다른 경우가 많다.
    - 단적인 예로 페이징 처리
- 결국 처음 선택한 데이터베이스 기술에 종속되고, 다른 데이터베이스로 변경하기는 매우 어렵다.
- JPA는 애플리케이션과 데이터베이스 사이에 추상화된 데이터 접근 계층을 제공해서 애플리케이션이 특정 데이터베이스 기술에 종속되지 않도록 한다.
    - 그림 1.10 참고 (p.59)
- 데이터베이스를 변경하면 JPA에게 다른 데이터베이스를 사용한다고 알려주기만 하면 된다.
    -  예로, JPA를 사용하면 로컬 개발 환경은 H2, 개발이나 상용 환경은 오라클이나 MySQL을 사용할 수 있다.

#### 표준
- JPA는 자바 진영의 ORM 기술 표준이다.
    - 표준을 사용하면 다른 구현 기술로 손쉽게 변경할 수 있다.
