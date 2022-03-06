---
tags: [Book/Effective Java/Ch4 - 클래스와 인터페이스]
title: '[Item 17] 변경 가능성을 최소화하라'
created: '2022-03-01T10:21:40.656Z'
modified: '2022-03-06T12:13:33.982Z'
---

# [Item 17] 변경 가능성을 최소화하라

불변 클래스는 인스턴스 내부 값을 수정할 수 없는 클래스이다. 불변 인스턴스에 저장된 정보는 고정되어서 객체가 파괴되는 순간까지 달라지지 않는다. 따라서 가변 클래스보다 설계하고 구현하기 쉬우며, 오류가 생길 여지가 적다.

## 불변 클래스를 만드는 방법
- 객체 상태를 변경하는 메서드(setter)를 제공하지 않는다.
- 클래스를 extend할 수 없도록 한다.
- 모든 필드를 final로 선언한다.
- 모든 필드를 private로 선언한다.
- 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.

## 불변 클래스 예시

아래 복소수를 표현하는 클래스에서는 사칙연산 메서드의 결과로, 인스턴스 자신을 수정하지 않고 새로운 Complex 인스턴스를 만들어 반환한다. 이렇게 연산 후에도 피연산자가 그대로인 패턴을 함수형 프로그래밍이라 한다. 메서드명이 add가 아니라 plus인 이유도 해당 메서드가 객체의 값을 변경하지 않는다는 것을 강조하려는 의도이다. **함수형 프로그래밍을 사용해 코드에서 불변이 되는 영역의 비율을 높일 수 있다.**

```java
public final class Complex {
  private final double re;
  private final double im

  public Complex(double re, double im) {
    this.re = re;
    this.im = im;
  }

  public double realPart()      { return re; }
  public double imaginaryPart() { return im; }

  public Complex plus(Complex c) {
    return new Complex(re + c.re, im + c.im);
  }
  ...
}
```

## 불변 클래스의 특징

### thread-safe해 안심하고 공유할 수 있다

불변 객체에 대해서는 어떤 스레드도 다른 스레드에 영향을 줄 수 없으므로 thread-safe하다고 할 수 있다. 이때 재사용성을 높이기 위해서는 아래 방법들 중 한 가지를 이용해 한번 만든 인스턴스를 최대한 재활용하는 것이 좋다.

1. 자주 사용하는 값들을 상수로 제공한다.
2. 자주 사용되는 인스턴스를 캐싱해 같은 인스턴스를 중복 생성하지 않게 해주는 정적 팩터리를 제공한다. 이 경우 여러 클라이언트가 같은 인스턴스를 공유해 메모리 사용량과 가비지 컬렉션 비용이 줄어든다.

불변 객체는 자유롭게 공유할 수 있고 아무리 복사해도 원본과 동일하므로 방어적 복사가 필요없다. 그러니 clone 메서드나 [복사 생성자](https://github.com/suin0730/active-reading/blob/main/%5B1%5D%20%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C%20%EC%9E%90%EB%B0%94%203%ED%8C%90/%5BChapter%203%5D%20%EB%AA%A8%EB%93%A0%20%EA%B0%9D%EC%B2%B4%EC%9D%98%20%EA%B3%B5%ED%86%B5%20%EB%A9%94%EC%84%9C%EB%93%9C/%5BItem%2013%5D%20clone%20%EC%9E%AC%EC%A0%95%EC%9D%98%EB%8A%94%20%EC%A3%BC%EC%9D%98%ED%95%B4%EC%84%9C%20%EC%A7%84%ED%96%89%ED%95%B4%EB%9D%BC.md)를 제공하지 않는 것이 좋다.

### 불변 객체끼리 내부 데이터를 공유할 수 있다

불변 객체들끼리는 내부 데이터를 공유할 수 있다. 예를 들어 `BigInteger` 클래스는 내부에서 값의 부호와 크기를 따로 표현한다. 반면 `negate` 메서드는 크기가 같고 부호만 반대인 새로운 `BigInteger`를 생성하는데, 값의 크기 배열을 원본 인스터스가 가리키는 크기 배열과 공유한다.

이렇게 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 아무리 복잡한 구조를 갖는 객체이더라도 불변식을 유지하기 훨씬 수월하다. 좋은 예로, 불변성이 깨지면 안되는 맵의 키나 집합의 원소로 불변 객체를 사용하기 좋다.

### 값이 다르면 다른 객체이다

당연한 말이지만, 객체의 값이 다르면 다른 불변 객체로 만들어야 한다. 만약 불변 객체로 만들어야 하는 값의 가짓수가 많다면 객체를 모두 생성하는 데 큰 비용이 들 것이다.

## 불변 클래스를 만드는 또다른 설계 방법

자신을 상속하지 못하게끔 해 클래스가 불변임을 보장하려면 final 클래스를 만들수도 있지만, 모든 생성자를 `private` 혹은 `package-private`로 만들고 public 정적 팩토리를 만드는 것이 더 유연한 방법이 될 수 있다.

아래 방식은 외부에서 확인할 수 없는 `package-private` 구현 클래스를 원하는 만큼 만들 수 있어서 final 클래스보다 유연하다. `public`이나 `protected` 생성자가 없어서 외부에서는 이 클래스를 구현하는 게 불가능하기 때문이다. **정적 팩터리 방식을 사용하면 여러 구현 클래스를 사용할 수 있고, 다음 릴리즈에서 객체 캐싱 기능을 추가해 성능을 올릴 수도 있다.**

```java
public class Complex {
  private final double re;
  private final double im;

  private Complex(double re, double im) {
    this.re = re;
    this.im = im;
  }

  public static Complex valueOf(double re, double im) {
    return new Complex(re, im);
  }
}
```

앞서 본 불변 클래스를 만드는 방법 중 모든 필드를 final로 선언하라는 조건이 있었다. 그러나 동작 성능을 향상시키기 위해, 계산 비용이 큰 값을 final이 아니도록 선언해두고 지연 초기화를 사용해 사용 시점에 값을 넣고 그 이후부터 캐싱된 값을 사용하도록 할 수 있다.

