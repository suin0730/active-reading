---
tags: [Book/Effective Java/Ch2 - 객체 생성과 파괴]
title: '[Item 5] 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라'
created: '2022-01-04T10:01:06.982Z'
modified: '2022-01-06T08:23:49.947Z'
---

# [Item 5] 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

## 자원을 직접 명시하지 않아야 하는 경우

> 사용하는 자원에 따라 동작이 달라지는 클래스에는 [정적 유틸리티 클래스](https://github.com/suin0730/active-reading/blob/main/%5B1%5D%20%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%20%EC%9E%90%EB%B0%94%203%ED%8C%90/%5BItem%204%5D%20%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4%ED%99%94%EB%A5%BC%20%EB%A7%89%EC%9C%BC%EB%A0%A4%EB%A9%B4%20private%20%EC%83%9D%EC%84%B1%EC%9E%90%EB%A5%BC%20%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC.md)나 [싱글턴](https://github.com/suin0730/active-reading/blob/main/%5B1%5D%20%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%20%EC%9E%90%EB%B0%94%203%ED%8C%90/%5BItem%203%5D%20private%20%EC%83%9D%EC%84%B1%EC%9E%90%EB%82%98%20%EC%97%B4%EA%B1%B0%20%ED%83%80%EC%9E%85%EC%9C%BC%EB%A1%9C%20%EC%8B%B1%EA%B8%80%ED%84%B4%EC%9E%84%EC%9D%84%20%EB%B3%B4%EC%A6%9D%ED%95%98%EB%9D%BC.md)이 적합하지 않다.

자원을 정적으로 명시해두는 것이 부자연스러울 때가 종종 있다. 예를 들어, 맞춤법을 검사하는 프로그램을 아래와 같이 만들었다고 생각해보자. 현실 세계에서는 언어가 바뀔수도, 특수한 사전을 사용할수도 있지만 이 코드는 단 하나의 사전만 자원으로 사용한다.

만약 `final` 한정자를 지우고 사전을 바꿀 수 있는 메소드를 추가한다면 여러 사전을 사용할 수 있겠지만 thread-safe하지 않다.

```java
// 정적 유틸리티 사용
public class SpellChecker {
  private static final Lexicon dictionary = ...;

  private SpellChecker() {}
  ...
}
```
```java
// 싱글턴 사용
public class SpellChecker {
  private final Lexicon dictionary = ...;

  private SpellChecker() {}
  public static SpellChecker INSTANCE = new SpellChecker(...);
  ...
}
```

## 자원 명시가 필요한 경우

> 클라이언트가 명시하는 자원을 클래스가 사용해야 한다면 의존성을 주입해야 한다. 이 경우 클래스의 유연성, 재사용성, 테스트 용이성이 크게 향상된다.

클래스(SpellChecker)가 클라이언트가 원하는 자원(dictionary)을 사용해야 한다면 아래 코드와 같이 **인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨줘야 한다.**

의존 객체 주입 패턴을 사용하면, [불변을 보장]()하여 같은 자원을 사용하려는 여러 클라이언트가 의존 객체를 공유할 수 있고 생성자, [정적 팩터리](https://github.com/suin0730/active-reading/blob/main/%5B1%5D%20%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%20%EC%9E%90%EB%B0%94%203%ED%8C%90/%5BItem%201%5D%20%EC%83%9D%EC%84%B1%EC%9E%90%20%EB%8C%80%EC%8B%A0%20%EC%A0%95%EC%A0%81%20%ED%8C%A9%ED%86%A0%EB%A6%AC%20%EB%A9%94%EC%84%9C%EB%93%9C%EB%A5%BC%20%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC.md), [빌더](https://github.com/suin0730/active-reading/blob/main/%5B1%5D%20%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%20%EC%9E%90%EB%B0%94%203%ED%8C%90/%5BItem%202%5D%20%EC%83%9D%EC%84%B1%EC%9E%90%EC%97%90%20%EB%A7%A4%EA%B0%9C%EB%B3%80%EC%88%98%EA%B0%80%20%EB%A7%8E%EB%8B%A4%EB%A9%B4%20%EB%B9%8C%EB%8D%94%EB%A5%BC%20%EA%B3%A0%EB%A0%A4%ED%95%B4%EB%9D%BC.md) 모두에 똑같이 응용할 수 있다.

```java
public class SpellChecker {
  private final Lexicon dictionary;

  public SpellChecker(Lexicon dictionary) {
    this.dictionary = Object.requireNonNull(dictionary);
  }
  ...
}
```

이 패턴을 변형해서 생성자에 특정 타입 인스턴스를 반복해서 넘겨주는 자원 팩터리를 넘길 수 있다. [한정적 와일드카드 타입]()을 사용해 팩터리 타입 매개변수를 제한하면 클라이언트는 자신이 명시한 타입의 하위 타입 객체를 생성할 수 있는 팩터리를 넘길 수 있다.

```java
Apartment create(Supplier<? extends house> houseFactory) { ... }
```


