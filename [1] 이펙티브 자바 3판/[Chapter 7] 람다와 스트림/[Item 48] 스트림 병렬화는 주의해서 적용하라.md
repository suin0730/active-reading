---
tags: [Book/Effective Java/Ch7 - 람다와 스트림]
title: '[Item 48] 스트림 병렬화는 주의해서 적용하라'
created: '2022-06-10T07:31:54.130Z'
modified: '2022-06-10T08:41:17.513Z'
---

# [Item 48] 스트림 병렬화는 주의해서 적용하라

## 자바의 동시성 프로그래밍 지원

- 릴리즈 시점: 스레드, 동기화, wait/notify 지원
- 자바 5: `java.util.concurrent` 라이브러리, `Excutor` 프레임워크 지원
- 자바 7: parallel decomposition, fork-join 패키지 지원
- 자바 8: parellel 메서드로 파이프라인을 병렬 실행할 수 있는 스트림 지원

## 동시성을 지키기 위해 중요한 것

자바에서 동시성 프로그램을 작성하기는 점차 쉬워지고 있지만, 프로그램을 올바르게 작성하는 것은 어려운 일이다. 특히 **안전성과 응답 가능 상태를 유지하는 것이 중요하다.** 아래 코드는 Stream.iterate와 limit를 모두 사용하므로 스트림 라이브러리가 내부적으로 코드를 병렬화하는 방법을 찾지 못해 성능이 오히려 느려질 수 있다.

```java
public static void main(String[] args) {
  primes().map(p ->TWO.pow(p.intValueExact()).subtract(ONE))
    .filter(mersenne -> mersenne.isProbablePrime(50))
    .limit(20)
    .forEach(System.out::println);
}
static Stream<BigInteger> primes() {
  return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}
```

## 병렬 처리에 적합한 자료구조

반면 데이터의 크기를 원하는대로 손쉽게 나눌 수 있는 특징을 가진 자료구조는 병렬화 연산을 하기에 적합하다. (예시로는 ArrayList, HashSet, HashMap, ConcurrentHashMap, int, long 범위 등이 있다.) 이런 경우에는 Stream이나 Iterator의 Spliterable로 자료를 나눌 수 있다. 또한 이 자료구조들은 데이터의 참조지역성이 높아서 대용량 데이터 처리를 하는 bulk 연산을 병렬화할 때 더 큰 이점을 갖는다.

## 병렬 처리에 적합한 종단 연산

스트림 파이프라인 연산에서 종단 연산은 비교적 병렬 연산 시간 중 큰 비중을 차지한다. 종단 연산 중 병렬화에 가장 적합한 것은 파이프라인에서 들어온 모든 연산을 합치는 축소(reduction) 연산이다. Stream의 reduce 메서드 중 하나, max, min, count, sum이나 조건에 맞는 값들 중 하나를 반환하는 anyMatch, allMatch 등도 병렬 연산에 적합하다.

> 가변 축소를 수행하는 Stream의 collection 메서드는 컬렉션들을 합치는 부담이 있어 병렬 연산에 적합하지 않다.

## 스트림을 잘못 병렬화하는 경우

병렬화 연산을 잘못 하용해서 예상했던 것과 다른 방식으로 동작한다면 안전 실패(safety failure)라 한다. 이는 성능이 나빠지거나 오동작하는 것을 말하는데 스트림 명세에 따르면 reduce 연산에 건네지는 1)누적, 결합 함수가 결합 법칙을 만족하고 2)간섭받지 않고 3)상태를 갖지 않아야 한다.

위에서 언급한 스트림 명세를 모두 만족하고 원하는 결과가 나왔다고 해도 **파이프라인이 수행하는 진짜 작업이 병렬화에 드는 추가 비용을 상쇄하지 못한다면 병렬화를 사용하는 효과가 없을 수도 있다.**

스트림 병렬화는 성능 최적화만을 위한 수단이므로 코드를 변경했다면 반드시 성능 테스트를 거쳐야 한다. 잘못 작성한 병렬화 코드는 시스템의 다른 부분에까지 영향을 줄 수 있지만, 적합한 환경에서 수행한다면 parellel 메서드 호출 하나로 코어 배수만큼의 성능 이점을 볼 수 있다. 

> 스트림 병렬화를 잘못 사용하면 프로그램 오동작이나 성능 하락을 낳기 때문에 성능이 개선될 것이라는 확신과 사전 모니터링이 없다면 적용하지 않는 것이 낫다.

