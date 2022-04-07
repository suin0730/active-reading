---
title: '[Item 22] 인터페이스는 타입을 정의하는 용도로만 사용하라'
created: '2022-04-06T08:37:41.974Z'
modified: '2022-04-06T08:53:03.028Z'
---

# [Item 22] 인터페이스는 타입을 정의하는 용도로만 사용하라

클래스가 어떤 인터페이스를 구현한다는 것은 클라이언트에게 자신의 인스턴스로 무엇을 할 수 있는지 설계도를 제시하는 것과 같다. 따라서 이 용도 외에는 인터페이스를 사용하면 안된다.

## 상수 인터페이스는 안티패턴이다

이를 어긴 예시로 상수 인터페이스가 있다. 상수 인터페이스는 메서드 없이 상수를 의미하는 `static final` 필드로만 찬 인터페이스를 말한다. 

```java
public interface PhysicalConstants {
  static final double AVOGADROS_NUMBER = ...;
  static final double BOLTSMANN_CONSTANT = ...;
  static final double ELECTROM_MASS = ...;
}
```

상수 인터페이스가 안티 패턴인 이유는 클래스 내부에서 사용하는 수를 불필요하게 외부에 노출시켰기 때문이다. 내부 구현을 API로 노출하는 것은 사용자에게 혼란을 줄 수 있고, 바이너리 수준 호환성에도 좋지 않은 영향을 미친다.

## 상수 인터페이스의 대안

1. 특정 클래스 혹은 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스 자체에 추가한다. (MIN_VALUE, MAX_VALUE)
2. 열거 타입이 적합한 상수라면 열거 타입으로 만들어 공개한다.
3. 공개가 목적이라면 인스턴스화할 수 없는 유틸리티 클래스에 담아 공개한다. 이 상수를 사용하려면 클래스 이름까지 함께 명시해야 하므로 정적으로 import해서 짧게 사용할수도 있다.

```java
public class PhysicalConstants {
  private PhysicalConstants() { }

  public static final double AVOGADROS_NUMBER = ...;
  public static final double BOLTSMANN_CONSTANT = ...;
  public static final double ELECTROM_MASS = ...;
}
```
