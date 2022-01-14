---
tags: [Book/Effective Java/Ch3 - 모든 객체의 공통 메서드]
title: '[Item 10] equals는 일반 규약을 지켜 재정의하라'
created: '2022-01-10T11:39:06.310Z'
modified: '2022-01-14T01:14:34.336Z'
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

[인스턴스 통제 클래스](https://github.com/suin0730/active-reading/blob/main/%5B1%5D%20%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%20%EC%9E%90%EB%B0%94%203%ED%8C%90/%5BItem%201%5D%20%EC%83%9D%EC%84%B1%EC%9E%90%20%EB%8C%80%EC%8B%A0%20%EC%A0%95%EC%A0%81%20%ED%8C%A9%ED%86%A0%EB%A6%AC%20%EB%A9%94%EC%84%9C%EB%93%9C%EB%A5%BC%20%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC.md), [Enum 클래스](34)는 값이 같은 인스턴스가 2개 이상 만들어지지 않으므로 객체 식별성을 확인하는 것이 논리적 식별성을 확인하는 것과 같은 의미이다.

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

따라서 CaseInsensitiveString과 String을 equals로 비교할 생각을 하지 말고,  CaseInsensitiveString의 equals를 좋은 예시처럼 분리해야 한다.

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
// 좋은 예시
@Override public boolean equals(Object o) {
  return o instanceof CaseInsensitiveString && ((CaseInseneitiveString o).s.equalsIgnoreCase(s);
}
```

### 추이성(transitivity)

> null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)도 true이면 x.equals(z)도 true이다.

점의 좌표를 멤버로 가지는 Point 클래스와 색깔 정보를 더해 확장한 ColorPoint 클래스가 있다고 할 때, 아래 예시 상황은 추이성을 만족하지 않는다. p1.equals(p2)와 p2.equals(p3)는 true이나 p1.equals(p3)는 false를 반환하기 때문이다.

```java
// 예시 상황
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
```
```java
public class Point {
  private final int x;
  private final int y;

  public Point(int x, int y) {
    this.x = x;
    this.y = y;
  }

  @Override public boolean equals(Object o) {
    if (!(o instanceof Point))
      return false;
    Point p = (Point)o;
    return p.x == x && p.y == y;
  }
}
```
```java
public class ColorPoint extends Point {
  private final Color color;

  public ColorPoint(int x, int y, Color color) {
    super(x, y);
    this.color = color;
  }

  @Override public boolean equals(Object o) {
    if (!(o instanceof Point))
      return false;

    if(!(o instanceof ColorPoint))
      return o.equals(this);
    
    return super.equals(o) && ((ColorPoint) o).color == color;
  }
}
```

> 문제는 구체 클래스를 확장해 새로운 값을 추가하며 equals 규약을 만족시킬 방법은 존재하지 않는다는 것이다.

만약 추이성을 만족시키기 위해 instanceof가 아니라 getClass를 사용한다면, **특정 상황에서 리스코프 치환 원칙을 위배한다.** Set과 같은 컬렉션 구현체에서는 contains는 equals를 기반으로 구현되어 있다. 따라서 주어진 원소를 가지는지를 확인하는 contains를 호출했을 때 **getClass를 사용해 재정의된 equals를 사용한다면 하위 클래스는 여전히 상위 클래스처럼 사용되어야 함에도 항상 false를 반환할 것이다.**

구체 클래스의 하위 클래스에서 값을 추가하는 대신, 아래 코드와 같이 컴포지션을 사용할 수 있다. Point를 ColorPoint의 private 필드로 두고, ColorPoint와 같은 위치의 Point를 반환하는 [뷰 메서드](https://github.com/suin0730/active-reading/blob/main/%5B1%5D%20%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%20%EC%9E%90%EB%B0%94%203%ED%8C%90/%5BItem%206%5D%20%EB%B6%88%ED%95%84%EC%9A%94%ED%95%9C%20%EA%B0%9D%EC%B2%B4%20%EC%83%9D%EC%84%B1%EC%9D%84%20%ED%94%BC%ED%95%98%EB%9D%BC.md)를 public으로 추가하는 것이다.

```java
public class ColorPoint {
  private final Point point;
  private final Color color;

  public ColorPoint(int x, int y, Color color) {
    point = new Point(x, y);
    this.color = Objects.requireNonNull(color);
  }

  // 뷰를 반환하는 메서드
  public Point asPoint() {
    return point;
  }

  @Override public boolean equals(Object o) {
    if (!(o instanceof ColorPoint))
      return false;
    ColorPoint cp = (ColorPoint) 0;
    return cp.point.equals(point) && cp.color.equals(color);
  }
}
```

### 일관성(consistency)

> null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.

일관성은 두 객체가 같다면 앞으로도 영원히 같아야 한다는 뜻이다. 가변 객체는 비교 시점에 따라 비교 결과가 달라질 수 있지만, 불변 객체는 항상 같은 결과를 반환해야 한다.

특히 일관성을 지키기 위해서는, equals의 판단에 신뢰할 수 없는 자원이 끼어들면 안된다. `java.net.URL`의 equals는 주어진 url과 매핑된 호스트의 IP 주소를 이용해 비교하는데, 호스트 이름은 네트워크를 통해야 변경할 수 있으므로 equals의 결과가 항상 같다고 보장할 수 없다. **이런 문제를 피하려면 equals는 항상 메모리에 존재하는 객체만을 사용해 결정적인 계산만 수행해야 한다.**

### null아님

> null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false이다.

이 규약은 모든 객체가 null과 같이 않아야 한다는 뜻이다. 아래 코드처럼 명시적으로 null을 검사할 수도 있겠지만, equals에 사용되는 instanceof는 첫번째 인자가 null이면 항상 false를 반환하므로 null을 명시적으로 검사하지 않아도 된다.

```java
@Override public boolean equals(Object o) {
  if (o == null)
    return false;
}
```

## 양질의 equals 메서드 구현법

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인하면 성능이 좋아질 것이다.
    - float과 double은 부동 소수값을 다뤄야 하므로 == 대신 compare()를 사용한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
    - 이때 올바른 타입은 equals가 정의된 클래스일 수도, 그 클래스가 구현한 인터페이스일 수도 있다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신에서 핵심 필드가 모두 일치하는지 확인한다.
    - 어떤 필드를 먼저 비교하느냐가 equals의 성능을 좌우하기도 한다.
    - 높은 성능을 요한다면 다를 가능성이 크거나 비교 비용이 싼 필드를 먼저 비교하자.
    - 파생 필드가 객체의 상태를 대표한다면 파생 필드를 사용하는 것도 좋다.
5. equals를 재정의했으면 [hashCode도 재정의하자.](11)
6. Object 외의 타입을 매개변수로 받는 equals 메서드는 작성하지 말아야 한다.
    - Object 외의 타입을 받으면 그것은 `Object.equals`를 재정의한 것이 아니라 새로운 메서드를 생성해 equals를 [다중정의](52)한 것이다.

AutoValue를 사용하지 않고 equals를 구현했다면 해당 메서드가 대칭적인지, 추이성이 있는지, 일관적인지 단위 테스트를 돌려보자. 아래 코드는 잘 작성된 equals의 예시이다.

```java
@Override public boolean equals(Object o) {
  if (o == this)
    return true;
  if (!(o instanceof PhoneNumber))
    return false;
  PhoneNumber pn = (PhoneNumber)o;
  return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
}
```
