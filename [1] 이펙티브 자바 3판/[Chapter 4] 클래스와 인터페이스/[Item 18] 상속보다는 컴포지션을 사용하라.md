---
tags: [Book/Effective Java/Ch4 - 클래스와 인터페이스]
title: '[Item 18] 상속보다는 컴포지션을 사용하라'
created: '2022-03-01T10:22:34.465Z'
modified: '2022-03-07T15:23:36.026Z'
---

# [Item 18] 상속보다는 컴포지션을 사용하라

상속은 코드를 재사용할 수 있는 좋은 수단이지만 잘못 사용하면 오류를 내기 쉽다. 상위 클래스와 하위 클래스를 모두 같은 패키지 내에서 상속한다면 문제가 없지만, 패키지 경계를 넘어 다른 패키지의 구체 클래스를 상속하는 일은 위험하다. 

## 상속은 캡슐화를 깨뜨린다

상속은 캡슐화를 깨뜨릴 수 있다. 상위 클래스 구현이 달라짐에 따라 하위 클래스 수정 없이 동작에 이상이 생길 수 있다. 따라서 상위 클래스 설계자가 문서화를 제대로 해둘 필요가 있다.


아래는 HashSet에 누적해서 저장한 원소 수를 알 수 있도록 재정의한 클래스이다. 만약 원소를 3개 가진 리스트를 추가한 후 `getAddCount`를 호출하면 예상했던 3개가 아닌 6개가 반환된다. 그 이유는 HashSet의 addAll이 add를 사용해서 구현되었기 때문이다. super.addAll(c)가 불린다면 재정의된 add를 사용해 값이 중복으로 더해진다. 이 예시와 같이 **클래스 자신의 다른 부분을 사용하는 self-use 여부는 해당 클래스의 내부 구현 방식에 해당되어 다음 릴리즈에서도 유지될지 알 수 없다.**

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
  private int addCount = 0;

  public InstrumentedHashSet() {
  }

  public InstrumentHashSet(int initCap, float loadFactor) {
    super(initCap, loadFactor);
  }

  @Override public boolean add(E e) {
    addCount++;
    return super.add(e);
  }

  @Override public boolean addAll(Collection<? extends E> c) {
    addCount += c.size();
    return super.addAll(c);
  }

  public int getAddCount() {
    return addCount();
  }
}
```

다음 릴리즈에서 새로운 메서드가 상위 클래스에 추가된다면 이를 상속받은 하위 메서드들은 모두 수정되어야 할 수 있다. 또는 상위 클래스가 이미 하위 클래스에 있는 메서드와 시그니처가 같은 메서드를 추가한다면 개발자의 예상대로 동작하지 않는다.

## 컴포지션을 사용하라

상기한 모든 문제를 피해가려면 **기존 클래스를 확장하는 대신 새로운 클래스를 만들고 private 필드로 기존 클래스 인스턴스를 참조하게 하면 된다.** 이러한 구성은 기존 클래스가 새로운 클래스의 구성으로 쓰인다는 의미로 컴포지션이라 한다.

## 상속과 컴포지션

단순히 상위 클래스가 하위 클래스를 포함할 때에 상속시키면 안된다. "클래스 B is a 클래스 A"일 때만 클래스 A를 상속해야 한다. 그렇지 않으면 A는 B의 필수 구성요소가 아니라 구현하는 방법 중 하나일 뿐이라 private 인스턴스로 만들어야 한다.

컴포지션을 써야 하는 상황에서 상속을 사용하는 것은 불필요하게 상위 클래스를 하위에 노출시키는 것이다. 하위 클래스는 상위 클래스의 변경에 종속되고 해당 API 성능도 영원히 제한된다. 
