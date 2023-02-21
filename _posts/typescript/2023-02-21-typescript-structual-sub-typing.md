---
layout: post
author: winverse
title:  "잉여속성 체크와 구조적 서브 타이핑 (Feat: Typescript)"
description: 이번 글을 통해서 타입스크립트의 타입 시스템에 대해서 알아보고 더욱 견고한 소프트웨어를 만들어보는 방법에 대해서 알아봅시다.
tags: Typescript Type-level-system
category: typescript
---


# 정보
이 글은 [시지프의 타입스크립트 도약하기](https://www.youtube.com/watch?v=kMuJz6N-Grwab_channel=%EC%9A%B0%EC%95%84%ED%95%9CTech)을 보고 작성한 글입니다.
자세한 내용은 영상을 보면 크게 도움이 됩니다.

# 1. 타입스크립트의 타입시스템 딥 다이브
## 1-1. 타입스크립트는 기본적으로 집합을 따르는 구조적 서브 타이핑(structual sub typing)을 따라감

{% highlight typescript %}
let unknownType: unknown = "";
unknownType = 123;
unknownType = "123123";
unknownType = () => {};

let neverType: never = ""; // error
neverType = 123; //error
{% endhighlight typescript %}

## 1-2. 그러나 잉여속성 체크와 (excess property check)와 any는 구조적 서브 타이핑을 따라가지 않음

{% highlight typescript %}
type Optional = {
name: string;
darkMode?: boolean;
};

function a(op: Optional) {
if (op.darkMode) {
  console.log("dark mode");
  return;
}
return op;
}

const option = {
name: "blue",
darkmode: true, // camelcase를 따르지 않음 type Optional과 맞지 않음
};

a(option); // 구조적 서브 타이핑을 따름

a({ name: "blue", darkmode: true }); // error => 객체 리터럴은 잉여속성체크 수행함
{% endhighlight typescript %}

## 1-3. any에 대해서
- any는 모든 타입에 할당 가능하다 -> any는 최하위 집합이다.
- any는 모든 타입에 할당 가능하다 -> any는 최상위 집합이다.
- 1과 2가 충돌한다 -> any는 기본적으로 타입 시스템을 따르지 않는다.
- 결론: 어떤 타입이 들어올지 모를 경우, 모든 타입이 들어올 수 있을 경우 any가 아닌 unknown을 사용한다.

# 2. 더 나은 타이핑을 위한 액션 아이템

## 2-1. 함수의 반환 타입을 명시하여 의도를 표기하자
> 타입 추론에 의존하지 않고, 의도를 타입으로 명시한다.

{% highlight typescript %}
const addZero = (num: number): stirng => { // 리턴 타입 명시
return Math.floor(num / 10) === 0 ? `${num}` : String(numm);
}
{% endhighlight typescript %}

## 2-2. 구별된 유니온 사용하기
> 여러가지 상황을 하나의 인터페이스에 다 담아서 애매모호하게 하지 말고, 각 상황별 타입을 나누어서 적용하자  

{% highlight typescript %}
// BAD
interface fetchResult<T> {
data: T | null;
isLoading: boolean;
error: Error | null;
}

// GOOD
interface IdleResult {
data: null;
isLoading: false;
error: null;
}

interface LoadingResult {
data: null;
isLoading: true;
error: null;
}

interface SuccessResult {
data: T;
isLoading: false;
error: null;
}

interface ErrorResult {
data: null;
isLoading: false;
error: Error;
}

type Result<T> = IdleResult | SuccessResult<T> | LoadingResult | ErrorResult;

// 이렇게 정의하는 것이 react-query useQuery의 반환 타입이다.
{% endhighlight typescript %}

> 유니온의 인터페이스 보다는 인터페이스의 유니온을 사용하자!!

## 2-3. any를 잘쓰기
> 함수 호출 결과로 any를 쓰게 되는 상황이 발생한다면 차라리 함수 내부에서 any를 하고   
> 반환 타입을 잘 명시해서 any가 외부로 전파되지 않도록 하자!

{% highlight typescript %}
type Member = {
  type: "front" | "back";
  name: string;
  age: number;
};

const noAnyParseMembers = (memberse: Member[]) => {
  return memberse.reduce((prev, member) => {
    const key = member.type;
    return { ...prev, [key]: [...prev[key], member] };
  }, {});
};

const allMember: Member[] = [
  { type: "front", name: "Sang", age: 30 },
  { type: "back", name: "hyun", age: 30 },
];

const member1 = noAnyParseMembers(allMember) as Record<
  "front" | "back",
  Member[]
>; // member1의 타입이 {}로 표기 되어 as 키워드가 필요함

const parseMembers = (
  memberse: Member[]
): Record<"front" | "back", Member[]> => {
  return memberse.reduce((prev, member) => {
    const key = member.type;
    return { ...prev, [key]: [...(prev as any)[key as any], member] };
  }, {}) as any;
};

const member2 = parseMembers(allMember); // 함수에 반환 타입이 명시되어 as 키워드를 사용하지 않음
{% endhighlight typescript %}

# Reference
1. [시지프의 타입스크립트 도약하기](https://www.youtube.com/watch?v=kMuJz6N-Grwab_channel=%EC%9A%B0%EC%95%84%ED%95%9CTech)