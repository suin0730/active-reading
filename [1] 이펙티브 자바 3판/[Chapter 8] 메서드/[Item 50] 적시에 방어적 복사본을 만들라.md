---
tags: [Book/Effective Java/Ch8 - 메서드]
title: '[Item 50] 적시에 방어적 복사본을 만들라'
created: '2022-06-14T00:08:20.489Z'
modified: '2022-06-14T01:07:58.945Z'
---

# [Item 50] 적시에 방어적 복사본을 만들라

## 방어적 프로그래밍의 필요성

네이티브 코드는 unmanaged code라고도 하고 컴파일하면 os에서 해석가능한 기계어로 번역된다. 반면 managed code라고도 불리는 자바는 컴파일 시 바이너리 코드로 변환되는 것이 아니라 jvm에서 해석가능한 `.class` 코드로 변경되므로 안전하다고 할 수 있다. 예를 들어 네이티브 메서드를 사용하는 C, C++ 등에서 보는 버퍼 오버플로우, 배열 오버런 등의 메모리 충돌 오류에서 안전하다.

일반적으로 자바로 작성한 클래스는 시스템 다른 부분에서 어떤 처리를 하던 불변식이 지켜지지만 **개발자는 클라이언트가 불변식을 언제든 깰 수 있다고 생각하고 방어적으로 프로그래밍해야 한다.**

## Period 예제

어떤 객체든 해당 객체의 허락 없이는 **외부에서 내부를 수정하는 일이 불가능하다.** 아래 Period를 표현하는 클래스는 한번 값이 정해지면 변하지 않도록 하는 것을 의도하고 있다.

```java
public final class Period {
  private final Date start;
  private final Date end;

  public Period(Date start, Date end) {
    if (start.compareTo(end) > 0 {
      throw new IllegalArgumentException("exception");
    }
    this.start = start;
    this.end = end;
  }

  public Date start() {
    return start;
  }

  public Date end() {
    return end;
  }
}
```

### 불변식이 깨지는 경우 1

start, end의 클래스인 `Date`가 가변인 경우 Period의 불변성을 깰 수 있다. 대안으로는 가변인 `Date` 대신 `Instant`, `LocalDateTime`, `ZonedDateTime` 등을 사용하면 된다.

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYest(78);  // 불변식 깨짐
```

만약 이미 `Date`로 작성된 코드가 있다면 **생성자에서 받은 가변 매개변수 각각을 방어적으로 복사해야 한다.** 아래처럼 방어적으로 복사하고 이 복사본으로 유효성을 검사한다면 멀티스레딩 환경에서 1)유효성을 검사한 후 2)복사본을 만드는 찰나의 순간 사이에 다른 스레드가 원본 객체를 수정할 가능성이 있으므로 위험하다. (해당 방법으로 TOCTOU 공격을 막을 수 있다.)

```java
public Period(Date start, Date end) {
  this.start = new Date(start.getTime());
  this.end = new Date(end.getTime());
  if (start.compareTo(end) > 0 {
    throw new IllegalArgumentException("exception");
  }
}
```

> `Date`는 final이 아니므로 상황에 따라 clone이 악의적인 하위 클래스 인스턴스를 반환할 가능성이 있기 때문이다. 예를 들어 하위 클래스가 start와 end의 참조를 private 정적 리스트에 담아두었다가 공격자에게 이 리스트를 제공할 수도 있다. 따라서 방어적 복사를 할 때 `Date`의 clone 메서드를 사용하지 않아야 한다.

### 불변식이 깨지는 경우 2

생성자를 수정하면 위 단락에서 설명한 공격은 막을 수 있지만 접근자 메서드가 내부 가변 정보를 드러내는 한 Period 인스턴스는 여전히 변경 가능하다.

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end.setYest(78);  // 불변식 깨짐
```

이 공격을 막아내려면 **접근자가 가변 필드의 방어적 복사본을 반환하도록 하면 된다.** 이렇게 조치하면 **Period 자신 말고는 가변 필드에 접근할 방법이 없고 모든 필드가 완벽히 캡슐화된 것이다.**

```java
public Date start() {
  return new Date(start.getTime());
}

public Date end() {
  return new Date(end.getTime());
}
```

## 매개변수를 방어적으로 복사하는 목적

불변 객체를 만들기 위해 매개변수를 방어적으로 복사하지만 클라이언트가 제공한 객체 참조를 내부 자료구조에 보관해야 할 때 객체가 변해도 클래스가 문제없이 동작한다는 확신이 없을 때에도 매개변수를 방어적으로 복사해야 한다. 같은 맥락으로 내부 객체를 클라이언트에 건네주기 전에 방어적 복사본을 만들어서 원본을 노출하지 말아야 한다.

## 방어적 복사를 할 때 고려해야 하는 것

방어적 복사는 성능 저하가 따른다. 또한 같은 패키지에 속할 때는 호출자가 컴포넌트 내부를 수정하지 않는다고 확신한다면 굳이 방어적 복사를 해야 할 이유도 없다. (다만 이 경우에도 호출자에서 해당 매개변수나 반환값을 수정하지 말아야 한다고 문서화를 해두어야 한다.)

다른 패키지에 속하는 경우에도 항상 방어적 복사를 수행해야 하는 것은 아니다. 매개변수로 값을 넘기는 것을 해당 객체 통제권을 넘기는 것이라 보면 된다. 따라서 해당 클래스와 클래스를 사용하는 클라이언트가 상호 신뢰할 수 있거나, 불변식이 깨져도 호출한 클라이언트에만 영향이 갈 때에 한정해서 생략해야 한다.

> 되도록이면 불변 객체들을 조합해 객체를 구성해야 방어적 복사를 할 일이 줄어든다.
