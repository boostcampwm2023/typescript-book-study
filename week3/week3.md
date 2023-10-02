# Week3

## 2.16, 2.17, 2.18 함수의 타이핑

함수에서는 배열이나 객체처럼 매개변수에 …(나머지) 문법을 사용할 수 있습니다. 배열과 객체와 같은 방법입니다.

```tsx
function test(a: string, ...arg: string[]) {}
function test1(...arg: number[], b: number) {}
```

대신 나머지 문법은 매개변수의 가장 마지막에 위치해야 합니다.

이를 이용해서 다음과 같이 이름이 지정되어 있지 않은 튜플을 만들 수도 있습니다.

```tsx
function tuple(...args: [number, boolean, string]) {}
```

다만 매개변수를 타이핑 해야할때 조심해야하는 점은 바로 객체에 구조분해 할당을 적용할 때 입니다.

```tsx
function test({ props: { prop1: string } }) {}
// error

function test2({ props }: { props: { prop1: string } }) {}
```

첫번째 방법으로 하면 prop1의 타입이 string이 되는것이 아니라 prop1의 별칭이 string으로 설정되는 것입니다. 따라서 두번째 방법으로 구조를 따로 타이핑 해줘야 합니다.

함수의 매개변수를 여러개로 오버로딩하여 사용할 수 있습니다. 실제 구현부에서는 오버로딩한 타입을 전부 합쳐서 명시해 주어야 합니다.

```tsx
function overloading(name: string): string;
function overloading(name: string | undefined): number;
function overloading(name: string | number | undefined): string | number {
  if (name) {
    return "string";
  } else {
    return 1;
  }
}

const result = overloading("string");
```

해당 매서드는 1개의 인자에 name이라는 프로퍼티의 타입에 따라 다른 return 타입을 주는것을 알 수 있습니다. 여기서 이 오버로딩을 하는 순서도 중요하는데요 위에서부터 읽기 때문에 같은 string을 입력하더라도 다음과 같은 차이가 발생 합니다.

```tsx
function overloading(name: string): string;
function overloading(name: string | undefined): number;
function overloading(name: string | number | undefined): string | number {
  if (name) {
    return "string";
  } else {
    return 1;
  }
}

const result = overloading("string");
// return type은 string

function overloading(name: string | undefined): number;
function overloading(name: string): string;
function overloading(name: string | number | undefined): string | number {
  if (name) {
    return "string";
  } else {
    return 1;
  }
}

const result = overloading("string");
// return type은 number;
```

인터페이스로도 오버로딩 사용을 할 수 있습니다.

```tsx
interface Add {
  (x: number, y: number): number;
  (x: string, y: string): string;
}

const add: Add = (x: any, y: any) => x + y;
```

![Untitled](../images/week3/week3/Untitled.png)

![Untitled](../images/week3/week3/Untitled1.png)

타입 별칭으로도 가능합니다.

```tsx
type a = (x: number, y: number) => number;
type b = (x: string, y: string) => string;

type Add = a & b;

const add: Add = (x: any, y: any) => x + y;
```

- 궁금한 점 - 태림

  ```tsx
  type Add = {
    (x: number, y: number): number;
    (x: string, y: string): string;
  };

  const add: Add = (x: any, y: any) => x + y;
  ```

  - 이렇게는 못 쓰나
  - 확인 결과 정상적으로 동작도 하는 것 같은데 인터페이스는 다음처럼 써 놓고 왜 타입은 굳이 `&` 로 예시를 보여 준 것일까?

    ```tsx
    interface Add {
      (x: number, y: number): number;
      (x: string, y: string): string;
    }

    const add: Add = (x: any, y: any) => x + y;
    ```

    → 인터페이스와 타입에서 많이 사용되는 패턴으로 설명해 준 것 같다는 결론~

![Untitled](../images/week3/week3/Untitled2.png)

![Untitled](../images/week3/week3/Untitled3.png)

함수가 콜백함수로 사용될때 타입스크립트는 자동으로 해당 매개변수를 추론해 줍니다.

이를 통해서 우리가 callback함수의 매개변수를 입력할때 해당 타입을 잘 지정해서 사용할 수 있는 것입니다.

```tsx
function func(callback: (res: string, req: number) => void) {}
```

![Untitled](../images/week3/week3/Untitled4.png)

## 공변성 반공변성

타입은 좁은타입에서 넓은 타입으로만 대입할수 있다고 했습니다. 그러면 함수도 좁은 타입에서 넓은 타입으로 대입할수 있을까요? 그러면 함수는 어떤게 좁은 타입일까요?

함수는 조금 특별하게? 단순히 좁은타입에서 넓은 타입으로 대입할 수 있는게 아니라 공변성과 반공변성을 따져야 합니다.

- 공변성(Covariance) : A가 B보다 좁은 타입이면, T<A>는 T<B>또한 대입할 수 있다.
- 반공변성(Contravariance) : A가 B보다 좁은 타입이면, T<B>는 T<A>또한 대입할 수 있다.
- 이변성(Bivariance) : A가 B보다 좁은 타입이면, T<A> → T<B>도 되고 T<B> → T<A>도 되는 경우
- 불변성(immutability) :A가 B보다 좁은 타입이더라도, T<A> → T<B>도 안 되고 T<B> → T<A>도 안 되는 경우

리턴값을 살펴 보면 number보다 number| string이 더 넓은 타입이므로 a타입은 B타입에 대입할 수 있습니다. 함수를 대입할때도 마찬가지로 대입할수 있죠 이는 리턴값은 공변성이기 때문입니다.

```tsx
function a(x: string): number {
  return 0;
}

type B = (x: string) => number | string;

let b: B = a;
```

그러면 매개변수를 살펴 보겠습니다. B의 매개변수가 a보다 더 넓은 타입이므로 a에서 b를 대입할 수 있습니다. 하지만 에러가 발생하죠

```tsx
function a(x: string): number {
  return 0;
}

type B = (x: string | number) => number;

let b: B = a;
```

이를 해결하려면 반대로 작성을 해주어야 하는데요 이는 매개변수가 반공변성이기 때문입니다.

```tsx
function a(x: string | number): number {
  return 0;
}

type B = (x: string) => number;

let b: B = a;
```

참고로 tsonfig의 `strictFunctionTypes` 을 꺼두게 된다면 매개변수는 반공변성이 아니라 이변성이 되어 좁은타입에서 넓은타입으로든 반대든 모두 대입이 가능하게 됩니다.

객체내부의 매서드 작성 방식에 따라 다른데요 함수로 작성할 경우에는 기존과 똑같이 반공변성을 갖지만 매서드로 정의할때는 이변성을 갖습니다.

```tsx
interface Method {
  say(a: string | number): string;
}

interface Func {
  say: (a: string | number) => string;
}

interface Obj {
  say: {
    (a: string | number): string;
  };
}

const func = (a: string) => "hi";

const methodObj: Method = {
  say: func,
};

const funcObj: Func = {
  say: func,
};
// err

const objObj: Obj = {
  say: func,
};
// err
```

이변성이라 양방향 다 가능합니다.

```tsx
interface Method {
  say(a: string): string;
}

const func = (a: string | number) => "hi";

const methodObj: Method = {
  say: func,
};
```

정리: 리턴값은 공변성을 가지고 매개변수는 반공변성을 가지면서 객체내부의 매서드형과 함수는 이변성을 갖는다.

공변성: 좁은타입에서 넓은 타입 대입가능

반공변성: 넓은 타입에서 좁은 타입 대입가능

이변성: 양방향 다 대입가능

- 궁금한 점 - 유석

  ```tsx
  interface SayMethod {
  	say(a:string|number):string
  }
  interface SayFunction  {
  	say : (a:string|number) => string;
  };
  interface SayCall {
  	say :{ (a:string|number):string}
  }
  const sayFunc = (a:string) => ”hello";

  const MyAddingMethod: SayMethod ={
  	say:sayFunc // 이변성
  };
  const MyAddingFunction: SayFunction ={say:sayFunc}; // 반공
  const MyAddingMethod: SayCall ={say:sayFunc}; // 반공
  ```

  - 함수 (매개변수) : 반환값으로 선언한 것은 매개변수가 이변성을 가짐
  - 함수 : (매개변수) ⇒ 반환값으로 선언한 것은 매개변수가 반공변성을 가짐
  - MyAddingMethod
    - sayFunc → SayMethod
      - 리턴 타입만 비교하면 ‘hello’가 string에 포함되니까 ok
      - 매개변수를 체크해보면 string|number를 string에 넣고 있으니까 X
      - but 함수(매개변수):반환값으로 선언한 것은 이변성을 가져서 ok.
  - MyAddingFunction
    - sayFunc → SayFunction
      - 함수 : (매개변수) ⇒ 반환값으로 반공변성만 체크하면 됨
      - 원래의 매개변수 체크하는 형식이니까 T<SayFunction> → T<sayFunc>인데
      - 매개변수를 체크해보면 string|number를 string에 넣고 있으니까 X

## class

타입스크립트에서 class는 값 이자 타입입니다.

```tsx
class Dog {
  constructor(public name: string) {}
  bark() {
    return "Woof!";
  }
}

// 값으로서의 사용
const myDog = new Dog("Buddy"); // Dog 클래스의 새 인스턴스를 생성합니다.

// 타입으로서의 사용
let anotherDog: Dog; // Dog 타입의 변수를 선언합니다.
anotherDog = new Dog("Max"); // Dog 인스턴스를 할당합니다.
```

자바의 타입과 같이 public protect private를 사용하여 프로퍼티를 제한 할 수 있습니다.

그러면 여기서 JS의 private과 중첩이 생기는데요. 저자는 책에서 JS의 private field(#)를 사용한다고 합니다. 둘의 큰 차이로는 JS의 기능이 extends될때 발생하는데요 JS의 private field는 부모의 private를 오버라이딩 할 수 있지만 Typescript의 private 키워드는 할 수 없습니다. 사실 private인데 오버라이딩 되는게 맞지 않다고 생각해서 저는 Typescript의 private 키워드를 사용할 것 같습니다.

**implement**

객체 지향의 interface를 구현하는것과 비슷합니다. class에 들어가야할 프로퍼티를 미리 지정해서 인터페이스를 준수하도록 강제시킬 수 있습니다.

```tsx
interface IPerson {
  name: string;
  sayHello(): void;
}

class Person implements IPerson {
  name: string;

  constructor(name: string) {
    this.name = name;
  }

  sayHello() {
    console.log(`Hello, ${this.name}`);
  }
}
```

abstract

반대로 객체의 추상클래스가 되도록 정의할 수 있는데요 자바의 abstract 키워드와 똑같습니다.

```tsx
abstract class AbstractAnimal {
  abstract makeSound(): void;

  move(): void {
    console.log("Moved a bit.");
  }
}

class Dog extends AbstractAnimal {
  makeSound() {
    console.log("Woof!");
  }
}
```

대신 implements는 js로 변환하지 않지만 abstract는 js로 변환합니다.

![Untitled](../images/week3/week3/Untitled5.png)

overriding

overriding을 할때 명시적으로 overriding한 매서드라고 지정할 수 있습니다. 설정의 `noImplicitOverride` 를 키면은 작동이 되는데요 overriding을 명시적으로 표현해 주어야 합니다.

```tsx
class Parents {
  sayHello() {
    console.log("hello");
  }
}

class Child extends Parents {
  override sayHello() {
    console.log("wow");
  }
}
```

- 궁금한 점 - 유석
  클래스의 생성자 함수에도 오버로딩을 적용할 수 있다. ⇒ 사용할까?
  클래스의 속성에도 인덱스 시그니처를 사용할 수 있다. ⇒ 사용할까?

## 2.21 enum

js에서는 enum이 없지만 타입스크립트에서는 지원을 해주고 있습니다. 양방향 map으로 서로 참조할 수 있습니다.

```tsx
"use strict";
var Level;
(function (Level) {
  Level[(Level["NOVICE"] = 0)] = "NOVICE";
  Level[(Level["INTERMEDIATE"] = 1)] = "INTERMEDIATE";
  Level[(Level["ADVANCED"] = 2)] = "ADVANCED";
  Level[(Level["MASTER"] = 3)] = "MASTER";
})(Level || (Level = {}));

enum Level {
  NOVICE,
  INTERMEDIATE,
  ADVANCED,
  MASTER,
}

const a = Level.ADVANCED;
const b = Level.INTERMEDIATE;
```

타입스크립트의 enum은 아직 완벽하지 않기 때문에 사용할때 조심해야 하고 대부분 값을 사용하는 것 보다는 타입을 정의할때 많이 쓰입니다. 특히 (브랜딩)

- 브랜딩 속성으로 사용할 때는 같은 enum의 멤버로 사용해야 한다
- 다른 enum의 멤버일 경우에는 가지고 있는 값(0, 1, 2와 같은)이 중복될 수 있기 때문

- 궁금한 점 - 유석
  enum의 타입을 사용하되 자바스크립트 코드가 생성되지 않게 할 수 있다.

  - const 키워드를 enum 앞에 붙인다.

  ```tsx
  "use strict";
  var Level;
  (function (Level) {
      Level[Level["NOVICE"] = 0] = "NOVICE";
      Level[Level["INTERMEDIATE"] = 1] = "INTERMEDIATE";
      Level[Level["ADVANCED"] = 2] = "ADVANCED";
      Level[Level["MASTER"] = 3] = "MASTER";
  })(Level || (Level = {}));

  const Mondy = ['WON','DOLLAR'];
  const Money ={
  	'WON':'0',
  ...
  }

  const enum Money {
  	WON, DOLLAR,
  }
  Money.WON; // O
  Money['WON'] // O
  Money[Money.WON]; // Error
  Money[0] // Error
  // 그냥 {WON:0, DOLLAR:1} 이렇게만 key value 생기는 게 아닐까
  ```

  - Money.won은 0 Money.DOLLAR은 1로 변환되지만 Money라는 자바스크립트 객체가 없기 때문에 Money[Money.WON]은 Error가 발생한다.
