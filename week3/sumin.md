<!-- TOC -->
- [2.16 함수와 메서드 타이핑](#216-함수와-메서드-타이핑)
  - [옵셔널 매개변수 타이핑](#옵셔널-매개변수-타이핑)
  - [나머지 매개변수 타이핑](#나머지-매개변수-타이핑)
  - [함수의 this 타이핑](#함수의-this-타이핑)
  - [메서드의 this 타이핑](#메서드의-this-타이핑)
  - [함수로 생성자를 만들었을 때 타이핑](#함수로-생성자를-만들었을-때-타이핑)
- [2.17 타입의 오버로딩](#217-타입의-오버로딩)
  - [오버로딩이 필요한 상황](#오버로딩이-필요한-상황)
  - [오버로딩 하는 방법](#오버로딩-하는-방법)
  - [오버로딩 순서](#오버로딩-순서)
  - [인터페이스로 오버로딩 표현하기](#인터페이스로-오버로딩-표현하기)
  - [타입 별칭으로 오버로딩 표현하기](#타입-별칭으로-오버로딩-표현하기)
  - [오버로딩할 때 주의할 점](#오버로딩할-때-주의할-점)
- [2.18 콜백 함수의 매개변수](#218-콜백-함수의-매개변수)
- [2.19 함수의 대입 가능 여부](#219-함수의-대입-가능-여부)
  - [공변성, 반공변성](#공변성-반공변성)
  - [변수, 함수, 메서드의 대입 가능 여부](#변수-함수-메서드의-대입-가능-여부)
- [2.20 클래스는 값이면서 타입](#220-클래스는-값이면서-타입)
  - [타입스크립트의 클래스](#타입스크립트의-클래스)
  - [클래스 인터페이스](#클래스-인터페이스)
  - [클래스에서 사용할 수 있는 수식어](#클래스에서-사용할-수-있는-수식어)
  - [클래스 메서드 오버라이딩](#클래스-메서드-오버라이딩)
  - [클래스 메서드 오버로딩](#클래스-메서드-오버로딩)
  - [클래스에서 인덱스 시그니처 사용하기](#클래스에서-인덱스-시그니처-사용하기)
  - [추상 클래스 사용하기](#추상-클래스-사용하기)
- [2.21 enum](#221-enum)
  - [열거형의 인덱스](#열거형의-인덱스)
  - [열거형의 문자열 인덱스 할당](#열거형의-문자열-인덱스-할당)
  - [enum이 사용되는곳](#enum이-사용되는곳)
<!-- TOC -->
## 2.16 함수와 메서드 타이핑
### 옵셔널 매개변수 타이핑
```typescript
function example(a: string, b?:number, c = false){}
example('hi', 123, true);    
example('hi', 123);          
example('hi');               
```
매개변수를 옵셔널로 지정하려면 매개변수 b처럼 물음표 기호를 사용해서 명시적으로 옵셔널을 표기하거나, 매개변수 c처럼 기본값을 지정해야합니다.
### 나머지 매개변수 타이핑
```typescript
function example1(a: string, ...b: number[]) {}
example1('hi', 123, 4, 56);
```
...기호를 이용해서 나머지 매개변수 문법을 사용할 때는 배열 혹은 튜플 타입으로 타이핑을 해야합니다.
왜냐하면 나머지 파라미터를 하나로 묶어 매개변수에 전달하기 때문에 배열 꼴일 수밖에 없기 때문입니다.
이 문법을 사용할 때 주의할 점은 spread 문법과 달리 나머지 매개변수는 항상 매개변수의 마지막 자리에만 위치해야 한다는 것입니다.
### 함수의 this 타이핑
```typescript
function example2(this: Window){
	console.log(this);
}

function example3(this: Document, a: string, b: 'this'){
	console.log(this);
}
example3('hello', 'this');    // 1번
//The 'this' context of type 'void' is not assignable to method's 'this' of type 'Document'.(2684)

example3.call(document, 'hello', 'this')  // 2번
```
this를 타이핑할때는 매개변수의 첫 번째 자리에 작성해야합니다. 이 경우 다른 매개변수는 한 자리씩 밀려납니다.
하지만 매개변수의 첫 번째 자리에 존재하는 this는 실제 매개변수가 아니기 때문에 example3의 1번 호출처럼 this의 타입을 실제와 다르게 작성하는 경우 에러가 발생합니다.
따라서 this의 값을 다르게 지정하고 싶은 경우 call 메서드 등을 활용해서 명시적으로 지정해야 합니다.
### 메서드의 this 타이핑
```typescript
type Animal = {
	age: number,
	type: 'dog',
}
const person = {
	name: 'zero',
	age: 28,
	sayName(){
		this.name;
	},
	sayAge(this: Animal){
		this.type;
	},
}

person.sayAge.bind({ age: 3, type: 'dog' });
```
메서드에서도 this를 사용할 수 있으나, 일반적으로 메서드의 this는 메서드를 갖고 있는 객체 자신으로 추론되므로 명시적으로 타이핑할 필요가 없습니다. 하지만 위 예시처럼 this가 바뀔 수 있을 경우에는 명시적으로 타이핑 해야 합니다.
### 함수로 생성자를 만들었을 때 타이핑
```typescript
type Person = {
	name: string,
	age: number,
	married: boolean,
}
interface PersonConstructor {
	new (name: string, age: number, married: boolean): Person;
}

const Person = function (this: Person, name: string, age: number, married: boolean){
	this.name = name;
	this.age = age;
	this.married = married;
} as unknown as PersonConstructor;
Person.prototype.sayName = function(this: Person){
	console.log(this.name);
}

const zero = new Person('zero', 28, false);
```
자바스크립트에서는 함수를 생성자로 사용할 수 있지만, 타입스크립트에서는 불가능하므로 class를 사용해야 합니다. 하지만 생성자의 타입과 인스턴스의 타입을 따로 만들고 생성자 함수를 `as unknown as PersonConstructor;` 를 사용해서 강제로 타입을 지정하는 방식으로 억지로 만들 수 있긴 합니다.
이는 부자연스러운 방법이므로 생성자 함수 대신 클래스를 사용하는 방법을 더 권장합니다.
## 2.17 타입의 오버로딩
### 오버로딩이 필요한 상황
```typescript
function add(x: string | number, y: string | number): string | number {
	return x + y;
}

add(1, 2);
add('1', '2');
add(1, '2'); 
add('1', 2);
```
만약 두 수를 더하거나, 두 문자열을 합치는 add라는 메서드를 타이핑할 때 위와 매개변수 타입을 유니언으로 작성하게 되면 문자열 + 숫자 조합의 매개변수 같은 원하지 않은 동작도 정상적으로 수행됩니다.
이 경우 타입의 오버로딩을 통해 호출할 수 있는 함수의 타입을 여러 개 타이핑 해둘 수 있습니다.
### 오버로딩 하는 방법
```typescript
function add(x: number, y: number): number
function add(s: stinrg, y: string): string
function add(x: any, y: any){
	return x + y;
}
```
첫 번째 줄과 두 번째 줄에는 함수의 선언부만 두고 이곳에 타입을 명시합니다. 그리고 최종적으로 함수의 구현부를 작성할 때 매개변수의 타입을 any로 선언하면 오버로딩이 완료됩니다.
구현부의 타입은 any로 작성했지만, 함수의 매개변수에 모든 타입이 가능한 것은 아닙니다. 함수의 선언부에서 명시한 타입의 조합만 가능합니다.
### 오버로딩 순서
```typescript
function example(param: string | null): number;
function example(param: string): string;
function example(param: string | null): string | number{
	if(param){
		return 'string';
	}else{
		return 123;
	}
}

const result = example('what');
```
위 예시에서 오버로딩 순서로 인해 문제가 발생합니다. 'what'은 string이기 때문에 첫 번째 오버로딩과 두 번째 오버로딩에 모두 해당될 수 있습니다. 이처럼 여러 오버로딩에 동시에 해당되는 경우 먼저 선언된 오버로딩에 해당됩니다. 따라서 result의 타입은 number가 되는데, 실제 반환값은 string이라서 오류가 발생합니다.
따라서 오버로딩의 순서는 좁은 타입부터 넓은 타입순으로 오게 해야합니다.
### 인터페이스로 오버로딩 표현하기
```typescript
interface Add{
	(x: number, y: number): number;
	(x: string, y: string): string;
}
const add: Add = (x: any, y: any) => x + y;

add(1, 2);
add('1', '2');
```
### 타입 별칭으로 오버로딩 표현하기
```typescript
type Add1 = (x: number, y: number) => number;
type Add2 = (x: string, y: string) => string;
type Add = Add1 & Add2;
const add: Add = (x: any, y: any) => x + y;

add(1, 2);
add('1', '2');
```

> #### warning::question
> 인터페이스와 타입 별칭 방법 중 어떤 방법을 더 선호하는지 궁금합니다!
> 그리고 타입스크립트로 프로젝트를 해보신 분들 중에 오버로딩을 사용해본적이 있는지 궁금해요.

### 오버로딩할 때 주의할 점
오버로딩이 필요하지 않은 부분에 지나치게 오버로딩을 했다가 문제가 발생하는 경우도 있습니다. 따라서 유니언이나 옵셔널 매개변수를 활용할 수 있는 경우는 오버로딩을 사용하지 않는 것이 좋습니다.
## 2.18 콜백 함수의 매개변수
```typescript
function example(callback: (error: Error, result: string) => void) {}
example((e, r) => {});
example(() => {});
example(() => true);
```
위와 같은 경우 example의 매개변수로 넘기게 되는 콜백함수의 매개변수가 error와 result인 경우도 있고, 존재하지 않는 경우도 있습니다. 이를 표현하기 위해 error?, result? 처럼 옵셔널로 적는 경우가 있는데 이렇게 되면 각각 매개변수의 타입이 undefined가 될 수 있으므로 의도와 맞지 않습니다. 따라서 콜백 함수의 매개변수는 생략이 가능합니다.
## 2.19 함수의 대입 가능 여부
### 공변성, 반공변성
|      속성       |              설명              |
|:--------------:|:----------------------------:|
|     공변성     | A -> B 일 때 T\<A> -> T\<B>인 경우 |
|   반공변성        | A -> B 일 때 T\<B> -> T\<A>인 경우 |
|    이변성     | A -> B 일 때 T\<A> -> T\<B>도 되고 T\<B> -> T\<A>도 되는 경우 |
|  무공변성    | A -> B 일 때 T\<A> -> T\<B>도 안 되고 T\<B> -> T\<A>도 안 되는 경우 |
### 변수, 함수, 메서드의 대입 가능 여부
| 종류 | 속성 | 
| :--: | :--: |
| 일반 변수 | 공변성 |
| 함수의 매개변수 (strict 활성화) | 반공변성 |
| 함수의 매개변수 (strict 비활성화) | 이변성 |
| 함수의 반환값 | 공변성 |
| 객체의 메서드 (strict 활성화, '함수: (매개변수) => 반환값' 형태) | 반공변성|
| 객체의 메서드 (strict 활성화, '함수(매개변수): 반환값' 형태) | 이변성|
## 2.20 클래스는 값이면서 타입
### 타입스크립트의 클래스
```typescript
class Person {
	name;
	age;
	married;
	constructor(name: string, age: number, married boolean){
		this.nmae = name;
		this.age = age;
		this.married = married;
	}
}
```
자바스크립트와 다르게 타입스크립트의 클래스는 클래스 내부에 멤버를 한번 적어주어야 하고, 멤버의 타입은 생략할 수 있습니다.
### 클래스 인터페이스
```typescript
interface Human{
	name: string;
	age: number;
	married: boolean;
	sayName(): void;
}
class Person implements Human {
	name;
	age;
	married;
	constructor(name: string, age: number, married: boolean){
		this.name = name;
		this.age = age;
		this.married = married;
	}
}
```
타입스크립트에서는 클래스에 대한 인터페이스를 작성하고 implements 예약어를 사용해 인터페이스를 구현한 클래스를 작성할 수 있습니다.
### 클래스에서 사용할 수 있는 수식어
![[Pasted image 20230928092946.png]]
### 클래스 메서드 오버라이딩
```typescript
class Human {
	eat(){
		console.log('냠냠');
	}
	sleap(){
		console.log('쿨쿨');
	}
}
class Employee extends Human {
	work() {
		console.log('끙차');
	}
	override sleap() {
		console.log('에고고');
	}
}
```
Ts Config 설정에서 noImplicitOverride 옵션이 체크되어 있는 경우 override 키워드를 사용할 수 있습니다.
### 클래스 메서드 오버로딩
```typescript
class Person {
	name?: string;
	age?: number;
	married?: boolean;
	constructor();
	constructor(name: string, married: boolean);
	constructor(name: string, age: number, married: boolean);
	constructor(name?: string, age?: boolean | number, married?: boolean){
		if (name) {
			this.name = name;
		}
		if (typeof age === 'boolean') {
			this.married = age;
		} else {
			this.age = age;
		}
		if (married) {
			this.married = married;
		}
	}
}
const person1 = new Person();
const person2 = new Person('nero', true);
const person3 = new Person('zero', 28, false);
```
함수처럼 클래스 생성자도 오버로딩을 사용할 수 있습니다. 단, 구현부는 한 번만 나와야 하고 구현부에서 모든 생성자의 파라미터에 대응할 수 있어야 합니다.
### 클래스에서 인덱스 시그니처 사용하기
```typescript
class Signature {
	[propName: string]: string | number | undefind;
	static [propName: string]: boolean;
}

const sig = new Signature();
sig.hellog = 'world';
Signature.isGood = ture;
```
### 추상 클래스 사용하기
abstract 키워드를 사용해서 추상 클래스를 사용할 수 있습니다. 추상 클래스 안에는 abstract 키워드가 붙은 추상 필드나 추상 메서드가 올 수 있습니다. 또한 실제 구현되어있는 메서드도 올 수 있습니다.
abstract 키워드가 붙은 타입스크립트 코드는 implements와 다르게 자바스크립트의 실제 클래스로 변환됩니다.
## 2.21 enum
```typescript
enum Level {
	NOVICE,
	INTERMEDIATE,
	ADVANCED,
	MASTER,
}
```
타입스크립트에서 enum 키워드를 사용해서 열거형을 사용할 수 있습니다. enum타입은 자바스크립트로 변환할 때 아래와 같은 형태가 됩니다.
```typescript
"use strict";
var Level;
(function (Level) {
	Level[Level["NOVICE"] = 0] = "NOVICE";
	Level[Level["INTERMEDIATE"] = 1] = "INTERMEDIATE";
	Level[Level["ADVANCED"] = 2] = "ADVANCED";
	Level[Level["MASTER"] = 3] = "MASTER";
})(Level || (Level = {}));
```
이렇게 변환되는 이유는 열거형은 양방향이기 때문입니다. 즉 `NOVICE`라는 멤버를 `Level[0]` 혹은 Level.NOVICE로 접근할 수 있습니다.
### 열거형의 인덱스
열거형은 맨 위쪽 멤버부터 차례대로 인덱스가 부여됩니다. 하지만 아래와 같이 = 연산자를 사용해서 직접 인덱스를 부여할 수도 있습니다.
```typescript
enum Level {
	NOVICE = 3,
	INTERMEDIATE,
	ADVANCED = 7,
	MASTER,    // 8
}
```
이 때 직접 인덱스를 부여하지 않은 멤버의 인덱스는 이전 값의 인덱스에 1을 더한 값이 됩니다.
### 열거형의 문자열 인덱스 할당
```typescript
enum Level {
	NOVICE,
	INTERMEDIATE = 'hello',
	ADVANCED = 'world',
	MASTER = 'bye',   
}
```
위 예제처럼 숫자 인덱스 대신에 문자열을 할당할 수도 있습니다. 다만 한 멤버를 문자열로 할당하면 그 다음 멤버부터는 전부 직접 값을 할당해야합니다.
### enum이 사용되는곳
```typescript
enum Money{
	WON,
	DOLLAR,
}

interface Won{
	type: Money.WON,
}
interface Dollar{
	type: Money.DOLLAR,
}
```
enum은 브랜딩을 할 때 자주 사용됩니다. 단 type속성에 오는 멤버는 같은 enum의 멤버여야만 구분이 가능합니다.
