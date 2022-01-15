---
tags: [Book/Effective Java/Ch3 - 모든 객체의 공통 메서드]
title: '[Item 11] equals를 재정의하려거든 hashCode도 재정의해라'
created: '2022-01-13T10:18:28.229Z'
modified: '2022-01-15T08:27:11.812Z'
---

# [Item 11] equals를 재정의하려거든 hashCode도 재정의해라

hashCode와 equals에 대한 Object 명세
- equals(Object)가 두 객체를 같다고 판단하면 두 객체의 hashCode는 같은 값을 반환해야 한다.
- equals(Object)가 두 객체를 다르다고 판단해도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

## hashCode를 재정의하지 않을 때 생기는 문제

[Item 10](https://github.com/suin0730/active-reading/blob/main/%5B1%5D%20%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%20%EC%9E%90%EB%B0%94%203%ED%8C%90/%5BChapter%203%5D%20%EB%AA%A8%EB%93%A0%20%EA%B0%9D%EC%B2%B4%EC%9D%98%20%EA%B3%B5%ED%86%B5%20%EB%A9%94%EC%84%9C%EB%93%9C/%5BItem%2010%5D%20equals%EB%8A%94%20%EC%9D%BC%EB%B0%98%20%EA%B7%9C%EC%95%BD%EC%9D%84%20%EC%A7%80%EC%BC%9C%20%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%9D%BC.md)에서 보았듯이 **equals는 물리적으로 다른 두 객체를 논리적으로 같다고 할 수 있다.** 그러나 `Object`에 정의된 기본 hashCode 메서드는 물리적으로 다르나, 논리적으로는 같은 이 객체를 전혀 다르다고 판단해 다른 값을 반환한다.

이전 Item 예시인 `PhoneNumber` 클래스를 생각해보면, 이 클래스는 hashCode를 재정의하지 않았기 때문에 논리적으로 같은 객체에 다른 해시코드를 반환한다. 예를 들어, 아래 코드에서 `m.get()`은 "제니"를 반환하지 않는다.

```java
Map<PhoneNumber, String> m = new Hashmap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");
// 아래 값은 "제니"를 반환하지 않는다.
m.get(new PhoneNumber(707, 867, 5309));
```

## 좋은 해시 함수

> 좋은 해시 함수는 서로 다른 인스턴스에 다른 해시코드를 반환한다.

### hashCode 작성 순서

1. 결과값을 c로 초기화한다. (c는 객체의 첫번째 핵심 필드를 2(1) 방식으로 계산한 해시 코드이다.)
2. 해당 객체의 나머지 핵심 필드 각각에 대해 다음 작업을 수행한다.
   1) 핵심 필드의 해시코드를 계산한다.
       - 기본 타입이면, `Type.hashCode(f)`를 수행한다.
       - 참조 타입이면서 클래스의 equals가 필드의 equals를 재귀적으로 호출한다면, 이 필드의 hashCode를 재귀적으로 호출한다.
       - 배열이라면, 배열 내 핵심 원소 각각을 별도의 필드처럼 다뤄 계산한다. 배열 해시코드를 계산하면 2(2) 방법대로 갱신한다. 모든 원소가 핵심 원소라면 `Arrays.hashCode`를 사용한다.
   2) 해시코드로 reault를 갱신한다.
3. result를 반환한다.

### 전형적인 hashCode 메서드

아래 메서드는 인스턴스의 핵심 필드만을 사용해 간단하게 hashCode를 계산한다. 

```java
@Override public int hashCode() {
  int result = Short.hashCode(areaCode);
  result = 31 * result + Short.hashCode(prefix);
  result = 31 * result + Short.hashCode(lineNum);
  return result;
}
```

### hashCode 작성시 주의할 점

- 동치인 인스턴스에 똑같은 해시코드를 반환해야 한다.
  - AutoValue로 생성한 것이 아니면 단위 테스트를 작성해 이 사실을 검증해야 한다.
- 파생 필드는 해시코드 계산에서 제외해도 된다.
- equals 비교에 사용되지 않는 필드는 **반드시** 제외해야 한다.
- `Object`에서 제공하는 `hash` 메서드를 사용할 수 있다.
  - 속도가 느리므로 성능에 민감하지 않을 때만 사용해야 한다.
- 성능을 높이기 위해 핵심 필드를 생략하면 안된다.
  - 해시테이블 속도가 느려질 수 있다.
- hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 않아야 한다.
  - 클라이언트가 값에 의존하지 않아야 추후에 계산 방식을 바꿀 수 있다.


