---
tags: [Book/Effective Java/Ch2 - 객체 생성과 파괴]
title: '[Item 4] 인스턴스화를 막으려면 private 생성자를 사용하라'
created: '2022-01-04T10:00:27.095Z'
modified: '2022-01-05T00:26:26.398Z'
---

# [Item 4] 인스턴스화를 막으려면 private 생성자를 사용하라

> 정적 멤버만 담은 유틸리티 클래스는 인스턴스화를 막기 위해 private 생성자를 만들어야 한다.

종종 정적 메서드와 정적 필드만 존재하는 클래스를 만들어야 할 때가 있다.

- 기본 타입 값이나 배열 관련 메서드를 모을 때
  - `java.util.Arrays`, `java.lang.Math`
- 특정 인터페이스를 구현하는 객체를 생성하는 정적 (팩터리) 메서드를 모을 때
  - `java.util.Collections`
- final 클래스와 관련된 메서드를 모을 때
  - final 클래스는 상속이 불가능하므로 하위 클래스에 메서드를 생성하지 못하기 때문

**정적 멤버만 담은 유틸리티 클래스에 생성자를 명시하지 않으면, 컴파일러가 자동으로 기본 생성자를 만든다.** 즉, 매개변수 없는 public 생성자가 만들어져 의도치 않게 API를 공개하게 될 수 있다.

아래 예시처럼 명시적으로 private 생성자를 만들어서 객체가 생성될 가능성을 없애면 인스턴스화를 막을 뿐 아니라 하위 클래스가 생성되는 것도 막을 수 있다. 다만 원래 private 생성자의 목적대로 쓰이고 있지 않기 때문에 따로 주석을 달아두는 것이 좋다.

```java
public class UtilityClass {
  // 인스턴스화 방지
  private UtilityClass() {
    throw new AssertionError();
  }
}
```
