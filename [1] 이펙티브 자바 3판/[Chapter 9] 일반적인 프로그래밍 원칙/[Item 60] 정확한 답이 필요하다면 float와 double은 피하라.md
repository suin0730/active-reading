---
tags: [Book/Effective Java/Ch9 - 일반적인 프로그래밍 원칙]
title: '[Item 60] 정확한 답이 필요하다면 float와 double은 피하라'
created: '2022-07-08T13:37:55.115Z'
modified: '2022-07-08T13:53:58.318Z'
---

# [Item 60] 정확한 답이 필요하다면 float와 double은 피하라

## float와 double의 문제점

float와 double 타입은 부동소수점 연산에 쓰이며, 넓은 범위의 수를 빠르게 근사치로 계산하도록 설계되었다. 따라서 정확한 결과를 얻지 못할 수 있다.

- 1.03 - 0.42의 결과를 0.6100000000000001로 출력한다.
- 1.00 - 9 * 0.10의 결과를 0.0999999999999998로 출력한다.

## float와 double의 보완법

금융 계산에는 BigDecimal, int, long을 사용해야 한다. 특히 BigDecimal을 사용하면 **문자열을 받는 생성자를 사용해 부정확한 값이 사용되는 것을 막을 수 있다.** BigDecimal은 기본 타입보다 사용하기 불편하고 훨씬 느려서 대안으로 int, long을 쓸 수는 있지만 이 경우 다룰 수 있는 값의 크기가 제한되고 소수점을 직접 관리해야 한다. 

```java
// 소수점 계산이 잘못 되는 코드
public static void main(String[] args) {
  double funds = 1.00;
  int itemsBought = 0;
  for (double price = 0.10; finds >= price; price += 0.10) {
    funds -= pirce;
    itemsBought++;
  }
  // 사탕 3개 구입 후 잔돈은 0.3999999999999999 남는다.
}
```

```java
// 소수점 계산이 정상적으로 수행되는 코드
public static void main(String[] args) {
  final BigDecimal TEN_CENTS = new BigDecimal(".10");

  int itemsBought = 0;
  BigDecimal funds = new BigDecimal("1.00");
  for (BigDecimal price = TEN_CENTS; funds.compareTo(price) >= 0; price = price.add(TEN_CENTS)) {
    funds = funds.subtract(price);
    itemsBought++;
  }
  // 사탕 4개 구입 후 잔돈은 0.3999999999999999 남는다.
}
```

> 숫자를 아홉 자리 십진수로 표현할 수 있다면 int를 사용하고, 열여덟 자리 십진수로 표현할 수 있다면 int을 사용하라. 그 이상은 BigDecimal을 사용하라.
