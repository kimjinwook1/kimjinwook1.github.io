# JPA - Java Persistence API
- 자바 진영의 ORM 기술 표준

**ORM** -> Object-relational mapping(객체 관계 매핑)
* 객체는 객체대로 설계
* 관계형 데이터베이스는 관계형 데이터베이스대로 설계
* ORM 프레임워크가 중간에서 매핑
* 대중적인 언어에는 대부분 ORM 기술이 존재

JPA는 인터페이스의 모음
JPA 2.1 표준 명세를 구현한 3가지 구현체(하이버네이트, EclipseLink, DataNucleus)

생산성 - JPA와 CRUD
* 저장: jpa.persist(member)
* 조회: Member member = jpa.find(memberId)
* 수정: member.setName(“변경할이름”)
* 삭제: jpa.remove(member)

JPA의 성능 최적화 기능

1. 1차 캐시와 동일성(identity)보장
2. 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)
3. 지연 로딩(Lazy Loading)

1차 캐시와 동일성 보장
1. 같은 트랜잭션 안에서는 같은 엔티티를 반환 - 약간의 조회 성능 향상
2. DB Isolation Level 이 Read Commit 이어도 애플리케이션에서 Repeatable Read 보장

```java
String memberId= “100”;
Member m1 = jpa.find(Member.class, memberId); //SQL
Member m2 = jpa.find(Member.class, memberId); //캐시

println(m1 == m2) //true
```

트랜잭션을 지원하는 쓰기 지연 - Insert
* 트랜직션을 커밋할 때 까지 INSERT SQL 을 모음
* JDBC BATCH SQL 기능을 사용해서 한 번에 SQL 전송

``` java
transaction.begin(); //[트랜잭션] 시작

em.persist(memberA);
em.persist(memberB);
em.persist(memberC);
//여기 까지 INSERT SQL 을 데이터베이스에 보내지 않는다.

//커밋하는 순간 데이터베이스에 INSERT SQL 을 모아서 보낸다.
transaction.commit(); //[트랜잭션] 커밋
```

지연로딩과 즉시 로딩
* 지연 로딩: 객체가 실제 사용될 때 로딩
* 즉시 로딩: JOIN SQL 로 한번에 연관된 객체까지 미리 조회

지연로딩
```java
Member member = memberDAO.find(memberId); // -> select * from member
Team team = member.getTeam();
String teamName = team.getName(); // -> select * from team
```

즉시 로딩
```java
Member member = memberDAO.find(memberId); // select m.*, t.* from member join team ...
Team team = member.getTeam();
String teamName = team.getName();
```