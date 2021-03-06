---
tags: [Book/Effective Java/Ch3 - 모든 객체의 공통 메서드]
title: '[Item 12] toString을 항상 재정의하라'
created: '2022-01-07T12:51:15.122Z'
modified: '2022-01-15T08:56:17.062Z'
---

# [Item 12] toString을 항상 재정의하라

## 기본 toString 메서드

`Object`의 기본 toString 메서드는 단순히 `클래스이름@해시코드`를 반환한다. 클라이언트가 toString을 제대로 재정의하지 않으면 쓸모없는 메시지만 로그에 남을 것이다. 일반 규약에 따르면, toString은 **간결하면서 사람이 읽기 쉬운 형태의 유익한 정보**를 반환해야 한다. 따라서 toString 일반 규약은 모든 하위 클래스에서 이 메서드를 재정의하는 것을 추천한다.

```
// 기본 toString
{Jenny=PhoneNumber@abcde}

// 재정의한 toString
{Jenny=010-1234-5678}
```

## 재정의한 toString 메서드

> toString을 잘 구현한 클래스를 사용한 시스템은 디버깅하기 수월하다.

toString은 클라이언트가 직접 호출하지 않더라도 다른 어딘가에서 사용된다. println, printf, 문자열 연결(+), assert 구문, 디버거가 객체를 출력할 때 등이 대표적인 사용 예시이다. 

### 포멧 문서화 장단점

toString을 구현할 때 **반환값의 포멧을 문서화할지** 정해야 한다.

포멧을 명시한 객체는 표준적이고, 사람이 읽을 수 있게 된다. 이 경우 문자열과 객체를 상호 전환할 수 있는 정적 팩터리나 생성자를 함께 제공할수도 있다. 자바 플랫폼의 `BigInteger`, `BigDecimal` 같은 클래스가 포멧을 명시해두었다.

그러나 포멧을 한번 명시하면 해당 시스템은 평생 그 포멧에 얽매이게 된다. 클라이언트는 모두 포멧에 맞추어 파싱하고, 객체를 만들고 저장하는 코드를 작성하게 되는데, **만약 다음 릴리즈에서 포멧이 바뀐다면 기존 코드는 엉망이 될 것이다.** 따라서 포멧을 명시하지 않으면 향후 릴리즈에서 정보를 더 넣거나 포멧을 개선할 수 있어 비교적 유연하다.
~~지번주소 포멧 레거시를 도로명주소로 바꾼다고 생각해보면...끔찍하다..~~

### 개별 값 접근 API

toString이 반환하는 객체 인스턴스를 얻어올 수 있는 API를 제공해야 한다. 예를 들어 `Address` 클래스에 시, 군, 구, 우편번호 등이 있다면 각각을 위한 접근자를 제공해야 한다. 그렇지 않으면 클라이언트는 각 정보가 필요할 때 toString 반환값을 파싱해서 각 값을 얻어야 하고 이는 성능 저하로 이어진다.
