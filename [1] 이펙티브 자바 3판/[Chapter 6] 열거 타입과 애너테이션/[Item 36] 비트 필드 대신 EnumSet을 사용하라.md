---
tags: [Book/Effective Java/Ch6 - 열거 타입과 애너테이션]
title: '[Item 36] 비트 필드 대신 EnumSet을 사용하라'
created: '2022-05-20T11:52:04.795Z'
modified: '2022-05-20T12:05:33.994Z'
---

# [Item 36] 비트 필드 대신 EnumSet을 사용하라

## 비트 필드의 문제점
enum 값들이 집합으로 사용될 경우, 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당하는 정수 열거 패턴을 사용했다. 아래와 같이 비트별 or를 사용해 여러 상수를 한 집합으로 모으는 방법을 사용했는데 이렇게 만들어진 집합을 비트 필드라 했다.

비트 필드를 사용하면 타입 안전성을 보장할 수 없고, 가독성이 낮아진다. 이 값이 그대로 출력된다면 그 의미를 해석하기 어렵고, 최대 몇비트가 필요한지 미리 예측할 필요가 있다.

```java
public class Text {
  public static final int STYLE_BOLD      = 1 << 0;
  public static final int STYLE_ITALIC    = 1 << 1;
  public static final int STYLE_UNDERLINE = 1 << 2;

  public void applyStyles(int styles) { ... }
}

text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

## EnumSet
대안으로 사용할 수 있는 방법이 `java.util.EnumSet` 클래스인데, 이 클래스는 enum으로 구현된 집합을 효과적으로 표현해준다. 다른 Set 구현체와도 함께 쓸 수 있고 `removeAll`, `retainAll` 등 대량 작업은 비트를 효율적으로 처리할 수 있도록 구현되어 있어 비트 필드만큼의 성능이 나온다.

```java
public class Text {
  public enum Style { BOLD, ITALIC, UNDERLINE }

  public void applyStyles(Set<Style> styles) { ... }
}

text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```
> `applyStyles`에서 `EnumSet`으로 받을 수도 있지만 이왕이면 인터페이스로 받는 것이 좋은 습관이다.
