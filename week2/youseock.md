# 2 주차 학습 ✨

작성일: 2023년 9월 16일 오후 11:24

### 2.10 `객체의 속성과 메서드에 적용되는 특징`

- 객체의 속성에도 옵셔널이나 readonly 수식어 가능
    
    ```tsx
    interface IObj  {
    	normalInfo? : string;   
    	readonly privateInfo : string; 
    }
    ```
    
    - readonly 수식어가 붙은 요소는 변경할 수 없다.
    - ? 수식어가 붙은 속성은 undefined | 할당한 타입 으로 추론됨
        - 그래서 ? 수식어가 붙은 속성은 undefined를 직접 할당가능
    - 여러 수식어를 한 번에 사용할 수 있다.
        - 왜 readonly 수식어는 앞에 붙이지란 의문이 있었는데, 여러 수식어를 붙일 때 식별할 수 있게 도와주네.
- 객체 리터럴과 객체 리터럴을 참조하는 변수에 따른 타입 검사 방식
    - 객체 리터럴
        
        ```tsx
        interface IType = {
        	name : string;
        }
        
        const isNotOk : IType = { 
        	name : 'hi', say:'hellow' // error say 없어
        }
        ```
        
    - 변수에 할당 후
        
        ```tsx
        const 변수 = { name : 'mahwin', say:'hellow' }
        const isOk : IType = 변수;
        ```
        
    - 매개변수에서도 정확히 같은 일이 일어난다.
        
        ```tsx
        interface ICoin {
        	coin :string,
        	price: number,
        }
        
        function getCoinInfoArr(coinInfo1:ICoin, coinInfo2:ICoin) :ICoin[] {
        	return [coinInfo1,coinInfo2]
        }
        const coinInfo = {coin:"에이다", price:"33"}
        
        getCoinInfoArr(coinInfo, {coin:"에이다", price:"33"})
        // 두 번째 매개변수에서 에러 나옴!
        // 정확히 같은 객체를 (참조값을 의미하는 것은 아님, 타입적으로)
        // 넣어도 다른 결과가 나오는 이유는 잉여 속성 검사 때문 
        ```
        
    - 잉여 속성 검사
        - 단순하게 타입을 검증할 때 선언하지 않은 속성이 있는지 체크해줌
        - only 객체 리터럴에서만 체크 해줌
        - 참조값으로 타입을 체크할 때는 객체 간 대입 가능성을 기준으로 함
- 객체에서도 전개 문법과 나머지 속성을 이용할 수 있다.
    
    ```tsx
    const {prop:{nested,...rest}} =
    {prop:{nested:'hi', name:'crong',age:4}};
    // nested의 타입 nested:string
    // rest의 타입 name:string, age: number
    ```
    

💡 타입스크립트의 error 메세지는 아래로 내려갈 수록 구체적인 정보를  포함하고 있다.

### 2.10.1 `인덱스 접근 타입`

- 자바스크립트 객체 처럼 타입의 속성도 접근할 수 있다.
    
    ```tsx
    type Animal = {
    	name: string;
    }
    type N1 = Animal['name' | "name"] // ok
    type N2 =Animal.name; // X 
    ```
    
- 인덱스 접근 타입을 활용한 객체의 키 타입 얻기
    
    ```tsx
    const myName ={
    	first:"정",
    	second:"유",
    	third:"석"
    }
    type keys = keyof typeof myName;
    // typeof 연산자는 객체의 타입을 가져오고,
    // keyof 연산자는 해당 객체의 속성 키들을 
    // 문자열 리터럴 유니온 타입으로 
    // "first"|"second"|"third"
    ```
    
- 인덱스 접근 타입을 활용한 객체의 타입 얻기
    
    ```tsx
    // 앞에서 얻은 keys
    type values = typeof myName[keys];
    // 해당 값들의 타입을 유니온 타입으로 
    // string  
    ```
    
- keyof 특성
    
    ```tsx
    type anyTypeKeys =  keyof any; //  string|number|symbol
    ```
    
    - string과 symbol만 key 값으로 가능하지만 타입스크립트는 배열을 위해 number 키도 허용한다.
    
    ```tsx
    type arrKeysType = keyof [1,2,3];
    ```
    
    - keyof 배열
        - number | 배열_속성_이름_유니언 | 배열_인덱스_문자열_유니언
        - 배열_속성_이름_유니언
            - 배열이 공통적으로 갖고 있는 속성 ( length, forEach, indexOf ….)
        - 배열_인덱스_문자열_유니언
            - 배열의 인덱스를 유니온한 값 ‘0’|’1’|’2’
    - arrKeysType에 ‘3’은 안 되지만 3은 포함된다
        - number
        - 배열_속성_이름_유니언
            - forEach | map | filter | reduce | indexOf | …
        - 배열_인덱스_문자열_유니언
            - ‘0’ | ‘1’ | ‘2’
    
    ```tsx
    type numArrType =  [1,3,5];
    type First = numArrType[0]; // 1
    type Length =numArrType['length']; // 3
    
    type booleanOrStringArrType = (string|boolean)[];
    type elementType =  booleanOrStringArrType[number]; 
    // elementType는 string | boolean으로 추론됨!
    ```
    
    - 인덱스 접근 타입을 활용해서 특정 키들의 값만 가져올 수도 있음
    
    ```tsx
    const obj ={
    	key1 : "hi"
    	key2 : "hello"
    	key3 : boolean;
    }
    
    type KeyType = typeof obj['key1'|'key2'];
    // KeyType는 string으로 추론!
    ```
    

### 2.10.2 `매핑된 객체 타입`

- 기존의 타입으로 부터 새로운 객체 속성을 만들어 내는 것을 의미한다.
    - 인터페이스(X) 타입 별칭 (O)
- in 연산자를 이용해서 인덱스 시그니처가 표현하지 못하는 타입을 표현할 수 있다.
    - 일부 속성에만 타입을 부여해보자
    - hello ,hi 를 키로 가진 값들의 속성을 string으로 하고 싶으면 어떻게 타입을 지정해줘야 할까
    
    ```tsx
    type errorType = {
    	[key : "hello"|"hi"] : string; //Error
    }
    // 인덱스 시그니처에 사용할 수 있는 타입은 string, number, symbol,
    // 템플릿 리터럴과 이들의 유니온 뿐임
    // 템플릿 리터럴 타입의 예시 type Greeting = `Hello, ${string}!`;
    ```
    
    - 맵핑된 객체 타입을 이용해서 해결 가능하다.
        - 위 코드처럼 타입을 지정해도 돌아가는 것이 문제 없어 보이지만,
        - in 연산자를 붙힘으로 ts가 유니온 값을 통해 타입을 반복적으로 지정해 줘야 하구나라고 알 수 있음.
    
    ```tsx
    type correctType = {
    	[key in "hello"|"hi"] : string; // correct
    }
    // 유니온 타입의 값이 하나씩 평가되어 객체의 속성이 된다.
    ```
    
    - 맵핑된 객체 타입의 예시
    
    ```tsx
    interface Original {
    	name: string;
    	age: number;
    	married:boolean;
    }
    
    type Copy ={ 
    	[key in keyof Original] : Original[key]
    }
    
    // Copy {name: string  age:number married:boolean}
    ```
    
    - 여기서 in 연산자가 확실히 필요하구나 알게 됨.
        - in 연산자가 포함되면 전체 구문이 평가되는 구나!
    - 튜플에서 매핑된 객체 타입 적용하기
    
    ```tsx
    type Tuple = [1,2,3]
    type CopyTuple = {
    	[key in keyof Tuple] = Tuple[key];
    }
    ```
    type CopyTuple = {
    	[x:number] : 2|1|3;  
    	0: 1;
    	1: 2;
    	2: 3;
    	length: 3;
    	// 내부적으로 포함된 메소드들
    	pop:()=> 2 | 1 | 3 | undefined 
    	...
    }
    ```
    // 왜 굳이 2|1|3 ? 
    ```
    
    - 배열에서 매핑된 객체 타입 적용하기
    
    ```tsx
    type Arr = number[];
    type CopyArr = {
    	[key in keyof Arr] : Arr[key];
    }
    
    const copyArr: CopyArr = [1,3,9];
    
    ```
    type CopyArr = {
    	[x:number] : number;
    	length: number;  	
    	// 내부적으로 포함된 메소드들
    	pop:()=> number|undefined; 
    	...	
    }
    ```
    ```
    
    - 맵핑된 객체 타입을 이용하면서 연산자 붙이기 가능!
    
    ```tsx
    type Copy = {
    	readonly [key in unionType]? : string;
    }
    ```
    type Copy = {
    	readonly key1? : string | undefined;
    	readonly key2? : string | undefined;
    	readonly key3? : string | undefined;
    }
    ```
    ```
    
    - - 연산자를 통해 수식어를 제거할 수도 있다.
    
    ```tsx
    type mutableType = {
    	-readonly [key in unionType]-? : string;
    }
    
    ```
    type mutableType = {
    	key1 : string; 
    	key2 : string;
    	key3 : string;
    }
    ```
    ```
    

### 2.11 `타입을 집합으로 생각하자`

- 집합개념을 타입에 적용하자
    - & 교집합 | 합집합
    - 모든 타입을 포함하는 타입 unknown
    - 모든 타입에 포함되지 않는 타입 never
    
    ```tsx
    type nev = string & number; // never로 추론
    ```
    
- 타입스크립트에서는 좁은 타입을 넓은 타입에 대입해야 한다
    - unknown에 모든 타입은 대입할 수 있다.
        - 반대로 any와 unknown을 제외하고는 unknown을 대입할 순 없다.
    - never에는 모든 타입에 대입할 수 없다.
        - 반대로 모든 타입에 never는 대입할 수 있다.
    - any 타입은 집합 관계를 무시하기 지양하자.
- & , | 연산자 사용 예시
    
    ```tsx
    type A = string | boolean;
    type B = boolean | number ;
    type C =  A & B;   // boolean;
    
    type D = {} & (string | null ); // string
    // {} 는 null과 undefined를 제외한 모든 값
    
    type E = string & boolean; // never
    
    type F = unknown | {};  // unknown 
    type G = never & {}; // never
    
    // 주의할 점
    
    type I = null & {a:'b'}; // never
    type J = {} & string // string
    
    type H = {a:'b'} & number; // type H는  {a:'b'} & number
    ```
    type H는 never로 추론되는 게 맞아 보이지만 
    원시 자료형(null, undefined 제외)과 채워 진 객체를 &해도 never가 나오진 않는다.
    ```
    ```
    

### 2.12 타입도 상속이 가능하다.

- extends를 이용한 타입 상속 with interface
    
    ```tsx
    interface Human {
    	name: string;
    }
    interface Mahwin extends Human {
    	speak():void
    }
    
    ```
    
- & 연산자를 이용한 타입 상속 with type
    - 상속 받는 다는 것은 더 좁은 타입이 만들어 진다는 것을 의미하기 때문에 &연산자를 사용
    
    ```tsx
    type Human {
    	name:string;
    }
    type Mahwin =  Human & {
    	speak():void;
    }
    ```
    
- type 별칭과 interface 섞어서 상속하기
    
    ```tsx
    type Human {
    	name:string;
    }
    interface Mahwin extends Human {
    	speak():void;
    }
    interface YouSeock extends Human {
    	coding():void;
    }
    
    // 한 번에 여러 타입 상속 받기
    interface Me extends Mahwin,YouSeock {}
    
    // 부모 속성의 타입을 변경할수도 있음
    interface Merge {
    	one:string
    }
    interface Merge2 extends Merge{
    	one:'a' // ok
    }
    // 대입 못하는 타입으로 변경하는 것은 불가능하다.
    // one:number  error
    ```
    

### 2.13 `객체 간에 대입할 수 있는지 확인하는 법`

- 좁은 타입은 넓은 타입에 대입할 수 있지만 넓은 타입은 좁은 타입에 대입할 수 없다.
    
    ```tsx
    interface IWide {
    	name:string;
    }
    
    interface INarrow extends IWide {
    	age:number;	
    }
    
    const wideObj = {name:"zero"};
    const narrowObj = {name:"zero" , age:32};
    
    // 알맞게 넣은 타입
    const wideToWide : IWide = wideObj // ok
    const narrowToNarrow : INarrow = narrowObj // ok
    
    // 잉여 속성 검사 무시 참조값으로 넣으니까
    const narrowToWide : IWide = narrowObj // ok
    const wideToNarrow : INarrow = wideObj // error 
    
    // 리터럴로 넣어줬다면 narrowToWide,wideToNarrow 둘다 에러남
    // narrowToWide age를 할당할 수 없다고 얘기하고
    // wideToNarrow는 더 넓은 타입을 좁은 타입에 할당할 수 없다고 나옴
    const narrowToWide : IWide = {name:"zero" , age:32}; // error
    const wideToNarrow : INarrow = {name:"zero"};        // error 
    ```
    
- 객체 타입에서 | 연산
    
    ```tsx
    interface A {
    	name :string;
    }
    interface B {
    	age:number;
    }
    function fn() A|B { 
    	 // .. 
    }
    
    // 합집합은 각자의 집합이나 교집합보다 넓다
    // 넓은 타입에 좁은 타입을 대입할 순 없다.
    
    const target1:A&B = fn(); // error
    const target2:A = fn(); // error
    const target3:B = fn(); // error
    ```
    
- 튜플과 배열
    - 튜플은 배열보다 좁은 타입이다.
    - 좁은 타입은 튜플을 배열에 대입할 순 있으나 넓은 타입인 배열을 튜플에 넣을 순 없다.
        
        ```tsx
        let a:['hi','read'] = ['hi','read'];
        let b: string[] = ['hi','read'];
        a=b; // error 넓은 타입인 배열을 튜플 타입에 넣으려니 error 발생
        b=a;
        ```
        
    - 배열과 튜플에서 readonly 수식어
        - readonly 수식어가 붙은 타입이 더 넓다.
        
        ```tsx
        let a : readonly string[] = ['a','b'];
        let b : string[] = ['a','b'];
        
        a=b; 
        b=a; // error readonly가 붙은 a가 더 넓은 타입이라 에러
        ```
        
    - readonly 튜플과 일반 배열의 타입 비교
        
        ```tsx
        let a:readonly ['a','b'] = ['a','b'];
        let b:string[] = ['a','b'];
        a=b; // error 넓은 타입을 좁은 타입에 넣으면 안된다 (튜플,배열)
        b=a; // error readonly가 붙으면 일반 배열보다 넓은 의미라서 
        ```
        
- {} 의 타입 비교
    - 옵셔널 속성에 따른 타입
    - 더 넓은 타입은 좁은 타입에 대입할 수 없다가 되려면 optionalObj에 a,b 프로펄티 둘 다 채워야하지 않을까..?
    
    ```tsx
    type optionalType {
    	a?:string;
    	b?:string;
    }
    type fixedType{ a:string; b:string }
    
    const optionalObj : optionalType = {a:"hi", b:"hello"}
    const fixedObj : fixedType = {a:"hi",b:"hellow"}
    
    const o2: optionalType = fixedObj;
    const m2: fixedType = optionalObj; // error
    // 옵셔널 때문에 a가 string | undefined로 더 넓은 객체가 됨.
    ```
    
    - readonly 속성에 따른 타입
        - readonly 속성은 {} 에서는 타입에 영향을 미치지 않는다.
    
    ```tsx
    type optionalType {
    	readonly a:string;
    	readonly b:string;
    }
    type fixedType{ a:string; b:string }
    
    const optionalObj : optionalType = {a:"hi", b:"hello"}
    const fixedObj : fixedType = {a:"hi",b:"hellow"}
    
    const o2: optionalType = fixedObj;
    const m2: fixedType = optionalObj;
    ```
    

### 2.13.1 구조적 타이핑

- 타입스크립트에서는 구조가 같다면 같은 타입으로 판단한다.
- 정확히는 충분 조건을 따짐.

```tsx
interface 주식 {name:string; price:number};
interface 코인 {name:string; price:number};
const 에이다 : 코인 = {amount:1,price:200};
const 셀트리온 : 주식 = 에이다;

interface A {a:string;}
interface B {a:string; b:string;}
// B 타입에 A 타입에 할당 o B는 A이기 위해 충분
// A 타입은 B 타입에 할당 x A는 B이기 위해 불충분
```

- 매핑된 객체 타입에 적용된 구조적 타이핑

```tsx
type numArrType = number[];
type CopyType = {
	[key in keyof numArrType] : numArrType[key];
}

const copy: CopyType = [1,3,9];
// CopyType는 객체 타입인데도 숫자 배열을 대입할 수 있음.
// 구조적 타이핑 때문.

type SimpleArr = {[key:number]:number, length:number};
const simpleArr :SimpleArr = [1,2,3]; 

```

- 구조는 같지만 다른 타입으로 분리하고 싶다면?
    - __type과 같이 브랜드 속성을 붙여준다.

```tsx
interface 주식 {__type:'주식'; name:string; price:number};
interface 코인 {__type:'코인'; name:string; price:number};
```

### 2.14 `제네릭으로 타입을 함수처럼 사용하기`

- 제네릭은 <>으로 표기하며 실제 타입 인수를 매개변수 처럼 사용할 수 있다.
- 타입 선언의 중복을 해결
- 일반적으로 대문자로 표현
    
    ```tsx
    interface Array<T>{
    	[key:number]:T,
    	length:number,
    	// 기타 속성
    }
    // T가 string string 배열
    // T가 boolean boolean 배열
    ```
    
- 클래스, 타입 별칭에서 제네릭 사용하기
    
    ```tsx
    type Person<N,A> = {
    	name:N,
    	age:A
    }
    
    class Person<N,A>{
    	name:N;
    	age:A;
    	constructor (name:N, age:A){
    		this.name = name;
    		this.age = age; 
    	}
    }
    ```
    
- 함수에서 제네릭 사용하기
    
    ```tsx
    //함수 표현식
    const fn = <N,A>(name:N,age:A) =>{};
    //함수 선언식
    function fn2<N,A>(name:N,age:N){};
    ```
    
- interface와 type간 교차도 가능
    
    ```tsx
    interface IPerseon<N,A>{
    	name:N,
    	age:A
    }
    type TPerson <N,A> = {
    	name:N,
    	age:A,
    }
    
    type Zero = IPerson<'zero', 29>;
    interface Nero extends TPerson('nero',32){}
    ```
    
- 제네릭의 위치
    - interface 이름<>
    - type 이름<>
    - class 이름<>
    - function 이름<>
    - const 함수 = <>()
- 특정 메서드에 제네릭 적용하기
    
    ```tsx
    class Person<N,A>{
    	...
    	method<B>(param:B){}
    }
    interface IPerson<N,A>{
    	...
    	method: <B>(param:B) => void;
    }
    ```
    
- 타입 매개변수에 default type 사용가능
    
    ```tsx
    iinterface P<N=string, A= number) ...
    ```
    
    - 예시
    
    ```tsx
    interface Person<N,A>{name:N,age:A};
    const getPerson = <N,A=unknown>(name:N,age:A) : Person=>({name,age})
    const zero = getPerson('z',222);
    // Person<string,number> 으로 추론
    // unknown보다 좁은 number로
    ```
    
- 상수 타입 매개변수
    
    ```tsx
    function values<T>(a:T[]){
        return {
            hasValue(value:T) {return a.includes(value)}
        }
    }
    const s = values(['a','b','c']);
    // T는 string으로 추론됨. 'a'| 'b'| 'c'와 같은 유니온으로 추론되게 하고 싶으면?
    
    // 4,9 버전 전에는
    function values<T>(a: readonly T[]){
        return {
            hasValue(value:T) {return a.includes(value)}
        }
    }
    const s = values(['a','b','c']);
    
    // 5.0 이상 버전에서는 
    function values<const T>(a:T[]){
        return {
            hasValue(value:T) {return a.includes(value)}
        }
    }
    const s = values(['a','b','c']);
    ```
    

### 2.14.1 `제네릭에 제약 걸기`

- 타입 매개변수에 extends 문법을 사용해서 매개변수 타입에 제약을 걸 수 있다.
    
    ```tsx
    interface E<A extends number, B= string>{
    	a:A,
    	b:B,
    }
    ```
    
- 하나의 매개변수 타입이 다른 매개변수의 제약이 될 수도 있다
    
    ```tsx
    interface E <A, B extends A> {}
    
    type u = E<string,'11'> // ok
    type u2 = E<number,11> // ok
    type u3 =E<number,'1'> // XX
    ```
    
- 자주 사용되는 제약
    
    
    | <T extends object> 모든 객체  |
    | --- |
    | <T extends any[]> 모든 배열 |
    | <T extends (…args:any⇒any))> 모든 함수 |
    | <T extends abstract new (…args:any)⇒ any> 생성자 타입 |
    | <T extends keyof any> 속성의 키  // string | number | symbol |
    - <T extends abstract new (…args:any)⇒ any>
        - T가 추상 클래스며 생성자 시그니처를 가져야하고, 어떤 타입의 인수도 받을 수 있고, 어떤 타입의 값도 반환할 수 있는 값으로 제한한다.
        - …으로 전개 구문 사용한 이유는 매개변수가 많을 경우를 대비.
- 제네릭에서 자주하는 실수
    
    ```tsx
    interface VO {
    	value:any;
    }
    const returnVO = <T extends VO>():T=>{
    	return {value:'text'}
    }
    // error T는 정확히 VO가 아니라 VO에 대입할 수 잇는 모든 타입을 의미한다.
    // 다시 말하면 {value:'text', name:'zzz', age:5} 같은 값도 T가 될 수 있음.
    
    const re = <T extends number>():T=>{
        return 3
    }
    // error도 마찬가지 T에 1,2,3,4,5,... 가 다 들어갈 수 있는데
    // 1,2,3,4,5,...,와 3은 같지 않으니까 error
    // re<5>() => T는 5인데 return은 3이니까
    ```
    
    - 천천히 생각해보면 T extends VO가 의미하는 것은 VO에 대입할 수 있는 모든 값들이 T가 될 수 있다는 것을 의미한다.
    - ts에서 대입은 좁은 타입에서 넓은 타입으로 이루어지니까 {value:’123’, age:3}같은 좁은 타입이 넓은 VO타입에 대입될 수 있는 것이다.
    
    ```tsx
    function onlyBoolean<T extends boolean>(arg:T = false): T {
    	return arg;
    } // error
    
    function onlyBoolean<T extends boolean>(arg:T = false as any): T {
    	return arg;
    } // ok
    
    function onlyBoolean<T extends boolean>(arg:T=false as never) :T{
    	return arg;
    }// ok
    
    // T extends boolean에서 
    // T는 true, false, never가 될 수 있다.
    // 만약에 never가 들어오면  
    ```
    
    - 더 좋은 해결 방법
        - 제네릭을 쓰지말자
        - 원시값 타입만 사용한다면 대부분 제네릭이나 제약을 걸지 않아도 된다.
    
    ```tsx
    function onlyBoolean(arg:true|false=true):true|false{
    	return arg
    }
    
    function onlyBooleana(arg:boolean=true):boolean{
    	return arg
    }
    
    interface VO {
    	value:any;
    }
    
    const f = () : VO=>{
    	return {value:'test'}
    }
    ```
    

### 2.15 `조건문과 비슷한 컨디셔널 타입`

- 조건에 따라 다른 타입이 되는 타입을 컨디셔널 타입이라고 한다
- extends 연산자를 이용해 삼항연자가 처럼 사용된다
    - 특정 타입 extends 다른 타입 ? 참일 때 타입 : 거짓일 때 타입
- extends해야 조건을 만족하는 것은 아니고, 대입 관계를 따진다.

```tsx
type A1 = string;
type B1 = A1 extends string ? number : boolean; // number 타입

type A2 = number ;
type B2 = A2 extends string ? number : boolean; // boolean 

type Start = string | number;
type New = Start extends string | number ? Start[] : never;

let n: New = ['hi']
n = [123]

type New = Start[] // 이랑 뭐가 달라?  never를 꼭 분리해 주고 싶을때!

// strArr 와 stringArr는 never 포함 여부에 따라 다름!
type strArr = string[] 
type Choose<T> = T extends string ? string[] : never;
type stringArr = Choose<string>
type Never = Choose<number>
```

- 매핑된 객체 타입에서 키가 never면 해당 속성은 제거된다
    - 이를 이용해서 특정 value의 타입만 필터링 할 수 있다!

```tsx
type OmitByType<O,T> = {
	[K in keyof O as O[K] extends T ? never :K] : O[K];
};

type Result = OmitByType<{
	name:string;
	age:number;
	married:boolean;
	rich:boolean
}, boolean>;

const A : Result = {name:'cccc',age:1231231231} // 대박
```

- 당연히 삼항연산자처럼 중첩해서 사용할 수 있다.

```tsx
type C<A> = A extends string 
? string[] 
: A extends boolean ? boolean[]:never;
// string이면 string[] boolean이면 boolean[] 그것도 아니면 never
```

- 인덱스 접근 타입으로 컨디셔널 타입 표현하기

```tsx
type A1 = string;
type B1 = A1 extends string ? number : boolean;
type B2 = {
	't':number;
	'f':boolean;
}[A1 extends string ?'t':'f']
```

### 2.15.1 `컨디셔널 타입 분배법칙`

- string | number 타입이 있는데 string[] 타입을 얻고 싶다면 어떻게 할까
- 어떻게 다른 값이 나올까?
- 검사하려는 타입이 제네릭이면서 유니언 타입이면 분배법칙이 실행된다.
    
    ```tsx
    type Start = string | number;
    
    type wrongResult = Start extends string ? Start[]: never;
    // never 타입이됨.
    
    type Result<Key> = Key extends string ? Key[] : never;
    let n : Result<Start> = ['hi'];
    let n : Result<Start> = [123]; // error
    
    // Result<string | number>
    // Result<string> | Result<number>
    
    // Result<Key:string> = Key extends string ? Key[] : never;
    // Result<Key:number> = Key extends string ? Key[] : never;
    ```
    
- boolean에 분배법칙이 적용될 때는 조심하자
    - boolean은 애초에 true | false의 유니온 집합이다
    
    ```tsx
    type Start = string | number | boolean;
    type Result<Key> = Key extends string | booelan ? Key[]:never;
    let n : Result<Start> = ["hi"]; 
    // let n:string[] | false[] | true[]
    ```
    
- 분배법칙 막기
    - 배열로 제네릭을 감싸면 막아짐
    
    ```tsx
    type IsString<T> = T extends string ? true : false;
    type Result = IsString<'hi'|3>; 
    // Result는 boolean
    
    type IsString<T> = [T] extends [string] ? true : false;
    type Result = IsString<'hi'|3>;  // false
    ```
    
- never도 분배법칙의 대상이다.
    - 기본적으로 never면 never extends 타입은 항상 참이라 true가 나와야하지만
    - never도 분배되면서 공집합이라 아무것도 실행되는 게 없어서 never가 나온다.
    
    ```tsx
    type R<T> = T extends string ? true : false;
    type RR = R<never>; // type RR = never
    ```
    
    - never의 분배법칙을 막아보자
        - never도 분배법칙을 막을 수 있네!
    
    ```tsx
    type R<T> = [T] extends [string] ? true : false;
    type RR = R<never>; // true!!
    ```
    
    - 주의사항
        - 제네릭과 컨디셔널 타입을 함께 쓸때는 분배법칙을 조심하자.
    
    ```tsx
    function test<T>(a:T){
    	type R<T> = T extends string ? T :T;
    	const b:R<T> = a;
    }
    
    // 타입스크립트는 제네릭이 들어 있는 컨디셔널 타입의 경우 판단을 뒤로 미룬다
    // type R<T> = T extends string ? T :T;에서 R<T>는 무조건 T 타입을 가진다고
    // 생각하겠지만 타입스크립트는 팓단을 뒤로 미루기 때문에
    // b: R<T> = a할때 a는 T라고만 생각하고 R<T>가 T라고 확신하지 못해 에러가 발생함.
    
    function test<T>(a:T){
    	type R<T> = [T] extends string ? T :T;
    	const b:R<T> = a;
    }
    // []로 제네릭을 감싸면 지금 즉시 타입 판단을 하라고 알려줌
    // 이렇게 하면 error가 없어진다.
    ```