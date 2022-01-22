---
layout: post
title: JPA - 순수 JPA와 Querydsl
description: 
summary: 
tags: JPA
minute: 1

---



### 실무 활용 - 순수 JPA와 Querydsl

- 순수 JPA 리포지토리와 Querydsl
- 동적쿼리 Builder 적용
- 동적쿼리 Where 적용
- 조회 API 컨트롤러 개발



순수 JPA 리포지토리

```java
public List<Member> findAll() {
		return em.createQuery("select m from Member m", Member.class)
								.getResultList();
}

public List<Member> findByUsername(String username) {
  	return em.createQuery("select m from Member m where m.username = :username", Member.class)
      					.setParameter("username", username)
      					.getResultList();
}
```





순수 JPA 리포지토리 - Querydsl 적용

```java
public List<Member> findAll_Querydsl() {
		return queryFactory
						.selectFrom(member)
						.fetch(;)
}

public List<Member> findByUsername_Querydsl(String username) {
		return queyrFactory
							.selectFrom(member)
							.where(member.username.eq(username)
							.fetch();
}
```



> **참고**
>
> 동시성 문제는 걱정하지 않아도 된다. 스프링이 주입해주는 엔티티 매니저는 실제 동작 시점에 진짜 엔티티 매니저를 찾아주는 **프록시용 가짜 엔티티 매니저**이다. 이 가짜 엔티티 매니저는 실제 사용 시점에 트랜잭션 단위로 실제 엔티티 매니저(영속성 컨텍스트)를 할당해준다.



### 동적 쿼리와 성능 최적화 조회 - Builder 사용

**`MemberTeamDto` - 조회 최적화용 DTO 추가**

-> `MemberTeamDto`에 `@QueryProjection`을 추가했다.

> **참고**
>
> `@QueryProjection`을 사용하면 해당 DTO가 Querydsl을 의존하게 된다. 이런 의존이 싫으면 해당 애노테이션을 제거하고, `Projection.bean(), Projection.fileds(), Projection.constructor()`을 사용하면 된다.



**동적쿼리 - builder 사용** (MemberSearchCondition -> 회원 검색조건)

```java
public List<MemberTeamDto> searchByBuilder(MemberSearchCondition condtion) {

		BooleanBuilder builder = new Booleanbuilder();
		if(StringUtils.hasText(condition.getUsername())) {
				builder.and(member.username.eq(condition.getUsername()));
		}
		if(StringUtils.hasText(condition.getTeamName)) {
				builder.and(team.name.eq(condition.getTeamName()));
		}
		if(condition.getAgeGoe() != null) {
				builder.and(member.age.goe(condtion.getAgeGoe()));
		}
		if(condition.getAgeLoe() != null) {
				builder.and(member.age.loe(condtion.getAgeLoe()));
		}
		
		return queryFactory
							.select(new QMemberTeamDto(
																	member.id,
																	member.username,
																	member.age,
																	team.id,
																	team.name))
							.from(member)
							.leftjoin(member.team, team)
							.where(builder)
							.fetch();
}
```





### 동적 쿼리와 성능 최적화 조회 - Where 절 파라미터 사용

```java
public List<MemberTeamDto> search(MemberSearchCondtion condtion) {
		return queryFactory
						.select(new QMemberTeamDto(
															member.id,
															member.username,
															member.age,
															team.id,
															team.name))
						.from(membeR)
						.leftjoin(member.team, team)
						.where(usernameEq(condtion.getUsername())
									,teamNameEq(condtion.getTeamName())
									,ageGoe(condtion.getAgeGoe())
									,ageLoe(condtion.getAgeLoe()))
						.fetch();
}

private BooleanExpression usernameEq(String username) {
  	return isEmpty(username) ? null : member.username.eq(username);
}

private BooleanExpression teamNameEq(String teamName) {
  	return isEmpty(teamName) ? null : team.name.eq(teamName);
}

private BooleanExpression ageGoe(Integer ageGoe) {
  	return ageGoe == null ? null : member.age.goe(ageGoe);
}

private BooleanExpression ageLe(Integer ageLoe) {
  	return ageLoe == null ? null : member.age.loe(ageLoe);
}
```





**참고** : where 절에 파라미터 방식을 사용하면 조건을 재사용할 수 있다.

**동적쿼리시 기본조건이 있는게 좋다, 혹은 리미트를 걸어줘야한다.**