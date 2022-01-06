---
tags: [Book/Effective Java/Ch2 - 객체 생성과 파괴]
title: '[Item 6] 불필요한 객체 생성을 피하라'
created: '2022-01-04T10:01:43.711Z'
modified: '2022-01-06T14:53:20.443Z'
---

# [Item 6] 불필요한 객체 생성을 피하라

똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는게 나을 때가 있다. 아래는 객체를 재사용하는 방향으로 개선할 수 있는 코드 예시이다.

## 동일한 문자열 반복 생성

아래 코드는 "IU is the best"라는 문자열 객체를 String 생성자에 넘겨 완전히 똑같은 객체를 하나 더 생성한다. 반복문이나 자주 사용되는 메서드에 아래 코드가 있다면, 쓸모없는 객체가 대량으로 만들어질 것이다. 좋은 예시는 새로운 인스턴스를 매번 만드는 대신 하나의 String 인스턴스만 사용하기 때문에 **같은 VM에서 똑같은 문자열을 사용하는 모든 코드가 같은 객체를 재사용하는 것이 보장된다.**

```java
// 나쁜 예시
String s = new String("IU is the best");

// 좋은 예시
String s = "bikini";
```

## 비싼 객체 반복 생성

생성 비용이 비싼 객체인지 매번 명확히 알 수는 없지만, 성능이 갑자기 떨어진다면 비싼 객체를 생성했는지 의심할 수 있다. 정규표현식을 사용해 문자열 매칭을 확인하는 아래 코드가 그 예시이다. `String.matches`는 정규표현식으로 문자열 형태를 확인하는 쉬운 방법이지만 정규표현식에 사용하는 `Pattern` 인스턴스는 한 번 사용하고 버려지기 때문에 불변으로 생성하는 것이 좋다. `Pattern`을 static final로 끄집어내 이름을 지으면 코드 의미도 훨씬 잘 드러난다.

```java
// 나쁜 예시
static boolean isRomanNumeral(String s) {
  return s.matches( ... );
}

// 좋은 예시
static boolean isRomanNumeral(String s) {
  private static final Patten ROMAN = Pattern.compile( ... );

  static boolean inRomanNumeral(String s) {
    return ROMAN.matcher(s).matches();
  }
}
```

## 불필요한 어댑터

일반적으로는 객체가 불변이면 재사용해도 안전하다. 그러나 어댑터(뷰)와 같이 제2의 인터페이스 역할을 하는 객체는 뒷단 객체 외에는 관리할 상태가 없어서, 객체 하나당 어댑터 하나씩만 만들어지면 된다.

예를 들어, `Map` 인터페이스의 `keySet` 메서드는 `Map` 객체 안의 키 전부를 담은 `Set`뷰를 반환한다. `Set`은 단순히 뷰 역할을 하므로, 매번 새로운 `Set` 인스턴스가 만들어질 것이라는 추측과 달리 매번 같은 객체를 반환해도 상관없다. 반환한 객체 중 하나를 수정하면 다른 모든 객체가 따라서 마뀌고 모두가 똑같은 `Map`을 대변하기 때문디ㅏ.

## 오토 박싱

0부터 `Integer.MAX_VALUE`까지의 합을 출력하는 아래 프로그램은 long에서 Long으로의 언박싱 때문에 불필요한 Long 인스턴스가 만들어진다. **박싱된 기본 타입 보다는 기본 타입을 사용하고, 의도치 않은 오토박싱을 주의해야 한다.**

```java
private static long sum() {
  Long sum = 0;
  for (long i = 0; i <= Integer.MAX_VALUE; i++)
    sum += i;

  return sum
}
```


