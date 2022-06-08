---
tags: [Book/Effective Java/Ch7 - 람다와 스트림]
title: '[Item 46] 스트림에서는 부작용 없는 함수를 사용하라'
created: '2022-06-08T13:03:27.108Z'
modified: '2022-06-08T13:49:16.439Z'
---

# [Item 46] 스트림에서는 부작용 없는 함수를 사용하라

> 스트림은 하나의 API라기보다 함수형 프로그래밍에 기초한 패러다임이다.

스트림 패러다임의 핵심은 계산의 일련의 변환으로 재구성하는 것이다. 여기에서 이뤄지는 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수여야 한다. 이 목표를 달성하기 위해서는 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않아야 한다. 

```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
  words.forEach(word -> {
    freq.merge(word.toLowerCase(), 1L, Long::sum);
  })
}
```

위 코드는 file에서 단어 수를 세어 빈도표를 반환하는 함수이지만 스트림 코드라 하기 어렵다. forEach문에서 스트림이 수행한 연산 결과를 보여주는 것 외에 **상태를 수정하는 등 외부에 영향을 준다면** 문제가 생길 것이다.

```java
Map<String, Long> freq;
try (Stream<String> words = new Scanner(File).tokens()) {
  freq = words
    .collect(groupingBy(String::toLowerCase, counting()));
}
```

위 코드는 스트림 코드이며 짧고 명확하다. 첫번째 잘못된 예시에서 사용된 forEach 종단 연산은 스트림 계산 연산을 보고할 때만 사용하고 계산하는 데는 사용하지 않는 것이 좋다. 반면 두번째 예시에서는 collector를 사용했는데 이는 스트림의 원소들을 객체 하나에 취합한다는 뜻이다. 

## stream 사용 예시

### 일반적인 예시

collect를 사용해서 가장 흔하게 사용되는 단어를 뽑아보면 아래 코드와 같다.

```java
List<String> topTen = freq.keySet().stream()
  .sorted(comparing(freq::get).reversed())
  .limit(10)
  .collect(toList());
```

### toMap 사용 예시

> toMap을 사용하면 스트림 각 원소가 고유한 키에 매핑되어 있을 때 편리하다.

인수 두 개가 있는 toMap은 스트림 원소 다수가 같은 키를 사용할 때 `IllegalStateException`을 던지며 사용되고, 인수 세 개가 있는 toMap은 특정 키에 연관된 원소들 중 하나를 골라 연관 짓는 맵을 만들 때 유용하다. 혹은 충돌이 나면 마지막 값을 취하는 수집기를 만들 수도 있다.

인수가 네 개인 toMap은 마지막 인수로 맵 팩터리를 받고 이 인수로 EnumMap이나 TreeMap처럼 원하는 구현체를 지정할 수 있다.

> toMap의 변종인 `toConcurrentMap`은 병렬 실행된 후 결과로 `ConcurrentHashMap`을 인스턴스로 생성한다.

```java
// 인수 두 개 toMap
private static final Map<String, Operation> stringToEnum = 
  Stream.of(values()).collect(
    toMap(Object::toString, e -> e)
  )

// 인수 세 개 toMap
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
```

### groupingBy 사용 예시

groupingBy는 입력으로 분류 함수를 받고 출력으로는 원소를 분류한 결과인 맵을 담은 collector를 반환한다. groupingBy는 여러 형태로 다중 정의 되어 있다. 

혹은 리스트 외의 값을 갖는 맵을 생성하려면 다운스트림 수집기도 명시해야 한다. 다운스트림 수집기는 해당 카테고리 모든 원소를 담은 스트림으로부터 값을 생성하고, 이 결과로는 groupingBy가 원소의 리스트가 아닌 set을 갖도록 할 수 있다. (toSet()을 넘겼을 경우)

혹은 groupingBy를 사용해서 맵 팩터리도 지정할 수 있다. 다만 점층적 인수 목록 패턴에 어긋단다는 점을 주의해야 한다.
```java
words.collect(groupingBy(word -> alphabetize(word)))
```

### 기타

다운스트림 수집기 전용으로 counting, summing, averaging, summarizing 등 collections 전용으로 사용할 수 있는 stream용 메서드가 있다.

혹은 maxBy, minBy 등 인수로 받은 비교자를 이용해 스트림에서 특정 값을 찾을 수 있다.

joining을 사용하면 문자열 등 CharSequence 인스턴스의 스트림에 적용해서 원소를 연결할 수 있다.
