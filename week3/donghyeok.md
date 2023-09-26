# 3주차 스터디 내용 정리

> #### info::출처
>
> 이 내용은 [타입스크립트 교과서](https://product.kyobobook.co.kr/detail/S000208416779) 책을 읽고 스터디한 내용을 바탕으로 정리했습니다.

<!-- TOC -->
- [3주차 스터디 내용 정리](#3주차-스터디-내용-정리)
  - [2.16 함수와 메서드를 타이핑하자](#216-함수와-메서드를-타이핑하자)
  - [2.17 같은 이름의 함수를 여러 번 선언할 수 있다](#217-같은-이름의-함수를-여러-번-선언할-수-있다)
  - [2.18 콜백 함수의 매개변수는 생략 가능하다](#218-콜백-함수의-매개변수는-생략-가능하다)
  - [2.19 공변성과 반공변성을 알아야 함수끼리 대입할 수 있다](#219-공변성과-반공변성을-알아야-함수끼리-대입할-수-있다)
  - [2.20 클래스는 값이면서 타입이다](#220-클래스는-값이면서-타입이다)
    - [private과 #의 차이점?](#private과-의-차이점)
    - [implements하는 인터페이스의 속성은 전부 public이어야 한다](#implements하는-인터페이스의-속성은-전부-public이어야-한다)
    - [Override](#override)
  - [2.20.1 추상 클래스(abstract class)](#2201-추상-클래스abstract-class)
  - [2.21 enum은 자바스크립트에서도 사용할 수 있다](#221-enum은-자바스크립트에서도-사용할-수-있다)
<!-- TOC -->

<br />

## 2.16 함수와 메서드를 타이핑하자

```typescript
function example(a: string, b?: number, c = false) {}

example('hi', 123, true);
example('hi', 123);
example('hi');

function example2(...a: string[], b: number) {} // error
```

이 함수의 파라미터를 확인해보면 b에는 옵셔널이 달려있어서 없어도 됩니다. 
그리고 **c의 경우 디폴트 값이 false로 지정되어있는데 디폴트 값이 제공된 파라미터는 자동으로 옵셔널 처리가 됩니다.**

example2 함수에서는 에러가 발생한다.
당연하다싶이 a는 첫 파라미터인데 전개연산자 …를 사용했으니 에러가 난다.
… 전개연산자는 파라미터 상에서 마지막 파라미터에만 사용이 가능하다.

```typescript
function destructuring({ prop: { nested } }) {} // nested error
destructuring({ prop: { nested: 'hi' }});

function destructuring({ prop: { nested: string } }) {} // string error
destructuring({ prop: { nested: 'hi' }});
```

이 앞장의 그 복잡했던 코드가 또 다시 등장했다.

매개변수에 구조분해 할당을 적용할 때 다음과 같이 작성하면 implicitAny에러가 발생한다.

두번째 함수의 경우 nested라는 변수를 string 타입으로 정의한 것 같지만 실상은 그렇지 않다.

이 문법은 nested 속성을 string 변수로 이름을 바꾼 것이기 때문에 오류가 발생한다.

```typescript
function destructuring({ prop: { nested } }: { prop: { nested: string }}) {}
destructuring({ prop: { nested: 'hi' }});
```

다음과 같이 코드를 작성해야한다.

다시봐도 굉장히 복잡하다,, ts에서 제공하는 에러코드에서도 이정도까지 친절하게 수정하라고 알려주는 것이 아니기 때문에
학습이 필요하다.

함수 내에서 this를 사용하려면 매개변수 첫 번째 자리에 표기를 해야하며 타입을 명시적으로 표기해야한다.

타입을 표기하지않으면 any 타입으로 추론되고 에러가 발생한다.

또한 자바스크립트에서는 함수를 생성자로 사용할 수 있고 new를 붙여서 객체를 만들 수 있다.

하지만 타입스크립트에서는 기본적으로 함수를 생성자로 사용할 수 없으며 class를 사용해야한다.

이는 처음 자바를 사용하다가 자바스크립트를 접했을 때 당황스러웠다는 분들이 많은데 타입스크립트를 사용하면 해결할 수 있는 문제 중 하나라고 생각한다.

그리고 나 역시 class 문법을 사용하고 prototype을 직접적으로 사용하지는 않기 때문에 (생성자를 함수로 만들지도 않는다.)
원래 사용하던대로 하면 될 것 같다.

```typescript
class Person {
    name: string;
    age: number;
    married: boolean;
    constructor(name: string, age: number, married: boolean) {
        this.name = name;
        this.age = age;
        this.married = married;
    }

    sayName() {
        console.log(this.name);
    }
}
const zero = new Person("zero", 28, false);
```

아주 친화적이다.
우리가 평소에 사용하던 그 class 문법에 타입을 지정해준 것 뿐

또한 교재에서도 생성자 함수 대신 클래스를 사용하는 방법을 권장한다고 한다.

<br />

## 2.17 같은 이름의 함수를 여러 번 선언할 수 있다

당연히 오버로딩의 개념인 것 같다.

같은 이름의 함수를 여러번 재작성할 수 있다 라면 오버라이딩일 것이다.

- 오버로딩(overloading)이란

호출할 수 있는 함수의 타입을 미리 여러 개 타이핑해두는 기법이다.
다소 다르다. 타입스크립트는 매개변수에 어떤 타입과 값이 들어올지 미리 타입을 선언해두어야하기 때문에 다음과 같다.

교재에서 x + y 함수를 작성했지만 타입을 string | number로 해서
문자열, 숫자 값을 더하게 되는 자바스크립트와 같은 모양새가 되는 것을 볼 수 있다.

그래서 타입스크립트에서는 다음과 같은 상황에 이렇게 작성할 수 있다.

```typescript
function add(x: number, y: number): number
function add(x: string, y: string): string
function add(x: any, y: any) {
	return x + y;
}

add(1, 2) // 3
add('1', '2') // 12
add(1, '2') // error
```

다소 생소하게 느낄 수 있다. 나 또한 그랬다.

함수에서 구현부는 없고 타입만 선언해두었다. 또한 구현부가 있는 함수의 경우 파라미터의 타입을 any로 지정하였다.

여기서는 any가 사용되었지만 해당 함수 add는 이미 정의되어있는 두 개의 함수를 오버로딩한 것이기 때문에 값은 any가 아니게 되는 것이다.

```typescript
function example(param: string): string;
function example(param: string | null): number;
function example(param: string | null): string | number {
	if (param) {
        return 'string';
    } else {
        return 123;
    }
}

const result = example('what');
```

오버로딩을 선언하는 순서에도 타입 추론에 영향을 끼친다.

코드는 위부터 아래로 순차적으로 실행된다.
그래서 보통 마지막으로 정의된 함수의 타입을 따를 것이라고 생각하지만
오버로딩에 동시에 해당될 수 있는 경우는 제일 먼저 선언된 오버로딩에 해당된다.

result는 string 타입을 띈다.
’what’을 집어넣었을 때 example의 첫 번째 선언한 함수에서 param: string이면 string을 반환하는 것이다.

저 위에서 L1, L2를 바꾸면 number 타입이 된다.

<br />

## 2.18 콜백 함수의 매개변수는 생략 가능하다

문맥적 추론에 대해서 읽고 지나가면 될 듯하다.

콜백 함수의 경우 매개변수가 생략하다라는 말은

```typescript
function example(callback: (error: Error, result: string) => void {}
example((e, r) => {});
example(() => {});
example(() => true);
```

콜백함수의 error, result의 타입을 정의해주었기 때문에 파라미터 작성이 생략 가능하다.

여기서 특이점은 옵셔널이 아닌데도 
L3, 4의 코드가 동작한다는 것이다.

콜백 함수의 파라미터 특징 중 하나는 함수를 포출할 때 사용하지 않아도 된다.

L3처럼 콜백 함수에 error, result 파라미터 자리가 없어도 호추링 가능하다.

> 여기서 error, result를 ?를 붙여서 옵셔널을 붙이는 실수를 가장 많이 한다고 하니 주의하자.
옵셔널로 만들면 error, result 타입이 각각 error | undefined, string | undefined가 되어버리기 때문에 의도가 달라질 수 있다.
> 

L1에서 ⇒ void로 반환값이 void일 경우에는 어떠한 반환값이 와도 상관없다.

그래서 L4를 해도 에러가 발생하지 않는다.

어떠한 반환값이든 상관이 없을 경우에는 void를 사용하자. (any는 사절)

<br />

## 2.19 공변성과 반공변성을 알아야 함수끼리 대입할 수 있다

2.7.6절에서 타입 간 대입 가능성에 대해 배웠다.
이 절에서는 함수 간 대입 가능성에 대해 배우면서 공변성과 반공변성에 대한 개념을 알아본다.

논리회로 게이트 연산과 느낌 비슷하다고 느꼈다.

공변성: A → B일 때 `T<A> → T<B>` (AND)

반공변성: A → B일 때 `T<B> → T<A>` (NOT, 인버터)

이변성: A → B일 때 `T<A> → T<B>, T<B> → T<A>` 둘다 가능 (OR)

무공변성 A → B일 때 `T<A> → T<B>, T<B> → T<A>` 둘다 불가능 (NOR?? 이건 애매)

TS Config 메뉴에서

strictFunctionTypes 옵션이 체크가 되어 있어야 한다.
이 옵션은 strict 옵션이 체크되어 있을 때 자동으로 활성화되며 strict, strictFunctionTypes 모두 체크되어 있지 않다면 타입스크립트는 파라미터에 대해 이변성을 갖는다.

```typescript
function a(x: string): number {
    return 0;
}

type B = (x: string) => number | string;
let b: B = a;
```

b는 a보다 넓은 타입이다.
이 두 타입의 차이는 반환값 뿐이며 a 함수를 b 타입에 대입할 수 있다.
a의 반환 값을 b의 반환값에 대입 가능 ⇒ a → b

여기서 T 타입을 함수<반환값>이라고 생각하면 a → b일 때 `T<a>와 T<b>`간 관계를 파악하면 된다.
해당 코드는 공변성을 가지고 있다고 볼 수 있다.

```typescript
function a(x: string): number | string {
    return 0;
}

type B = (x: string) => number;
let b: B = a;
```

이렇게 바꾸면 대입이 불가능해진다.

b는 a보다 좁은 타입이다.
이 두 타입의 차이는 반환값 뿐이며 a 함수를 b 타입에 대입할 수 없다.
b의 반환 값을 a의 반환값에 대입 불가능 b → a 불가능

- 항상 공변성을 가짐을 역 명제를 통해 추론 가능

이정도까지만 하고 뒷 내용은 각자 교재를 유심히 읽어보자.

<br />

## 2.20 클래스는 값이면서 타입이다

```typescript
class Person {
    name: string;
    age: number;
    married: boolean;
    constructor(name: string, age: number, married: boolean) {
        this.name = name;
        this.age = age;
        this.married = married;
    }
}
```

위에서 다음과 같은 class를 보았을 것이다.

타입스크립트에서 작성하는 class이다.

자바스크립트의 class와 차이점은 타입스크립트는 name,. age, married 같은 멤버를 클래스 내부에서 한 번 적어야 한다.

위의 코드에서 name, age, married에 직접 작성한 타입은 생략할 수 있다. 타입스크립트가 알아서 추론할 수 있다.

```typescript
const Person = class {
    name;
    age;
    married;
    constructor(name: string, age: number, married: boolean) {
        this.name = name;
        this.age = age;
        this.married = married;
    }
}
```

이렇게 class 표현식으로도 작성할 수 있다.

여기서 주의할 점은 멤버는 항상 contructor 내부와 짝이 맞아야한다.

```typescript
class Person {
    name: string;
    married: boolean;
    constructor(name: string, age: number, married: boolean) {
        this.name = name;
        this.age = age;
    }
}
```

다음 코드에서 문제점은 한 눈에 봐도 확인이 가능하다.

1. age는 멤버로 선언이 되어있지 않았다.
2. married는 멤버로만 사용이 되었고 생성자 내부에서 할당이 되지 않았다.

```typescript
interface Human {
	name: string;
	age: number;
	married: boolean;
	sayName(): void;
}

class Person implements Human { // error
    name;
    age;
    married;
    constructor(name: string, age: number, married: boolean) {
        this.name = name;
        this.age = age;
        this.married = married;
    }
}
```

이렇게 implements 예약어를 사용하면 엄격하게 멤버 확인을 할 수 있다.

현재 human 인터페이스 안의 sayName을 사용하지않았다는 오류가 발생한다.

interface에 있는 것을 전부 다 구현해야한다.

클래스 멤버로 옵셔널, readonly뿐만 아니라 public, protected, private 수식어가 추가 되었다.

- public 속성: 선언한 자신, 자손 클래스, new 호출로 만들어낸 인스턴스에서 속성 사용 가능
- protected 속성: 자신의 클래스와 자손 클래스에서는 사용 가능하나 인스턴스에서 사용 불가능
- private: 자신의 클래스에서만 속성 사용 가능

protected는 아직 사용해본 경험이 없고, private, public은 사용을 해본 경험이있다.

그런데 private는 MVC 패턴을 구성하면서 service, repository class를 가져와서 그 곳에서만 사용할 것이기 때문에 private를 사용했는데 public은 보통적으로 붙여도 되지만 그냥 안붙이고 코드를 작성하고 있다.

<br />

### private과 #의 차이점?

자바스크립트에서는 private속성을 나타내기 위해서 속성 앞에 #을 붙이는 것을 자주 보았다.

private 수식어로 선언한 속성은 자손 클래스에서 같은 이름으로 선언 불가능, #은 가능
위에서 내 견해를 좀 적었는데 교재에서도 public 수식어는 생략해도 되므로 사용하지 않는다고 한다.
그리고 private는 #으로 사용하는 것을 선호하신다고 한다.

그런데 나는 #보단 private을 좀 더 쓸 것 같다.

<br />

### implements하는 인터페이스의 속성은 전부 public이어야 한다

애초에 인터페이스 속성은 protected, private이 될 수 없기 때문에 
인터페이스를 implments해서 사용할 경우 해당 클래스에서도 인터페이스의 속성들은 전부 public이어야한다.

<br />

### Override

override 수식어를 활용하려면 TS Config에서 noImplicitOverride 옵션이 체크되어 있어야 한다.

오버라이딩을 사용하려면 해당 메서드 앞에 override 수식어를 붙여야한다.

만약 한다면 나는 이 기능을 활용해볼 생각이다.
코드를 읽다가 오버라이딩 되었음을 확인할 수 있기 때문

클래스에서도 2.17절에 나온대로 생성자 함수를 여러번 선언해서 오버로딩을 할 수 있다.

```typescript
class Person {
    name?: string;
    age?: number;
    married?: boolean;
    constructor();
    constructor(name: string, married: boolean);
    constructor(name: string, age: number, married: boolean);
    constructor(name?: string, age?: boolean | number, married?: boolean) {
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

똑같이 타입 선언을 여러번 하면 되고 함수의 구현부는 한 번만 나와야 한다.

하지만 코드의 가독성이 상당히 떨어지는 것을 확인 가능하고 인덱스 시그니처를 사용할 수 있다.

- 이게 맞는 지 확인해보자,,

```typescript
class Person {
    [key: string]: string | number | boolean;

    constructor(name?: string, age?: number, married?: boolean) {
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

클래스나 인터페이스의 메서드에서는 this를 타입으로 사용할 수 있다.

```typescript
class Person {
    age: number;
    married: boolean;
    constructor(age: number, married: boolean) {
        this.age = age;
        this.married = married;
    }

    sayAge() {
        console.log(this.age); // this: this
    }

    sayMarried(this: Person) {
        console.log(this.married); // this: Person
    }

    sayCallback(callback: (this: this) => void) {
        callback.call(this);
    }
}
```

sayAge() 파라미터 안에 this: this를 사용해도 된다.

```typescript
class A {
    callbackWithThis(cb: (this: this) => void) {
        cb.call(this);
    }
    callbackWithoutThis(cb: () => void) {
        cb();
    }
}

new A().callbackWithThis(function() {
    this;
})

new A().callbackWithoutThis(function() {
    this;
})
```

this: this를 넣지 않은 경우 this를 사용할 수 없다.

콜백 함수에서 this를 사용하고 싶다면 this를 타이핑해야하고, 그 this가 클래스 자신 것이라면 this: this로 타이핑하면 된다.

<br />

## 2.20.1 추상 클래스(abstract class)

Java에서 많이 들어본 클래스이다.
정작 추상 클래스에 대한 이해도가 전무하다.

코드를 보고 기억난 점은 abstract class에서 선언되어있는 값들을 extends 상속 받아서 사용하면 
abstract가 달려있는 메서드들은 구현해야 한다는 것이다.

implements보다 조금 더 구체적으로 클래스의 모양을 정의한다.
코드를 작성하면서 마땅히 사용을 해보지가 않아서 한번 읽고 넘어간다.

<br />

## 2.21 enum은 자바스크립트에서도 사용할 수 있다

내가 아는 지식으로는 JS에서는 enum을 사용하지 않는다.

제대로 정독을 해본적은 없지만 사용하지 않는게 좋다는 글의 제목들도 몇 번 본 기억이 있다.

enum(열거형)이란 여러 상수를 나열하는 목적으로 쓰인다.

enum은 보통 첫 자를 대문자로 사용하는 네이밍 컨벤션을 가지고 그 안의 값들을 멤버라고 한다.

지금까지 배웠던 타입들은 JS로 변환할 때 모두 사라졌지만, enum은 사라지지 않고 자바스클비트 코드로 남는다.

하지만 코드 가독성이 굉장히 떨어지고 실제로 나도 사용하지 않는다.
또한 타입스크립트의 enum도 아직 완전하지가 않고 사용을 하지 않는 추세여서 넘긴다.