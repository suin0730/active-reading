---
tags: [Book/Effective Java/Ch9 - 일반적인 프로그래밍 원칙]
title: '[Item 61] 박싱된 타입보다는 기본 타입을 사용하라'
created: '2022-07-12T11:54:05.608Z'
modified: '2022-07-12T12:27:45.235Z'
---

# [Item 61] 박싱된 타입보다는 기본 타입을 사용하라

자바 데이터 타입은 기본 타입과 참조 타입으로 나뉘고 각 기본 타입에 대응하는 참조 타입이 하나씩 존재한다. 이를 박싱된 기본 타입이라 한다. 

## 기본 타입과 박싱된 기본 타입의 차이

1) 기본 속성은 값만 가지지만 박싱된 기본 타입은 같은 값도 구분할 수 있는 식별성을 갖는다.
2) 기본 타입과 달리 박싱된 기본 타입은 유효하지 않은 값(null)을 가질 수 있다.
3) 기본 타입이 시간과 메모리를 사용하는 데에 있어 효율적이다.

## 박싱된 기본 타입 사용 시 주의할 점

아래 코드는 `Comparator.naturalOrder()`와 같이 동작한다. 첫번째 원소가 두번째 원소보다 작으면 음수, 같으면 0, 크면 양수를 반환한다. 문제는 박싱된 기본 타입이 **식별성**을 가진다는 점이다. 만약 `naturalOrder.compare(new Integer(42), new Integer(42))`를 호출한다면 아래 프로그램은 1을 출력한다. 두 객체는 크기를 가지기 때문이다. 원래 의도대로 동작시키려면 명시적으로 언박싱을 해서 식별성 검사가 이뤄지지 않도록 해야 한다.

```java
Comparator<Integer> natrualOrder = (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);
```

또한 박싱된 기본 타입을 사용한 연산에서는 NPE를 주의해야 한다. 예시로 아래 코드는 print하기 전, NullPointerException을 일으킨다.

```java
public class Unbelievable {
  static Integer i;

  public static void main(String[] args) {
    if (i == 42)
      System.out.println("i == 42");
  }
}
```

마지막으로 무의미하게 객체가 계속 생성되지 않도록 주의해야 한다. 아래 프로그램은 sum이라는 변수를 참조 타입으로 선언해서 박싱과 언박싱이 계속 일어나 성능이 느려지는 예시이다.

```java
public static void main(String[] args) {
  Long sum = 0L;
  for (ling i = 0; o <= Integer.MAZ_VALUE; i++) {
    sum +=i;
  }
}
```

## 박싱된 기본 타입을 사용해야 하는 경우

1) 컬렉션은 기본 타입을 담을 수 없기 때문에 컬렉션의 원소, 키, 값으로 써야 한다.
2) 일반화하자면 매개변수화 타입이나 메서드의 타입 매개변수로는 박싱된 기본 타입을 사용해야 한다.
