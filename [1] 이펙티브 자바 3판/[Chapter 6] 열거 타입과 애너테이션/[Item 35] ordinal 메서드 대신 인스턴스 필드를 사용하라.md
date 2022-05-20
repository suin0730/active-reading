---
tags: [Book/Effective Java/Ch6 - 열거 타입과 애너테이션]
title: '[Item 35] ordinal 메서드 대신 인스턴스 필드를 사용하라'
created: '2022-05-20T11:43:40.434Z'
modified: '2022-05-20T11:51:53.590Z'
---

# [Item 35] ordinal 메서드 대신 인스턴스 필드를 사용하라

모든 enum 타입은 한 원소가 해당 열거 타입에서 몇 번째 위치인지를 반환하는 `ordinal`이라는 원소를 제공한다. 따라서 열거 타입에서 정수값이 필요하다면 `ordinal`을 사용하는 것을 고려하게 될 수도 있는데, 좋은 방법이 아니다. 상수 선언 순서를 바꾸면 정수값이 바뀌면서 `numberOfFruits`가 오동작하고, 중간에 빈 정수값을 두기 위해서 더미 상수를 추가해야 하거나, 중복된 정수값을 나타낼수도 없다.

```java
// ordinal의 잘못된 사용
public enum Fruit {
  APPLE, ORANGE, PEACH;

  public int numberOfFruits() { return ordinal() + 1; }
}

// enum에서 정수값을 표현하는 방법
public enum Fruit {
  APPLE(1), ORANGE(2), PEACH(3);

  private final int numberOfFruits;
  Fruit(int size) { this.numberOfFruits = size; }
  public int numberOfFruits() { return numberOfFruits; }
}
```
