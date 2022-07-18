---
tags: [Book/Effective Java/Ch9 - 일반적인 프로그래밍 원칙]
title: '[Item 63] 문자열 연결은 느리니 주의하라'
created: '2022-07-18T13:22:00.223Z'
modified: '2022-07-18T13:27:49.173Z'
---

# [Item 63] 문자열 연결은 느리니 주의하라

문자열을 연결하기 위해 +를 사용하면 여러 문자열을 합치기 쉽지만, 너무 자주 사용한다면 성능이 저하된다. 문자열 연결 연산자로 n개 문자열을 잇는다면 그 시간은 n^2에 비례한다.

예를 들어 아래 코드는 item을 전부 하나의 문자열로 연결하므로 양쪽 내용을 모두 복사해야 해서 성능이 굉장히 느려진다. 따라서 StringBuilder를 사용하는 것이 낫다.

```java
public String statement() {
  String result = "";
  for (int i = 0; i < numItems(); i++)
    result += lineForItem(i);
  return result;
}
```

```java
public String statement() {
  StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
  for (int i = 0; i < numItems(); i++)
    b.append(lineForItem(i));
  return b.toString();
}
```

> 많은 문자열을 연결할 때는 문자열 연결 연산자(+)를 피하고, append 메서드를 사용하라.
