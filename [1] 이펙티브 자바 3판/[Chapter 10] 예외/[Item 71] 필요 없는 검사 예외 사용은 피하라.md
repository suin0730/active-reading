---
tags: [Book/Effective Java/Ch10 - 예외]
title: '[Item 71] 필요 없는 검사 예외 사용은 피하라'
created: '2022-07-26T14:25:34.508Z'
modified: '2022-07-26T14:50:00.734Z'
---

# [Item 71] 필요 없는 검사 예외 사용은 피하라

## 검사 예외의 문제

문제를 프로그래머가 처리해 프로그램 안정성을 높이고자 할 때는 checked exception이 유용하다. API를 올바르게 사용해도 발생하는 예외이거나 API를 사용하는 클라이언트가 예외에 의미 있는 처리를 할 수 있다면 checked exception을 사용해도 무방하다.

그러나 checked exception은 catch 블록을 두어 예외를 처리하거나 전파해야 하고, 스트림에서 사용할 수 없기 때문에 API 호출자에게 부담을 준다. 특히 한 메서드가 하나의 checked exception을 던지는 경우에는 해당 예외를 위해 try문을 작성해야 하고 스트림에서 사용할 수 없게 되기 때문에 단점이 더 크다고 할 수 있다.

## 검사 예외를 우회하는 법

메서드 동작에 따라 적절한 결과 타입을 담은 Optional 객체를 반환하는 방법으로 검사 예외를 우회할 수 있다. 그러나 Optional을 반환할 때는 예외가 발생한 이유를 알려주는 메시지 등을 담을 수 없다.

혹은 아래 예외처럼 검사 예외를 반환하는 메서드를 비검사 예외로 리팩터링할 수 있다. 그러나 `actionPermitted`는 상태 검사 메서드로, 외부 요인에 의해 바뀔 수 있다면 사용하는 것이 부적절하다.

```java
// 리팩터링 전
try {
  obj.action(args);
} catch (TheCheckedException e) {
  // 예외 처리
}

// 리팩터링 후
if (obj.actionPermitted(args)) {
  obj.action(args);
} else {
  // 예외 처리
}
```
