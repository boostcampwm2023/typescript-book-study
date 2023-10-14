# 3 주차 학습 ✨

작성일: 2023년 10월 7일 오전 11:58

### 2.16  `함수와 메서드 타이핑하기`

- 일반적인 타이핑 방법과 일치
- 매개 변수에 옵셔널 수식어 사용가능
- 나머지 문법 사용 가능
    - 나머지 문법을 사용할 경우 꼭 해당 매개변수는 마지막에 위치해야 함.
    - 튜플도 사용가능
        
        ```tsx
        function ex(a:string,...b:number[]){} // O
        function ex(...b:number[],a:string){} // X
        
        // 튜플
        function ex(...args:[number,string,boolean]){} //O
        // 해당 매개변수의 이름을 지정할 수도 있음.
        // 실용성은 없다.
        function ex(...args:[a:number,b:string,c:boolean]){}
        ```
        
- this를 사용하는 경우
    - 명시적으로 표기해야 함
    - 명시적으로 표기하지 않으면 any로 추론
        - 전역 this
            - node 환경 ⇒ global
                - console.log(this)는 왜 빈 객체냐?
                    - Node에서 js 파일은 하나의 모듈로 취급함. 그렇기에 지역 scope를 가지게 됨.
            - 브라우저 환경 ⇒ window
            - 통합 ⇒ globalThis
    - 매개변수로 사용된 this는 실제 매개변수가 아니라 타입 지정을 위해서 들어간다.
    
    ```tsx
    function ex(){console.log(this)}// this가 any로 추론됨.
    
    function ex1(this:Window){}
    ```
    
    - 당연히 this에 타입을 지정했다고 사용할 수 있는 것은 아니다.
        
        bind, call, apply를 사용해서 this를 붙여줘야 함
        
    
    ```tsx
    function ex(this:Document, a:string, b:'this'){}
    ex('he','this') // X error  this가 Documemt 타입일 수 없어.
    ex.call(document,'hello','this') // O
    ```
    
    - call, apply와 bind 비교
        - call과 apply는 매개변수 할당을 배열로 할지 나열할 지의 차이
        
        ```tsx
        ex.call(document,'ad','this');
        ex.apply(document,['ad','this']);
        ```
        
        - bind는 특정 객체를 this로 사용하는 새로운 함수를 반환
        
        ```tsx
        const ex2 = ex.bind(document);
        ex2('ad','this')
        ```
        
- 메서드에서 this 타이핑하기
    
    ```tsx
    type Coin = {
    	amount:number;
    	price:number;
    }
    const bitcoin = {
    	name:'bitcoin',
    	age:13,
    	sayInfo(this:Coin){
    		this; // this:Coin
    		this.price // price:number
    	}
    }
    ```
    
- 타입스크립트에서는 함수를 new 키워드를 이용해 객체를 만들 수 없다
    - 예외적으로 만들순 있지만 class를 쓰자.

### 2.17 `같은 이름의 함수를 여러 번 선언할 수 있다.`

- 사용 예시
    - 두 개의 매개변수를 받고, 매개변수의 타입이 같은 경우에 두 매개변수를 합치는 함수를 구현한다고 가정하자 ( string, number에 대해서만)
    - 타입 narrowing 없이 타입 오버로딩으로 해결 가능
    - 실제 함수의 기능이 구현되는 곳에선 선언부에 any를 명시적으로 타이핑한다
    
    ```tsx
    function add(x:number,y:number):number; // 선언부
    function add(x:string,y:string):string; // 선언부
    function add(x:any,y:any){return x+y}   // 선언부 + 구현부
    add(1,2)
    add(1,'2') // X Error 만약에 오버로딩 안 하고 유니온 타입을 사용했다면 오류 검출 XXXXX
    ```
    
    - 오버로딩을 선언하는 순서도 타입 추론에 영향을 줌
    
    ```tsx
    function e(p:string):string;
    function e(p:string|null):number;
    function e(p:string|null):string|number {
    	if(param){ return 'string'}
    	else return 123;
    }
    e('string') // => 리턴 타입이 string으로 제일 위에 선언과 일치함!
    ```
    
    ```tsx
    function e(p:string|null):number;
    function e(p:string):string;
    function e(p:string|null):string|number {
    	if(param){ return 'string'}
    	else return 123;
    }
    e('string') // => 리턴 타입이 number으로 제일 위에 선언과 일치함!
    ```
    
- interface에서 오버로딩 적용하기
    
    ```tsx
    interface Add {
    	(x:number,y:number}:number;
    	(x:string,y:string):string;
    }
    const add : Add = (x:any,y:any) => x+y;
    ```
    
- 타입 별칭으로 오버로딩 적용하기
    
    ```tsx
    type Add1 = (x:number,y:number} => number;
    type Add2 = (x:string,y:string} => string;
    type Add = Add1 & Add2;
    const add :Add = (x:any,y:any)=>x+y;
    ```
    
- 잘못된 오버라이딩의 예시
    - 필요없는 곳에 오버라이딩을 하지말자
    - 결국 타입 오버라이딩하고 싶으면 타입 선언부의 타입이 정확히 일치해야 해!
    
    ```tsx
    function a(param:string):void
    function a(param:number):void
    // function a(param:string|number):void 이걸 넣어야 ok
    function a(param:string|number){}
    
    function errorA(param:string|number){
    	a(param); 
    }
    // a의 param은 유니온 타입인데 
    // 타입 선언부의 타입과 일치하는 타입이 없음 에러를 해결하기 위해서는
    // function a(param:string|number):void
    
    function b(p1:string):void;
    function b(p1:string, p2:number):void
    function b(p1:string, p2?:number){}
    
    function errorB(p1:string, p2:number|undefined){
    	b(p1,p2);
    }
    
    // b의 p1은 string이고 p2는 number|undefined인데 
    // 타입 선언부의 최초 p1은 string만 있어서 넘어가고 
    // function b(p1:string, p2?:number):void 추가하면 OK
    ```
    

2.18 `콜백함수의 매개변수는 생략 가능하다`

- 인수로 제공하는 콜백 함수의 매개변수에는 타입을 표기하지 않아도 된다.
- 함수를 선언할 때 타이핑한 값을 토대로 추론할 수 있음
- 문맥적 추론이라고 한다
- 콜백함수의 매개변수는 자동으로 옵셔널 처리되기 때문에 ? 키워드를 생략하자.

```tsx
function e(cb:(error:Error, result:string) => void){};
e((err,res)=>{}); // ok
e(()=>{}); // ok
e(()=>true); // ok  void 값은 어떤 값이 와도 괜찮.
```

- 고차함수 forEach
    - 얘도 매개변수 안 써도 괜찮잖아.

```tsx
[1,2,3].forEach((el,index,origin)=>{});
[1,2,3].forEach((el,idx)=>{})
[1,2,3].forEach(()=>{})
```

### 2.19 `공변성과 반공변성`

- 공변성 : A → B일 때 T<A> → T<B>인 경우
- 반공변성: A → B일 때 T<B> → T<A>인 경우
- 이변성: A → B일 때 T<A> → T<B>나  T<B> → T<A>가 만족 되는경우 (하나만 만족해도 ok)
- 무공변성: A → B일 때 T<A> → T<B>도 안 되고 T<B> → T<A>도 안 되는 경우
- 기본적으로 타입스크립트는 공변성을 갖고 있다.
- 함수의 매개변수는 반공변성을 갖고 있다.

> strictFunctionTypes 옵션이 체크되어 있어야 매개변수에 대해 반공변성을 체크한다.
아닐 경우 이변성을 갖는 지 체크한다.
> 

- 예시로 이해하기
    
    ```tsx
    function a(x:string):number {return 0;}
    type B = (x:string) => number | string;
    let b: B = a; // Ok
    ```
    
- a 함수를 B에 대입할 수 있냐
- a → B
    - 일반 매개변수는 같기 때문에 pass
    - 반환 타입만 따지면 된다.
    - A의 리턴 타입은 number고 B의 리턴 타입은 number | string이라 T<a> → T<B>이기 때문에 대입할 수 있다.
    
    ```tsx
    function a(x:string):number | string {return 0;}
    type B = (x:string) => number;
    let b: B = a; // XX
    ```
    
- a→b
    - T<B> → T<A>이기 때문에 반공변성을 가진다.
    - 리턴 타입의 경우 공변성을 가져야 할당할 수 있으므로 XXXX
    
    ```tsx
    function a(x:string|number):number{return 0;}
    type B = (x:string)=>number;
    let b:B =a; //OO
    ```
    
- strict 옵션 on
    - 매개변수의 경우 반공변성을 가져야 할당할 수 있다.
        - a → B하고 싶은데
        - 반공변성의 경우 T<B> → T<a>가 만족해야 할당가능.
        - B의 매개변수 보다 a의 매개변수가 더 넓은 타입을 가지니 할당가능!
    
    ```tsx
    function a(x:string):number{return 0;}
    type B = (x:string|number)=>number;
    let b:B =a;  //XX 
    ```
    
- strict 옵션 on
    - 매개변수의 경우 반공변성을 가져야 할당할 수 있다.
        - a → B하고 싶은데
            - 반공변성의 경우 T<B> → T<a>가 만족해야 할당가능.
            - b의 매개변수가 a보다 넓기 때문에 반공변성을 가지지 못 한다. 고로 Error

- 객체의 메서드 타이핑

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
    

### 2.20 `클래스는 값이면서 타입`

- 타입스크립트는 클래스의 멤버변수를 클래스 내부에 한 번 적어야 한다는 사실을 제외하고는 같다.
    - 생략하고 생성자 함수에 타입을 할당해도 됨.
    
    ```tsx
    class P {
    	name:string; // 타입 지정을 생략가능
    	constructor(name:string){
    		this.name=name;
    	}
    }
    ```
    
    - 타입을 지정한 멤버는 항상 constructor 내부와 짝이 맞아야 한다.
    - interface를 사용해 필요한 멤버와 메소드가 모두 들어 있는지 체크 가능하다.
        - implements keyword 사용

```tsx
interface Coin {
	name:string;
	rising():void;
}
class BitCoin implements Coin{
	name;
	constructor(name:string){
		this.name = name;  // Error rising 메소드가 빠졌어 
	}
}
```

- 타입스크립트에선 클래스가 타입으로 사용 되기도 한다.
    - 타입으로 사용된다면 클래스 자체의 타입이 아니라 인스턴스의 타입이다.
    - 클래스 자체의 타입이 필요하다면 typeof classname
    
    ```tsx
    const coin1: Coin = new Coin('bitcoin');
    const C: typeof Coin = Coin;
    const coin2 : Coin = new C('riple')
    ```
    
- 클래스 멤버에는 옵셔널, readonly, public, protected, private 수식어를 사용할 수 있다.

```tsx
class Parent {
	name?:string
	readonly age:number;
	protected married: boolean;
	private value: number;
	constructor(name:string,age:number,married:boolean){
			this.name=name;
			this.age=age;
			this.married= married;
			this.value =0;
	}
	changeAge(age:number){
		this.age =age; // Error readonly propetry는 변경 XX
	}
}

class Child extends Parent {
	consturctor(name:string,age:number,married:boolean){
		super(name,age,married);
	}
	sayName(){console.log(this.name)} // ? 
	sayMarried(){console.log(this.married)} // protected
	sayValue(){console.log(this.value)} // private Error. 
}

const child = new Child('z',28,false);
child.name;   // ? 키워드 붙은 멤버라서 ok
child.married; // protected 붙어 있어서 No
child.value; // private 붙어 있어서 No
```

- public : 아무 keyword도 없으면 public이다. 모든 경우에 접근 가능하다.
- protected : 자신이나 상속된 클래스 내부에서 사용 가능하다. 인스턴스화 후에는 사용 불가능하다.
- private : 자신의 클래스 내부에서만 사용 가능하다.

| 수식어 | 자신 class | 자손 class | 인스턴스 |
| --- | --- | --- | --- |
| public | O | O | O |
| protected | O | O | X |
| private | O | X | X |
- js에서 private field(#)과 ts에서 private 수식어의 차이점
    - private 수식어로 선언한 속성은 자손 클래스에서 같은 이름으로 선언할 수 없다.
    - 뭐가 나쁘고 좋고의 문제가 아님.
    
    ```tsx
    class P {private priv:string='priv'}
    class C extends P {private priv:string='priv'} // Error
    
    class P { #priv:string='priv'}
    class C extends P {#priv:string='priv'} // no Error
    ```
    
    - 인터페이스를 implements하는 경우에는 모든 속성이 public이다.
        - 당연하게 interface에는 private이나 protected이 없다.

> TS Config 메뉴에서 noImplicitOverride 옵션을 키면 메소드가 오버라이드된 경우에 알려준다.
이때 override를 붙여 명시적으로 개발자가 오버라이드를 시행 했음을 알려야 한다.
반대로 override를 명시적으로 표현했지만, 상속받은 메소드가 없을 경우에도 알려준다.
> 

```tsx
class H {say(){console.log('안녕')}}
class HH extends H {
	override say() {
		console.log('안녕')
	}
}
```

> 생성자 함수에도 오버로딩을 적용할 수 있다.
일반 함수와 비슷하게 타입 선언을 여러번 한다.
> 

```tsx
class P {
	constructor()
	constructor(name:string,married:boolean);
	constructor(name:string,age:number,married:boolean);
	constructor(name?:string,age?:boolean|number,married?:boolean){
		if(name){this.name=name;}
		if(typeof age ==='boolean'){this.married=age}
		else {this.age=age}
		if(married){this.married=married}
	}
}

const p1 = new P() // ok 1번째
const p1 = new P('nero',true) // ok 2번째
const p1 = new P('zero',28,false) // ok 3번째
```

- 클래스의 속성에도 인덱스 시그니처를 사용할 수 있다.
    - static 키워드를 이용해서 static 속성에도 인덱스 시그니처를 적용할 수 있다.

```tsx
class Signature {
	[propName:string]: string|number|undefined;
	static [propName:string]:boolean;
}

const sig = new Signature();
sig.hello = 'world'
Signature.isGood = true;

```

- this를 타입으로 사용할 수 있다.
    - 명시적으로 this를 타이핑할 때 필요.
    - cb.call(this)해야 콜백함수의 this는 P 인스턴스가 됨.

```tsx
class P {
	...
	sayAge(){console.log(this.age)}
	sayMarried(this:Person){ // this: Person
	sayCallback (cb:(this:this)=>void){
			cb.call(this);
		}		
	}

	callThis(cb:(this:this)=>void){cb.call(this)};
	noCallThis(cb:(this)=>void){cb()};
}

new P().callThis(function(){this}) // 만든 인스턴스가 this로 할당됨.
new P().noCallThis(function(){this}) // any가 할당됨.
```

- 인터페이스로 클래스 생성자를 타이핑할 수 있다.

```tsx
interface PersonConstructor {
	new (name:string,age:number):{
		name :string;
		age:number;
	};
}

class Person {
	name: string;
	age: number;
	constuctor(name:string,age:number){
		this.name = name;
		this.age = age;
	}
}

function createPerson(ctor: PersonConstructor,name:string,age:number){
	return new ctor(name,age);
}
createPerson(Person,'z',2);
```

- 인터페이스로 클래스 생성자를 타이핑할 수 있는데, 이를 활용해 생성자 함수에도 타이핑이 가능하다.
    - ????? 그냥 class로 쓰자

```tsx
interface PersonInterface {
	name:string;
	age:number;
	married:boolean;
}

function Person(this:PersonInterface,name:string,age:number,married:boolean){
	this.name =name;
	this.age=age;
	this.married=married;
}

type PersonType = typeof Person & { new (name:string,age:number,married:booelan):PersonInterface}
new (Person as PersonType)('zero',28,false);
```

### 2.20.1 `추상클래스`

- 구체적으로 클래스의 모양일 정의하는 방법
- class 앞에 abstract 키워드를 붙여서 사용한다.
    - abstract class안의 멤버와 메서드는 abstract를 키워드로 가질 수 있다.
    - abstract가 붙은 멤버와 메서드는 구현 없이 타이핑만 되어있다.
    - 상속받는 곳에서 abstract 멤버와 메서드를 꼭 구현해야 한다.

```tsx
abstract class AbstractPerson {
	name:string;
	age:number;
	married:boolean= false;
	abstract value:number;
  constructor(name:string,age:number,married:boolean){
		this.name=name;
		this.age=age;
		this.married = married;
	}
	abstract sayAge():void;
}

class ReadlPerson extends AbstractPerson { // Error value를 구현안 했어!
	sayAge(){
		console.log(this.age);
	}
}
```

- implements와 다르게 abstract 클래스는 실제 자바스크립트 코드로 변환된다.
    - 객체의 타이핑을 위해 interface를 사용하냐 클래스를 사용하느냐는 취향 차이다.
    - js로 변환 후에도 코드로 남길 바라면 class를 사용하자

```tsx
"use strict";
class AbstractPerson {
    constructor(name, age, married) {
        this.married = false;
        this.name = name;
        this.age = age;
        this.married = married;
    }
}
```

### 2.21 `enum은 자바스크립트에서도 사용할 수 있다.`

- enum은 열거형이라는 타입이다.
- js에서는 없는 타입이지만 자바스크립트의 값으로 사용할 수 있다.
- 여러 상수를 나열하는 목적으로 사용된다.
- enum 예약어로 선언한다.

```tsx
enum Level {
	NOVICE,
	INTERMEDIATE,
	ADVANCED,
	MASTER,
	}
```

- enum을 js로 변환하면

```tsx
"use strict";
var Level;
(function (Level) {
    Level[Level["NOVICE"] = 0] = "NOVICE";
    Level[Level["INTERMEDIATE"] = 1] = "INTERMEDIATE";
    Level[Level["ADVANCED"] = 2] = "ADVANCED";
    Level[Level["MASTER"] = 3] = "MASTER";
})(Level || (Level = {}));
```

- Level[Level["NOVICE"] = 0] = "NOVICE";은 양방향으로 key, value를 묶는 코드임
    - level.novice =0 //  level[’0’] = ‘novice’
- 기본적으로 enum은 0부터 값을 할당하지만 =을 사용해서 시작 key를 지정할 수 있다.

```tsx
enum Level {
	ONE=1,
	TWO,  // 2
	THREE // 3
}
```

- 문자열도 할당할 수 있지만, 할당한 후 부터는 모든 값을 직접 넣어줘야 한다.

```tsx
enum Level{
	ONE;
	TWO='T'
	THREE // ERROR Enum member must have initializer !
}
```

> 💡 tip   enum[enum의 멤버]는 enum의 멤버 이름
      enum[enum.ONE] ⇒ ‘ONE’
> 

```tsx
function whatsYourLevel(leveL:Level){
	console.log(Level[level]);
}

const myLevel = Level.ONE; // myLevel 은 인덱스 0이 들어감
whatsYourLevel(myLevel) // ONE
```

- enum은 개선되고 있지만, 완벽하지 않다.

```tsx
enum Role {
	USER,
	GUEST,
	ADMIN,
}

enum Role2 {
	USER = 'USER',
	GUEST = 'GUEST',
	ADMIN = 'ADMIN', 
}

function changeUserRol(rol:Role){}
function changeUserRol2(rol:Role2){}

changeUserRol(2);
changeUserRol(4); // Error 4는 없어. 원래는 에러가 아니었지만 개선된 사항!
changeUserRol2(Role2.USER);
changeUserRol2('USER'); // Error 개선이 필요한 사항 왜 안됨?
```

- 엉망인 enum도 브랜딩에 사용하기엔 좋다!

```tsx
enum Money { WON, DOLLAR }
interface Won {type:Money.WON,}
interface Dollar {type:Money.DOLLAR,}
function moneyOrDollar(param: Won|Dollar){
    if(param.type===Money.WON){param} // Won으로 타입추론
    else {param} // Dollar로 타입추론
}
```

- 같은 enum의 멤버가 아니면 구분되지 않는다.
    - 당연한게 enum은 0부터 자동으로 인덱스를 부여한다.
    - [Money.WON] = 1
    - [Water.LITER] =1

```tsx
enum Money{WON}
enum Water{LITER}
interface M {type: Money.WON,}
interface W {type: Water.LITER,}

function moneyOrLiter(param:M|W){
	if(param.type === Money.WON){param}
	else param
}

moneyOrLiter({type:Money.WON})
moneyOrLiter({type:Water.LITER})
```

- enum의 타입을 사용하되 자바스크립트 코드가 생성되지 않게 할 수 있다.
    - const 키워드를 enum 앞에 붙인다.

```tsx
const enum Money {
	WON, DOLLAR,
}
Money.WON; // O
Money['WON'] // O
Money[Money.WON]; // Error
Money[0] // Error
// 그냥 {WON:0, DOLLAR:1} 이렇게만 key value 생기는 게 아닐까

// 1.5 => 1
```

- Money.won은 0 Money.DOLLAR은 1로 변환되지만 Money라는 자바스크립트 객체가 없기 때문에 Money[Money.WON]은 Error가 발생한다.