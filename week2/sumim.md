> #### info::출처
>
> 이 내용은 [타입스크립트 교과서](https://product.kyobobook.co.kr/detail/S000208416779) 책을 읽고 스터디한 내용을 바탕으로 정리했습니다.

<!-- TOC -->
- [타입과 집합의 다른 점](#타입과-집합의-다른-점)
	- [interface 상속하기](#interface-상속하기)
	- [type 상속하기](#type-상속하기)
	- [상속시 기억할 것](#상속시-기억할-것)
	- [어떤 타입이 더 넓은 타입일까?](#어떤-타입이-더-넓은-타입일까)
	- [타입 이름이 다르더라도 구조가 같으면 대입할 수 있다](#타입-이름이-다르더라도-구조가-같으면-대입할-수-있다)
	- [타입에서 중복된 부분은 제네릭으로 해결](#타입에서-중복된-부분은-제네릭으로-해결)
	- [제네릭 표기 위치](#제네릭-표기-위치)
		- [함수 표현식에서 제네릭 표기](#함수-표현식에서-제네릭-표기)
		- [함수 선언문에서 제네릭 표기](#함수-선언문에서-제네릭-표기)
	- [제네릭에 기본값 지정해두기](#제네릭에-기본값-지정해두기)
	- [제네릭의 타입 추론](#제네릭의-타입-추론)
	- [타입 매개변수 (T)](#타입-매개변수-t)
	- [제네릭에 제약 걸기](#제네릭에-제약-걸기)
		- [자주 사용되는 제약들](#자주-사용되는-제약들)
	- [제약은 타입 매개변수가 아님!](#제약은-타입-매개변수가-아님)
	- [컨디셔널 타입 설정](#컨디셔널-타입-설정)
	- [컨디셔널 타입이 자주 사용되는 곳](#컨디셔널-타입이-자주-사용되는-곳)
		- [타입을 검사할 때](#타입을-검사할-때)
		- [nerver와 함께 사용](#nerver와-함께-사용)
<!-- TOC -->

### optional 타이핑

```typescript
interface Exmaple{
	hello: string;
	world?: number;
}
```

위와 같이 객체의 타입을 선언하면, optional 타이핑을 할 수 있습니다.
이 타입을 사용하는 객체는 아래와 같은 형태가 될 수 있습니다.

```typescript
const example1: Example = { hello: 'hello' };
const example2: Example = { hello: 'hello', world: 'world' };
```

optional로 선언한 word 프로퍼티의 타입은  number | undefined가 됩니다.
따라서 속성이 존재하지 않아도 되고, 아래와 같이 undefined를 대입할 수도 있습니다.

```typescript
const example2: Example = { hello: 'hello', world: undefined };
```

### readonly 타이핑

```typescript
interface Exmaple{
	hello: string;
	readonly wow: boolean;
	readonly multiple?: symbol;
}
```

위와 같이 readonly 키워드를 사용해서 객체 안에 있는 프로퍼티를 읽기 전용 타입으로 정의할 수 있습니다.
그리고 multiple 프로퍼티처럼 optional과 readonly 이렇게 동시에 여러개의 수식어를 붙일 수 있습니다.

### 잉여 속성 검사 (Excess Property Checking)

```typescript
interface Exmaple{
	hello: string;
}

const example: Example = {
	hello: 'hi',
	why: '에러 발생',
}
/*
Type '{ hello: string; why: string; }' is not assignable to type 'Example'.  
Object literal may only specify known properties, and 'why' does not exist in type 'Example'.(2322)
*/

const obj = {
	hello: 'hi',
	why: '이건 에러 안남',
}
const example2: Example = obj;
```

Example 인터페이스에 why 속성이 없으므로 example 변수에서 에러가 발생합니다. 하지만 example2에서는 에러가 발생하지 않습니다. 왜냐하면, example 객체는 객체 리터럴을 대입했고, example2는 obj 객체 자체를 대입했기 때문입니다. 이 두 방식은 타입을 검사하는 방법이 다릅니다.
객체 리터럴을 이용해서 대입하면 잉여 속성 검사가 실행됩니다. 잉여 속성 검사는 타입에서 선언하지 않은 속성을 사용할 때 에러를 표시하는 것을 의미합니다. 하지만 변수를 대입할 때는 객체 간 대입 가능성을 비교하게 됩니다.

### 구조 분해 할당 타이핑시 주의점

```typescript
const { prop: { nested: string } } = {
	props: { nested: 'hi' },
}
console.log(nested);
```

구조분해 할당의 타입은 위와같이 하면 안됩니다. 위 코드는 nested의 타입을 string으로 표기한 것이 아니라 nested의 속성 값을 string이라는 변수 이름에 대입한 것입니다.

> #### info::INFO
>
> ``` typescript
> const obj = { id: 'sumin', age: 22 };
> const { id, age } = obj;
> const { id: username, age: year } = obj;
> console.log(username)
> ```
>
> 3번째 줄에 보이는 것 처럼 구조분해 할당한 속성의 이름을 다른 변수명으로 치환할 수 있습니다.
> 때문에  `{ prop: { nested: string } }` 으로 타입을 표기하게 된다면 nested로 구조분해 할당된 속성의
> 변수명을 string에 대입하는 꼴이 됩니다.

구조분해 할당된 객체의 타입을 올바르게 표기 하려면 아래와 같이 해야합니다.

```typescript
const { prop: { nested } }: { prop: { nested: string } } = {
	prop: { nested: 'hi' },
}

console.log(nested);
```

### 인덱스 접근 타입 (Indexed Access Type)

```typescript
type Animal = {
	name: string;
}

type N = string;
```

객체 안에 있는 속성에 커스텀 타입을 붙이고 싶을 때는 어떻게 해야 할까요? 위와 같이 하게 되면 타입을 변경해야 할 때 마다 두곳 모두 변경해야 합니다. 하지만 이건 번거로운 작업입니다. 특정 속성에 연동되도록 타입을 작성해서 한곳에서 변경하면 두곳 모두 바뀌도록 하는 것이 좋은 방법인 것 같습니다.
이럴 때는 아래와 같이 작성할 수 있습니다.

```typescript
type Animal = {
	name: string;
}

type N1 = Animal['name'];
```

자바스크립트에서 객체의 속성에 접근하듯 접근해서 타입을 지정해주면 됩니다. 다만 '객체.속성' 꼴의 방식은 사용할 수 없습니다.

> #### warning::타입 표기법에 대해 이야기하고싶은 내용
>
> ``` typescript
> type Animal = {
> name: Name;
> }
> type Name = string;
> ```
>
> 위처럼 표기하는게 더 가독성도 높고 편하지 않을까요??

### 인덱스 접근 타입의 타입 구하기

자바스크립트에서 변수의 타입이 궁금할 때는 `typeof 변수명` 키워드를 통해 구할 수 있습니다.
타입스크립트에서도 동일한 방법을 사용해서 타입을 구할 수 있습니다.
그리고 keyof 키워드를 사용해 키의 타입도 구할 수 있습니다.

```typescript
const obj = {
	hello: 'world',
	name: 'zero',
	age: 28,
}
type Keys = keyof typeof obj;
// type Keys = 'hello' | 'name' | 'age'
type Values = typeof obj[Keys];
// type Values = string | number
```

### keyof 속성 더 알아보기

### 객체에 인덱스 시그니처 사용하기

```typescript
type HelloAndHi = {
	[key: 'hello' | 'hi' ]: string;
}
/*
An index signature parameter type cannot be a literal type or generic type. Consider using a mapped object type instead.(1337)
*/

type HelloAndHi2 = {
	[key in 'hello' | 'hi' ]: string;
}
```

인덱스 시그니처를 사용할 때 처럼 객체에서도 속성에 전부 타입을 지정하지 않고 일부 속성에만 타입을 부여할 수 있습니다. 하지만 `HelloAndHi` 처럼 키의 타입을 지정할 순 없습니다.
왜나하면 인덱스 시그니처에 사용할 수 있는 타입은 string, number, symbol, 템플릿 리터럴 타입과 이들의 유니언 뿐입니다.

그래서 매핑된 객체 타입을 사용해서 `HelloAndHi2` 처럼 타입을 지정해야합니다. in 연산자를 사용하면 인덱스 시그니처가 표현하지 못하는 타입을 표기할 수 있습니다.
매핑된 객체 타입이란 기존의 다른 타입으로부터 새로운 객체 속성을 만들어내는 타입을 의미합니다.
인터페이스에서는 사용하지 못하고 타입 별칭에서만 사용할 수 있습니다.

### 매핑된 객체 타입 사용 예시

```typescript
interface Original = {
	name: string;
	age: number;
	married: boolean;
}
type Copy = {
	[key in keyof Original]: Original[key];
}
```

기존의 타입을 복사해야 할 때 위와 같이 사용할 수 있습니다.
그리고 매핑된 객체 타입으로 다른 타입을 가져올 때도 readonly나 optional 설정을 할 수 있습니다.

반대로 타입을 복사해서 가져올 때 수식어를 제거한채로 가져올 수도 있습니다.

```typescript
interface Original = {
	readonly name?: string;
	readonly age?: number;
	readonly married?: boolean;
}
type Copy = {
	-readonly [key in keyof Original]-?: Original[key];
}
```

타입을 복사해서 가져올 때 이름을 변경할 수도 있습니다.

```typescript
interface Original = {
	name: string;
	age: number;
	married: boolean;
}
type Copy = {
	[key in keyof Original as Capitalize<key>]: Original[key];
}
```

### 타입과 집합의 유사한 점

| 집합 | 타입스크립트 타입 |
| ----| --------------|
| 전체집합 | unknown |
| 공집합 | never |
| 교집합 | & |
| 합집합 | \| |

### 유니언 타입

타입스크립트에서 유니언 타입은 일종의 합집합 역할을 합니다.

```typescript
let strOrNum: string | number = 'hello';
strOrNum = 123;
```

위 예시의 경우 `strOrNum` 의 타입은 string과 number의 합집합이라고 생각할 수 있습니다.
하지만 변수에 대입된 값이 두개의 타입을 모두 만족할 수는 없습니다. 이 경우 교집합을 나타내는 부분은 never 타입이 됩니다.

### 인터섹션 타입

```typescript
type nev = string & number;
// type nev = never;
```

타입스크립트에서 인터섹션 타입은 교집합을 나타냅니다.

## 타입과 집합의 다른 점

```typescript
type H = { a: 'b' } & number;
// type H = { a: 'b' } & number;
```

null/undefined를 제외한 원시 자료형과 비어 있지 않은 객체를 & 연산할 때는 교집합이 never가 되지 않습니다.
즉, 타입 H는 never가 되지 않습니다. 객체의 타입과 number는 겹치는 타입이 없어서 never가 되야하는데 그렇지 않습니다. 이는 타입스크립트의 예외 사항입니다. 자세한 것은 뒤에 공부할 2.28절에서 알아보겠습니다.
자바스크립트에서 extends 키워드를 사용해서 클래스를 상속하듯이 타입도 상속할 수 있습니다.

### interface 상속하기

```typescript
interface Animal {
	name: string;
}
interface Dog extends Animal {
	bark(): void;
}
interface Cat extends Animal {
	meow(): void;
}
```

### type 상속하기

```typescript
type Animal = {
	name: string;
}
type Dog = Animal & {
	bark(): void;
}
type Cay = Animal & {
	meow(): void;
}
type Name = Cat['name'];
```

### 상속시 기억할 것

- 타입 별칭이 인터페이스를 상속할 수도 있고, 그 반대도 가능합니다.
- 여러개 타입을 상속할 수도 있습니다.
- 상속할 때 부모의 타입을 변경할 수도 있습니다. 단, 더 좁은 타입으로만 변경 가능합니다.
  - ex) string -> 'hello'

```typescript
interface A {
	name: string;
}
interface B {
	name: string;
	age: number;
}
```

[[객체를 타이핑하기#잉여 속성 검사 (Excess Property Checking)]] 에서 보면 객체 리터럴이 아닌 변수를 대입할 때는 잉여 속성 검사가 진행되지 않는다고 합니다. 따라서 변수를 대입할 때는 객체 간 대입 여부를 판단해야 합니다.
위 인터페이스의 경우 B 타입 객체를 A타입 변수에 할당할 수는 있지만, A 타입 객체를 B타입 변수에 할당할 수 없습니다.
왜냐하면 타입은 좁은 타입에서 넓은 타입으로 대입하는 것만 가능하기 때문입니다.
A타입이 B타입보다 더 넓은 타입이므로 B타입에서 A타입으로 대입이 가능합니다.

### 어떤 타입이 더 넓은 타입일까?

![어떤 타입이 더 넓은 타입일까?](../images/week2_sumin_img1.png)

### 타입 이름이 다르더라도 구조가 같으면 대입할 수 있다

제곧내

```typescript
interface Money {
	amount: number;
	unit: string;
}

interface Liter {
	amount: number;
	unit: string;
}

const liter: Liter = { amount: 1, unit: 'liter' };
const circle: Money = liter;
```

위의 예시처럼 `Money` 와 `Liter`는 서로 다른 타입이지만 구조가 같기 때문에 대입할 수 있습니다.

```typescript
interface Money {
	__type: 'money';
	amount: number;
	unit: string;
}

interface Liter {
	__type: 'liter';
	amount: number;
	unit: string;
}
```

구조는 같지만 이름은 다른 타입 간에 대입을 막으려면 위의 예시처럼 \_\_type 같은 브랜드 속성을 추가할 수 있습니다.

### 타입에서 중복된 부분은 제네릭으로 해결

```typescript
interface Person<N, A> {
	type: 'human',
	race: 'yellow',
	name: N,
	age: A,
}

interface Zero extends Person<'zero', 28> {}
interface Nero extends Person<'nero', 32> {}
```

위와 같이 중복되는 부분은 제네릭 타입을 통해 해결할 수 있습니다.
제네릭 표기는  <>로 하며 인터페이스 이름 바로 뒤에 위치합니다.

### 제네릭 표기 위치

#### 함수 표현식에서 제네릭 표기

```typescript
const personFactoryE = <N, A>(name: N, age: A)=> ({
	type: 'human',
	race: 'yellow',
	name,
	age,
});
```

#### 함수 선언문에서 제네릭 표기

```typescript
function personFactoryE<N, A>(name: N, age: A) {
	return ({
		type: 'human',
		race: 'yellow',
		name,
		age,
	})
};
```

### 제네릭에 기본값 지정해두기

```typescript
interface Person<N = string, A = number> {
	type: 'human',
	race: 'yellow',
	name: N,
	age: A,
}
```

함수 매개변수에 기본 값을 지정해두는 것 처럼 제네릭도 이와 마찬가지로 기본 타입을 지정해둘 수 있습니다.

### 제네릭의 타입 추론

```typescript
interface Person<N = string, A = number> {
	type: 'human',
	race: 'yellow',
	name: N,
	age: A,
}

const personFactoryE = <N, A = unknown>(name: N, age: A): Person<N, A> => ({
	type: 'human',
	race: 'yellow',
	name,
	age,
});

const zero = personFactoryE('zero', 28);
```

`personFactoryE` 에서 A에 기본값으로 unknown이 지정되어 있지만 함수 호출시 대입한 number 타입이 더 좁은 타입이므로 number로 추론됩니다.

### 타입 매개변수 (T)

```typescript
function values<T>(initial: T[]){
	return {
		hashValue(value T){ return initial.includes(value) },
	};
}

const savedValues = values(['a', 'b', 'c']);
savedValues.hasValue('x');
```

위와 같이 T를 사용해 타입 매개변수를 선언할 경우 타입스크립트가 타입을 추론합니다.
위 예시의 경우 savedValues의 타입은 string[]으로 추론됩니다.
만약 T를 string 대신 'a' | 'b' | 'c'와 같은 유니언으로 추론되게 하려면 타입 매개변수 앞에 const 키워드를 붙이면 됩니다.

### 제네릭에 제약 걸기

타입 매개변수에는 extends 문법으로 타입에 제약을 표시할 수 있습니다. 제약을 걸어두면 그 제약과 동일하거나 혹은 더 구체적인 타입만 대입할 수 있습니다.

#### 자주 사용되는 제약들

| 의미 | 코드 |
| ------- | ---------------- |
| 모든 객체 | `<T extends object>` |
| 모든 배열 | `<T extends any[]>` |
| 모든 함수 | `<T extends (...args: any)>` |
| 생성자 타입 | `<T extends abstract new (...args: any) => any>` |
| 모든 타입 | `<T extends keyof any>` |

### 제약은 타입 매개변수가 아님!

```typescript
interface V0 {
	value: any;
}

const returnV0 = <T extends V0>(): T => {
	return { value: 'test' };
}
```

위 예시처럼 타입을 선언할 수 없습니다. 왜냐하면 현재 제네릭 타입 매개변수는 V0라는 제약이 걸려있기 때문에 V0 뿐만 아니라 V0보다 더 좁은 타입도 대입이 가능합니다. 예를들어 { value: any, key: string } 타입도 V0보다 좁은 의미의 타입이기 때문에 대입 가능합니다. 하지만 returnV0 함수의 반환값은 V0만을 만족하기 때문에 에러가 발생합니다.

```typescript
function onlyBoolean<T extends boolean>(arg: T = false): T{
	return arg;
}
```

위와 같은 경우에 arg의 기본값으로 false를 넣으면 에러가 발생합니다. 왜냐하면 타입 매개변수 T에 boolean 제약이 걸려있기 때문입니다. boolean 타입 제약이면never 타입도 대입이 가능합니다. 따라서 T가 boolean이라고 확정된 것이 아니기 때문에 기본값을 설정할 수 없습니다.

### 컨디셔널 타입 설정

```typescript
type A1 = string;
type B1 = A1 extends string ? number : boolean;
```

자바스크립트에서 삼항연산자를 사용할 때 처럼 타입 또한 조건에 따라 다르게 설정할 수 있습니다.
그리고 자바스크립트처럼 중첩해서 사용할 수도 있습니다.

### 컨디셔널 타입이 자주 사용되는 곳

#### 타입을 검사할 때

```typescript
type = Result = 'hi' extends string ? true : false;
type = Result2 = [1] extends [string] ? true : false;
```

#### nerver와 함께 사용

```typescript
type OmitByType<O, T> = {
	[K in keyof O as O[k] extends T ? never : K]: O[K];
};
type Result = OmitByType<{
	name: string;
	age: number;
	married: boolean;
	rich: boolean;
}, boolean>;
```

매핑된 객체 타입에서 키가 nerver이면 해당 속성이 제거되기 때문에 위 예시처럼 사용할 수도 있습니다.
