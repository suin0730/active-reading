---
tags: [Book/Effective Java/Ch2 - 객체 생성과 파괴]
title: '[Item 2] 생성자에 매개변수가 많다면 빌더를 고려해라'
created: '2022-01-04T09:59:07.777Z'
modified: '2022-01-04T12:27:32.614Z'
---

# [Item 2] 생성자에 매개변수가 많다면 빌더를 고려해라

정적 팩토리 메서드와 생성자는 선택적 매개변수가 많을 때 적절히 대응하기 어렵다. 빌더 패턴이 나오기까지의 과정을 살펴보자.

## 생성자에 매개변수가 많을 때 사용할 수 있는 방법

### 점층적 생성자 패턴

> 점층적 생성자 패턴을 사용할 때 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 의미를 파악하기 어렵다.

점층적 생성자 패턴은 필수 매개변수만 받는 생성자를 기반으로 선택 매개변수를 하나씩 늘린 생성자를 만드는 패턴이다. 

`Circles` 클래스에서 필수 매개변수인 `radius`와 `count`를 받는 생성자를 기본으로 선택 매개변수인 `color`, `solidLine`, `shadow`를 하나씩 추가한 생성자를 만들었다. 이 클래스를 사용하기 위해서는 원하는 선택 매개변수를 모두 포함한 생성자 중 가장 짧은 생성자를 호출하게 되는데, 이 방법은 사용하지 않는 매개변수도 결국 채워진다는 점에서 비효율적이다. 자료형이 같은 값이 연속되면 각 값의 의미도 헷갈릴 것이다. 

```java
public class Circles {
  private final int radius;          // 반지름       (필수)
  private final int count;           // 원 개수      (필수)
  private final String color;        // 원 색깔      (선택)
  private final boolean solidLine;   // 실선 여부    (선택)
  private final boolean shadow;       // 그림자 여부 (선택)
  
  public Circles (int radius, int count) {
    this(radius, count, '');
  }

  public Circles (int radius, int count, Stirng color) {
    this(radius, count, color, 0);
  }

  public Circles (int radius, int count, String color, boolean solidLine) {
    this(radius, count, color, solidLine 0);
  }

  public Circles (int radius, int count, String color, boolean solidLine, boolean shadow){
    this.radius = radius;
    this.count = count;
    this.color = color;
    this.solidLine = solidLine;
    this.shadow = shadow;
  }
}
```

### 자바빈즈 패턴

> 자바빈즈 패턴을 사용하면 객체 하나를 만들기 위해 여러 메소드를 호출해야 하고 객체가 완전히 생성되기 전에 일관성이 무너진다.

자바빈즈 패턴은 매개변수가 없는 생성자로 객체를 만든 후, 세터 메서드를 호출해 원하는 매개변수 값을 설정하는 패턴이다. 반지름이 3이고 그림자가 있는 노란색 원을 7개 만들려면 아래 예시와 같이 생성하면 된다.

이 방법은 인스턴스를 만들기 쉽고 각 매개변수를 따로 설정하기 때문에 읽기에도 쉽다. 그러나 `circles` 객체를 하나 만들기 위해 세터 메서드를 5개나 호출해야 하고 모든 메서드 호출이 완료되기 전까지는 객체 일관성이 깨진다는 단점이 있다. 객체 일관성이 깨지면 스레드 안정성을 얻기 위해 수동으로 객체를 얼리고 녹여야 하는데 이 방법은 런타임 오류에 취약하므로 사용하기 어렵다.

```java
Circles circles = new Circles();
circles.setRadius(3);
circles.setCount(7);
circles.setColor('yellow');
circles.setShadow(true);
```
```java
public class Circles {
  // 기본값으로 초기화
  private int radius = 1;
  private int count = 1;
  private String color = 'white';
  private boolean solidLine = true;
  private boolean shadow = false;
  
  public Circles() { }

  // 세터 메서드
  public void setRadius(int val)          { radius = val; }
  public void setCount(int val)           { count = val; }
  public void setColor(String val)        { color = val; }
  public void setSolidLine(boolean val)   { solidLine = val; }
  public void setShadow(boolean val)      { shadow = val; }
}
```

### 빌더 패턴

> 빌더 패턴은 점층적 생성자 패턴의 안정성과 자바빈즈 패턴의 가독성을 둘 다 가진다. 생성자나 정적 팩터리 메서드가 처리할 매개변수가 많다면 빌더 패턴을 사용하는 것이 좋다.

빌더 패턴에서 클라이언트는 필요한 객체를 직접 만들지 않고, 필수 매개변수를 사용해 생성자(혹은 정적 팩토리 메서드)를 호출해 빌더 객체를 얻는다. 빌더 객체는 세터 메서드로 선택 매개변수를 설정하고 `build()`를 호출해 객체를 만든다.

아래 예제를 보면 **빌더의 세터 메서드는 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다.** 이 코드는 점층적 생성자 패턴보다 읽고 쓰기 쉽다. 

> 잘못된 매개변수가 넘어오는 것을 발견하려면 빌더의 생성자와 메서드에서 매개변수 유효성을 검사하고, 여러 매개변수에 걸친 오류는 `build()`에서 불변식 여부를 검사해야 한다.
잘못된 점이 있다면 `IllegalArgumentException`을 던지면 된다.

```java
Circles circle = new Circles.Builder(3, 7).color('yellow').shadow(true);
```

```java
public class Circles {
  private final int radius;
  private final int count;
  private final String color;
  private final boolean solidLine;
  private final boolean shadow;

  public static class Builder {
    // 필수 매개변수
    private final int radius;
    private final int count;

    // 선택 매개변수 (기본값으로 초기화)
    private String color = 'white';
    private boolean solidLine = true;
    private boolean shadow = false;

    public Builder(int radius, int count) {
      this.radius = radius;
      this.count = count;
    }
    public Builder(String val) { color = val; return this; }
    public Builder(boolean val) { solidLine = val; return this; }
    public Builder(boolean val) { shadow = val; return this; }

    public Circles build() {
      return new Circles(this);
    }
  }

  private Circles(Builder builder) {
    radius = builder.redius;
    count = builder.count;
    color = builder.color;
    solidLine = builder.solidLine;
    shadow  =builder.shadow;
  }
}
```

단, 빌더 객체를 만들려면 다른 방법에 비해 장황한 빌더부터 만들어야 한다. 따라서 매개변수가 4개 이상인 경우에 빌더 사용을 고려하는 것이 낫다.

**빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.** 빌더 클래스에는 [재귀적 타입 한정]()을 이용하는 제네릭 타입을 반환 타입으로 사용할 수 있고, 여기에 추상 메서드인 `self`를 더해 **하위 클래스 형변환 없이 메서드 연쇄를 지원할 수 있다.**


