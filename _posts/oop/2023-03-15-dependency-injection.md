---
author: winverse
title:  "Dependency Injection과 IoC container"
description: Dependency Injection, DIP, IoC container의 관계를 알아보자.
tags: 
  - Dependency-Injection 
  - IoC-container 
  - 의존성주입 
  - 제어역전 
  - DIP
categories: 
  - oop
is_published: true
toc: true
---

# 1. 문제 인식
```ts
class B {
  // blar blar
}

class A {
  priavate b;
  public Hello() {
    this.b = new B();
    // Do something
  }
}
```
기존 방식에서는 상위 클래스가 하위 클래스를 contructor에서 직접적으로 호출하여 사용하는 방식이 있을 수 있는데, 문제는 이런식으로 코드를 작성하게 되면,
상위 클래스가 하위 클래스에 의존성이 커지게 되어, 코드의 확장과 유지보수의 어려움 그리고 테스트 코드를 짜기 힘들다는 단점이 발생하게 됩니다.

# 2. 제어의 역전(Inversion of Control / IoC)과 Container
직접적으로 하위 클래스를 호출하여 사용하는 방식은 문제가 있기 때문에 중간에 매개체를 두어 의존성들 생성, 메모리 해제와 같은 의존성 관리를 하게 됩니다. 이렇게 되면 개발자가 직접적으로 의존성을 관리하던 것을 이젠 매개체에게 제어권을 일임하게 되어 더 이상 제어의 주체가 개발자 아닌 매개체가 주체가 되기 때문에 제어의 역전이 발생한다고 할 수 있습니다. 이런 매개체는 Nestjs, Spring과 같은 framework들이 IoC container들을 포함하고 있습니다.

# 3. IoC를 구현하는 패턴들
## 1. 생성자 주입(Constructor Injection)
생성자 주입은 객체가 생성될 때 의존성을 전달하는 방식입니다. 이 방식은 객체가 생성되는 시점에 필요한 모든 의존성이 전달되므로 객체의 불완전한 상태를 방지할 수 있습니다.

```ts
class SomeClass {
  private dependency: Dependency;

  constructor(dependency: Dependency) {
    this.dependency = dependency;
  }
}
```

## 2. Setter 주입(Setter Injection)
Setter 주입은 객체가 생성된 후 setter 메소드를 통해 의존성을 전달하는 방식입니다. 이 방식은 객체가 생성된 후에도 의존성을 변경할 수 있어 유연성이 높습니다.

```ts
class SomeClass {
  private dependency: Dependency;

  setDependency(dependency: Dependency) {
    this.dependency = dependency;
  }
}
```

## 3. Interface 주입(Interface Injection)
Interface 주입은 인터페이스를 통해 의존성을 전달하는 방식입니다. 이 방식은 인터페이스를 구현하는 클래스들이 동일한 방법으로 의존성을 전달받게 되어 일관성이 유지됩니다. 

```ts
interface SomeInterface {
  setDependency(dependency: Dependency): void;
}

class SomeClass implements SomeInterface {
  private dependency: Dependency;

  setDependency(dependency: Dependency) {
    this.dependency = dependency;
  }
}
```

> NestJS는 내장된 Inversion of Control (IoC) 컨테이너를 가지고 있어서 provider들 사이의 관계를 해결해줍니다. Dependency Injection (DI)은 IoC 기술로 사용자 자신의 코드로 종속성을 인스턴스화하는 대신 IoC 컨테이너(NestJS 런타임 시스템)로 위임합니다. NestJS에서는 생성자 기반의 의존성 주입을 사용하여 클래스에 인스턴스(주로 서비스 제공자)를 주입하는 방식으로 DI가 구현되어 있습니다.

# 4. DIP(Dependency Inversion Principle)
DIP (Dependency Inversion Principle)는 의존성 역전 원칙이라는 뜻으로, 상위 모듈이 하위 모듈에 의존하는 것이 아니라 추상화된 인터페이스에 의존하도록 만드는 것을 의미합니다.

예를 들어, 다음과 같은 코드에서 SomeClass는 ConcreteDependency 클래스에 직접 의존하고 있습니다.
```ts
class SomeClass {
  private dependency: ConcreteDependency;

  constructor(dependency: ConcreteDependency) {
    this.dependency = dependency;
  }
}
```
하지만 DIP를 적용하면 다음과 같이 SomeClass는 추상화된 인터페이스인 AbstractDependency에 의존하게 됩니다.

```ts
interface AbstractDependency {}

class ConcreteDependency implements AbstractDependency {}

class SomeClass {
  private dependency: AbstractDependency;

  constructor(dependency: AbstractDependency) {
    this.dependency = dependency;
  }
}
```

# 5. Dependency Injection
의존성 주입이라는 것은, IoC container에 모든 의존성을 등록을 해두고, 더 이상 상위 클래스에서 의존성들을 호출하는 것이 아니라 IoC Container가 상위에 의존성들을 주입 하는 것을 말해준다. 이 과정에서 의존하는 모듈의 생성과 해제 등의 일련의 제어 과정을 IoC 컨테이너가 진행을 하면서 제어의 역전이 일어나게 됩니다. 

# 6. IoC와 DIP와 Dependency Injection의 관계
IoC와 DIP와 Dependency Injection은 서로 밀접한 관계가 있습니다. IoC 컨테이너가 객체의 생성과 생명주기 관리를 담당함으로써 Dependency Injection을 가능하게 합니다. 또한 DIP 원칙을 적용함으로써 추상화된 인터페이스에 의존함으로서 유연한 구조가 가능해집니다.

# 7. 결론 
IoC (Inversion of Control), DIP (Dependency Inversion Principle), 그리고 Dependency Injection은 객체지향 프로그래밍에서 중요한 개념들입니다. 이들 개념들을 적용함으로서 유연하고 확장 가능한 구조의 프로그램을 만들 수 있습니다.