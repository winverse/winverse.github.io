---
layout: post
author: winverse
title:  "공변성(covariant)과 반공변성(contravariant)"
description: 이번글을 통해서 공변성과 반공변성에 대해서 알아보며, Rust의 API를 인용하여 Typescript에서 어떻게 타입 안전(type-safety)한 코드를 작성할 수 있는지 알아봅시다.
tags: 공변성 반공변성 covariant contravariant type level system Typescript Rust
---


> 공변성과 반공변성은 객체 지향에서 LSP(리스코프 치환 원칙)과 연관이 있으며 Type level system에서 어떤 인자 타입을 받고 어떤 인자 타입을 반환할건지에 대한 논의와도 연관되어 있습니다.

# 공변성(Covariant)

- 다음 세가지 클래스가 있다고 가정하면
{% highlight typescript %}
class Animal {}

class Dog extends Animal {}

class Greyhound extends Dog {}
{% endhighlight typescript %}

이것을 슈퍼(super)타입과 서브(sub)타입으로 표현한다면

> Greyhound&nbsp;&nbsp; < &nbsp;&nbsp; Dog &nbsp;&nbsp; < &nbsp;&nbsp; Animal

Greyhound는 Dog의 서브 타입이고 Dog는 Animal의 서브 타입입니다. 추이적 관계에 따라 Greyhound는 Animal의 서브타입니다.

{% highlight typescript %}
let animal = new Animal();
let dog = new Dog();

animal = dog // OK
dog = animal // Error
{% endhighlight %}

위 예제를 보면 슈퍼 타입에 서브 타입을 할당하려하면 문제가 없지만 서브 타입을 슈퍼 타입에 할당하려고 하면 문제가 생기는데 이를 공변성이라고 하고 정리하면 아래와 같습니다.


* T'가 T의 서브 타입이라면 K<T'>는 K<T>의 서브 타입이 성립합니다.


# 반공변성(Contravariant)
반공변성은 정확히 공변성과 반대로 작용하며 타입스크립트에서 사용하려면 `--strictFunctionTypes` 옵션을 사용하면 됩니다. 

{% highlight typescript %}
let animal = new Animal();
let dog = new Dog();

animal = dog // Error
dog = animal // OK
{% endhighlight %}

다시 말하면, 공변성과 완전 반대로 작동합니다. 슈퍼 타입에 서브 타입을 할당하려고하면 문제가 행기고 서브 타입에 슈퍼 타입을 할당하려고 하면 문제가 생기지 않습니다. 이를 반공변성이라고 하고 정리하면 다음과 같습니다.

* T'가 T의 서브 타입이면, K<T>는  K<T'>의 서브 타입이 성립합니다.

# 이변성(Bivariant)
이변성은 공변성의 특징과 반공변성의 특징을 가지는 것으로 타입스크립트에서는 `--strictFunctionTypes`옵션을 사용하지 않으면 나타납니다. 슈퍼 타입과 서브 타입 모두 어떤 식으로 사용하든지 문제가 생기지 않습니다.
다만 타입스크립트에서 `--strictFunctionTypes` 켜놓고 타입 표현 방식에 따라서 이변적으로 작동할지 공변적으로 작동할지 선택할 수 있도록 [의도](https://github.com/microsoft/TypeScript/pull/18654) 했는데 줄여쓰기(shorthand) 방식 (예시: `push(item: T): void`)는 메서드 파라미터를 이변적으로 동작 시키기 위한 표기법이고, 프로퍼티 방식 (예시: `push: (item: T) => void`)는 메서드 파라미터를 반공변적으로 동작시키기 위한 표기법으로 선택할 수 있도록 하였습니다.

# Type level System과의 관계
Type level System이라는 것은 결국 사전에 버그를 잡아주고 에러를 줄여주는 역할을 합니다. 다시 말하면 개발의 안전성을 올려주기 위해서 필요한 것이며 이것은 타입 안전`(type safe)`을 통해서 이루어집니다. 그럼 공변성과 이변성을 어떻게 이용하는 것이 타입 안전한 개발과 이어지는 것일까요? 여기서 질문입니다.

함수 Dog -> Dog 타입을 가진 함수 `g` 를 인자로 가지는 함수 `f` 있다. 구체적으로 적으면 다음과 같습니다.

{% highlight typescript %}
class Animal {}
class Dog extends Animal {}
class Greyhound extends Dog {}
class ShibaInu extends Dog {}

type F = (g: (a: Dog) => Dog) => string;
{% endhighlight typescript %}

이제 f를 다른 함수인 g와 함께 호출해보고 어떤 일이 발생하는지 알아봅시다.

* 다음 중 `(a: Dog) => Dog`의 타입을 가지는 함수 g 의 타입이 될 수 있는 인자는?

1. (a: Greyhound) => Greyhound (`서브 타입`을 받아서 -> 동일 `서브 타입` 반환)
2. (a: Greyhound) => Animal (`서브 타입`을 받아서 -> `슈퍼 타입`을 반환)
3. (a: Animal) => Animal (`슈퍼 타입`을 받아서 -> 동일 `슈퍼 타입`을 반환)
4. (a: Animal) => Greyhound (`슈퍼 타입`을 받아서 -> `서브 타입`을 반환)

---

1. (a: Greyhound) => Greyhound로 가정하면 f(g)는 타입 안전(type safe)한가?  
_함수 f는 인자 g를 사용하면서 Dog의 다른 서브 타입 예를 들면 `ShibaInu`가 인자로 들어갈 수 있기 때문에 아닙니다._   

2. (a: Greyhound) => Animal 가정하면 f(g)는 타입 안전한가?  
_1과 동일합니다._

3. (a: Animal) => Animal 가정하면 f(g)는 타입 안전한가?  
_f(g(x))이 호출 되면 반환값으로 Dog 메소드를 활용할지도 모릅니다. 예를 들어 bark() 같은 그러나 모든 Animal이 bark() 메소드를 가지고 있지 않기 때문에 안전하지 않습니다._

4. (a:Animal) => Greyhound 가정하면 f(g)는 타입 안전한가?  
_이 경우는 안전합니다. f(g(x))가 호출되고 나면 어떤 종류의 Dog의 메소드를 사용할 수 있습니다. 모든 Dog는 Animal이기 때문입니다._   

# 정리
결과적으로 이렇게 말할 수 있으면 안전하다(type safe)고 할 수 있습니다.

{% highlight typescript %}
  type F = (a: Dog) => Dog;
  type G = (a: Animal) => Greyhound;
{% endhighlight typescript %}

> G&nbsp;&nbsp;<&nbsp;&nbsp;F 이면 함수 합성에서 안전하다고 할 수 있습니다.
  
반환 타입은 간단합니다. Greyhound는 Dog의 서브타입이다. 하지만 인자 타입은 그 `반대`이며 `Animal은 Dog의 슈퍼 타입입니다!!` 이 동작 방식을 설명하면 함수 타입에서 반환 타입은 ***공변적(covariant)*** 이고, 인자 타입은 ***반공변적(contravariant)*** 인 것이 됩니다. 반환 타입의 공변성은 `A < B가 (T -> A) < (T -> B)로 적용된다는 뜻`이고 인자 타입의 반공변성은 `A < B가 (B -> T) < (A -> T)로 적용된다는 의미입니다.` 


# 적용
다음 예시는 제가 작성한 [rs-style-monads-ts](https://github.com/winverse/rs-style-monads-ts)라는 repo에 포함되어 있는 result.ts 코드 일부이며 [Rust result API](https://doc.rust-lang.org/std/result/enum.Result.html#)의 일부 구현체입니다. 또한 공변성과 반공변성 적용 사례이기도 합니다.  

{% highlight typescript %}
// 성공했을떄
export type Ok<T> = {
  readonly _tag: "Ok";
  readonly result: T;
};

// 에러가 발생했을때
export type Err<E> = {
  readonly _tag: "Err";
  readonly error: E;
};

// 부수효과를 일으키는 함수를 처리하는 기법으로 기본적으로 성공과 실패 타입을 가지고 있다.
export type Result<T, E> = Ok<T> | Err<E>;

// ok 함수는 부수효과를 일으키는 함수의 처리가 성공했다는 의미입니다.
// 반환 타입은 공변적이라는 것에 따라 사용하지 않는 Err 타입에 never를 사용하였습니다.
export const ok = <T>(value: T): Result<T, never> => ({
  _tag: "Ok",
  result: value,
});

// err 함수는 부수효과를 일으키는 함수의 처리가 실패했다는 의미입니다.
// 반환 타입은 공변적이라는 것에 따라 사용하지 않는 Ok 타입에 never를 사용하였습니다.
export const err = <E>(value: E): Result<never, E> => ({
  _tag: "Err",
  error: value,
});

// 인자 타입은 반공변적이라는 말에 따라서 사용하지 않는 Err 타입에 unknown을 사용하였습니다.
export const isOk = <T>(value: Result<T, unknown>): value is Ok<T> =>
  value._tag === "Ok";

// 인자 타입은 반공변적이라는 말에 따라서 사용하지 않는 Ok 타입에 unknown을 사용하였습니다.
export const isErr = <E>(value: Result<unknown, E>): value is Err<E> =>
  value._tag === "Err";
{% endhighlight typescript %}
  
  
# Reference
- https://github.com/microsoft/TypeScript/pull/18654
- https://seob.dev/posts/%EA%B3%B5%EB%B3%80%EC%84%B1%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80/
- https://velog.io/@lsb156/covariance-contravariance
- https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-6.html
- https://www.stephanboyer.com/post/132/what-are-covariance-and-contravariance
- https://doc.rust-lang.org/std/result/enum.Result.html