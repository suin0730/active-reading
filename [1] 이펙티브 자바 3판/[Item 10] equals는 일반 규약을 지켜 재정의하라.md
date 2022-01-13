---
tags: [Book/Effective Java/Ch3 - 모든 객체의 공통 메서드]
title: '[Item 10] equals는 일반 규약을 지켜 재정의하라'
created: '2022-01-10T11:39:06.310Z'
modified: '2022-01-13T14:11:56.971Z'
---

# [Item 10] equals는 일반 규약을 지켜 재정의하라

equals는 주의 깊게 재정의해야 한다.

## equals를 재정의하지 않아야 하는 경우

### 각 인스턴스가 본질적으로 고유할 때

`Thread`와 같이 값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스는 각 인스턴스가 고유할 때는 equals를 재정의하지 않아야 한다.

### 인스턴스의 논리적 동치성을 검사할 일이 없을 때

각 인스턴스가 논리적으로 동일한지 검사할 필요가 없을 수 있다. 예를 들어 `java.util.regex.Pattern`은 equals를 재정의해서 두 Pattern 인스턴스가 같은 정규표현식을 나타내는지 알아볼 수도 있겠지만, 굳이 그러한 종류의 검사가 필요하지 않을 수 있다. 이 경우는 Object의 기본 equals로도 충분하다.

### 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞을 때

상위 클래스의 equals가 하위 클래스에 알맞을 때, 하위 클래스의 equals를 재정의할 필요가 없다. 예를 들어 `Set` 구현체는 `AbstractSet`이 구현한 eqauls를 상속받아 쓴다.

### 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없을 때

어느 곳에서도 equals가 호출될 일이 없다고 가정하고, 재정의할 필요가 없다. 만약에라도 위험을 피하고 싶다면 아래와 같이 재정의할 수 있다.

```java
@Override public boolean equals(Object o) {
  throw new AssertionError(); // 호출 금지!
}
```

### 같은 인스턴스가 둘 이상 만들어지지 않을 때

[인스턴스 통제 클래스](), [Enum 클래스]()는 값이 같은 인스턴스가 2개 이상 만들어지지 않으므로 객체 식별성을 확인하는 것이 논리적 식별성을 확인하는 것과 같은 의미이다.

## equals를 재정의해야 하는 경우

객체가 물리적으로 다른지 식별할 때가 아니라, **논리적으로 다른지 식별해야 하는데 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때** 재정의가 필요하다. 주로 Integer, String같은 값 클래스가 여기에 해당되는데, 일반적으로 클라이언트는 값 클래스 객체가 동일한 객체인지가 아니라 같은 값을 갖는지 검사하고 싶어할 것이다.

equals가 논리적 동치성을 확인하도록 재정의하면 우리가 equals 메서드를 쓸 때 예상하는 결과에 부응함은 물론 Map, Set의 원소로도 사용할 수 있을 것이다.

## equals 재정의 시 따라야 하는 일반 규약

equals 메서드는 동치관계를 구현한다.

> Object 명세에서 말하는 동치관계란 집합을 서로 같은 원소로 이뤄진 부분집합으로 나누는 연산이다. 올바르게 구현된 equals 메서드는 모든 원소를 같은 동치류에 속한 어떤 원소와 교환해도 만족해야 한다. 아래 다섯 가지 성질을 만족하면 동치관계를 만족한다.

### 반사성(reflexivity)

> null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true이다.

객체는 자기 자신과 같아야 한다.

### 대칭성(symmety)

> null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true이다.

아래 코드에서 CaseInsensitiveString은 String 클래스 객체와도 비교하기 위해 equals를 재정의했다. 그러나 String 클래스의 equals는 CaseInsensitiveString의 존재를 모르기 때문에 대칭성을 위반한다.

```java
// 나쁜 예시
public final class CaseInsensitiveString {
  private final String s;

  public CaseInsentiveString(String s) {
    this.s = Objects.requireNonNull(s);
  }

  @Override public boolean eqauls(Object o) {
    if (o instanceof CaseInsensitiveString)
      return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
    if (o instanceof String)
      return s.equalsIgnoreCase((String) o);
    return false;
  }  
}
```
```java
@Override public boolean
```

### 추이성(transitivity)

null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)도 true이면 x.equals(z)도 true이다.

### 일관성(consistency)

null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.

### null아님

null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false이다.


