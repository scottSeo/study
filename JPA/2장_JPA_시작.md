# 2장. JPA 시작
## 환경 셋팅
- P. 65~74 참고

## 객체 매핑 시작
- 테이블 생성
```
CREATE TABLE MEMBER {
    ID VARCHAR(255) NOT NULL,
    NAME VARCHAR(255), 
    AGE INTEGER,
    PRIMARY KEY (ID)
}
```

- 객체 매핑
```
import javax.persistence.*;

@Entity
@Table(name = "MEMBER")
public class Member {
    @Id
    @Column(name = "ID")
    private String id;
    
    @Column(name = "NAME")
    private String username;
    
    //매핑정보가 없는 필드
    private Integer age;
    ...
}
```

- 그림 2.10 참고 (p.76)
- @Entity
    - 이 클래스를 테이블과 매핑한다고 JPA에게 알려줌
    - @Entity가 사용된 클래스를 엔티티 클래스라고 함.
- @Table
    - 엔티티 클래스에 매핑할 테이블 정보를 알려줌
    - name 속성을 사용해서 Member 엔티티를 MEMBER 테이블에 매핑함.
    - 생략한다면, 클래스 이름을 테이블 이름으로 매핑함.
        - 정확히는 엔티티 이름을 사용
- @Id
    - 엔티티 클래스의 필드를 테이블의 기본 키에 매핑
    - 여기서는 엔티티의 id필드를 티에블의 ID 기본 키 컬럼에 매핑
    - 식별자 필드
- @Column
    - 필드를 컬럼에 매핑
    - name 속성을 사용해 Member 엔티티의 username 필드를 MEMBER 테이블의 NAME과 매핑
- 매핑 정보가 없는 필드
    - age는 매핑 어노테이션이 없음
    - 생략하면 필드명을 사용해서 컬럼명으로 매핑함
        - 데이터베이스가 대소문자를 구분하지 않는다고 가정(AGE == age)
        - 구분한다면 @Column(name = "AGE") 추가

## 2.5 persistence.xml 설정
```
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" version="2.1">

    <persistence-unit name="jpabook">

        <properties>

            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect" />

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.format_sql" value="true" />
            <property name="hibernate.use_sql_comments" value="true" />
            <property name="hibernate.id.new_generator_mappings" value="true" />

            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </properties>
    </persistence-unit>

</persistence>
```

- 설정 파일은 persistence로 시작한다.

```
<persistence-unit name="jpabook">
```

- JPA 설정은 영속성 유닛(persistence-unit)이라는 것부터 시작하는데, 일반적으로 연결할 데이터베이스당 하나의 영속성 유닛을 등록한다.
- 그리고 영속성 유닛에는 고유한 이름을 부여해야 하는데 위에서는 jpabook이라는 이름을 사용함.

### JPA 표준속성
- javax.persistence.jdbc.*
    - JPA 표준 속성으로 특정 구현체에 종속되지 않음.

### 하이버네이트 속성
- hibernate.dialect: 데이터 베이스 방언 설정
    - 하이버네이트에서만 이 속성 사용 가능

### 2.5.1 데이터베이스 방언
- JPA는 특정 데이터베이스에 종속적이지 않은 기술.
    - 따라서 다른 데이터베이스로 손쉽게 교체 가능
    - 그런데 각 데이터베이스가 제공하는 SQL 문법과 함수가 조금씩 다름

***
- 데이터 타입 : 가변 문자 타입으로 MySQL은 VARCHAR, 오라클은 VARCHAR2를 사용
- 다른 함수명 : 문자열을 자르는 함수로 SQL 표준은 SUBSTRING(), 오라클 SUBSTR() 사용
- 페이징 처리 : MySQL은 LIMIT, 오라클은 ROWNUM

- 이처럼 SQL 표준을 지키지 않거나 특정 데이터베이스만의 고유한 기능을 JPA서는 `방언`이라함.
- 그림 2.11참고 (p.80)
    - 개발자는 JPA가 제공하는 표준 문법에 맞추어 JPA를 사용하면 되고,
    - 특정 데이터베이스에 의존적인 SQL은 데이터베이스 방언이 처리해줌.
    - 즉, 데이터베이스가 변경되어도 애플리케이션 코드는 변경 X, 방언 설정만 수정

## 2.6 애플리케이션 개발
```
package jpabook.start;

import javax.persistence.*;
import java.util.List;

/**
 * @author holyeye
 */
public class JpaMain {

    public static void main(String[] args) {

        //엔티티 매니저 팩토리 생성
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
        EntityManager em = emf.createEntityManager(); //엔티티 매니저 생성

        EntityTransaction tx = em.getTransaction(); //트랜잭션 기능 획득

        try {


            tx.begin(); //트랜잭션 시작
            logic(em);  //비즈니스 로직
            tx.commit();//트랜잭션 커밋

        } catch (Exception e) {
            e.printStackTrace();
            tx.rollback(); //트랜잭션 롤백
        } finally {
            em.close(); //엔티티 매니저 종료
        }

        emf.close(); //엔티티 매니저 팩토리 종료
    }

    //비지니스 로직
    public static void logic(EntityManager em) {...}
}
```

- 코드는 크게 3부분으로 나뉘어짐.
    - 엔티티 매니저 설정
    - 트래잭션 관리
    - 비즈니스 로직

### 2.6.1 엔티티 매니저 설정
- 그림 2.12 참고 (p.82)
***
- 엔티티 매니저 팩토리 생성
```
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
```
- META-INF/persistence.xml 에서 이름이 jpabook인 영속성 유닛을 찾아서 엔티티 매니저 팩토리를 생성함.
    - 설정 정보를 읽어서 JPA를 동작시키기 위한 기반 객체를 만들고
    - JPA 구현체에 따라서는 데이터베이스 커넥션 풀도 생성하므로 
    - 이 엔티티 매니저 팩토리 생성하는 비용은 아주 큼.
    - 따라서, **엔티티 매니저 팩토리는 애플리케이션 전체에서 딱 한 번만 생성하고 공유해서 사용해야 함**

***
- 엔티티 매니저 생성
```
EntityManager em = emf.createEntityManager();
```
- 엔티티 매니저 팩토리에서 엔티티 매니저를 생성함.
- JPA 대부분 기능은 엔티티 매니저가 제공
    - 대표적으로 이것을 이용하여 데이터베이스에 등록/수정/삭제/조회할 수 있음.
- 엔티티 매니저는 내부에 데이터소스(커넥션)을 유지하면서 데이터베이스와 통신함
- 참고로 **엔티티 매니저는 데이터베이스 커넥션과 밀접한 관계가 있으므로 스레드간에 공유하거나 재사용하면 안된다**.
***
- 종료
```
em.close();
emf.close();
```

### 2.6.2 트랜잭션 관리
- JPA를 사용하면 항상 트랜잭션 안에서 데이터를 변경해야함.
- 트랜잭션 없이 데이터를 변경하면 예의가 발생

```java
EntityTransaction tx = em.getTransaction(); // 트랜잭션 API
try {
    
    tx.begin(); //트랜잭션 시작
    logic(em);  //비즈니스 로직 실행
    tx.commit(); //트랜잭션 커밋
} catch (Exception e) {
    tx.rollback(); //예외 발생 시 트랜잭션 롤백
}
```

### 2.6.3 비즈니스 로직
```
 public static void logic(EntityManager em) {

        String id = "id1";
        Member member = new Member();
        member.setId(id);
        member.setUsername("지한");
        member.setAge(2);

        //등록
        em.persist(member);

        //수정
        member.setAge(20);

        //한 건 조회
        Member findMember = em.find(Member.class, id);
        System.out.println("findMember=" + findMember.getUsername() + ", age=" + findMember.getAge());

        //목록 조회
        List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();
        System.out.println("members.size=" + members.size());

        //삭제
        em.remove(member);

    }
```

- 출력 결과는 아래와 같음
```
findMember=지한, age=20
members.size=1
```

***
- 등록
```
String id = "id1";
Member member = new Member();
member.setId(id);
member.setUsername("지한");
member.setAge(2);

//등록
em.persist(member);
```

- 엔티티를 저장하려면 엔티티 매니저의 persist() 메소드에 저장할 엔티티를 넘겨주면 된다.
- JPA는 회원 엔티티의 매핑 정보(어노테이션)을 분석해서 아래와 같은 SQL을 생성한다.
```
INSERT INTO MEMBER (ID, NAME, AGE) VALUES ('id1', '지한', 2);
```

***

- 수정
```
member.setAge(20);
```

- 엔티티를 수정한 후에 수정 내용을 반영하려면, em.update() 같은 매소드를 호출해야할 것 같음..
- 하지만, JPA는 어떤 엔티티가 변경되었는지 추적하는 기능을 갖추고 있음.
    - 또한 em.update()라는 것도 없음
- JPA는 아래와 같은 쿼리 생성
```
UPDATE MEMBER
SET AGE=20, NAME='지한'
WHERE ID = 'id1'
```

***
- 삭제
```
em.remove(member);
```

- 엔티티를 삭제하려면 remove() 호출
- JPA는 아래와 같은 쿼리 호출

```
DELETE FROM MEMBER WHERE ID = 'id1'
```

***
- 한 건 조회
```
Member findMember = em.find(Member.class, id);
```

- find()는 조회할 엔티티 타입과 @Id로 데이터베이스 테이블의 기본 키와 매핑한 식별자 값으로 엔티티 하나를 조회하는 가장 단순한 조회 메소드
- JPA는 아래와 같은 쿼리 호출

```
SELECT * FROM MEMBER WHERE ID='id1'
```

### 2.6.4 JPQL
- 하나 이상의 회원 목록을 조회하는 다음 코드를  자세히 보자
```
//목록 조회
TypedQuery<Member> query = em.createQuery("select m from Member m", Member.class);
List<Member> members = query.getResultList();
```

- JPA를 사용하면 애플리케이션 개발자는 엔티티 객체를 중심으로 개발하고 데이터베이스에 대한 처리는 JPA에 맡겨야 한다.
- CRUD는 앞선 예시처럼 SQL을 사용하지 않지만, 문제는 `검색 쿼리`
- JPA는 엔티티 객체를 중심으로 개발하므로 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색해야함.
- 그런데 테이블이 아닌 엔티티 객체를 대상으로 검색하려면 데이터베이스의 모든 데이터를 애플리케이션으로 불러와서 엔티티 객체로 변경한 다음 검색해야 하는데 사실상 **불가능**
- 따라서 이런 경우에는 검색 조건이 포함된 SQL을 사용해야 하는데, 이것이 JPQL이고 쿼리 언어로 이 문제를 해결한다.

***
- JPA는 SQL을 추상화한 JPQL이라는 객체지향 쿼리 언어를 제공한다.
- JPQL은 SQL과 문법이 거의 유사해서 SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 등을 사용할 수 있따.
- 둘의 차이점은 아래와 같다.
    - JPQL은 `엔티티 객체`를 대상으로 쿼리한다.
        - 즉, 클래스와 필드를 대상으로 쿼리한다.
    - SQL은 `데이터베이스 테이블`을 대상으로 쿼리한다.
- 즉 위의 목록 조회 예시에서 JPQL의 `from Member` 부분은 회원 엔티티 객체를 말하는 것이지 데이터베이스 테이블이 아니다.
    - **JPQL은 데이터베이스 테이블을 전혀 알지 못한다.**
- JPA는 JPQL을 분석해서 아래와 같이 쿼리를 생성한다.
```
SELECT M.ID, M.NAME, M.AGE FROM MEMBER M
```
