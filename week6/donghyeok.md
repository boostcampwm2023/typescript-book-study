# 6주차 스터디 내용 정리

> #### info::출처
>
> 이 내용은 [타입스크립트 교과서](https://product.kyobobook.co.kr/detail/S000208416779) 책을 읽고 스터디한 내용을 바탕으로 정리했습니다.

<!-- TOC -->
- [6주차 스터디 내용 정리](#6주차-스터디-내용-정리)
- [3장 lib.es5.d.ts 분석하기](#3장-libes5dts-분석하기)
  - [3.1 Partial, Required, Readonly, Pick, Record](#31-partial-required-readonly-pick-record)
    - [Partial 타입](#partial-타입)
    - [Required 타입](#required-타입)
    - [Readonly 타입](#readonly-타입)
    - [Pick 타입](#pick-타입)
    - [Record 타입](#record-타입)
  - [3.2 Exclude, Extract, Omit, NonNullable(분배법칙을 활용하는 타입들)](#32-exclude-extract-omit-nonnullable분배법칙을-활용하는-타입들)
    - [Exclude 타입](#exclude-타입)
    - [Extract 타입](#extract-타입)
    - [Omit 타입](#omit-타입)
    - [NonNullable 타입](#nonnullable-타입)
  - [3.3 Parameters, ConstructorParameters, ReturnType, InstanceType (infer를 활용한 타입들)](#33-parameters-constructorparameters-returntype-instancetype-infer를-활용한-타입들)
  - [3.4 ThisType](#34-thistype)
  - [3.5 ~ 3.9 배열 내장 메서드 타이핑하기(forEach, map, filter, reduce, flat)](#35--39-배열-내장-메서드-타이핑하기foreach-map-filter-reduce-flat)
  - [3.9 flat 분석하기](#39-flat-분석하기)
  - [3.10 Promise, Awaited 타입 분석하기](#310-promise-awaited-타입-분석하기)
  - [3.11 bind 분석하기](#311-bind-분석하기)

<!-- TOC -->


<br />

# 3장 lib.es5.d.ts 분석하기

‘타입스크립트는 어떻게 타입을 선언했는지’를 보면서 타입 선언 방법이나 기술을 익히면 좋다.

> 확장자가 단순히 ts가 아니라 .d.ts인 이유는, 타입스크립트의.d.ts 파일에는 타입 선언만 있고 실제 구현부가 없기 때문이다. 자바스크립트 문법은 따로 구현되어 있기에 타입스크립트에서는 타입 선언만 제공하는 것이다.
> 

<br />

## 3.1 Partial, Required, Readonly, Pick, Record

`Partial, Required, Readonly, Pick, Record` 는 타입스크립트 공식 사이트의 유틸리티 타입에서 매핑된 객체 타입을 사용하는 것만 추린 것이다.

→ 유틸리티 타입이 너무 많고 Record를 뺴고 실질적으로 아직 사용을 해본 적이 없어서 학습하기에 정말 좋다.

<br />

### Partial 타입

기존 객체의 속성을 전부 옵셔널로 만든다.

```typescript
type MyPartial<T> = {
    [P in keyof T]? : T[P];
};

type Result = MyPartial<{ a: string, b: number }>;
type A = Partial<{ a: string, b: number }>;
```

A와 Result는 같다. Partial을 MyPartial로 구현한 것이다.

이전 학습에도 나왔듯이 MyPartial로 한 이유는 이미 Partial이 lib.es5.d.ts에 선언되어있기 때문에 이미 선언되어있다는 declared 오류가 나타나기 때문에 My로 테스트한다.

<br />

### Required 타입

Partial과 반대로 모든 속성을 옵셔널이 아니게 만든다.

```typescript
type MyRequired<T> = {
    [P in keyof T]-? : T[P];
}

type Result = MyRequired<{ a?: string, b?: number}>;
type A = Required<{ a?: string, b?: number }>;
```

<br />

### Readonly 타입

P.188 교재 오류

```
type MyReadonly<T> = {
    readonly [P in keyof T]:
};

type Result = MyReadonly<{ a: string, b: number }>;
```

교재에서 이렇게 나와있는데 타이핑을 해보니 오류가 나와서 직접 까봤다.

```typescript
/**
 * Make all properties in T readonly
 */
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};
```

이렇게 구현부를 작성하는 것이 맞다.

```typescript
type MyReadonly<T> = {
	readonly [P in keyof T]: T[P];
};

type ReverseMyReadonly<T> = {
	-readonly [P in keyof T]: T[P];
};

type Result = MyReadonly<{ a: string, b: number }>;
```

반대로 Readonly가 아니게 되려면 구현부에 -readonly로 작성하면 된다.

<br />

### Pick 타입

객체에서 지정한 속성만 추리기

```typescript
type MyPick<T, K extends keyof T> = {
    [P in K]: T[P];
};

type Result = MyPick<{ a: string, b: number, c: number }, 'a' | 'c'>;
```

K 타입 파라미터는 T 객체의 속성 이름이어야 하므로 `extends keyof T` 제약을 주었다.
MyPick 타입을 사용해 keyof T인 a, b, c 중에서 a, c만 추릴 수 있게 된다.

<br />

### Record 타입

```typescript
type MyRecord<K extends keyof any, T> = {
    [P in K]: T;
};

type Result = MyRecord<'a' | 'b', string>;
```

`K extends keyof any`를 통해 K에 string | number | symbol로 제약조건을 설정

제약은 가능하면 엄격하게 거는 것이 좋다.
속성 이름으로 사용할 수 없는 값을 K로 제공하는 실수를 막을 수 있기 때문이다.

<br />

## 3.2 Exclude, Extract, Omit, NonNullable(분배법칙을 활용하는 타입들)

2.15.1절에서 컨디셔널 타입일 때 유니언인 기존 타입과 제네릭이 만나면 분배법칙이 실행된다고 배웠다.

이 절에서 배우는 타입은 모두 분배법칙을 활용한다.

<br />

### Exclude 타입

어떠한 타입에서 지정한 타입을 제거한다.

```typescript
type MyExclude<T, U> = T extends U ? never : T;
type Result = MyExclude<1 | '2' | 3, string>; // type Result = 1 | 3
```

1 | ‘2’ | 3은 유니언 타입으로 분배법칙이 실행된다.

여기서 Exclude의 특징으로 1 | 3이 된다.

<br />

### Extract 타입

Exclude 타입과 반대이다.
컨디셔널 타입의 참, 거짓 부분만 서로 바꾸면 된다.

```typescript
type MyExtract<T, U> = T extends U ? T : never;
type Result = MyExtract<1 | '2' | 3, string>; // type Result = "2"
```

<br />

### Omit 타입

특정 객체에서 지정한 속성을 제거한다.

```typescript
type MyOmit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
type Result = MyOmit<{ a: '1', b: 2, c: true }, 'a' | 'c'>; // type Result = { b: 2 }
```

Pick 타입은 객체에서 지정한 속성만 추리는 특징을 가지고 있다.

Omit 타입은 Pick 타입과 반대되는 행동을 한다.
그러면서 Omit은 Pick과 Exclude 타입을 활용한다.

먼저 `Exclude<keyof T, K>`를 하여 지정한 속성을 제거한다.
’a’ | ‘b’ | ‘c’ 중에 ‘a’ | ‘c’를 지정했으니 ‘b’만 추려진다.
Pick 타입을 통해 객체에서 추려낸 속성을 선택한다. 최종적으로 ‘b’ 속성만 있는 객체 타입이 남게 된다.

<br />

### NonNullable 타입

타입에서 null, undefined를 제거한다.

```typescript
type MyNonNullable<T> = T extends null | undefined ? never : T;
type Result = MyNonNullable<string | number | null | undefined>;

type MyNonNullable<T> = T & {}; // 요즘은 이렇게 변경되었음
```

요즘 바뀐 문법을 보면 {}의 경우 `null, undefined`는 포함하지 않는다.
따라서 T와 {}의 교집합은 `string | number`이다.

<br />

## 3.3 Parameters, ConstructorParameters, ReturnType, InstanceType (infer를 활용한 타입들)

이 유틸리티 타입들은 Infer를 활용한 타입들이다.

infer을 저번에 학습했지만 워낙 어려워서 적당히 읽고 넘겼는데 한번 더 복습을 할 수 있게 되었다.

```typescript
type MyParameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never;
type MyConstructorParameters<T extends abstract new (...args: any) => any> = T extends abstract new (...args: infer P) => any ? P : never;
type MyReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
type MyInstanceType<T extends abstract new (...args: any) => any> = T extends abstract new (...args: any) => infer R ? R : any;
```

`abstract new (…args: any) ⇒ any`는 무엇일까요?
우선 `new (…args: any) ⇒ any`는 모든 생성자 함수를 의미하는 타입이다. 클래스를 포함해서, 다만 추상 클래스는 포함하지 않는다.
추상 클래스까지 포함하려면 `abstract new (…args: any) ⇒ any`로 타이핑해야한다.

<br />

## 3.4 ThisType

메서드들에 this를 한 방에 주입하는 타입

<br />

```typescript
const obj = {
    data: {
        money: 0,
    },
    methods: {
        addMoney(amount: number) {
            this.money += amount;
        },
        useMoney(amount: number) {
            this.money -= amount;
        }
    }
}

// mone에서 Property 'money' does not exist on type '{ addMoney(amount: number): void; useMoney(amount: number): void; }'.(2339)
// 이런 오류가 발생한다.
```

addMoney, useMoney 같은 메서드에서 this를 쓰고 싶은 상황이다.

하지만 이 예제에서 this는 obj 객체가 아니라 data, methods 객체를 합친 타입이다.
즉 this.data.money로 접근하는 것이 아니라 this.money로 바로 접근할 수 있게 하고 싶은 것

```typescript
type Data = { money: number };
type Methods = {
    addMoney(amount: number): void;
    useMoney(amount: number): void;
};
type Obj = {
    data: Data;
    methods: Methods & ThisType<Data & Methods>;
};
const obj: Obj = {
    data: {
        money: 0,
    },
    methods: {
        addMoney(amount: number) {
            this.money += amount;
        },
        useMoney(amount: number) {
            this.money -= amount;
        }
    }
}
```

<br />

## 3.5 ~ 3.9 배열 내장 메서드 타이핑하기(forEach, map, filter, reduce, flat)

위의 내용들은 교재를 읽으며 흐름만 확인하였습니다.

<br />

## 3.9 flat 분석하기

배열에 있는 메서드로 ES2019에 추가된 메서드이다.
사용하려면 TS config에서 target을 ES2019로 변경해야한다.

이 부분도 교재를 정독하자,,

정리하기에는 양이 방대하고 따라 적는 느낌만 들 것 같아서 생략한다.

(참고로 flat 메서드는 나는 사용을 할 일이 없다고 판단되어 스킵했다.)

<br />

## 3.10 Promise, Awaited 타입 분석하기

<br />

## 3.11 bind 분석하기

- "typescript": "^5.2.2”

```typescript
/**
* For a given function, creates a bound function that has the same body as the original function.
* The this object of the bound function is associated with the specified object, and has the specified initial parameters.
* @param thisArg An object to which the this keyword can refer inside the new function.
* @param argArray A list of arguments to be passed to the new function.
*/
bind(this: Function, thisArg: any, ...argArray: any[]): any;

/**
 * For a given function, creates a bound function that has the same body as the original function.
 * The this object of the bound function is associated with the specified object, and has the specified initial parameters.
 * @param thisArg The object to be used as the this object.
 */
bind<T>(this: T, thisArg: ThisParameterType<T>): OmitThisParameter<T>;

/**
* For a given function, creates a bound function that has the same body as the original function.
* The this object of the bound function is associated with the specified object, and has the specified initial parameters.
* @param thisArg The object to be used as the this object.
* @param args Arguments to bind to the parameters of the function.
*/
bind<T, A extends any[], B extends any[], R>(this: (this: T, ...args: [...A, ...B]) => R, thisArg: T, ...args: A): (...args: B) => R;

/**
* For a given function, creates a bound function that has the same body as the original function.
* The this object of the bound function is associated with the specified object, and has the specified initial parameters.
* @param thisArg The object to be used as the this object.
*/
bind<T>(this: T, thisArg: any): T;

/**
* For a given function, creates a bound function that has the same body as the original function.
* The this object of the bound function is associated with the specified object, and has the specified initial parameters.
* @param thisArg The object to be used as the this object.
* @param args Arguments to bind to the parameters of the function.
*/
bind<A extends any[], B extends any[], R>(this: new (...args: [...A, ...B]) => R, thisArg: any, ...args: A): new (...args: B) => R;
```

교재보다 코드가 아주 많이 간략화되었음을 확인할 수 있다.# 3장 lib.es5.d.ts 분석하기

‘타입스크립트는 어떻게 타입을 선언했는지’를 보면서 타입 선언 방법이나 기술을 익히면 좋다.

> 확장자가 단순히 ts가 아니라 .d.ts인 이유는, 타입스크립트의.d.ts 파일에는 타입 선언만 있고 실제 구현부가 없기 때문이다. 자바스크립트 문법은 따로 구현되어 있기에 타입스크립트에서는 타입 선언만 제공하는 것이다.
> 

<br />
