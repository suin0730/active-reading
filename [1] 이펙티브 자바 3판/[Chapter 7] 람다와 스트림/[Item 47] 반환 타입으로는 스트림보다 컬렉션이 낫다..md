---
tags: [Book/Effective Java/Ch7 - 람다와 스트림]
title: '[Item 47] 반환 타입으로는 스트림보다 컬렉션이 낫다.'
created: '2022-06-09T13:10:38.099Z'
modified: '2022-06-09T13:41:01.081Z'
---

# [Item 47] 반환 타입으로는 스트림보다 컬렉션이 낫다.

## 기존에 사용하던 반환 타입

- 일반적으로는 컬렉션 인터페이스 사용
- 반환된 원소 시퀀스가 일부 Collection 메서드를 구현할 수 없을 때는 Iterable 사용
- 원소가 기본 타입이거나 성능이 중요하면 배열 사용

## 스트림 등장 후의 반환 타입

스트림은 반복을 지원하지 않아서 스트림과 반복을 알맞게 조합해야 좋은 코드가 나온다. 어댑터 메서드를 사용해 `Stream<E>`를 `Itreable<E>`로 적절히 중개해준다면 별도 형변환 없이 스트림을 forEach 문으로 반복할 수 있다.

```java

public static <E> Iterable<E> iterableOf(Stream<E> stream) {
  return stream::iterator;
}
for (ProcessHandle p : iterableOf(ProcessHandle.allProcesses())) {
  // 각 프로세스 처리
}
```

반대로 API가 itreable만 반환하면 스트림 파이프라인에서 처리하기 어려울 것이다. 아래처럼 객체 시퀀스를 반환하는 메서드를 작성해서 스트림 파이프라인을 사용하려는 사람과 반복문에서 쓰려는 사람 모두를 배려할 수 있다.

```java
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
  return StreamSupport.stream(iterable.spliterator(), false);
}
```

## 반환할 시퀀스가 클 때

Collection 인터페이스는 Iterable의 하위 타입이고 stream 메서드도 지원하므로 Collection 혹은 그 하위 타입을 사용하는게 합리적인 방법이다. 그러나 반환하는 시퀀스의 크기가 크다면 **컬렉션을 반환한다는 이유만으로 덩치 큰 시퀀스를 메모리에 올리면 안된다.**

반환할 시퀀스가 클 때는 표현을 간결하게 할 수 있는 전용 컬렉션을 구현할 수도 있다.

- 주어진 집합의 멱집합을 반환하기: 멱집합을 구성하는 각 원소 인덱스를 비트 벡터로 사용
- 입력 리스트의 연속적인 부분리스트를 모두 반환하기: 리스트의 프리픽스의 서픽스에 빈 리스트를 추가하는 방식으로 구현

> 반복을 사용하는게 더 자연스러운 상황에 스트림으로 코드를 작성하거나 어댑터를 사용하면 클라이언트 코드르 ㄹ복잡하게 만들고 느려진다.


