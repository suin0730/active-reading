---
tags: [Book/Effective Java/Ch2 - 객체 생성과 파괴]
title: '[Item 7] 다 쓴 객체 참조를 해제하라'
created: '2022-01-04T10:02:08.698Z'
modified: '2022-01-07T01:26:34.807Z'
---

# [Item 7] 다 쓴 객체 참조를 해제하라

Java의 가비지 컬렉터는 다 쓴 객체를 알아서 회수해주지만, 여전히 클라이언트는 메모리를 관리해야 한다.

특정 조건으로 인해 가비지 컬렉터가 다 쓴 객체 메모리를 회수하지 않으면 점차 가비지 컬렉션 활동과 메모리 사용량이 늘어나 성능이 저하될 것이다. 성능 저하가 심해진다면 디스크 페이징이나 OutOfMemoryError를 일으켜 예기치 않게 프로그램이 종료된다. 

메모리 누수가 일어나는 대표적인 원인을 살펴보자.

## 메모리를 직접 관리하는 클래스

아래 `Stack` 클래스는 element 메모리를 직접 관리한다. 나쁜 예시 `pop()` 메서드에서는 스택이 줄어들 때 인덱스가 size보다 큰 배열 요소를 GC하지 못한다. 따라서 좋은 예시 `pop()`과 같이 다 쓴 참조를 null 처리하는 과정이 필요하다.

```java
public class Stack {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;

  public Stack() {
    elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }

  public void push(Object e) {
    ensureCapacity();
    elements[size++] = e;
  }

  // 나쁜 예시
  public Object pop() {
    if (size == 0) throw new EmptyStacException();
    return elements[--size];
  }

  // 새로 들어올 원소를 위한 공간 확보
  private void ensureCapacity() {
    if (elements.length == size)
      elements = Arrrays.copyOf(elements, 2 * size + 1);
  }
}
```
```java
// 좋은 예시
public Object pop() {
  if (size == 0) throw new EmptyStacException();
  Object result = elements[--size];
  element[size] = null; // 참조 해제
  return result;
}
```

그러나 이렇게 객체 참조를 수동으로 null 처리하는 일은 예외적인 경우에만 발생해야 한다. **다 쓴 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것으로, [변수의 범위를 최소로 정의]()했다면 이 일은 자연스럽게 이뤄진다.**

일반적으로 자기 메모리를 스스로 관리하는 클래스라면 클라이언트는 항상 메모리 누수에 주의해야 한다.

## 캐시

객체 참조를 캐시에 넣고 클라이언트가 잊는다면 메모리 누수가 날 수 있다.

만약 캐시 외부에서 키를 참조하는 동안만 엔트리가 살아 있어야 한다면, `WeakHashMap`을 사용해서 캐시를 만들면 된다. 다 쓴 엔트리가 자동으로 제거될 것이다.

일반적으로는 캐시 엔트리 유효 기간을 알 수 없으므로 쓰지 않는 엔트리를 가끔 청소한다. `ScheduledThreadPoolExecutor`같은 백그라운드 스레드를 활용하거나 캐시에 새 엔트리를 추가할 때 메서드를 호출해 부수 작업으로 수행할 수 있다.

## 콜백 리스너

클라이언트가 콜백을 등록만 하고 해지하지 않는 경우 콜백이 쌓여 메모리 누수가 날 수 있다. 

이 경우 콜백을 `WeakHashMap`과 같은 약한 참조로 저장하면 가비지 컬렉터가 메모리를 수거한다.

