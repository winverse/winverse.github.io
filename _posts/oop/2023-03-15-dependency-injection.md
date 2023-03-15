---
layout: post
author: winverse
title:  "Dependency Injection과 IoC container"
description: Dependency Injection, IoC container에 대해서 알아보자
tags: Dependency-Injection IoC-container, 의존성주입, 제어의역전 
category: oop
is_published: false
---

# 문제 발생
<img src="https://i.imgur.com/xaDVoZU.png"  alt="dependeny-injection-before" style="width: 100%; height: 400px;">

기존 방식에서는 상위 클래스가 하위 클래스를 contructor에서 직접적으로 호출하여 사용하는 방식이 있을 수 있는데, 문제는 이런식으로 코드를 작성하게 되면,
상위 클래스가 하위 클래스에 의존성이 커지게 되어, 코드의 확장과 유지보수의 어려움 그리고 테스트 코드를 짜기 힘들다는 단점이 발생하게 됩니다.

# IoC(Inversion of Control) container와 제어의 역전
직접적으로 하위 클래스를 호출하여 사용하는 방식은 문제가 있기 때문에 중간에 매개체를 두어 의존성들 생성, 메모리 해제와 같은 
의존성 관리를 하게 됩니다. 이렇게 되면 개발자가 직접적으로 의존성을 관리하던 것을 이젠 매개체에게 제어권을 일임하게 되어 더 이상 제어의 추체가 개발자 아닌
매개체가 주체가 되기 때문에 제어의 역전이 발생한다고 할 수 있습니다. 
이런 매개체는 nestjs, spring과 같은 framework들이 ioc container들을 포함하고 있습니다.

# Dependency Injection
의존성 주입이라는 것은, IoC container에 모든 의존성을 등록을 해두고, 더 이상 상위 클래스에서 의존성들을 호출하는 것이 아니라
IoC Container가 상위에 의존성들을 주입 하는 것을 말해준다. 이 과정에서 의존하는 모듈의 생성과 해제 등의 일련의 제어 과정을 
IoC 컨테이너가 진행을 하면서 제어의 역전이 일어나게 된다.