---
tags: [Book/Effective Java/Ch10 - 예외]
title: '[Item 72] 표준 예외를 사용하라'
created: '2022-07-28T15:46:17.482Z'
modified: '2022-07-28T15:56:13.365Z'
---

# [Item 72] 표준 예외를 사용하라

표준 예외를 사용하면 다른 사람도 해당 코드를 읽거나 유지보수하기 쉬워진다는 장점이 있다. 또한 예외 클래스 수가 적을수록(따로 정의한 예외가 적을수록) 메모리 사용량과 클래스 적재 시간이 줄어든다.

## 자주 사용되는 표준 예외

- IllegalArgumentException: 클라이언트가 비즈니스 로직상 받아들일 수 없는 인자를 넘길 때 사용한다.
- IllegalStateException: 클라이언트가 객체를 사용하기에 적합하지 않은 상태일 때 사용된다. 객체가 초기화되지 않았을 때 사용하려 하면 해당 예외가 발생한다.
- NullPointerException: null값을 허용하지 않은 메서드에 인자로 null을 넘기면 해당 예외가 발생한다.
- ConcurrentModificationException: 한 스레드에서 재사용하기 위한 목적으로 만든 객체를 여러 스레드에서 사용하려 할 때 발생하는 예외이다. (경고 수준에 불가하긴 함..)
- UnsupportedOperationException: 구현하려는 인터페이스의 메서드 일부를 지원하지 않는 경우 발생한다. (원소를 넣을 수만 있는 List에 remove를 호출한 경우)

> 예외의 쓰임들은 서로 배타적이지 않기 때문에 맥락을 잘 파악하고 표준 예외를 참고해 적절한 예외를 발생시켜야 한다.

