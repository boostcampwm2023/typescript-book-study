# 6주차 스터디 내용 정리

## 2.29 직접 타입 만들기

### 판단하는 타입

타입스크립트에서 특정 타입을 판단하는 타입을 만들 수 있습니다.

#### IsNever

```typescript
type IsNever<T> = [T] extends [never] ? true : false;
```

제네릭 인수 T가 never로 확장될 수 있다면 never를 반환하고 그렇지 않으면 false를 반환하는 타입입니다.
제네릭 인수를 배열로 감싸는 이유는 T에 never를 넣을 때 분배법칙이 일어나는 것을 막기 위해서입니다.

> #### info::QUIZ
>
> IsNever 타입을 만들 때 제네릭 인수와 never를 배열로 감싸지 않는다면 Res의 타입은 어떻게 될까?
> (참고로 `type Never = never extends never ? true : false` 에서 Never의 타입은 ture입니다.)
>
> ```typescript
> type IsNever\<T> = T extends never ? true : false;
> type Res = IsNever<string|number>;
> ```
>
> 1. false
> 2. true
> 3. never
> 4. any
> 5. unknown
>
> - 정답
>   - 3번, 타입스크립트에서 조건부 타입에서 유니온은 분배법칙이 적용됩니다. 하지만 never는 비어있는 유니온으로 간주되어서 아무런 멤버가 없기 때문에 조건부 타입이 적용되지 않고 never가 반환됩니다.
> [Generic conditional type T extends never ? 'yes' : 'no' resolves to never when T is never. · Issue #31751 · microsoft/TypeScript](https://github.com/microsoft/TypeScript/issues/31751)

#### IsAny

```typescript
type IsAny<T> = string extends (number & T) ? true : false;
```

- string과 number는 겹치지 않아서 extends 할 수 없지만, T가 any라면 `number & any` 의 타입이 `any`가 되므로 위 타입이 true가 될 수 있습니다.

> #### info::QUIZ
>
> **아래 내용은 string & any가 string이 된다는 잘못된 증명입니다.
> 1번부터 5번중 어떤 전제가 잘못되었는지 찾아보세요**
>
> 명제: string & any의 타입은 string이 된다.
>
> 1. any는 전체집합이다.
> 2. &는 교집합이다.
> 3. 만약 집합 A가 집합 B의 부분집합일 때, A와 B의 교집합은 A가 된다.
> 4. string은 any의 부분집합이다.
>
> 결론: 따라서 string & any의 타입은 string이다.
>
> - 정답
>   - any는 전체집합이 아니라 모든 타입의 슈퍼타입이 될 수도 있고 모든 타입의 서브 타입이 될 수도 있는 특수한 타입입니다.

#### IsArray

```typescript
type IsArray<T> = IsNever<T> extends true
 ? false
 : T extends readonly unknown[]
  ? IsAny<T> extends true
   ? false
   : true
  : false;
```

타입이 배열인지 판단하려면 아래와 같은 조건을 모두 통과해야 합니다. 왜냐하면 T가 never, any, readonly [] 타입일 때는 false가 되지 않기 때문입니다.

- `IsArray<never>`가 never가 되는 것을 막기 위해 `IsNever<T> extends true`
- `IsArray<any>`가boolean이 되는 것을 막기 위해 `IsAny<T>extends true`
- any는 모든 타입으로 extends 될 수 있기 때문에 조건부 타입에서 true와 false의 유니언 타입을 반환합니다.
- `IsArray <readonly[]>`가 false가 되는 것을 막기 위해 `T extends readonly unknown[]`

#### IsTuple

```typescript
type IsTuple<T> = IsNever<T> extends true
 ? false
 : T extends readonly unknown[]
  ? number extends T["length"]
   ? false
   : true
  : false;
```

배열은 length가 number타입이지만, 튜플은 개별 숫자입니다. 따라서 `number extends T["length"]`를 통해 배열과 튜플을 구분할 수 있습니다.

#### IsUnion

```typescript
type IsUnion<T, U = T> = IsNever<T> extends true
 ? false
 : T extends T
  ? [U] extends [T]
   ? false
   : true
  : false;
```

### 집합 관련 타입

#### 차집합

```typescript
type Diff<A, B> = Omit<A & B, keyof B>;
type R1 = Diff<{ name: string ,age: number }, { name: string, married: boolean }>;
// type R1 = { age: number }
```

Omit 타입을 사용해서 차집합(Diff) 타입을 만들 수 있습니다.
Omit 타입은 특정 객체에서 지정한 속성을 제거하는 타입입니다. 위 예시에서 A&B의 타입은 `{ name: string, age: number, married: boolean }`인데 keyof B는 `name | married`이므로 name과 married 속성이 제거된 { age: number만 남게 됩니다}

#### 대칭차집합

- 객체의 대칭 차집합

```typescript
type SymDiff<A, B> = Omit<A & B, keyof (A | B)>;
type R2 = SymDiff<{ name: string, age: number }, { name: string, married: boolean }>;
```

- 유니언의 대칭 차집합

```typescript
type SymDiffUnion<A, B> = Exclude<A | B, A & B>;
type R3 = SymDiffUnion<1 | 2 | 3, 2 | 3| 4>;
```

- Exclude는 어떤 타입에서 다른 타입을 제거하는 타입입니다.

#### Equal

아래 예제는 두 타입이 동일하다는 것을 판단할 때 사용하는 타입입니다.
하지만 이 타입은은 any와 다른 타입을 구별하지 못합니다.

```typescript
type Equal<A, B> = [A] extends [B] ? [B] extends [A] ? true : false : false;

type ResAny = Equal<any, 1>; // ture
```

- any를 구별하도록 Equal 타입을 수정하기

```typescript
type Equal<X, Y>
 = (<T>()=> T extends X ? 1 : 2) extends (<T>() => T extends Y ? 1 : 2)
  ? true
  : false
```

- `(<T>()=> T extends X ? 1 : 2)`와 `(<T>() => T extends Y ? 1 : 2)`의 타입이 동일하다면 X와 Y는 동일한 타입입니다.
- 여기서 제네릭 인수 T는 타입을 비교하기 위한 장치로 사용됩니다.

## 2.31 데코레이터 함수

### 데코레이터 예제코드

```typescript
function startAndEnd<This, Args extends any[], Return>(
 originalMethod: (this: This, ...args: Args) => Return,
 context: ClassMethodDecoratorContext<This, (this: This, ...args: Args) => Return>
){
 function replacementMethod(this: This, ...args: Args): Return {
  console.log('start');
  const result = originalMethod.call(this, ...args);
  console.log('end');
  return result;
 }
 return replacementMethod;
}

class A {
 @startAndEnd
 eat() {
  console.log('Eat');
 }
 
 @startAndEnd
 work() {
  console.log('Work');
 }
 
 @startAndEnd
 sleap() {
  console.log('Sleap');
 }
}
```

- 데코레이터 메서드는 기존 메서드의 this(This), 매개변수(Args), 반환값(Return)을 타입 매개변수로 선언합니다.
- 데코레이터의 두번째 인자인 context는 아래와 같은 종류가 있습니다.
- ClassDecoratorContext: 클래스 자체를 장식할 때  
- ClassMethodDecoratorContext: 클래스 메서드를 장식할 때  
- ClassGetterDecoratorContext: 클래스의 getter를 장식할 때
- ClassSetterDecoratorContext: 클래스의 setter를 장식할 때  
- ClassMemberDecoratorContext: 클래스 멤버를 장식할 때  
- ClassAccessorDecoratorContext: 클래스 accessor를 장식할 때
- ClassFieldDecoratorContext: 클래스 필드를 장식할 때

#### 데코레이터에서 매개변수를 받는 방법

```typescript
function startAndEnd(start = 'start', end = 'end'){
 return function RealDecorator<This, Args extends any[], Return>(
  originalMethod: (this: This, ...args: Args) => Return,
  context: ClassMethodDecoratorContext<This, (this: This, ...args: Args) => Return>
 ){
  function replacementMethod(this: This, ...args: Args): Return {
   console.log(context.name, start);
   const result = originalMethod.call(this, ...args);
   console.log(context.name, end);
   return result;
  }
 }
 return replacementMethod;
}
```

데코레이터 자체도 함수라서 매개변수를 받을 수 있습니다. 다만, 이 경우 고차함수를 활용해서 데코레이터 함수를 한겹 더 감싸야합니다.

#### context.addInitializer

```typescript
function bound(originalMethod: unknown, context: ClassMethodDecoratorContext<any>){
 const methodName = context.name;
 if(context.kind === 'method'){
  context.addInitializer(function () {
   this[methodName] = this[methodName].bind(this);
  });
 }
}

export class C {
 @bound
 eat() {
  console.log('Eat');
 }

 work(){
  console.log('Work');
 }
}

const c = new C();
```

context.addInitializer에 등록한 함수는 클래스의 인스턴스를 생성할 때 호출됩니다.
즉 위 코드에서 C 클래스가 초기화 될 때 `this.eat = this.eat.bind(this)`가 호출됩니다.

## 2.32 앰비언트 선언

### 앰비언트 선언을 사용하는 이유

타입스크립트에서 라이브러리를 사용할 때 라이브러리에 타입이 정의되어있지 않으면 직접 타이핑을 해야 되는 경우가 생깁니다.
이 때 사용하는 것이 앰비언트 선언입니다. 아래 코드에서 보는 것 처럼 앰비언트 선언을 위해서는 declare 예약어를 사용해야 합니다.

```typescript
declare namesplace NS{
 const v: string
}
declare enum Enum{
 ADMIN = 1
}
declare function func(param: number): string;
declare const variable: number;
declare class C {
 constructor(p1: string, p2: string);
};

new C(func(varable), NS.v);
```

앰비언트 선언 파일에는 구현부가 없고 타입만 존재합니다. 하지만 맨 아래줄에 보이는 것 처럼 값으로 사용할 수 있습니다.
왜냐하면 타입스크립트가 외부 파일에 실제로 구현부가 존재한다고 믿기 때문입니다.
이 때 외부 파일에 값이 없으면 코드 실행시 런타임 에러가 발생합니다. 따라서 앰비언트 선언을 할 때는 해당 값이 실제로 존재하는지 확인하는 것이 중요합니다.

### namespace, enum을 declare로 선언하는 이유

- namespace를 declare로 선언하면 내부 멤버의 구현부를 생략할 수 있습니다.
- enum을 declare로 선언하면 자바스크립트로 변환할 때 실제 코드가 생성되지 않습니다.

### 같은 이름의 다른 선언과 병합 가능 여부

| 병합 가능 여부 | 네임스페이스 | 클래스 | enum | 인터페이스 | 타입 별칭 | 함수 | 변수 |
|:--------------:|:------------:|:------:|:----:|:----------:|:---------:|:----:|:----:|
|  네임스페이스  |      O       |   O    |  O   |     O      |     O     |  O   |  O   |
|     클래스     |      O       |   X    |  X   |     O      |     X     |  O   |  X   |
|      enum      |      O       |   X    |  O   |     X      |     X     |  X   |  X   |
|   인터페이스   |      O       |   O    |  X   |     O      |     X     |  O   |  O   |
|   타입 별칭    |      O       |   X    |  X   |     X      |     X     |  O   |  O   |
|      함수      |      O       |   O    |  X   |     O      |     O     |  O   |  X   |
|      변수      |      O       |   X    |  X   |     O      |     O     |  X   |  X   |

### 선언 병합을 활용하면 좋은 경우???

```typescript
declare class A {
 constructor(name: string);
}
function A(name: string){
 return new A(name);
}

new A('milk717');
A('milk717');
```

위와 같이 선언하면 클래스가 있을 때 new 키워드를 붙이지 않고도 사용할 수 있게 해줍니다.
> [!question]
> 이 코드 실행해보면 콜스택 에러가 나는데 왜그럴까요???

```typescript
function Ex() { return 'hello'; }
namespace Ex {
 export const a = 'wordl';
 export type B = number;
}
Ex();
Ex.a;
const b: Ex.B = 123;
```
