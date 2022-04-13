---
tags: [Book/Effective Java/Ch5 - 제네릭]
title: '[Item 26] 로 타입은 사용하지 말라'
created: '2022-04-13T08:10:34.834Z'
modified: '2022-04-13T09:44:48.756Z'
---

# [Item 26] 로 타입은 사용하지 말라

## 제네릭 타입

클래스나 매개변수 선언에 **타입 매개변수가 쓰이면 제네릭 타입이라고 한다.** 각 제네릭 타입은 클래스(혹은 인터페이스) 이름 뒤에 꺾쇠괄호에 실제 타입 매개변수를 나열한다. List<String>은 원소의 타입이 String인 리스트를 뜻하는 매개변수화 타입니다. 제네릭 타입을 하나 정의하면 그에 따른 로 타입(raw type)도 함께 정의된다. **다만 로 타입은 제네릭이 생기기 전 코드와 호환시키기 위한 방법이므로 특정 상황을 제외하고는 사용하지 않아야 한다.

## 로 타입보다 매개변수화된 컬렉션 타입을 사용하라

로 타입은 제네릭이 지원되기 전에 사용하던 방법으로 아래처럼 코드를 사용하면 Stamp 대신 Coin을 넣으면 꺼내기 전까지 아무 오류가 나오지 않는다. 아래 for문에서 Coin을 다시 꺼내려 시도하면 `ClassCastException`이 나온다. 이렇게 런타임에 나타나는 오류는 **원인이 되는 코드가 물리적으로 떨어져 있을 가능성이 높기 때문에 오류를 찾기 어렵다.** 

```java
private final Collection stamps = ...;

stamp.add(new Coin(...));

// 여기!
for(Iterator i = stamps.iterator(); i.hasNext(); ) {
  Stamp stamp = (Stamp) i.next();
  stamp.cancel();
}
```

아래와 같이 제네릭을 사용해 매개변수화된 컬렉션 타입을 사용하면 stamps에는 Stamp의 인스턴스만 넣어야 한다는 것을 컴파일러가 알 수 있으므로 타입 안정성이 보장된 코드라 할 수 있다. 컴파일러는 컬렉션에서 원소를 꺼내는 모든 곳에 보이지 않는 형변환을 하므로 절대 실패하지 않음을 보장한다.

```java
// 매개변수화된 컬렉션 타입
private final Collection<Stamp> stamps = ...;
```

## List vs List\<Object\>

List 같은 로 타입은 사용하면 안되지만, List\<Object\>와 같은 임의 객체를 허용하는 타입은 사용해도 괜찮다. List은 제네릭 타입과 상관없는 타입이고 List\<Object\>는 모든 타입을 허용하는 제네릭이라는 것을 컴파일러에 알려주는 타입이다. List에는 List\<String\>을 넘길 수 있지만, List\<Object\>에는 List\<String\>을 넘길 수 없다. 제네릭 하위 타입 규칙에 따라 List\<String\>은 List의 하위 타입이기 때문이다.

## Set vs Set\<?\>

만약 제네릭을 사용하고 싶지만 실제 타입 매개변수가 무엇인지 신경쓰고 싶지 않을 때 비한정적 와일드카드 타입을 사용하면 된다. Set\<E\>의 비한정적 와일드카드 타입은 Set<?>이다. 비한정적 와일드카드 타입과 로 타입은 안정성에서 차이가 난다. Collections\<?\>에는 null 외에는 어떤 원소도 넣을 수 없다. 아래 printList는 List에 어떤 매개변수가 있던지 상관쓰지 않고 메서드를 사용할 수 있다.

```java
public static void printList(List<?> list) {
    for (Object o: list)
        System.out.print(o + " ");
}
```

## 로 타입을 사용해야 하는 경우

로 타입을 사용해야 하는 경우가 몇가지 있다. 

1. class 리터럴

클래스 타입 정보를 구할 때는 class 리터럴을 사용하는데, 자바 명세는 class 리터럴에 매개변수화 타입을 사용하지 못한다. 따라서 로 타입을 사용해야 한다.

2. instanceof

런타임시에는 제네릭 타입 관련 정보가 모두 지워진 후이므로 instanceof 연산자를 사용할 때는 비한정적 와일드카드 타입 외의 매개변수에는 적용할 수 없다. 따라서 아래와 같이 사용하는 것이 좋다.

```java
if (o instanceof Set) {
  Set<?> s = (Set<?>) o;
}
```

> 로 타입은 제네릭이 도입되기 이전 코드와의 호환성을 위해 제공되는 것으로, 런타임에 예외가 일어날 수 있으니 사용하면 안된다.

