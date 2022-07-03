---
tags: [Book/Effective Java/Ch8 - 메서드]
title: '[Item 55] 옵셔널 반환은 신중히 하라'
created: '2022-07-03T12:44:56.981Z'
modified: '2022-07-03T13:24:23.192Z'
---

# [Item 55] 옵셔널 반환은 신중히 하라

## 자바 8 이전

자바 8 이전에 메서드를 사용할 때는 값을 반환할 수 없을 때 취할 수 있는 선택지가 두 가지 있었다.

1. 예외 던지기: 예외는 진짜 예외적인 상황에서만 사용해야 하고 예외를 생성할 때 stack trace를 모두 캡처하기 때문에 큰 비용이 든다.
2. null 반환하기: null을 반환하면 이런 비용이 발생하는 문제가 생기지 않지만, **null이 반환될 일이 없다고 확신하지 않는 한** null 처리 코드를 추가해야 한다. 그렇지 않으면 실제 null을 반환하게 한 원인과 상관없는 부분에서 NPE가 발생할 수 있다.

## 자바 8 이후

자바 버전이 올라가며 생긴 Optional<T>에는 null이 아닌 T 타입 참조를 하나 담거나 아무것도 담지 않을 수 있다. **보통은 T를 반환해야 하지만 특정 조건에서는 아무것도 반환하지 않아야 할 때 `Optional`을 반환하도록 선언한다.** 옵셔널을 반환하면 예외를 던지는 것보다 사용하기 쉬우며 null을 반환하는 메서드보다 오류 가능성이 적다. 아래는 옵셔널을 반환하는 예시이다.

적절한 정적팩터리를 사용해 옵셔널을 생성해 사용하면 된다. 아래 코드에서는 `Optional.empty()`, `Optional.of(value)`를 사용했다. 옵셔널은 null을 반환해 NPE가 발생하는 것을 방지하기 위해 도입된 것으로 **옵셔널을 반환하는 메서드에서는 절대 null을 반환하면 안된다.**
```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
  if (c.isEmpty())
    return Optional.empty();
  E result = null;
  for(E e : c)
    if (result == null || e.compareTo(result) > 0)
      result = Objects.requireNonNull(e);
  return Optional.of(result);
}
```

## 옵셔널을 반환해야 하는 경우

옵셔널은 반환값이 없을 수도 있음을 API 사용자에게 명확히 알려주는 **검사예외**와 취지가 비슷하다. API 사용자는 검사 예외에 대처하는 코드를 반드시 작성해넣어야 하기 때문이다. 메서드가 옵셔널을 반환한다면 클라이언트는 값을 받지 못한 경우에 어떤 행동을 취할지 선택해야 한다.

만약 기본값을 설정하는 비용이 부담된다면 `Supplier<T>`를 인수로 받는 orElseGet을 사용하면 값이 처음 필요할 때 생성하므로 초기 설정 비용을 낮출 수 있다. 유사하게 특별한 쓰음에 대비한 메서드도 있다. (filter, map, flatMap, ifPresent)

적합한 메서드를 찾지 못했다면 isPresent 사용해 옵셔널이 채워져 있는지, 비어있는지 여부를 확인할 수 있다. 그러나 isPresent보다 더 구체화된 메서드로 대체할 수 있는 경우가 대다수이므로 실제 필요한지 여부를 잘 확인해야 한다.

```java
// 기본값을 정함
String lastWordInLexicon = max(words).orElse("단어 없음");

// 원하는 예외를 던짐
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);

// 옵셔널에 항상 값이 채워져있다고 가정
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```

## 옵셔널 사용 시 주의해야 할 경우

### 컨테이너를 옵셔널로 감쌀 때
컬렉션, 배열, 스트림, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안된다. 빈 Optional<List<T>>를 반환하는 것보다 빈 List<T>를 반환하는게 좋다.

확실하게 T 대신 Optional<T>를 사용하는 것이 나은 경우는 **결과가 없을 수 있으며, 클라이언트가 이 상황을 특별히 처리해야 하는 경우이다.** 다만 Optional도 새로 할당하고 초기화해야 하는 객체이므로 성능이 중요한 상황에서는 알맞지 않을 수 있다.

### 박싱된 기본 타입을 옵셔널로 감쌀 때
또한 박싱된 타입을 담는 옵셔널은 값을 두 번 감싸기 때문에 기본 타입보다 훨씬 무겁다. 따라서 int, ling, double 전용 옵셔널 클래스인 OptionalInt, OptionalLong, OptionalDouble을 사용하는 것이 좋다.

### 옵셔널을 컬렉션 원소로 사용할 때
옵셔널을 맵의 값으로 사용하면 맵에 키가 없다는 사실을 나타내는 방법이 두 가지가 되기 때문에 복잡성이 커져서 오류 가능성이 높아진다. 옵셔널을 컬렉션의 키, 값, 원소로 사용하는게 적절한 상황은 거의 없다.

### 인스턴스 필드 타입이 옵셔널일 때
클래스의 인스턴스 필드에 옵셔널을 사용하는 상황 대부분은 필수 필드를 갖는 클래스를 부모로 두고 이를 확장해 자식 클래스를 따로 만들어야 한다는 것을 의미한다.
