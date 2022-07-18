---
tags: [Book/Effective Java/Ch9 - 일반적인 프로그래밍 원칙]
title: '[Item 65] 리플렉션보다는 인터페이스를 사용하라'
created: '2022-07-18T13:35:44.203Z'
modified: '2022-07-18T14:03:52.811Z'
---

# [Item 65] 리플렉션보다는 인터페이스를 사용하라

## 리플렉션을 사용하면 할 수 있는 것

- 임의의 클래스에 접근해 생성자, 메서드, 피드에 접근하거나 조작할 수 있다.
- 인스턴스로 클래스의 멤버 이름, 필드 타입, 메서드 시그니처를 가져올 수 있다.
- 컴파일 당시에 존재하지 않던 클래스도 이용할 수 있다.

## 리플렉션을 사용했을 때의 단점

- 컴파일 타임에 수행되는 검사로써 얻을 수 있었던 이점을 누릴 수 없다. (예외 검사, 접근불가능한 메서드 등)
- 코드가 지저분하고 장황해진다.
- 리플렉션을 이용하면 일반 메서드를 호출하는 것보다 훨씬 느리다. (약 10배)

만약 컴파일타임에 사용할 수 없는 클래스를 사용해야 하는 프로그램은 해당 클래스 대신 적절한 인터페이스나 상위 클래스를 사용하는 것을 고려해야 한다. 이 경우에 리플렉션은 인스턴스를 생성하는 데만 사용하고 인터페이스 등으로 참조해서 사용해야 한다.

## 리플렉션을 사용한 예시

아래 예시는 명령줄 첫번째 인수로 클래스 명을 받고, Set<String> 인터페이스로 인스턴스를 생성하는 예제이다. 그러나 리플렉션을 사용함으로써 너무 다양한 종류의 예외를 던질 가능성이 생겼고, 코드 양이 너무 많아진다.

```java
public static void main(String[] args) {
  Class<? extends Set<String>> cl = null;
  try {
    cl = (Class<? extends Set<String>>) Class.forName(args[0]);
  } catch (ClassNotFoundException e) {
    fatalError("클래스를 찾을 수 없습니다.");
  }

  Constructor<? extends Set<String>> cons = null;
  try {
    cons = cl.getDeclaredConstructor();
  } catch (NoSuchMethodException e) {
    fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
  }

  Set<String> s = null;
  try {
    s = cons.newInstance();
  } catch (IllegalAccessException e) {
    fatalError("생성자 접근 불가");
  } catch (InstantiationException e) {
    fatalError("클래스 인스턴스화 불가");
  } catch (InvocationTargetException e) {
    fatalError("생성자가 예외를 던짐");
  } catch (ClassCastException e) {
    fatalError("Set을 구현하지 않은 클래스");
  }

  s.addAll(Arrays.asList(args).subList(1, args.length));
  System.out.println(s);
}
```

> 리플렉션은 드물게 런타임에 존재하지 않을 수 있는 클래스, 메서드 등과의 의존성을 관리하는 데에 쓰이기도 한다. 이 경우에는 새 클래스나 메서드가 런타임에 존재하지 않을 수 있다는 것을 감안해야 한다.
