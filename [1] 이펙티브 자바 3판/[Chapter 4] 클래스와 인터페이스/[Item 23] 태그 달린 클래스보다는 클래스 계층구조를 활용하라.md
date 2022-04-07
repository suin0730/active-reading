---
title: '[Item 23] 태그 달린 클래스보다는 클래스 계층구조를 활용하라'
created: '2022-04-06T08:53:06.723Z'
modified: '2022-04-07T01:47:29.855Z'
---

# [Item 23] 태그 달린 클래스보다는 클래스 계층구조를 활용하라

두 가지 이상의 의미를 표현할 수 있고, 그 중 표현하고자 하는 의미를 태그 등으로 알려주는 클래스를 볼 수 있다.

## 태그 달린 클래스의 단점

> 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다.

- 열거 타입을 선언해야 한다.
- 태그를 나타내는 필드가 추가된다.
- 어떤 태그인지에 따라 switch문으로 나타낸다.
- 메모리를 많이 사용한다.
- 필드를 final로 선언하려면 쓰이지 않는 필드까지 초기화해야 한다.

태그 달린 클래스는 클래스가 계층구조를 사용해 다양한 의미의 객체를 표현하는 것을 어설프게 흉내내는 과정이다.

## 태그 달린 클래스를 계층구조로 바꾸는 방법

1. 계층구조의 루트가 될 추상 클래스를 정의한다.
2. 태그 값에 따라 동작이 달라지는 메서드를 루트 클래스에 추상 메서드로 정의한다.
3. 태그 값에 상관없이 동작이 일정한 메서드를 루트 클래스에 일반 메서드로 추가한다.
4. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드를 루트 클래스 필드로 올린다.
5. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다.

## 클래스 계층구조

클래스 계층구조는 태그 달린 클래스의 단점을 모두 없앤다. 쓸데없는 코드가 모두 사라져서 간결하고 명확하다. 각 하위 클래스에서 필드를 모두 초기화하고 추상 메서드를 모두 구현했는지 컴파일러가 확인해준다. 타입 사이의 자연스러운 계층 구조를 반영할 수 있는 적절한 방법이다.

```java
abstract class Figure {
  abstract double area();
}

class Circle extends Figure {
  final double radius;

  Circle(double radius) { thhis.redius = radius; }

  @Override double area() { return Math.PI * (radius * radius); }
}

class Ractangle extends Figure {
  final double length;
  final double width;

  Ractangle(double length, double width) {
    this.length = length;
    this.width = width;
  }

  @Override double area() { return length * length; }
}
```
