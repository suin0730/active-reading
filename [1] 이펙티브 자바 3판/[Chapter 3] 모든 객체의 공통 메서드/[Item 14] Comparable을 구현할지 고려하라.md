---
tags: [Book/Effective Java/Ch3 - 모든 객체의 공통 메서드]
title: '[Item 14] Comparable을 구현할지 고려하라'
created: '2022-01-13T10:19:49.258Z'
modified: '2022-01-19T16:18:26.610Z'
---

# [Item 14] Comparable을 구현할지 고려하라

## Comparable 인터페이스

Comparable의 compareTo는 Object의 equals와 유사하다. **compareTo는 단순한 동치성 비교에 더해 순서까지 비교할 수 있고, 제네릭하다.** 검색, 극단값 계산, 자동 정렬되는 컬렉션도 `Array.sort(a);`를 사용해 손쉽게 정렬할 수 있다. 자바 플랫폼 라이브러리 대부분 값 클래스와 열거타입이 Comparable을 구현했으므로 알파벳, 숫자와 같이 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable을 구현하자.

## compareTo 일반 규약

- 모든 x, y에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))를 만족해야 한다. -> 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다.
- 추이성을 보장해야 한다. 즉, (x.compareTo(y) > 0 && y.compareTo(z) > 0)이면 x.compareTo(z) > 0이다. -> equals와 같은 추이성에 대한 규약이다.
- 모든 z에 대해 z.compareTo(y) == 0이면 sgn(x.compareTo(x)) == sgn(y.compareTo(z))다. -> 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다.
- (x.compareTo(y) == 0) == (x.equals(y))여야 한다. -> 정렬된 컬렉션들의 동치성을 비교할 때는 equals 대신 compareTo를 사용하므로 둘이 일관된 결과를 내야 한다.

## compareTo 메서드 작성 요령

- 제네릭: Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 compareTo 메서드의 인수 타입은 컴파일타임에 정해진다. 따라서 인수의 타입이 잘못되었다면 컴파일이 불가능하다.
- Comparable 커스텀: 인터페이스를 구현하지 않은 필드가 있거나 표준이 아닌 순서로 비교해야 한다면 비교자(Comparator)를 사용한다. 
- 관계 연산자: 박싱된 기본 타입클래스에 관계 연산자인 <, >을 사용하면 오류를 발생시키므로 compare를 사용하는 것이 좋다. 
- 핵심 필드: 핵심 필드가 여러 개라면 가장 핵심적인 필드부터 비교하자.
- 값의 차: 이따금 값의 차를 기준으로 첫번째 인자가 두번째 인자보다 작으면 음수를, 같으면 0을, 크면 양수를 반환하는 compareTo를 볼 수 있을 것이다. 정수 오버플로를 일으키거나 부동소수점 계산 방식에 따른 오류를 일으키므로 사용하지 않는 것이 좋다. 대신 아래 두 예시 중 하나를 사용하자.

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  public int compare(Object o1, Object o2) {
    return INteger.compare(o1.hashCode(), o2.hashCode());
  }
}
```

```java
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```

## 비교자 생성 메서드

compareTo에 Java8부터 생긴 비교자 생성 메서드를 사용할 수 있다. 아래 코드에서는 비교자 생성 메서드가 객체 참조를 int 타입 키에 매핑하는 키 추출 함수를 인수로 받아, 그 키를 기준으로 순서를 정하는 비교자를 반환하는 정적 메서드이다.

```java
private static final Comparator<PhoneNumber> COMPARATOR =
  comparingInt((PhoneNumber pn) -> pn.areaCode)
    .thenComparingInt(pn -> pn.prefix)
    .thenComparingINt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
  return COMPARATOR.compare(this, pn);
}
```

해시코드 차 사용하지 마세요
올바른 예씨
