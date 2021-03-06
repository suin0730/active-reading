---
tags: [Book/Effective Java/Ch4 - 클래스와 인터페이스]
title: '[Item 21] 인터페이스는 구현하는 쪽을 생각해 설계하라'
created: '2022-04-06T08:29:23.189Z'
modified: '2022-04-07T02:37:55.203Z'
---

# [Item 21] 인터페이스는 구현하는 쪽을 생각해 설계하라

자바 8 이후로 인터페이스에 메서드를 추가할 수 있게 되었다. 

## 디폴트 메서드

디폴트 메서드를 선언하면 그 인터페이스를 구현한 모든 클래스에서 디폴트 메서드를 사용할 수 있게 된다. 그러나 아무리 설계를 고려해서 추가된 디폴드 메서드라도 기존에 인터페이스를 구현한 클래스에 모두 다 맞기는 어렵다. 새로 만들어진 역할에 알맞게 메서드가 재정의되어야 할 필요가 있다.

디폴트 메서드는 컴파일에 성공하더라도 기존 구현체에 런타임 오류를 일으킬 수 있으므로 꼭 필요한 경우가 아니면 기존 인터페이스에 디폴트 메서드를 추가하는 일을 피해야 한다.

새로운 인터페이스라면 릴리즈 전에 테스트를 반드시 거쳐야 한다. 각 인터페이스의 인스턴스를 다양한 작업에 활용하는 클라이언트를 만들어보고, 개발자가 의도한 용도에 부합하는지 확인해야 한다.
