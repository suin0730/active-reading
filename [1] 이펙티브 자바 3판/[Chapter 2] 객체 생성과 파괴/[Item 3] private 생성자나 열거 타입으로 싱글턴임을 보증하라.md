---
tags: [Book/Effective Java/Ch2 - 객체 생성과 파괴]
title: '[Item 3] private 생성자나 열거 타입으로 싱글턴임을 보증하라'
created: '2022-01-04T09:59:56.373Z'
modified: '2022-01-04T12:54:17.056Z'
---

# [Item 3] private 생성자나 열거 타입으로 싱글턴임을 보증하라

**싱글턴(singleton)은 인스턴스를 오직 하나만 생성할 수 있는 클래스이다.** 설계상 유일하게 존재해야 하는 시스템 컴포넌트는 싱글턴으로 구현되어야 하고 무상태 객체(함수 등)도 싱글턴으로 구현할 수 있다.

## public static final 필드

Iu를 생성하는 private 생성자는 public static final 필드인 Iu.INSTANCE를 초기화할 때 한번만 호출된다. public, protected 접근제한자인 다른 생성자가 없으므로 `Iu` 클래스를 초기화할 때 만들어진 인스턴스가 전체 시스템에서 단 하나뿐임이 보장된다.

```java
public class Iu {
  public static final Iu INSTANCE = new Iu();
  private Iu() { ... }

  public void leaveTheBuilding() { ... }
}
```

이 방법은 API에 public static 필드가 final임이 명백하게 드러나 싱글턴이라는 것을 쉽게 확인할 수 있고 코드가 간결하다는 장점이 있다.

## 정적 팩토리 메서드

`Iu.getInstance`는 항상 같은 객체의 참조를 반환하므로 여전히 Iu는 유일하게 존재한다. 

```java
public class Iu {
  private static final Iu INSTANCE = new Iu();
  private Iu() { ... }
  public static Iu getInstance() { return INSTANCE; }

  public void leaveTheBuilding() { ... }
}
```

이 방법은 API를 변경하지 않고도 싱글턴이 아니도록 변경할 수 있다. 정적 팩터리 메서드가 내부적으로 다른 인스턴스를 반환하도록 변경할 수 있고 원한다면 [제네릭 싱글턴 팩터리]()로도 만들 수 있다. 마지막으로 정적 팩터리 메서드 참조를 공급자로 사용할 수 있다. **이러한 장점이 필요없다면 간결한 public 필드 방식을 사용하는 것이 낫다.**

위 두 방법 중 한 방법을 사용하려면, [싱글턴 클래스 직렬화]()에 주의해야 한다.

## 원소가 하나인 열거타입

> 대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.

이 방법은 간결하고 복잡한 직렬화 상황이나 리플렉션 공격에도 제2의 인스턴스가 생기는 일을 완벽히 막아준다.

```java
public enum Iu {
  INSTANCE;

  public void leaveTheBuilding() { ... }
}
```
