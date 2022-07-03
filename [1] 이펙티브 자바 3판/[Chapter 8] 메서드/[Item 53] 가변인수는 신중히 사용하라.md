---
tags: [Book/Effective Java/Ch8 - 메서드]
title: '[Item 53] 가변인수는 신중히 사용하라'
created: '2022-07-03T12:00:19.451Z'
modified: '2022-07-03T12:27:13.064Z'
---

# [Item 53] 가변인수는 신중히 사용하라

가변인수 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다. 가변인수 메서드를 호출하면 **인수의 개수와 길이가 같은 배열을 만들고 인수들을 배열에 저장해 가변인수 메서드에 전달한다.** 그러나 최솟값을 찾는 메서드처럼 인수가 0개만 들어왔을 때 실패해야 하는 경우 아래와 같이 런타임에 실패한다. 이 경우에는 첫번째 인자로 평범한 매개변수를 받아서 해결할 수 있다.

```java
static int min(int... args) {
  if (args.length == 0)
    throw new IllegalArgument("인수가 1개 이상 필요");
  int min = args[0];
  for (int i = 1; i < args.length; i++)
    if (args[i] < min)
      min = args[i];
  return min;
}
```

```java
static int min(int firstArgs, int... args) {
  int min = firstArg;
  for (int arg : args)
    if (arg < min)
      min = arg;
  return min;
}
```

가변인수 메서드는 호출될 때마다 배열을 새로 할당하고 초기화하므로 성능에 좋지 않은 영향을 미친다. 따라서 인수가 0개인 것부터 4개인 것까지 다중정의를 해두고, 일부 메서드만 호출시 가변 인수를 받는 배열을 생성하도록 하는 패턴을 사용할 수 있다.
