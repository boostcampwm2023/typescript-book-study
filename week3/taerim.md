## 2.16 함수와 메서드를 타이핑하자

- 함수에 옵셔널과 기본값 사용 가능
    
    ```tsx
    function orderFood(
      itemName: string,
      quantity: number = 1, // quantity = 1도 number로 추론됨
      special?: string,
    ) {
      console.log(`Order: ${quantity} ${itemName}`);
    
      if (special) {
        console.log(`Special: ${special}`);
      }
    }
    
    orderFood('ham')
    orderFood('ham', 2)
    orderFood('ham', 1, 'cheese')
    ```
    
    - 옵셔널 사용하려면 기본값 사용한 매개변수도 명시해 주어야 한다는 점
    - 웬만하면 옵셔널 매개변수를 뒤에 배치함
- `...` 나머지 매개변수
    
    ```tsx
    function calcSum(title: string, ...nums: number[]) {
        const total = nums.reduce((sum, num) => sum + num, 0)
        console.log(`${title}: ${total}`)
    }
    
    calcSum("ages", 20, 21, 22)
    calcSum('scores', 90, 85, 50, 75)
    ```
    
    - 나머지 매개변수는 항상 배열/튜플 타입
    - 매개변수의 마지막 자리에만 사용 가능
- 나머지 매개변수와 전개 문법
    
    ```tsx
    function example(...args: [number, string, boolean]){
        console.log(args[0], args[1], args[2])
    }
    
    function example2(...args: [a: number, b: string, c: boolean]){
        console.log(a, b, c)
    }
    ```
    
    - 튜플 타입으로 전개 문법 사용 가능
    - 이때는 나머지 매개변수여도 개수 제한
    - 매개변수 이름 직접 지정도 가능
- `this` 사용할 때는 명시적으로 표기
    
    ```tsx
    function example(this: Document, a: string) {}
    example.call(document, 'hello');
    ```
    
    - `this`는 언제나 첫 번째 매개변수
    - `call` 메소드로 `this` 의 값을 명시적 지정
    
    🌟 어렵..
    
- 메서드에서도 `this` 사용 가능
    - 일반적으로는 `this`가 바뀔 때만 명시적으로 타이핑
        
        ```tsx
        type Animal = {
          age: number;
          type: "dog";
        };
        
        const person = {
          name: "zero",
          age: 28,
          sayName() {
            this;
            console.log(this.name);
          },
          sayAge(this: Animal) {
            this;
            console.log(this.type);
          },
        };
        
        person.sayAge.bind({ age: 12, type: "dog" });
        ```
        
        - `bind` 없이 `person.sayAge({ age: 12, type: "dog" })` 만 작성할 때는 Expected 0 arguments, but got 1. 에러가 발생한다
        - **`this`** 매개변수는 인자를 받지 않도록 설계되어 있다
- 타입스크립트에서는 생성자 함수를 사용할 수 없다
    - 대신 다음과 같이 만들어서 사용 가능
        
        ```tsx
        type Person = {
            name: string,
            age: number,
            married: boolean
        }
        
        interface PersonConstructor {
            new (name: string, age: number, married: boolean): Person;
        }
        
        const Person = function (this: Person, name: string, age: number, married: boolean) {
            this.name = name;
            this.age = age;
            this.married = married;
        } as unknown as PersonConstructor;
        
        Person.prototype.sayName = function(this: Person) {
            console.log(this.name)
        }
        
        const zero = new Person('zero', 28, false)
        ```
        
    - 그러나 클래스 사용을 권장

## 2.17 같은 이름의 함수를 여러 번 선언할 수 있다

- 오버로딩
    
    ```tsx
    function add(x: number, y: number): number;
    function add(x: string, y: string): string;
    function add(x: any, y: any) {
      return x + y;
    }
    
    add(1, 2);
    add("1", "2");
    ```
    
    - 매개변수를 `any` 로 사용했지만 오버로딩한 타입의 조합만 가능하다
- 오버로딩의 순서는 좁은 타입에서 넓은 타입 순으로
    
    ```tsx
    function example(param: string): string;
    function example(param: string | null): number;
    function example(param: string | null): string | number {
      if (param) {
        return "string";
      } else {
        return 12;
      }
    }
    ```
    
- 인터페이스로 오버로딩 표현하기
    
    ```tsx
    interface Add {
        (x: number, y: number): number;
        (x: string, y: string): string;
    }
    
    const add: Add = (x: any, y: any) => x+y;
    ```
    
- 타입 별칭으로 오버로딩 표현하기
    
    ```tsx
    type Add1 = (x: number, y: number) => number;
    type Add2 = (x: string, y: string) => string;
    type Add = Add1 & Add2;
    
    const add: Add = (x: any, y: any) => x + y;
    
    add(1, 2);
    add("1", "2");
    ```
    
    - 궁금
        
        ```tsx
        type Add = {
            (x: number, y: number): number;
            (x: string, y: string): string;
        }
        
        const add: Add = (x: any, y: any) => x+y;
        ```
        
        이렇게는 못 쓰나
        
- 유니언이나 옵셔널로 해결 가능할 때는 오버로딩 사용하지 않기

## 2.18 콜백 함수의 매개변수는 생략 가능하다

```tsx
function example(callbak: (error: Error, result: string) => void) {}

example((e, r) => {});
example(() => {});
example(() => true);
```

- **문맥적 추론**: 선언할 때 작성한 콜백 함수의 타입으로 추론된다
- 매개변수를 사용하지 않아도 된다
    - but, 옵셔널로 만들면 타입 추론이 `Error | undefined`, `string | undefined` 로 되어 의도와 달라짐
- 콜백 함수의 반환값이 void일 때는 어떤 반환값이 와도 상관없다 (2.7장 void 참고)

- `forEach` 의 내부 구현 구조로 예를 들어 알아보자
    
    ```tsx
    function forEach(
      callbackfn: (value: number, index: number, array: number[]) => void,
      thisArg?: any,
    ): void {}
    
    [1, 2, 3].forEach((item, index) => {});
    [1, 2, 3].forEach((item) => item);
    ```
    
    - `forEach` 의 사용 예시를 보면 콜백 함수의 구현체의 매개변수가 옵셔널이 아님에도 매개변수를 옵셔널로 사용할 수도 있고, 반환값이 있어도 없어도 사용할 수 있다

## 2.19 공변성과 반공변성을 알아야 함수끼리 대입할 수 있다

- **공변성**: A → B, T<A> → T<B>
- **반공변성**: B → A, T<A> → T<B>
- 이변성: A → B, T<A> → T<B> / T<B> → T<A>
- 무공변성: A → B, T<A> → T<B> 불가능 / T<B> → T<A> 불가능

- 타입스크립트는 공변성을 가지고 있지만 함수의 매개변수는 반공변성을 가지고 있다
    - TS Config strictFunctionTypes 옵션 활성화
        
        

```tsx
function a(x: string): number {
  return 0;
}

type B = (x: string) => number | string;
let b: B = a;
```

- a는 `number`를 반환하고 b는 `number | string` 을 반환하므로 a의 반환값을 b의 반환값에 대입할 수 있다
    
    ⇒ **a → b 관계**
    
- T<a>는 a 함수의 반환값으로 생각하고 a → b 일 때 T<a> T<b> 관계 파악해 보기 🌟 이해 안 감
- 함수 a를 타입 b에 대입할 수 있으므로 **T<a> → T<b>**
    
    ⇒ 함수의 반환값은 공변성을 가지고 있다
    
- b → a 관계에서 a를 b에 대입한다면?
    
    ```tsx
    function a(x: string): number | string {
      return 0;
    }
    
    type B = (x: string) => number;
    let b: B = a;
    ```
    
    ![image](./images/week3/taerim/image2.png)
    
    - 에러 발생
    - `strict(strictFunctionTypes)` 옵션 해제해도 에러 발생
    - b → a 일 때는 T<b> → T<a>만 가능
    
    **⇒ 반환값은 항상 공변성을 가진다**
    

- 매개변수
    
    ```tsx
    function a(x: string | number): number {
      return 0;
    }
    
    type B = (x: string) => number;
    let b: B = a;
    ```
    
    - 여기서 매개변수는 `string` → `string | number` 이므로 b → a
    - a → b는 가능
    - 매개변수의 경우는 b → a 일 때 T<a> → T<b>이므로 반공변성을 가짐
    
    ```tsx
    function a(x: string): number {
      return 0;
    }
    
    type B = (x: string | number) => number;
    let b: B = a;
    ```
    
    - 반대의 경우는 에러 발생
        ![image](./images/week3/taerim/image1.png)
        
    - `strictFunctionTypes` 옵션 해제하면 에러 발생하지 않음
    
    ⇒ **매개변수는 strict 옵션일 때 반공변성, strict 옵션일 때 이변성을 가진다**
    
- 객체의 메서드
    
    ```tsx
    interface SayMethod {
      say(a: string | number): string;
    }
    
    interface SayFunction {
      say: (a: string | number) => string;
    }
    
    interface SayCall {
      say: {
        (a: string | number): string;
      };
    }
    
    const sayFunc = (a: string) => "hello";
    const MyAddingMethod: SayMethod = {
      say: sayFunc,
    };
    const MyAddingFunction: SayFunction = {
      say: sayFunc,
    };
    const MyAddingCall: SayCall = {
      say: sayFunc,
    };
    ```
    
    ![image](./images/week3/taerim/image3.png)
    ![image](./images/week3/taerim/image4.png)
    
    - `함수(매개변수): 반환값` : 이변성
    - `함수: (매개변수) => 반환값` : 반공변성

## 2.20 클래스는 값이면서 타입이다

```tsx
class Person {
  name: string; // name;
  age: number; // age;
  married: boolean; // married;
  constructor(name: string, age: number, married: boolean) {
    this.name = name;
    this.age = age;
    this.married = married;
  }
}
```

- 클래스 내부에 멤버를 선언해 주어야 하고 타입은 생략할 수 있다
    - 타입 생략하면 생성자 함수로 추론된다
- 멤버는 constructor 내부와 짝이 맞아야 한다

- 인터페이스와 `implements` 예약어로 클래스의 멤버 검사
    
    ```tsx
    interface Human {
      name: string;
      age: number;
      married: boolean;
    }
    
    class Person implements Human {
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
    
- 타입스크립트는 생성자 함수 방식으로 객체를 만들 수 없기 때문에 `new` 키워드로 객체를 호출할 수 있는 것은 클래스뿐이다

- 클래스를 타입으로 사용하려면 `typeof` 키워드와 함께 사용
    
    ```tsx
    const person1: Person = new Person("zero", 28, false);
    const P: typeof Person = Person;
    const person2 = new P("nero", 32, true);
    ```
    
- `readonly`, `public`, `protected`, `private`
    - `readonly`: 변경할 수 없음
    - `public`: 다른 키워드가 붙지 않으면 기본값
        - 선언한 클래스, 자손 클래스, new로 만든 인스턴스에서 속성을 사용할 수 있음
    - `protected`: 클래스, 자손 클래스 내에서는 속성 사용 가능, 인스턴스에서는 사용 불가능
    - `private`: 자신의 클래스에서만 속성 사용 가능, 자손 클래스와 인스턴스에서는 사용 불가능
    
    | 수식어 | 자신 class | 자손 class | 인스턴스 |
    | --- | --- | --- | --- |
    | public | O | O | O |
    | protected | O | O | X |
    | private | O | X | X |
- 자바스크립트의 `#` 기능과 `private`
    
    ```tsx
    class PrivateMember {
      private priv: string = "priv";
    }
    
    class ChildPrivateMember extends PrivateMember {
      private priv: string = "priv";
    }
    ```
    
    ![image](./images/week3/taerim/image5.png)
    
    ```tsx
    class PrivateField {
      #priv: string = "priv";
      sayPriv() {
        console.log(this.#priv);
      }
    }
    
    class ChildPrivateMember extends PrivateField {
      #priv: string = "priv";
    }
    ```
    
    - `private` 으로 선언한 속성은 자손 클래스에서 같은 이름으로 사용할 수 없음
- `implements` 하는 인터페이스의 속성은 `public` 만 허용
    
    ```tsx
    interface Human {
      name: string;
      age: number;
      married: boolean;
    }
    
    class Person implements Human {
      name;
      protected age;
      married;
    
      constructor(name: string, age: number, married: boolean) {
        this.name = name;
        this.age = age;
        this.married = married;
      }
    }
    ```
    
    ![image](./images/week3/taerim/image6.png)
    
    - `implements` 한 클래스에서 속성이 `protected` 나 `private` 인 경우에는 에러 발생
- `override`
    - `{ 'noImplicitOverride': true }`
    
    ```tsx
    class Human {
      eat() {
        console.log("냠냠");
      }
      sleep() {
        console.log("쿨쿨");
      }
    }
    
    class Employee extends Human {
      work() {
        console.log("끙차");
      }
      override sleep() {
        console.log("에구");
      }
    }
    ```
    
    - 명시적으로 오버라이드 하면 부모 클래스에서 메서드 변경되었을 때 바로 확인 가능
- 오버로딩
    - 타입 선언을 여러 번 하면 된다
        
        ```tsx
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
            if (typeof age === "boolean") {
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
        const person2 = new Person("nero", true);
        const person3 = new Person("zero", 28, false);
        ```
        
        - 모든 constructor에 대응시키기 위해 옵셔널로 속성을 선언
- 클래스 내 인덱스 시그니처
    
    ```tsx
    class Signature {
      [propName: string]: string | number | undefined;
      static [propName: string]: boolean;
    }
    
    const sig = new Signature();
    sig.hello = "world";
    Signature.isGood = true;
    ```
    
- 메서드에서 `this` 를 타입으로 사용하기
    
    ```tsx
    class Person {
      age: number;
      married: boolean;
      constructor(age: number, married: boolean) {
        this.age = age;
        this.married = married;
      }
      sayAge() {
        console.log(this.age);
      }
      sayMarried(this: Person) {
        console.log(this.married);
      }
      sayCallback(callback: (this: this) => void) {
        callback.call(this);
      }
    }
    ```
    
    - `sayCallback` 의 매개변수는 콜백 함수인데, 이 콜백 함수의 this 타입은 Person 인스턴스가 된다

## 2.21 enum은 자바스크립트에서도 사용할 수 있다

- 타입스크립트에서 다음과 같이 작성한 enum은 자바스크립트로는 오른쪽 코드로 변환된다.

```tsx
enum Level {
	NOVICE,
	INTERMEDIATE,
	ADVANCED,
	MASTER,
}
```

```jsx
var Level = {
	0: 'NOVICE',
	1: 'INTERMEDIATE',
	2: 'ADVANCED',
	3: 'MASTER',
	NOVICE: 0,
	INTERMEDIATE: 1,
	ADVANCED: 2,
	MASTER: 3,
}
```

- 기본적으로 순서대로 0부터 숫자가 할당된다
- `=` 연산자로 다른 숫자를 할당할 수도 있다
    
    ```jsx
    enum Level {
    	NOVICE = 3,
    	INTERMEDIATE, // 4
    	ADVANCED = 7
    	MASTER, // 8
    }
    ```
    
    - 다음 값으로는 이전에 할당한 값 + 1
- 문자열 할당
    
    ```tsx
    enum Level {
      NOVICE,
      INTERMEDIATE = "hello",
      ADVANCED = "ho",
      MASTER, // 문자열 할당 뒤로는 전부 직접 값을 할당해야 함
    }
    ```
    
- 값으로 사용하기
    
    ```tsx
    enum Level {
    	NOVICE,
    	INTERMEDIATE,
    	ADVANCED,
    	MASTER,
    }
    
    const a = Level.NOVICE // 0
    const b = Level[Level.NOVICE] // NOVICE
    ```
    
- 타입으로 사용하기
    - 값보다 타입으로 사용하는 경우가 더 많다
    
    ```tsx
    enum Level {
    	NOVICE,
    	INTERMEDIATE,
    	ADVANCED,
    	MASTER,
    }
    
    function getLevel(level: Level) {
    	return Level[level];
    }
    
    const myLevelNum = Level.NOVICE;
    const myLevel = getLevel(myLevelNum) 
    ```
    
    - enum을 타입으로 사용하면 멤버의 유니언과 비슷한 역할을 한다
        
        `Level.NOVICE | Level.INTERMEDIATE | Level.ADVANCED | Level.MASTER`
        
- 브랜딩 속성으로 사용하기
    - 브랜딩 속성으로 사용할 때는 같은 enum의 멤버로 사용해야 한다
    - 다른 enum의 멤버일 경우에는 가지고 있는 값(0, 1, 2와 같은)이 중복될 수 있기 때문
    
    ```tsx
    enum City {
    	SEOUL,
    	INCHEON,
    	BUSAN,
    }
    
    interface Seoul {
    	type: City.SEOUL,
    }
    interface Incheon {
    	type: City.INCHEON,
    }
    interface Busan {
    	type: City.BUSAN,
    }
    
    function classifyByCity(city: City) {
    	if (city.type === City.SEOUL) {
    		console.log("서울 사람");
    	} else if (city.type === City.INCHEON) {
    		console.log("인천 사람");
    	} else {
    		console.log('부산 사람');
    	}
    }
    ```
    
- 자바스크립트 코드를 생성하지 않고 enum 사용하기
    
    ```tsx
    const enum City {
    	SEOUL,
    	INCHEON,
    	BUSAN,
    }
    
    const seoul = City.SEOUL;
    City[City.SEOUL]; // City 객체가 없어서 불가능한 구문
    ```