---
tags: [Book/Effective Java/Ch5 - 제네릭]
title: '[Item 30] 이왕이면 제네릭 메서드로 만들라'
created: '2022-05-11T13:37:30.637Z'
modified: '2022-05-11T14:10:16.380Z'
---

# [Item 30] 이왕이면 제네릭 메서드로 만들라

## 제네릭하지 않은 메서드의 문제점

앞서 본 클래스와 마찬가지로, 메서드도 제네릭으로 만들 수 있다. 로 타입을 사용하는 아래 예시를 살펴보면, unchecked error를 뱉을 것이라 예상할 수 있다.

```java
public static Set union(Set s1, Set s2) {
  Set result = new HashSet(s1);
  result.addAll(s2);
  return result;
}
```

## 메서드 제네릭하게 만들기

경고를 없애기 위해서는 타입 매개변수를 명시해 메서드를 type-safe하게 변경해야 한다. 어떤 타입 매개변수를 사용할지는 메서드의 매개변수 목록과 반환 타입을 살펴보면 된다. 코드는 아래와 같다. 아래 메서드는 단순하고 타입 안전하다.

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
  Set<E> result = enw HashSet<>(s1);
  result.addAll(s2);
  return result;
}
```

다만 이 코드는 입력 집합들과 반환 집합의 타입이 모두 같아야 한다는 제약이 있는데, 이 문제는 한정적 와일드카드 타입을 사용해 개선할 수 있다. 

## 불변 객체를 여러 타입으로 활용하기

제네릭은 런타임에 타입 정보가 소거되므로 한 객체를 어떤 타입으로든 매개변수화할 수 있다. 프로그래머가 요청할 타입 매개변수에 맞게끔 매번 객체의 타입을 바꾸는 정적 팩터리를 사용함으로써 만들 수 있다. **이를 제네릭 싱글턴 팩터리라 부르고, `Collections.emptySet` 같은 컬렉션용으로 사용할 수 있다.**

자바 라이브러리의 `Function.identity`와 같이 항등 함수를 담은 클래스가 있다고 했을 때, 항등함수 객체는 **상태가 없기 때문에 요청때마다 새로 생성하는 것은 낭비이다.** 만약 자바 제네릭을 실체화해서 코드를 작성했다면 타입별로 항등함수가 하나씩 필요했겠지만 소거 방식을 사용했으므로 제네릭 싱글턴을 재사용할 수 있다.

```java
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressedWarning("unchecked")
public statuc <T> UnaryOperator<T> identityFunction() {
  return (UnaryOperator<T>) IDENTITY_FN;
}
```

위 함수는 항등함수로, 입력값을 수정 없이 그대로 반환한다. 따라서 T가 어떤 타입이든 상관없이 타입 안전하다. 아래 예시와 같이 쓸 수 있다.

```java
public static void main(Strign[] args) {
  String[] strings = {"아", "이", "유"};
  UnaryOperator<String> sameString = identityFunction();
  for(String s : strings) 
    System.out.println(sameString.apply(s));
  
  Number[] numbers = {1, 2.0, 3L};
  UnaryOperator<Number> sameNumber = identityFunction();
  for(Number n : numbers)
    System.out.println(sameNumber.apply(n));
}
```

## 타입 매개변수의 허용 범위 한정

**재귀적 타입 한정**을 사용해 자기 자신이 들어간 표현식을 사용해서 타입 매개변수의 허용 범위를 한정할 수 있다. 대표적인 예시로는 타입 인스턴스의 순서를 정하는 `Comparable`과 함께 사용된다. 거의 모든 타입은 자신과 같은 타입의 원소와만 비교할 수 있으므로 `String`은 `Comparable<String>`을 구현하는 식으로 사용한다. 정렬이나 검색, 최대 최솟값을 구하는 연산에 사용되려면 컬렉션에 담긴 모든 원소는 상호 비교 가능해야 한다. 따라서 그 제약을 `<E extends Comparable<E>>`로 나타내어 사용할 수 있다. **이는 모든 타입 E는 자신과 비교할 수 있다**는 의미이다.


