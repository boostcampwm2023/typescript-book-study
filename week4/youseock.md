# 4 주차 학습 ✨

작성일: 2023년 10월 7일 오후 8:11

### 2.22 `infer로 타입 추론`

- infer 예약어를 사용하면 타입스크립트 추론 기능을 더 잘 활용할 수 있다.
- 타입스크립트가 특정 타입을 추론 하라고 명령내리는 것.

```tsx
function fn (){
	return 123
}

typeof fn  // () => number 

type fnReturnType = ReturnType<typeof fn> // number라고 잡아줌.

type ReturnType<T extends (...args:any) => any> 
= T extends (...args:any)=>infer R ? R : any;

좀 더 어렵게

function fn1 (num1:number){
	return num1
}

type ReturnType<T extends (...args:any) => any> 
= T extends (...args:infer U)=>infer R ? [U,R] : never;

// [number,number]라고 리턴 타입이 추론됨!
```

- 사용 예시
    - 배열이면 배열안의 요소의 타입을 리턴하고 배열이 아니라면 never를 리턴하는 타입을 만들어 보자

```tsx
type El<T> = T extends (infer E)[] ? E : never;
type Str = El<string[]>
type NOB = El<(number |boolean)[]>
```

- 컨디셔널 타입에서 타입 변수는 참 부분에서만 사용할 수 있다

```tsx
type El<T> = T extends (infer E)[] ? never : E; // 이름을 못 찾는다는 에러가 나옴
```

- 여러 예시

```tsx
// 매개변수 (함수 형태면 무조건 P 타입을 리턴하네)
// 여러 매 개변수면 tuple 형식으로 타입이 리턴 됨!
T extends (...args:infer P) => any ? P : never;
// 생성자의 매개변수
T extends abstract new (...args:infer P) => any ? p : never;
// 반환값
T extends (...agrs:any) => infer R ? R : any;
// 인스턴스 타입
T extends abstract new (...args:any)=> infer R ? R :any;

```

- 질문
    
    ```tsx
    class A {
      a:string;
      
      constructor(str:string){
        this.a = str;
      }
    }
    
    type AInstance = typeof A
    
    type B<T> = T extends abstract new (...args:any)=> infer R ? R : any;
    type A1Instance = B<new (a:string)=> 123 >;
    // 이런식으로 사용하는 게 의미가 있을까
    // A1Instance이 123으로  추론됨. 
    ```
    
- 같은 이름의 infer 변수를 생성하면 유니온 타입이 된다
    - 매개변수의 경우 반공변성을 갖고 있기 때문에 intersection 됨.
    - U ← 1|2, 2|3 공변성으로 따지면 U는 1|2|3이 됨
    - 1|2 ← U 도 만족하고 2|3 ← U 도 만족해야하니까 U는 2다.

```tsx
type Union<T> = T extends {a:infer U, b:infer U} ? U : never;
type Result1 = Union<{a:1|2, b:2|3}> // => 1|2|3 으로 유니온 타입으로 추론 

type Intersection<T> = T extends {
	a: (pa:infer U)=> void,
	b: (pb:infer U)=> void,
} ? U : never;

type Result2 = Intersection<{a(pa:1|2):void, b(pb:2|3}:void}>; 
// 2로 추론
```

- infer 변수가 매개변수와 리턴 값의 조합이면?
    - 반환값의 타입이 매개변수의 타입의 부분집합인 경우에만 그 둘의 교집합이 된다.
    - 그 외엔 never

```tsx
type R<T> = T extends { a: () => infer U, b: (pb:infer U)=>void} ? U: never;
type Result3 = R<{a:()=> 1|2, b(pb:1|2|3):void}>; // 1|2가 됨.
```

- 매개변수의 타입을 Infer로 추론하면 반공변성을 기준으로 타입이 추론된다. 이를 바탕으로 타입의 교집합을 만드는 type을 만들 수 있지 않을까?

```tsx
type UnionIntersetion<U> =
(U extends any ? (p:U) => void:never) extends (p:infer I)=> void ? I : never;

type r = UnionIntersetion<{a:number}|{b:string}>
type r = UnionIntersetion<boolean|true> //
```

- U는 유니온이기 때문에 일단 분배법칙이 실행된다
    - <{a:number}|{b:string}>
        - UnionIntersetion<{a:number}>|UnionIntersetion<{b:string}>
        - {a:number} ⇒ (p:{a:number})⇒void가 되고 함수 형태의 타입처럼 만들어서 {a:number}부분을 매개변수로 판단하게 만드네!
        - 결과는 intersection이다!
    - boolean | true
        - boolean 자체가 union이라 true|false|true가 됨
        - 인터섹션하면 true & false & true라서 never !!

### 2.23 타입을 좁혀 정확한 타입을 얻어내자

- 다양한 타입 좁히기 방법을 알아보자
- typeof 만으로 완벽하게 타입을 통제할 수 없다
    - `typeof null => 'object'`
- 자바스크립트 코드를 섞어서 사용하면 더 쉽고 정확한 타입 좁히기를 할 수 있다.

```tsx
function A(p:string|null|undefined|number[]){
	if(p===undefined){}
	else if(p===null){}
	else if(Array.isArray(number){}
	else {}
	}
}

class A {}
class B {}
function AOrB(instance:A|B){
	if(instance instanceof A){}
	if(instance instanceof B){} 
}
```

- 프로퍼티가 다른 객체를 타입스크립트에선 어떻게 구별할까?
    - in 연산자를 사용한다
    - 여기서의 문제점은 프로퍼티가 일치하지 않는 key를 찾아서 그 키 값의 존재 유무로 판별해야 함.
    
    ```tsx
    const aObj = {name:string,age:number}
    const bObj = {width:number,height:number}
    
    function aOrbObj(obj :aObj|bObj){
    	if ('age' in obj){// aObj  }
    	else {// bObj}
    }
    ```
    
    - 브랜드 속성을 이용하기
    
    ```tsx
    const aInterface = {__type='A', name:string,age:number}
    const bInterface = {__type='B', width:number,height:number}
    
    function aOrbObj(p :aInterface|bInterface){
    	if(p.__type==='A'){}
    	else {}
    }
    
    //좁히기 함수 만들기
    
    function isA(p :aInterface|bInterface}){
    		if(obj.__type==='A') return true
    		return false
    }
    
    // if 문에서 타입을 판별하는 함수를 갖고와 사용할 때 타입 추론 정상 작동하지 않는다.
    function aOrbObj(p :aInterface|bInterface){
    	if(isA(p)){} // aInterface|bInterface
    	else {} // aInterface|bInterface
    }
    // 해결하기 위해 판별 함수에 특별한 작업을 해준다.
    
    function isA(p :aInterface|bInterface): param is aInterface{
    		if(obj.__type==='A') return true
    		return false
    }
    ```
    
    - param is aInterface의 의미
        - 서술 함수 (Type Predicate)라고 부름
        - 매개변수 하나를 받아 boolean을 반환하는 함수를 의미한다.
        - 서술 함수를 사용하면 반환 값이 true일 때 is 뒤에 명시한 타입이 반환되어 타입 좁히기가 가능하다.
    - 타입 서술로 타입을 좁히려는 시도는 다른 방법을 시도하고, 불가능 하다면 적용하자!

### 2.24 `자기 자신을 타입으로 사용하는 재귀 타입`

- 자기 자신의 타입을 지정할 때 자기 자신의 타입을 이용하는 타입!
- js와 마찬가지로 무한 루프면 에러가 발생함
    - 특별한 점은 타입을 선언할 때 발생하는 것이 아니라 타입을 사용할 때 발생한다.

```tsx
type Recursive ={
	name:string;
	children:Recursive[];
} 

const reDepth1 : Recursive { name:'test',children:[]}
const re2Depth2 :Recursive { 
	name:'test',children:
	[
		{ name:'test2',children:[]},
		{ name:'test3',children:[]}
	]
}
```

- 컨디셔널 타입에도 재귀 타입을 사용할 수 있다

```tsx
type E<T> = T extends any[] ? E<T[number]> : T;
```

- 타입 인수로 사용하는 것은 불가능하다

```tsx
type Y = number | string | Record<string,Y> // Y xxxxx
// 인수를 쓰지 않는 방식으로 변경
type Y = number |string | {[key:string]:T}
```

- 무한 루프도는 재귀 타입
    
    ```tsx
    type Infi<T> = {item: Infi<T>};
    type Unwrap<T> = T extends {item: infer U} ? Unwrap<U> : T;
    type Result = Unwrap<Infi<any>>;
    ```
    
    - Infi<T>는 {item:{item: … }}
    - Unwrap<T>는 {item:U }의 형태면 U를 다시 Unwrap하는 타입.
    - Unwrap는 {item: } 이런식으로 최상위 껍질을 벗기는데, Infi는 {item}가 무한히 중첩되어 있기 때문에 error!!
- 유용하게 사용해보자
    - 다음과 같이 재귀 타입을 이용하면 복잡한 JSON 객체를 쉽게 타이핑할 수 있다!
    
    ```tsx
    type JSONType = 
    | string
    | boolean
    | number
    | null 
    | JSONType[]
    | {[key:string]:JSONType};
    ```
    
- 배열 타입을 뒤집어 보자
    
    ```tsx
    type R<T> = T extends [...infer L, infer R] ? R [R,...R<L>]:[];
    ```
    

### 2.25 `템플릿 리터럴 타입의 유용성`

- 템플릿 리터럴을 사용해서 타입을 지정하면 리터럴 타입을 쉽게 지정할 수 있다

```tsx
type programmingLan = 'J'|'S'|'O';
type camperNumber =  '1'|'2'|'3';
type bostCamper = `${programmingLan}${camperNumber}`;

const id:bostCamper ='J1';
```

- infer를 사용해서 확장하기

```tsx
type RemoveX<Str> = Str extends `x${infer Rest}`
	? RemoveX<Rest>
	: Str extends `${infer Rext}x` ? RemoveX<Rest> : Str;
```

### 2.26 `satisfies 연산자`

- 타입 추론을 그대로 활용하면서 추가로 타입 검사를 하고 싶을 때 사용한다.
- 아래와 같이 key 값의 오타는 쉽게 찾아내지만 earth의 value가 오브젝트라고 추론하지는 못 한다.
    - universe.earth.type ⇒ error
    - {type:string,parent:string} | string의 유니온 타입이라고 생각하기 때문이다.

```tsx
const universe = {
	sun: "star",
	sriius: "star",
	earth:{type:"planet",parent:"sun"},
};

const universe : {
	[key in 'sun' | 'sirius'|"earth"]: {type:string,parent:string} | string
} = {
	sun:"star",
	sriius:"star",
	earth:{type:"planet",parent:"sun"},
}
```

- satisfies로 해결해보자
- 객체 리터럴 뒤에 satisfies 타입을 표시하면 해결됨
- 아래와 같다면 universe.earth.type OK

```tsx
const universe = {
	sun: "star",
	sriius: "star",
	earth:{type:"planet",parent:"sun"},
} satisfies {
	[key in 'sun' | 'sirius'|"earth"]: {type:string,parent:string} | string
}
```

### 2.27 `타입스크립트의 건망증`

- 타입 주장은 해당 명령줄에서만 유효하다.
- 타입 주장한 타입의 영속성을 부여하고 싶다면, 변수에 할당해서 사용하자.
- 예시로 알아보자
    
    ```tsx
    try {} catch (error) {
    	if (error) { error.message; }
    }
    ```
    
    1. error는 unknown 타입이다.
    2. if 문을 통과하면 { } 타입이 된다.
    3. { } 타입에 message라는 프로펄티를 찾으려고 하니 에러가 난다.
- 개선해보자
    
    ```tsx
    try {} catch (error as Error) {
    	if (error) { error.message; }
    }
    ```
    
    1. 똑같이 error은 {}로 추론되고 message 프로펄티에 접근할 수 없다.
    2. 타입을 강제로 주입하는 경우 그 라인의 타입만 as로 강제하고 다음부턴 다시 원래 타입으로 돌아간다.
- 더 개선해보자
    
    ```tsx
    try {} catch (error) {
    	const err = error as Error;
    	if (err) { err.message; }
    }
    ```
    
    1. 중간에 변수로 할당하는 과정을 둬서 타입을 붙여버리자.
- 마지막으로 개선해보자
    - error의 타입을 as로 강제하지 말고 Error라는 클래스로 타입핑 하는 것이 더 좋다.
    
    ```tsx
    try {} catch (error) {
    	if (error instanceof Error) { error.message; }
    }
    ```
    

### 2.28 `브랜딩을 확장해보자`

- 코인을 다른 코인으로 환산 해주는 함수를 짠다고 해보자

```tsx
function 비트To이더(비트:number){
	return 비트 * 17
}
```

- 잘 돌아갈텐데 만약 개발자가 비트가 아닌 이더리움을 넣고 비트To이더를 돌리면 발생하는 문제를 어떻게 방지할 수 있을까(약 17배 손해)
- number면서 tag를 달고있는 변수가 있으면 좋지않을까?
- 3 & __name=’비트’ 이런 식이면 해결할 수 있네.

```tsx
type Coin<T,B> = T & {__name: B };
type 이더타입 = Brand<number,'이더리움'>
type 비트타입 = Brand<number,'비트'>;

function 비트To이더(비트:비트타입){
	return 비트 * 17 as 이더타입
}
const 비트 = 3 as 비트타입
const 환산한이더리움 = 비트To이더(3);
const 이더리움 = 1 as 이더타입;
비트To이더(이더리움) // Error 비트 타입만 넣을 수 있다. 17배 손해 막아줌
```