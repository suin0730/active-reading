---
tags: [Book/Effective Java/Ch8 - 메서드]
title: '[Item 52] 다중정의는 신중히 사용하라'
created: '2022-06-21T08:18:11.831Z'
modified: '2022-06-27T14:04:01.145Z'
---

# [Item 52] 다중정의는 신중히 사용하라


## 오버로딩과 오버라이딩한 메서드의 차이

아래 코드에서 `classify` 메서드를 호출하면 실제로는 collection만 세 번 출력된다. 클라이언트는 오버로딩한 `classify`가 각 collections 원소 타입에 따라 호출되기를 기대하지만 **세 메서드 중 어느 메서드를 호출할지는 컴파일타임에 정해지기 때문이다.** 이렇게 다중정의한 메서드가 정적으로 선택되는 것과 달리, 오버라이드한 메서드는 런타임에 동적으로 선택된다.

```java
public class CollectionClassifier {
  public static String classify(Set<?> s) {
    return "set";
  }

  public static String classify(List<?> lst) {
    return "list";
  }

  public static String classify(Collection<?> c) {
    return "collection";
  }

  public static void main(String[] args) {
    Collection<?>[] collections = {
      new HashSet<String>(),
      new ArrayList<BigInteger>(),
      new HashMap<String, String>().values()
    };

    for (Collection<?> c : collections)
      System.out.println(classify(c));
  }
}
```

이 문제를 해결하고 원하는 대로(런타임에 동적으로 타입을 지정해서 각기 다른 결과를 내기) 구현하고 싶다면 모든 `classify` 메서드를 하나로 합친 뒤 `instanceof`로 명시적으로 타입을 검사하면 된다. 그러나 재정의와 다중정의가 직관적으로 생각했을때 유사하게 동작하지 않기 때문에 **혼동을 일으킬 수 있는 코드는 작성하지 않는 게 좋다.** 보수적으로 코드를 작성하려면 1)매개변수 수가 같은 다중정의를 만들지 않거나 2)다중정의하는 대신 메서드 이름을 다르게 지으면 된다.

혹은 `ObjectOutputStream` 클래스처럼 다중정의를 사용하지 않고, 모든 메서드에 다른 이름을 지어주는 방법을 사용한다면 메서드 통일성을 지키면서 클라이언트가 메서드를 잘못 사용하는 것을 방지할 수 있다. (writeBoolean, writeInt, writeLong)

## 다중정의가 혼란을 일으킬 수 있는 다른 상황

### radically different

radically different는 근본적으로 다른 타입을 뜻한다. 두 타입이 null이 아닌 값을 서로 어느 쪽으로도 형변환할 수 없다면 근본적으로 다르다고 할 수 있고, 런타임에서도 어떤 오버로딩 메서드를 호출할지 결정할 수 있었다. (ArrayList에서 int를 받는 생성자와 Collection을 받는 생성자는 헷갈릴 일이 없었다.)

그러나 java5부터 오토박싱이 도입되면서 근본적으로 다른 매개변수 타입도 오토박싱되어 의도한 바와 다른 메서드를 호출하는 일이 생겼다. (list.remove(int index)와 list.remove(Object)) List<E> 인터페이스가 remove(Object)와 remove(int)를 다중정의해 취약해졌다.

### 람다와 메서드 참조

넘겨진 인수가 같고 둘 다 Runnable을 받아도 아래 두 코드는 다른 결과를 낸다. 2번은 적절하지 않은 println과 적절하지 않은 submit을 호출해 오버로딩 resolution 알고리즘을 적용하지 못했다. **중요한 것은 다중정의된 메서드가 함수형 인터페이스를 인수로 받을 때, 서로 다른 함수형 인터페이스라도 위치가 같으면 혼란을 줄 수 있다는 것이다.**

```java
// 1번. Thread의 생성자 호출
new Thread(System.out::println).start();
// 2번. ExcutorService의 submit 메서드 호출
ExecutorService exec = Executors.newCachedThreadPool();
exec.submit(System.out::println);
```

> 상대적으로 더 특수한 다중정의 메서드에서 일반적인 메서드를 호출한다면 어떤 오버로딩 메서드가 호출되는지 몰라도 수행하는 기능이 같으므로 신경쓸 필요가 없다. 그러나 String 클래스의 valueOf(char[]), valueOf(Object)는 수행하는 역할이 매우 다르므로 주의해야 한다.
