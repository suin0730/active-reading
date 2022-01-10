---
tags: [Book/Effective Java/Ch2 - 객체 생성과 파괴]
title: '[Item 8] finalizer와 cleaner 사용을 피하라'
created: '2022-01-04T10:02:28.998Z'
modified: '2022-01-10T01:51:32.111Z'
---

# [Item 8] finalizer와 cleaner 사용을 피하라

자바에서는 두가지 객체 소멸자를 제공한다.
- `finalizer`: 예측 불가능하고, 상황에 따라 위험할 수 있어 자바 9에서 deprecated API로 지정되었다.
- `cleaner`: `finalizer`를 대체하는 덜 위험한 API이지만, 여전히 예측할 수 없고 느리다.

## 사용하지 말아야 할 이유

### 수행 시점

`finalizer`와 `cleaner`가 수행되기까지 얼마나 걸릴지 알 수 없으므로 제때 실행되어야 하는 작업을 맡기면 오류를 낼 수 있다. 객체 소멸자가 동작하는 시점은 GC 알고리즘에 달렸으며 그 구현 방법에 따라 천차만별이므로 애초에 사용하지 않는게 좋다.

만약 클래스에 `finalizer`를 사용한다면 인스턴스 자원 회수가 지연되어 OutOfMemory를 내며 프로그램이 죽을 수 있다.

### 수행 여부

`finalizer`와 `cleaner`는 객체 소멸 수행 여부를 보장하지 않으므로 데이터베이스 공유 락 해제와 같이 상태를 수정해야 하는 작업에는 사용하면 안된다.

### 보완 클래스가 제 역할을 못함

`System.gc`나 `System.runFinalization` 메서드를 사용하면 `finalizer`와 `cleaner`가 실행될 가능성을 높일 수는 있으나 보장하진 않아 사용하지 않는게 낫다.

### 낮은 성능

`finalizer`가 객체를 소멸시키면 AutoClosable 객체를 사용해서 가비지 컬렉터가 수거하도록 할 때보다 50배 정도 느리다.

### 보안 취약

객체 생성을 막으려면 원래는 생성자에서 예외를 던지면 되지만 `finalizer`는 다른 방식으로 동작한다. 만약 객체 직렬화 화정에서 예외가 발생하면, 생성되던 객체에서 `finalizer`가 수행될 수 있고 일그러진 객체가 만들어진다. final 클래스는 하위 클래스를 만들 수 없으니 만약 final이 아닌 클래스를 공격에 안전하게 만들려면 아무 일도 하지 않는 `finalizer` 메서드를 만들고 final로 선언해야 한다.

## 대안

파일이나 스레드 등 종료해야 하는 자원을 담고 있는 클래스에서 `finalizer`, `cleaner`를 대체하는 방법은 `AutoClosable을` 구현하고 `close` 메서드를 호출하는 것이다.  [(Item 9)](https://github.com/suin0730/active-reading/blob/main/%5B1%5D%20%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%20%EC%9E%90%EB%B0%94%203%ED%8C%90/%5BItem%209%5D%20try-finally%EB%B3%B4%EB%8B%A4%EB%8A%94%20try-with-resources%EB%A5%BC%20%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC.md)

## 그럼 `finalizer`와 `cleaner`는 대체 언제 쓰는가?

### close 메서드 안전망

자원을 가진 클라이언트가 `close` 메서드를 호출하지 않는 경우를 대비해, `cleaner`와 `finalizer`를 객체를 수거하는 안전망으로 사용할 수 있다.

### 네이티브 피어와 연결된 객체

자바 객체가 네이티브 메서드를 통해 네이티브 객체를 생성하면 가비지 컬렉터는 이 객체의 존재를 알지 못한다. 네이티브 객체를 회수하려면 성능 저하를 감수하고 `close` 메서드를 사용할 수 있다.


