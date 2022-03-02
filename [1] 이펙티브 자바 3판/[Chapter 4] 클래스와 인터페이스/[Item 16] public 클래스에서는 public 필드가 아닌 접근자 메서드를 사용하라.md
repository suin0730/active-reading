---
tags: [Book/Effective Java/Ch4 - 클래스와 인터페이스]
title: '[Item 16] public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라'
created: '2022-03-01T10:20:45.541Z'
modified: '2022-03-02T04:03:24.145Z'
---

# [Item 16] public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

## public 메서드에 public 필드를 두지 마라

아래 클래스는 데이터 필드에 직접 접근할 수 있어 캡슐화의 이점을 갖지 못한다. 내부 표현을 바꾸기 위해 API를 수정해야 하고 불변식도 보장할 수 없다.
```java
class Point {
  public double x;
  public double y;
}
```

패키지 바깥에서 접근할 수 있도록 의도된 클래스라면 아래와 같이 설계되어야 맞다. 이렇게 접근자가 있다면 내부 구현을 자유롭게 바꿀 수 있다.
```java
class Point {
  private double x;
  private double y;

  public Point(double x, double y) {
    this.x = x;
    this.y = y;
  }

  public double getX() { return x; }
  public double getY() { return y; }

  public void setX(double X) { this.x = x; }
  public void setY(double Y) { this.y = y; }
}
```
> 그러나 만약 packege-private이거나 private 중첩 클래스라면 필드를 노출해도 상관없다. 클라이언트 코드가 이 변수에 묶이지만, 어차피 API 내부 코드이기 때문이다.

