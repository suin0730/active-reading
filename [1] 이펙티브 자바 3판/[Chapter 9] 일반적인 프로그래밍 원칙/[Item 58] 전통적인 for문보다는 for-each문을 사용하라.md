---
tags: [Book/Effective Java/Ch9 - 일반적인 프로그래밍 원칙]
title: '[Item 58] 전통적인 for문보다는 for-each문을 사용하라'
created: '2022-07-07T12:03:07.568Z'
modified: '2022-07-07T12:58:34.217Z'
---

# [Item 58] 전통적인 for문보다는 for-each문을 사용하라

## 일반적인 for문의 문제

아래 컬렉션 순회, 배열 순회하기 방법은 while문보다는 낫지만 **반복자와 인덱스 변수는 코드를 지저분하게 만든다.** 1회 반복 시 반복자는 세 번, 인덱스는 네 번 등장해 변수를 잘못 사용하게 될 가능성이 높아진다. 컬렉션인지 배열인지에 따라서 순회 방법도 달라지므로 사용 방법에도 주의해야 한다.

```java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
  Element e = i.next();
}
```

```java
for (int i = 0; i < a.length ; i++) {
  ...
}
```

## 향상된 for문

아래 반복문은 향상된 for문으로, for each문이라고도 한다. 이 반복문은 elements 안의 각 원소 e에 대해 원하는 동작을 할 수 있다.

```java
for (Element e : elements) {
  ...
}
```

반복자를 사용하면 중첩 for문에서 쉽게 발생하는 버그를 피할 수 있다. 아래 코드는 바깥 컬렉션인 suits의 반복자에서 next 메서드가 필요 이상으로 호출되어 `NoSuchElementException`을 발생시킨다. 

```java
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
  for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
    deck.add(new Card(i.next(), j.next()));
```

for each문을 사용하면 간단하게 문제를 해결할 수 있다.

```java
for (Suit suit : suits)
  for (Rank rank : ranks)
    deck.add(new Card(suit,rank));
```

## 향상된 for문을 사용할 수 없는 상황

- destructive filtering: 컬렉션을 순회하며 선택된 원소를 제거해야 하면 반복자의 remove 메서드를 호출해야 하므로 사용할 수 없다.
- transforming: 리스트나 배열을 순회하면서 원소 값 일부 혹은 전체를 교체해야 한다면 리스트 반복자나 배열 인덱스를 사용해야 한다.
- parallel iteration: 여러 컬렉션을 병렬 순회해야 한다면 각각의 반복자 혹은 인덱스 변수를 사용해 명시적으로 제어해야 한다.

> for each문은 Iterable 인터페이스를 구현한 객체면 무엇이든 순회할 수 있으므로 원소 묶음을 표현하는 타입을 작성해야 한다면 Iterable을 구현하는 것이 좋다.
