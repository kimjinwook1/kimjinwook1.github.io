---

layout: post
title: JPA - 페이징 한계 돌파
description: 
summary: 
tags: JPA
minute: 1

---



### 엔티티를 DTO로 변환 - 페치 조인 최적화

- 페치 조인으로 SQL은 1번만 실행된다.

- `distinct`를 사용한 이유는 1대다 조인이 있으므로 데이터베이스 row가 증가한다. 그 결과 같은 order 엔티티의 조회 수도 증가하게 된다. JPA의 `distinct`는 SQL에 `distinct`를 추가하고 더해서 같은 엔티티가 조회되면, 애플리케이션에서 중복을 걸러준다.

  -> `Order` 1개에 연결되어있는 `OrderItem`이 2개이기 때문에 `Order`도 2개가 조회된다. 

- 참고로 DB에서는 2번 조회된다.
  

### 단점

- `distinct`를 사용하므로 페이징이 불가능

**참고** 

- 컬렉션 페치 조인을 사용하면 **페이징이 불가능하다**. 하이버네이트는 경고 로그를 남기면서 모든 데이터를 DB에서 읽어오고, 메모리에서 페이징 해버린다(매우 위험).
- 컬렉션 페치 조인은 1개만 사용할 수 있다. 컬렉션 둘 이상에 페치 조인을 사용하면 안된다. 데이터가 부정확하게 조회될 수 있기 때문이다.

### 페이징과 한계 돌파

- 컬렉션을 페치 조인하면 페이징이 불가능하다.
  - 컬렉션을 페치 조인하면 일대다 조인이 발생하므로 데이터가 예측할 수 없이 증가한다.
  - 일대다에서 일(1)을 기준으로 페이징을 하는 것이 목적이다. 그런데 데이터는 다(N)를 기준으로 row가 생성된다.
  - Order를 기준으로 페이징하고 싶은데, 다(N)인 OrderItem을 조인하면 OrderItem이 기준이 되어버린다.
- 이 경우 하이버네이트는 경고 로그를 남기고 모든 DB 데이터를 읽어서 메모리에서 페이징을 시도한다. -> 최악의 경우 장애로 이어질 수 있다.

### 한계 돌파

- 먼저 ToOne(OneToOne, ManyToOne)관계를 모두 페치조인한다. ToOne 관계는 row수를 증가시키지 않으므로 페이징 쿼리에 영향을 주지 않는다.
- 컬렉션은 지연 로딩으로 조회한다.
- 지연 로딩 성능 최적화를 위해 `hibernate.default_batch_fetch_size`,`@BatchSize`를 적용한다.
  - hibernate.default_batch_fetch_size: 글로벌 설정
  - @BatchSize: 개별 최적화 -> 컬렉션은 컬렉션 필드에, 엔티티는 엔티티 클래스에 적용
  - 이 옵션을 사용하면 컬렉션이나, 프록시 객체를 한꺼번에 설정한 size 만큼 IN쿼리로 조회한다.

### `hibernate.default_batch_fetch_size`,`@BatchSize`의 장점

- 쿼리 호출수가 1+N -> 1+1로 최적화 된다.

- 조인보다 DB데이터 전송량이 최적화 된다.

  (Order와 OrderItem을 조인하면 Order가 OrderItem 만큼 중복해서 조회한다. 이 방법을 사용하면 각각 조회하므로 전송해야할 중복 데이터가 없다.)

- 페치 조인 방식과 비교해서 쿼리 호출 수가 약간 증가하지만, DB 데이터 전송량이 감소한다.

- 컬렉션 페치조인은 페이징이 불가능 하지만 **이 방법은 페이징이 가능하다.**

### **결론**

- ToOne 관계는 페치 조인해도 페이징에 영향을 주지 않는다. 따라서 ToOne 관계는 페치조인으로 쿼리 수를 줄이고, 나머지는 `hibernate.default_batch_fetch_size`,`@BatchSize`로 최적화 하자.

- **참고**

  :`hibernate.default_batch_fetch_size` 의 크기는 100~1000사이를 선택하는것이 권장된다. 이 전략은 SQL IN 절을 사용하는데, 데이터베이스에 따라 IN 절 파라미터를 1000으로 제한하기도 한다. 1000으로 잡으면 한번에 1000개를 DB에서 애플리케이션에 불러오므로 DB에 순간 부하가 증가할 수 있다. 하지만 애플리케이션은 100이든 1000이든 결국 전체 데이터를 로딩해야하므로 메모리 사용량은 같다.

  일단 100으로 설정한 후 점차 늘려나가면서 성능 테스트를 해보는 것이 좋다.

```java
@GetMapping("/api/v1/orders")
public List<Order> ordersV1() {
    List<Order> all = orderRepository.findAllByString(new OrderSearch());

    for (Order order : all) {
        order.getMember().getName();
        order.getDelivery().getAddress();

        List<OrderItem> orderItems = order.getOrderItems();
        orderItems.forEach(o -> o.getItem().getName());
    }
    return all;
}
```





```java
@GetMapping("/api/v2/orders")
public List<OrderDto> ordersV2() {
    List<Order> orders = orderRepository.findAllByString(new OrderSearch());

    List<OrderDto> result = orders.stream()
            .map(o -> new OrderDto(o))
            .collect(toList());

    return result;
}
```





```java
@GetMapping("/api/v3/orders")
public List<jpabook.jpashop.service.query.OrderDto> ordersV3() {
    return orderQueryService.ordersV3();
}

public List<OrderDto> ordersV3() {
        List<Order> orders = orderRepository.finAllWithItem();

        List<OrderDto> result = orders.stream()
                .map(o -> new OrderDto(o))
                .collect(toList());

        return result;
    }

 public List<Order> finAllWithItem() {
        return em.createQuery(
                        "select distinct o from Order o" +
                                " join fetch o.member m" +
                                " join fetch o.delivery d" +
                                " join fetch o.orderItems oi" +
                                " join fetch oi.item i", Order.class)
                .getResultList();

    }
```





```java
@GetMapping("/api/v3.1/orders") // 컬렉션은 페치조인 X -> 대신에 지연로딩
public List<OrderDto> ordersV3_page(
        @RequestParam(value = "offset", defaultValue = "0") int offset,
        @RequestParam(value = "limit", defaultValue = "100") int limit
) {
    List<Order> orders = orderRepository.findWithMemberDelivery(offset, limit);

    List<OrderDto> result = orders.stream()
            .map(o -> new OrderDto(o))
            .collect(toList());

    return result;
}

  public List<Order> findWithMemberDelivery(int offset, int limit) {

        return em.createQuery(
                        "select o from Order o" +
                                " join fetch o.member m" +
                                " join fetch o.delivery d", Order.class)
                .setFirstResult(offset)
                .setMaxResults(limit)
                .getResultList();
    }
```



### 엔티티 조회

- 엔티티를 조회해서 그대로 반환: V1
- 엔티티 조회 후 DTO 반환: V2
- 페치 조인으로 쿼리 수 최적화: V3
- 컬렉션 페이징과 한계 돌파: V3.1
  - 컬렉션은 페치 조인시 페이징이 불가능
  - ToOne 관계는 페치 조인으로 쿼리 수 최적화
  - 컬렉션은 페치 조인 대신에 지연 로딩을 유지하고, `hibernate.default_batce_fetch_size`,`@BatchSize`로 최적화





