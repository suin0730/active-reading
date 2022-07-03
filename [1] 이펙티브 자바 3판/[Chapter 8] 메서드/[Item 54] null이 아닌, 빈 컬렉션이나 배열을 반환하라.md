---
tags: [Book/Effective Java/Ch8 - 메서드]
title: '[Item 54] null이 아닌, 빈 컬렉션이나 배열을 반환하라'
created: '2022-07-03T12:27:59.029Z'
modified: '2022-07-03T12:44:36.669Z'
---

# [Item 54] null이 아닌, 빈 컬렉션이나 배열을 반환하라

아래 코드처럼 컬렉션이 비어있을 때 null을 반환한다면 클라이언트는 null을 처리하는 코드를 추가로 작성해야 한다. 빈 컨테이너를 반환하는 것이 null을 반환하는 것보다 나은 이유는 다음과 같다.

1. 빈 컨테이너를 할당하는 것이 유의미한 성능 저하를 일으키지 않는다.
2. 빈 컬렉션과 배열은 새로 할당하지 않고도 반환할 수 있다.

```java
private final List<Cheese> cheesesInStock = ...;

// 컬렉션이 비어있으면 null을 반환하는 메서드
public List<Cheese> getCheese() {
  return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock);
}

// 클라이언트에서 null을 따로 처리
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON))
  System.out.println("");
```

> null을 반환하는 API는 사용하기 어렵고 오류 처리 코드가 늘어나지만 성능이 좋은 것은 아니다.
