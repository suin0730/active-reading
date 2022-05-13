---
tags: [Book/Effective Java/Ch5 - 제네릭]
title: '[Item 32] 제네릭과 가변인수를 함께 쓸 때는 신중하라'
created: '2022-05-13T13:44:45.871Z'
modified: '2022-05-13T14:20:14.416Z'
---

# [Item 32] 제네릭과 가변인수를 함께 쓸 때는 신중하라

가변인수 메서드와 제네릭을 함께 쓴다면 부작용이 생길 수 있다.

## 가변인수 방식의 문제점

가변인수 메서드를 사용하면 메서드에 넘기는 인수 개수를 클라이언트가 조절할 수 있다. 이 메서드를 호출하면 가변인수를 담기 위한 배열이 자동으로 만들어지는데, 힙 오염을 방지하기 위해서 이 배열은 외부로부터 감추어야 하지만 종종 클라이언트에 노출시키는 실수를 할 수도 있다. 

거의 모든 제네릭과 매개변수화 타입은 실체화되지 않으므로 컴파일타임보다 런타임시 타입 관련 정보를 더 적게 담고 있다. 따라서 메서드를 선언할 때 실체화 불가 타입을 사용하면 컴파일러가 경고를 띄운다. 예를 들어 매개변수화 타입 변수가 다른 타입 객체를 참조하면 힙 오염이 발생할 수 있기 때문이다. **이러한 경우에는 컴파일러가 생성한 자동 형변환이 실패할 수 있으므로, 제네릭 타입 시스템이 약속한 타입 안정성이 흔들리게 된다.**

아래 코드는 매개변수화 타입의 변수가 타입이 다른 객체를 참조해 힙 오염이 발생한 상황으로, 제네릭 varargs 배열 매개변수에 값을 저장하는 것이 안전하지 않다는 것을 알 수 있다.

```java
static void dangerout(List<String>... stringLists) {
  List<Integer> intList = List.of(42);
  Object[] objects = stringLists;
  objects[0] = intList;               // 힙 오염
  String  s = stringLists[0].get(0);  // ClassCastException
}
```
## @SaveVarargs

자바 7 전에는 제네릭 가변인수 메서드 호출자에서 발생하는 경고를 작성자가 처리할 수 없었다. 그래서 이런 경고를 그대로 두거나 호출 부분마다 `@SuppressedWarning`을 달아야 하는 부작용이 생겼다. 자바 7부터 등장한 `@SavaVarargs`를 달면 메서드 작성자가 호출측에서 발생하는 경고를 숨길 수 있다. **메서드 작성자가 이 메서드가 타입 안전함을 보장함으로써 컴파일러는 타입 경고를 없애게 된다.**

> 메서드가 타입 안전한지 알려면, 1)메서드가 배열에 아무것도 저장하지 않고(값을 덮어쓰지 않고) 2)그 배열 참조가 밖으로 노출되지 않아야 한다.

## 제네릭과 가변인자가 충돌하는 부분

아래 메서드는 배열 참조를 반환해 외부에 노출한다는 점에서 문제가 있다. 이 메서드가 반환해야 하는 배열 타입은 컴파일타임에 결정되는데, 그 시점에는 **컴파일러에 충분한 정보가 주어지지 않아 타입을 잘못 판단할 수 있다.** 타입을 잘못 판단해서 발생한 힙 오염은 호출 메서드 측 콜스택으로까지 전이될 수 있다.

`pickTwo`는 T는 `String`이지만 컴파일러는 가변 인자로 무엇이든 받기 위해 `Object[]`를 만들고 함수는 그 배열 참조를 그대로 반환한다. main 메서드에서 `pickTwo`를 호출하면 반환값은 `Object[]`이고 이를 `String[]`으로 변환하는 코드가 컴파일러에 의헤 추가된다. **제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않으므로 주의해야 한다.**

```java
static <T> T[] pickTwo(T a, T b, T c) {
  switch(ThreadLocalRandom.current().nextInt(3)) {
    case 0: return toArray(a, b);
    case 1: return toArray(b, c);
    case 2: return toArray(c, a);
  }
}

public static void main(String[] args) {
  String[] attribute = pickTwo("아", "이", "유");
}
```

## 제네릭과 가변인자를 안전하게 사용하는 방법

안전하지 않은 varargs 메서드는 절대 작성해서는 안된다.

```java
@SafeVarargs
static<T> List<T> flatten(List<? extends T>... lists) {
  List<T> result = new ArrayList<>();
  for (List<? extends T> list : lists)
    result.addAll(list);
  return result;
}
```

> @SafeVarargs는 재정의할 수 없는 메서드에만 달아야 한다. 재정의한 메서드가 안전할지는 보장할 수 없기 때문이다.


## 배열보다는 리스트를 사용해보자

매개변수 선언만 수정해서 varargs 매개변수를 List 매개변수로 바꿀 수도 있다. 정적 팩터리 메서드인 `List.of()`를 사용해서 `flatten` 메서드에 임의 개수의 인수를 넘길 수 있다. 이 방식을 사용하면 우리의 판단이 아닌, 기존 메서드에 달린 `@SafeVarargs`만으로도 타입 안정성을 판단할 수 있다. 배열 없이 제네릭만 사용한다면 더욱 안전하게 코드를 작성할 수 있다.

```java
@SafeVarargs
static<T> List<T> flatten(List<List<? extends T>> lists) {
  List<T> result = new ArrayList<>();
  for (List<? extends T> list : lists)
    result.addAll(list);
  return result;
}
```

> 가변인수 기능은 배열을 노출해 추상화가 완벽히 이뤄지지 못하고, 배열과 제네릭은 타입 규칙이 서로 다르다. 

