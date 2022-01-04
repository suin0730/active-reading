---
tags: [Book/Effective Java/Ch2 - 객체 생성과 파괴]
title: '[Item 1] 생성자 대신 정적 팩토리 메서드를 고려하라'
created: '2022-01-03T11:16:35.634Z'
modified: '2022-01-04T02:22:47.604Z'
---

# [Item 1] 생성자 대신 정적 팩토리 메서드를 고려하라

## 정적 팩토리 메서드로 인스턴스 생성 시 장점

### 1. 이름 사용

메서드 명에 클라이언트에 반환할 객체 특성을 나타낼 수 있어서 이름이 없는 생성자에 비해 개발자들이 각 팩토리 메서드가 어떤 역할을 하는지 이해하기 쉽다. 

### 2. 인스턴스 통제

호출될 때마다 동일한 객체가 반환되도록 해서 인스턴스를 통제할 수 있고 [싱글톤]() 혹은 [인스턴스화 불가]()하게 만들어서 성능을 끌어올릴 수 있다.

### 3. 하위 객체 반환

정적 팩토리 메소드를 사용하면 반환 타입의 하위 객체를 반환할 수 있다. 이 특징은 API를 더 유연하고 가볍게 만든다.

#### [예시] EnumSet

> 클라이언트는 팩터리가 전달하는 객체가 어떤 클래스의 인스턴스인지 알 필요 없다. 단지 `EnumSet`의 하위 클래스이기만 하면 된다.

`EnumSet`은 public 생성자 대신 정적 팩터리 메서드를 사용한다. 아래 `noneOf` 메서드를 보면 `universe.length`가 64보다 큰지에 따라 `EnumSet`의 하위 클래스인 `RegularEnumSet` 또는 `JumboEnumSet을` 반환할 수 있다.

```java
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
  Enum<?>[] universe = getUniverse(elementType);
  if (universe == null)
      throw new ClassCastException(elementType + " not an enum");

  if (universe.length <= 64)
      return new RegularEnumSet<>(elementType, universe);
  else
      return new JumboEnumSet<>(elementType, universe);
}
```

#### [예시] Collections

> 정적 팩토리 메서드를 응용해 구현 클래스를 숨기면 API를 논리, 물리적으로 가볍게 만들 수 있다. [인터페이스 기반 프레임워크]()의 핵심 기술이다.

`java.util.Collections`는 `java.util.Collection` 인터페이스의 구현체이며 동반 클래스(companion class)라고도 한다. `unmodifiableSet` 반환 타입은 인터페이스이고, 실제로 반환하는 것은 인터페이스 하위 객체인 `UnmodifiableSet`의 인스턴스이다. 여기서 클라이언트는 인스턴스가 어떻게 구현되었는지 알 필요가 없으므로 API를 사용하기 위해 인터페이스 정보만 알면 된다.

```java
public class Collections {

  // 정적 팩토리 메서드
  public static <T> Set<T> unmodifiableSet(Set<? extends T> s) {
        return new UnmodifiableSet<>(s);
    }

  // 구현 클래스
  static class UnmodifiableSet<E> extends UnmodifiableCollection<E> implements Set<E>, Serializable{/* 중략 */}
}
```

 단, Java8 이전에는 인터페이스에 static 메서드를 사용할 수 없었으므로 동반 클래스를 사용했다는 점에 유의한다. 

#### [예시] JDBC

> 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다. [추가 예정](https://github.com/suin0730/active-reading/issues/1)

## 정적 팩토리 메서드로 인스턴스 생성 시 단점

### 1. 하위 클래스 생성 불가

생성자 대신 정적 팩토리 메서드만 사용한다면 하위 클래스를 만들 수 없다. 상속하기 위해서는 상위 클래스에 public이나 protected 생성자가 필요하기 때문이다. (상속보다 [컴포지션]() 사용을 권장한다는 점에서 장점일 수 있다.)

### 2. 객체를 생성하는 메서드인지 알기 어려움

생성자와 비교해보면 정적 팩토리 메서드는 API 설명에서 객체를 생성하는 역할을 하는지 알기 어렵다. 


