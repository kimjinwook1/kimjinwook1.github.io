---
layout: post
title: JPA - Spring Data JPA - Query Method
description: 
summary: 
tags: JPA
minute: 1

---

# 쿼리 메소드 기능

- 메소드 이름으로 쿼리 생성
- NamedQuery
- @Query - Repository 메소드에 쿼리 정의
- 파라미터 바인딩
- 반환 타입
- 페이징과 정렬
- 벌크성 수정 쿼리
- @EntityGraph
  

### 쿼리 메소드 기능 3가지

- 메소드 이름으로 쿼리 생성
- 메소드 이름으로 JPA NamedQuery 호출
- `@Query` 어노테이션을 사용해서 Repository 인터페이스에 쿼리 직접 정의



### 메서드 이름으로 쿼리



 **쿼리 메소드 필터 조건**

-> 스프링 데이터 JPA 공식 문서 참고
[https://docs.spring.io/spring-data/jpa/docs/current/ reference/html/#jpa.query-methods.query-creation]()

**스프링 데이터 JPA가 제공하는 쿼리 메소드 기능**

- 조회: find...By, read...By, query...By, get...By
  - ex) findHelloBy 처럼 위의 ...에 식별하기 위한 내용(설명)이 들어가도 된다.
- COUNT: count...By -> 반환타입: `long`
- EXISTS: exists...By -> 반환타입: `boolean`
- 삭제: delete...By, remove...By -> 반환타입: `long`
- DISTINCT: findDistinct, findMemberDistinctBy
- LIMIT: findFirst3, findFirst, findTop, findTop3

> **참고**
>
> - 이 기능은 엔티티의 필드명이 변경되면 인터페이스에 정의한 메서드 이름도 꼭 함께 변경해야 한다.
>
>   그렇지 않으면 애플리케이션을 시작하는 시점에 오류가 발생한다.
>
>   이렇게 애플리케이션 로딩 시점에 오류를 인지할 수 있는 것이 스프링 데이터 JPA의 장점이다.



### JPA NamedQuery -> NamedQuery를 직접 등록해서사용하는일이 드물다.

:JPA의 NamedQuery를 호출할 수 있음.

`@NamedQuery` 어노테이션으로 Named쿼리 정의

```java
@Entity
@NamedQuery(
	name = "Member.findByUsername",
	query="select m from Member m where m.username = :username"
)
public class Member {
	...
}
```

**스프링 데이터 JPA로 NamedQuery 사용**

```java
@Query(name = "Member.findByUsername") // -> 생략가능
List<Member> findByUsername(@Param("username") String username);
```

`@Query`를 생략하고 메서드 이름만으로 Named 쿼리를 호출할 수 있다.



### @Query, 값, DTO 조회하기

**단순히 값 하나를 조회**

```java
@Query("select m.username from Member m")
List<String> findUsernameList();
```

**DTO로 직접 조회**

```java
@Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) from Member m join m.team t")
List<MemberDto> findMemberDto();
```

- 주의! DTO로 직접 조회하려면 JPA의 `new`명령어를 사용해야 한다. 파라미터에 적합한 생성자가 필요하다.



### 파라미터 바인딩

- 위치 기반
- **이름 기반** -> 이 방식을 사용하자

```java
select m from Member m where m.username = ?0 //위치 기반
select m from Member m where m.username = :name //이름 기반
```

**파라미터 바인딩 사용**

```java
public interface MemberRepository extends JpaRepository<Member, Long>{

		@Query("select m from Member m where m.username = :name")
		Member findMembers(@Param("name") String username);
}
```

> 참고: 코드 가독성과 유지보수를 위해 이름 기반의 파라미터 바인딩을 사용하자
> (위치기반은 순서가 바뀌면 실수할 가능성이 크다.)

**컬렉션 파라미터 바인딩** -> `Collection`타입으로 in절 지원

```java
@Query("select m from Member m whree m.username in :names")
List<Member> findByNames(@Param("name") List<String> names) 
```

**반환타입**

```java
List<Member> result = memberRepository.findListByUsername("CCC"); 
//리스트 조회는 무조건 Null이 아님 -> 없으면 empty컬렉션이 반환됨
System.out.println("result.size() = " + result.size());

Member findMember = memberRepository.findMemberByUsername("CCC"); //단건 조회는 없으면 Null 반환
System.out.println("findMember = " + findMember);

Optional<Member> findOptionalMember = memberRepository.findOptionalByUsername("CCC");
//Optional 조회 -> 없으면 Optional.empty 반환
System.out.println("findOptionalMember = " + findOptionalMember);
```



**조회 결과가 많거나 없으면?**

- 컬렉션

  - 결과 없음: 빈 컬렉션을 반환
- 단건 조회

  - 결과없음: `null`반환
  - 결과가 2건 이상: `javax.persistence.NonUniqueResultException` 예외 발생




### 순수 JPA 페이징과 정렬

**JPA 페이징 Repository 코드**

```java
public List<Member> findByPage(int age, int offset, int limit) {
   return em.createQuery("select m from Member m where m.age = :age order by m.username desc")
      .setParameter("age", age)
      .setFirstResult(offset)
      .setMaxResults(limit)
      .getResultList();
}

public long totalCount(int age) {
   return em.createQuery("select count(m) from Member m where m.age = :age", Long.class)
      .setParameter("age", age)
      .getSingleResult();
}
```



**페이징과 정렬 파라미터**

- `org.springframework.data.domain.Sort`: 정렬 기능
- `org.springframework.data.domain.Pageable`: 페이징 기능(내부에 `sort`포함)

**특별한 반환타입**

- `org.springframework.data.domain.Page`: 추가 count 쿼리 결과를 포함하는 페이징
- `org.springframework.data.domain.Slice`: 추가 count 쿼리 없이 다음 페이지만 확인 가능 -> 내부적으로 limit +1 조회
- `List`(자바 컬렉션): 추가 count 쿼리 없이 결과만 반환

**페이징과 정렬 사용 코드**

```java
Page<Member> findByUseername(String name, Pageable pageable);
Slice<Member> findByUsername(String name, Pageable pageable);
List<Member> findByUsername(String name, Pageable pageable);
List<Member> findByUsername(String name, Sort sort);
```

**Page 사용 예제 코드**

```java
@Test
	void paging() {
	    //given
		memberJpaRepository.save((new Member("member1", 10)));
		memberJpaRepository.save((new Member("member2", 10)));
		memberJpaRepository.save((new Member("member3", 10)));
		memberJpaRepository.save((new Member("member4", 10)));
		memberJpaRepository.save((new Member("member5", 10)));

		int age = 10;
		PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));

		//when
		Page<Member> page = memberRepository.findByAge(age, pageRequest);

		
    Page<MemberDto> map = page.map(
			m -> new MemberDto(m.getId(), m.getUsername(), null)); //map을 통해 엔티티를 DTO로 쉽게 변경할 수 있다.

    //then
    List<Member> content = page.getContent(); //조회된 데이터
		assertThat(content.size()).isEqualTo(3); //조회된 데이터 수
		assertThat(page.getTotalElements()).isEqualTo(5); //전체 데이터 수
		assertThat(page.getNumber()).isEqualTo(0); //페이지 번호 -> 페이지 번호는 0부터 시작한다.
		assertThat(page.getTotalPages()).isEqualTo(2); //전체 페이지 번호
		assertThat(page.isFirst()).isTrue(); //첫번째 항목인가?
		assertThat(page.hasNext()).isTrue(); //다음 페이지가 있는가?
	}

```



- 파라미터로 받은 `Pagable`은 인터페이스이다. 따라서 실제 사용할 때는 해당 인터페이스의 구현체인 `org.springframework.data.domain.PageReuqest`객체를 사용한다.

- `PageRequest`생성자의 첫 번째 파라미터로 현재 페이지를, 두 번째 파라미터로 조회할 데이터 수를 입력한다.

  여기에 추가로 정렬 정보도 파라미터로 사용할 수 있다.

> **주의**: Page는 **0**부터 시작한다!

Paging 할때는 Pageable인터페이스를 구현한 것을 많이 씀 -> 구현체 PageRequest



### 벌크성 수정 쿼리

```java
@Test
   void bulkUpdate() {
      //given
      memberRepository.save(new Member("member1", 10));
      memberRepository.save(new Member("member2", 19));
      memberRepository.save(new Member("member3", 20));
      memberRepository.save(new Member("member4", 21));
      memberRepository.save(new Member("member5", 40));

      //when
      int resultCount = memberRepository.bulkAgePlus(20);
//    em.clear();

      List<Member> result = memberRepository.findByUsername("member5");
      Member member5 = result.get(0);
      System.out.println("member5 = " + member5);

//    then
      assertThat(resultCount).isEqualTo(3);
   }
```

-> 영속성 컨텍스트에 있는 상태와 엔티티와 DB에 있는 상태가 달라질 수 있다.

- **벌크성 수정, 삭제 쿼리는 `@Modifying` 어노테이션을 사용!**
  - `@Modifying`를 사용하지 않으면 `org.hibernate.hql.internal.QueryExecutionRequestException` 예외 발생
- 벌크성 쿼리를 실행하고 나서 영속성 컨텍스트 초기화 -> `@Modifying(clearAutomatically = true)`
  

- **참고**

  벌크 연산은 영속성 컨텍스트를 무시하고 실행하기 때문에 영속성 컨텍스에 있는 엔티티의 상태와 DB에 있는 엔티티 상태가 달라질 수 있다.

- **권장하는 방안**

  1. 영속성 컨텍스트에 엔티티가 없는 상태에서 벌크 연산을 먼저 실행한다.
  2. 부득이하게 영속성 컨텍스트에 엔티티가 있으면 벌크 연산 직후 영속성 컨텍스트를 초기화 한다.

  > **참고**
  >
  > - 위의 코드에서 `save()`메서드로 호출된 `new Member`들은 영속성 컨텍스트에서 관리된다.
  >
  >   하지만 벌크성 수정 쿼리인 `bulkAgePlus()`는 영속성컨텍스트를 거치지 않고 바로 DB로 넘어가기 때문에 
  >
  >   DB에 있는 데이터와 영속성 컨텍스트에 있는 데이터가 다르다.
  >
  >   그렇기 때문에 만약 다시 조회를 해야 하면 영속성 컨텍스트를 초기화 해야한다!



### @EntityGraph

연관된 엔티티들을 SQL로 한번에 조회하는 방법

**EntityGraph 사용 코드**

```java
//공통 메서드 오버라이드
@Override
@EntityGraph(attributetPaths = {"team"})
List<Member> findAll();

//JPQL + 엔티티 그래드
@EntityGraph(attributetPaths = {"team"})
@Query("select m from Member m")
List<Member> findMemberEntityGraph();

//메서드 이름으로 쿼리에서 특히 편리하다.
@EntityGraph(attributePaths = {"team"})
List<member> findByUsername(String username)
```

**EntityGraph 정리**

- 사실상 페치조인의 간편버전

**간단하면 @EntityGraph 사용, 복잡해지면 JPQL의 페치 조인 사용**