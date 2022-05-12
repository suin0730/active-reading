---
tags: [Book/Effective Java/Ch5 - 제네릭]
title: '[Item 31] 한정적 와일드카드를 사용해 API 유연성을 높이라'
created: '2022-04-25T08:02:40.281Z'
modified: '2022-05-12T14:56:36.775Z'
---

# [Item 31] 한정적 와일드카드를 사용해 API 유연성을 높이라

## 매개변수 타입은 불공변이다

매개변수화 타입은 invarient하다. 즉, Type1가 Type2의 상위타입이라 할 때, List<Type2>는 List<Type1>이 하는 역할을 모두 수행하지 못하므로 하위 타입이라 할 수 없다.

원소를 스택에 추가하는 아래 메서드는 스택 원소 타입이 매개변수인 E 타입과 같다면 잘 동작할 것이다. 그러나 `Stack<Number>`를 선언하고 `push(intval)`을 수행한다면 기대와는 다르게 타입 오류를 뱉을 것이다. 매개변수화 타입은 불공변이기 때문이다.

> 불공변이란 서로 다른 타입1과 타입2가 있을 때 List<타입1>과 List<타입2>는 하위 타입도 상위 타입도 아니라는 것이다.

```java
pyblic void pushAll(Iterable<E> src) {
  for (E e : src)
    push(e);
}
```

## 한정적 와일드카드 타입

따라서 메서드를 유연하게 사용하고 싶을 때 *한정적 와일드카드 타입*을 사용할 수 있다. 우리가 위 상황에서 기대했던 바는 pushAll()의 입력 매개변수 타입이 E의 하위 타입인 상황이다. 따라서 아래와 같이 `Iterable<? extends E>`를 매개변수 타입으로 사용하면 원하는 바를 정확히 구현할 수 있다.

```java
public void pushAll(Iterable<? extends E> src) {
  for(E e : src)
    push(e);
}
```

한가지 더 고려해야 할 것은 스택 원소를 꺼내는 popAll() 메서드이다. 만약 `Stack<Number>`에 있는 원소를 `Object`용 컬렉션에 옮기려면 popAll()의 입력 매개변수 타입이 E 상위 타입 컬렉션이어야 한다. 원하는 상황을 구현하려면 `Collection<? super E>`를 사용하면 된다.

```java
public void popAll(Collection<? super E> dst) {
  while (!isEmpty())
    dst.add(pop());
}
```

> 메서드 유연성을 극대화하려면 원소의 생산자(pushAll)나 소비자(popAll) 입력 매개변수에 와일드카드 타입을 사용해야 한다. PECS(producer-extend, consumer-super)를 생각하면 된다.

> 클라이언트 코드에서도 와일드카드 타입을 써야 할수도 있기 때문에 반환 타입에는 한정적 와일드카드 타입을 사용하면 안 된다. 

## java7과 java8

java7까지는 타입 추론 능력이 충분히 강력하지 않아서 문맥에 맞는 반환 타입을 명시해야 할 수도 있다. 명시적 타입 인수를 사용해서 타입을 알려주면 java7 이하에서도 무리 없이 컴파일할 수 있다.

## 한정적 와일드카드 타입이 꼭 필요한 경우

항상 소비자로 사용되는 Comparable은 일반적으로 `Comparable<? super E>`로 사용하고, 항상 생산자로 사용되는 Comparator은 `Comparator<? super E>`를 사용하는 편이 낫다. 그러면 아래 코드와 같은 함수 선언이 나온다.

```java
public static <E extends Comparable<? super E>> E max(List<? extends E> list)
```

## 타입 매개변수 vs 와일드카드

타입 매개변수와 와일드카드는 타입을 정해줄 수 있다는 점에서 비슷한 역할을 한다. 메서드를 정의할 때 둘 중 어느 것을 사용해도 괜찮을 때가 많다. public API로 공개할 예정이라면 간단한 와일드카드 방식이 낫지만, 방금 꺼낸 원소를 바로 넣을 수 없다는 부작용이 생길 수 있다. 따라서 와일드카드 타입의 실제 타입을 알려주는 메서드를 제네릭한 private 도우미 메서드로 따로 작성해서 활용하면 된다.

```java
public static void swap(List<?> list, int i, int j) {
  swapHelper(list, i, j);
}
// 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
private static <E> void swapHelper(list<E> list, int i, int j) {
  list.set(i, list.set(j, list.get(i)));
}
```

