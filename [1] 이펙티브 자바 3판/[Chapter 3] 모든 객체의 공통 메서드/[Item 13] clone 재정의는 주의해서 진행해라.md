---
tags: [Book/Effective Java/Ch3 - 모든 객체의 공통 메서드]
title: '[Item 13] clone 재정의는 주의해서 진행해라'
created: '2022-01-13T10:18:57.077Z'
modified: '2022-01-19T15:39:48.733Z'
---

# [Item 13] clone 재정의는 주의해서 진행해라

## Cloneable은 무엇인가?

Cloneable은 복제해도 되는 클래스임을 명시하기 위한 용도의 인터페이스지만, clone 메서드가 Cloneable이 아닌 Object이므로 Cloneable을 구현하는 것만으로는 외부 객체에서 clone 메서드를 호출할 수 없다.

**Cloneable 인터페이스는 Object의 protected인 메서드인 clone의 동작 방식을 결정한다.** Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 내부적으로 객체 필드 하나하나를 복사한 객체를 반환하고, Cloneable을 구현하지 않는 클래스 인스턴스에서 clone을 호출하면 `CloneNotSupportException`을 던진다.

문제는 clone 메서드의 일반 규약이 허술하다는 것이다. 이 허술한 규약만을 따르면 생성자를 호출하지 않고도 객체를 생성하는, 위험하고 모순적인 매커니즘이 탄생한다.

## 가변 상태를 참조하지 않는 클래스용 clone 메서드

가변 상태를 참조하지 않는 클래스용 clone 메서드를 보자. 이 메서드가 동작하려면 PhoneNumber의 클래스 선언에 Cloneable을 구현하고 clone 메서드가 PhoneNumber를 반환하도록 형변환을 해주어야 한다. 

먼저 super.clone을 호출하면 원본의 완벽한 복제본을 가질 수 있을 것이다. 이때 모든 필드가 기본 타입이거나 불변 객체를 참조한다면 이 객체는 완벽히 clone 메서드에 기대하는 기능을 수행하므로 손볼 것이 없다.

```java
@Override public PhoneNumber clone() {
  try {
    return (PhoneNumber) super.clone();
  } catch (CloneNotSupportedException e) {
    throw new AssertionError();
  }
}
```

쓸데없는 복사를 지양한다는 관점에서 보면 불변 클래스는 굳이 clone 메서드를 제공하지 않는 것이 좋긴 하다.

## 가변 상태를 참조하는 클래스용 clone 메서드

반대로 가변 객체도 참조하는 Stack 클래스를 생각해보자. clone 메서드가 단순하게 super.clone을 반환하면 **원시 타입인 size 필드는 올바른 값을 갖겠지만 element 필드는 원본 Stack 인스턴스와 같은 배열을 참조할 것이다.** 결론적으로 clone 메서드는 원본 객체에 아무런 해를 끼치지 않으면서 복제된 객체의 불변식을 보장해야 한다. 

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

  public Object pop() {
    if (size == 0) throw new EmptyStacException();
    Object result = elements[--size];
    element[size] = null; // 참조 해제
    return result;
  }

  // 새로 들어올 원소를 위한 공간 확보
  private void ensureCapacity() {
    if (elements.length == size)
      elements = Arrrays.copyOf(elements, 2 * size + 1);
  }
}
```

### clone 메서드 재귀 호출

이 문제를 해결하려면 clone 메서드 내부에서 재귀적으로 clone 메서드를 호출해야 한다. 따라서 잘 작성된 Stack의 clone 메서드는 아래와 같다.

그러나 한편, elements 필드가 final이라면 여기에는 새로운 값을 할당할 수 없으므로 아래 코드가 동작하지 않는다. 그래서 클래스를 복제 가능하게 만들기 위해 일부 필드에서 final을 제거해야 할 수도 있다.

> 따라서 Cloneable 아키텍처는 '가변 객체를 참조하는 필드는 final로 선언하라'는 일반 용법과 충돌한다.

```java
@Override public Stack clone() {
  try {
    Stack result = (Stack) super.clone();
    result elements = elements.clone();
    return result;
  } catch (CloneNotSupportedException e) {
    throw new AssertionError();
  }
}
```

### 원본 객체의 상태를 다시 생성하는 고수준 메서드 호출

해시테이블에서는 clone을 재귀적으로 호출하는 것만으로 충분하지 않을 수 있다. 해시테이블 내부는 버킷들의 배열이고, 각 버킷은 키-값 쌍을 담는 연결 리스트의 첫번째 엔트리를 참조한다. Stack에서처럼 단순히 버킷 배열의 clone을 재귀적으로 호출한다면, **복제본은 자신만의 버킷 배열을 갖지만 원본과 같은 연결 리스트를 참조하므로 원본과 복제본 모두 예상치 못한 방식으로 동작한다.** 

이를 해결하려면 각 버킷을 구성하는 연결 리스트를 복사해야 한다. 코드에는 나와있지 않지만 private 클래스인 HashTable.Entry는 deepCOpy를 지원하도록 보강되었다. Entry의 deepCopy가 자신이 가리키는 연결 리스트 전체를 복사하기 위해 자신을 재귀적으로 호출한다면 원본 객체와 완전히 같은 객체를 생성할 수 있을것이다.

```java
@Override public HashTable clone() {
  try {
    HashTable result = (HashTable) super.clone();
    result.buckets = new Entry[buckets.length];
    for(int i = 0; i < buckets.length; i++){
      if(buckets[i] != null){
        result.buckets[i] = buckets[i].deepCopy();
      }
    }
    return result;
  } catch (CloneNotSupportedException e) {
    throw new AssertionError();
  }
}
```

```java
Entry deepCopy() {
  return new Entry(key, value, next == null ? null : next.deepCopy());
}
```

## 기타 clone 메서드 관련 주의점

### 재정의될 수 있는 메서드

생성자와 같이, clone 메서드도 재정의될 수 있는 메서드를 호출하지 않아야 함다. 만약 clone이 하위 클래스에서 재정의한 메서드를 호출한다면, 하위 클래스는 복제되는 과정에서 자신의 상태를 올바르게 바꿀 수 있는 기회 없이 잘못된 상태를 가진다. 

### Exception

Object의 clone은 CloneNotSupportedException을 던지지만 clone을 재정의한 메서드는 throws 절을 없애야 더 편하게 사용할 수 있다.

### 상속

일반적으로 상속용 클래스는 Cloneable을 구현하면 안되지만, 하위 클래스가 구현 여부를 선택하도록 하려면 clone 메서드를 구현해 protected로 두고 CloneNotSupportedException을 던지게 할 수 있다.

### 멀티스레드

스레드 안전한 클래스를 작성하려면, clone 메서드를 또다른 방식으로 재정의해야 한다.

## 복사 생성자, 복사 팩터리

객체의 복사가 필요하면 언어 모순적이고 위험한 객체 생성 방법인 Cloneable을 구현하기보다 복사 생성자, 복사 팩터리를 사용하는 것이 좋다. 심지어 이 방법들은 해당 클래스가 구현한 인터페이스 타입 인스턴스를 인수로 받을 수 있다. 이 방법을 이용하면 클라이언트는 원본의 구현 타입에 상관없이 복제본 타입을 선택할 수 있다.
