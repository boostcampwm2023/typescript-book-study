# 1 주차 학습✨

작성일: 2023년 10월 7일 오후 8:17

### 2.1 `변수, 매개변수, 반환값에 타입을 붙이자`

- 기본 타입에는
    - string, number, boolean, null, undefined, symbol, bigint, object가 있다
    - 함수, 배열도 object에 포함된다.
    

### 2.2`타입 추론을 적극 활용하자`

- 타입을 명시하지 않으면 타입스크립트가 적절한 타입을 추론함.
- 매개변수에는 꼭 타입을 부여하자.
    - 매개변수에 타입을 부여하지 않으면 any 형식으로 추론되는데, 이는 전체 타입 시스템을 무너뜨림.
- 변수 선언 방법에 따라 추론된 타입이 달라지기도 한다.
    
    ```css
    const name =  "유석" => "유석"으로 타입 추론
    let name = "유석"    => string으로 타입 추론
    
    const n : null => null로 추론
    let n : null => any로 추론
    
    const n : undefined => undefined로 추론
    let n : undefined => any로 추론
    
    // const 심볼은 그 고유한 심볼 자체지만 let으로 선언한 심볼은 다른 심볼 값으로 바뀔 수도 있음!
    // ts에서는 const로 선언한 심볼을 유니크 심볼이라고 하며 유니크 심볼 끼리 비교를 막고 있다.
    const s: Symbol('s'); => const s: typeof s
    let s: Symbol('s'); => let s: symbol
    ```
    
- 타입스크립트 추론을 최대한 이용하며, 잘못 추론될 경우에만 명시적으로 타입을 지정해주자.
- 적절히 타입추론을 사용하고, 추론된 타입의 범위가 좁다면 타입 넓히기를 적극 활용하자.
    
    > : { } 로 타이핑하면 null | undefined 를 제외한 모든 타입을 의미한다. 
    @ts-ignore를 사용해서 ts에러를 무시할 수 있다.
    @ts-expect-error 를 사용해서 에러가 날 상황을 무시할 수 있다.
    반대로 에러가 발생하지 않으면 에러를 발생 시킨다.
    > 

### 2.3`값 자체가 타입인 리터럴 타입이 있다.`

- 값 자체를 타입으로 지정할 수 있으며, 이는 타입의 안정성을 위해 자바스크립트가 가진 자유도 제한하는 행위이다.
- 원시타입 외에 리터럴 타입도 있다 (객체 리터럴 타입)
    - 함수 리터럴 타입에서는 반환값을 ⇒ 로 표현한다
    
    ```tsx
    const obj : {name:'유석'} = {name:'유석'};
    const nums : [1,2,3] = [1,2,3];
    const fn : (n:number) => number =(n)=>n*2);
    ```
    
    - 일반적인 타입스크립트의 타입 추론과 반대로 더 부정확하게 추론한다.
        - 객체타입은 항상 수정 가능성이 있기 때문
        
        ```tsx
        const obj ={name:'유석'} // => {name:string} 
        ```
        
        - 일반적인 타입에서 타입 넓히기를 했던 것과 반대로 타입 좁히기를 해서 명확한 타입을 명시할 수 있다.
            - 이런식으로 타입을 명시하면 readonly 변수가 됨.
        
        ```tsx
        const obj ={name:'유석'} as const;
        ```
        

### 2.4 `배열 말고 튜플도 있다.`

- 배열은 동적으로 길이가 달라지기 때문에 일반적인 방법으로 타입을 지정할 수 없다.
    - ts에서 채택한 방법
    
    ```tsx
    const arr: number[] = [1,2,3];
    const arr: Array<number> = [1,2,3];
    ```
    
- 빈 배열은 안 요소가 any로 추론된다
    
    ```tsx
    const arr = []; // => any[]
    ```
    
- 배열의 특정 요소에 접근할 때 생기는 에러에 대해서 어떻게 런타임 전에 알 수 있을까? 튜플로 해결!
    - 튜플은 각 요소 자리에 타입이 고정되어 있는 배열
    - 길이의 고정은 아니다.
    - 배열의 특정 요소에 접근할 때 생길 수 있는 에러의 예시
        - 배열의 인덱스를 넘어가는 값에 접근할 때
        - 요소의 타입과 관계없는 메소드 호출
    
    ```tsx
    const tuple : readonly [number,boolean,string]=[1,false,'유석'];
    ```
    
    - 위와 같이 타입을 직접 명시하면 특정 위치에 값을 넣거나 꺼내 사용할 때 발생할 수 있는 에러를 런타임 전에 알 수 있다.
    - push, pop, unshift, shift 등의 메서드에 대해서는 열려있지만 readonly 수식어를 붙여 push 사용도 막을 수 있다.
    - 튜플도 하나의 타입처럼 사용할 수 있다.
        
        ```tsx
        const 이름나이튜플배열 : [string,number][] = [['크롱',5],['뽀로로',6]]
        
        ```
        
    - 튜플과 배열의 조합 예시
        
        ```tsx
        cconst 회원정보와헬스장체크인 : [string,number,...boolean[]]
        = ['크롱',5,false,false,false,false,false];
        const 회원이름과몸무게변화와현재피티여부체크:[string,...number[],boolean]
        = ['크롱',3,4,1,2,3,3,4,1,3,true];
        const 유저닉네임변경내역과변경횟수및유료회원체크 :[...string[],number,boolean]
        = ['크롱','크로옹','크','크롱',4,true];
        ```
        
    - 타입이 아닌 객체에 전개 문법을 사용해도 타입을 추론 해낸다
        
        ```tsx
        const 에이다 = ['에이다',350];
        const coinPriceArr : 에이다[] = [['에이다',350], ['리플'1000],['루나',0.1]]
        ```
        
    - 옵셔널 수식어를 사용해서 유연한 객체 타입 만들기
        - 있어도 되고 없어도 되는 경우에 ? 연산자를 뒤에 붙인다.
        
        ```tsx
        let 비밀정보 :[string,boolean?,string?]=['마윈',false,'password']
        비밀정보 = ['크롱',true] // ok
        비밀정보 = ['크롱'] // ok
        비밀정보 = ['크롱','passs'] // x boolean|undefined에는 passs를 할당 x
        ```
        

### 2.5 `타입으로 쓸 수 있는 것을 구분하자`

- 값은 타입으로 쓸수도 있다.
- 타입은 값으로 쓸수 없다
    
    ex) 타입으로 쓸 수 있는 예
    
    - 클래스
    - 대부분의 리터럴 값
    - 내장 메서드
        - Math
        - Error
        - ~~Symbol~~
        - ~~String~~
        - ~~Object~~
        - ~~Number~~
        - ~~Boolean~~
    - 내장 메서드를 타입으로 사용할 때 발생할 수 있는 문제
    
    ```tsx
    // Number는 메서드기 때문에 Number + Number는 타입스크립트가 이해할 수 없다
    function add(n1:Number,n2:Number):Number{return n1+n2} // XXX
    // String와 string도 다른 타입으로 추론한다.
    const str:String='name'
    const str2:string= str // String 은 string에 할당할 수 없다.
    const obj:Object='what?'; // 왜 문자열 대입도 되냐..? 
    ```
    
    - 함수의 반환값을 타입으로 사용할 수 있다.
    - 함수의 호출로 타입을 지정할 순 없다.
        - 함수의 리턴값을 타입으로 사용은 나중에 배움 🌻
    
    ```tsx
    function returnNumber:(num:number) => number { return 3 };
    function a: typeof returnNumber = ()=>4;
    function a :typeof returnNumber(4) = ()=>4; // xxxx
    ```
    

### 2.6 `유니언 타입으로 OR 관계를 표현하자`

- 파이프 연산자 | 를 사용해서 유니온 타입을 만들 수 있다.
    - 유니온 타입이란 하나의 변수가 여러 타입을 가지는 것을 의미.
- union 타입 사용에 대한 예시

```tsx
fucntion fn(value:string|number){
	return parserInt(value);
}
fn(1);
fn('1');

// js에서는 문제 없이 돌아가지만
// ts에서는 에러남. 1을 넣어도 그대로 1인데 왜 하냐? 라고 타입스크립트는 알려줌
// 이럴 경우 type narrowing을 사용해서 parserInt를 거쳐야할 경우를 명확히 알려줘야 함.

fucntion fn(value:string|number){
	if(typeof value === 'string') value = parseInt(value);
	return value;
}
```

- string 값을 toString()하는 것은 막지 않음.
- 여러 줄려 union 타입을 적고 싶을 때
    
    ```tsx
    type union =
    | string
    | number
    | boolean
    ```
    

### 2.7 `타입스크립트에만 있는 타입을 배우자`

### 2.7.1 `any`

- any 타입은 모든 동작을 허용한다.
    - any로 타입을 지정하면 런타임에 가서 에러를 뱉는데, 그럴거면 타입스크립트 왜 씀.
    - any 타입으로 타이핑된 변수는 연산 과정에서 주변 타입도 오염시킴.
    - any 타입은 타입 검사를 포기한다는 선언과 같다. 타입스크립트가 any를 추론하는 타입이 있다면 타입을 직접 표기해야 한다.
    - any 의 재밌는 사실
    
    ```tsx
    const arr =[]; // any[]
    arr.push(boolean); // boolean[];
    arr.push(3); // (booelan|number)[]
    // 요소가 빠진다고 타입이 이전으로 돌아가진 않음.
    arr.pop(); // (booelan|number)[]
    
    // but concat은 사용못함!
    const arr=[]; // any[]
    const newArr = arr.concat('123'); // 
    
    // any 타입은 연산과정에서 변하기도 한다.
    
    const a: any='123';
    const a1 = a + 1; // any로 추론   => 얘도 무조건 string 아니냐?
    const a2 = a-1; // number로 추론
    const a3 = a*1; // number로 추론
    const a4 = a + '1' // string으로 추론
    ```
    
    - 타입스크립트가 명시적으로 any를 반환하는 경우
        - JSON.parse와 fetch
    
    ```tsx
    fetch('url').then((res)=>{
    	return res.json();
    }).then((data)=>{     // data가 any가 됨.
    	
    	})
    )
    
    // JSON.parse를 어떻게 타이핑 할까
    interface IObj {
        name: string;
        age: number;
    }
    
    let obj: IObj = JSON.parse('{ "name": "klong", "age": 4 }');
    ```
    
    - ❓❓❓❓
        - fetch나 api로 가져오는 데이터의 타입을 명확히 지정하는 게 가능함?

### 2.7.2 `unknown`

- 모든 타입에 대입할 수 있지만, 그 후 어떠한 동작도 할 수 없게 된다.
    
    ```tsx
    const a : unknown;
    a = true; // ok
    const b:boolean = a; // Error 
    ```
    
- 직접 unknown 타입을 지정하는 일은 거의 없고 try catch문에서 사용됨.

```tsx
try{
} catch(e){   // e : unknown으로 추론됨.
	console.log(e.message);}
```

- try catch문의 e의 타입은 any와 unknown 외의 타입을 직접 표기할 수 없고 ~ 타입이라고 주장할 순 있다.

```tsx
try{

} catch(e){   
	const error = e as Error;    // Error타입이라고 주장 <Error>e와 같지만 지양해라.
	console.log(error.message);
}
```

- 타입 주장
    - 불가능한 상황에서는 타입 주장을 할 순 없다
    - 불가능한 상황에서도 타입 주장을 할 수 있는 트릭은 있지만, 책임은 본인의 몫

```tsx
const a: number='123' as number; // error 
const a: number='123' as unknown as number; // ok
```

- ! 연산자를 이용해서 null과 undefined가 아님을 주장할 수 있다.

```tsx
type aType = string | null | undefined;

function a (param:aType){
	param.slice(3);    // error null or undefined이면 에러나
}

function a (param:aType){
	param!.slice(3);    // !를 붙임으로 명시적으로 null과 undefined이 아닐거라고 알려줌.
}
```

### 2.7.3 `void`

- js에서는 연산자지만 ts에서는 타입으로 사용된다.
    - js에서 void 연산자
        - void 연산자는 뒤에 나타나는 표현식을 실행하고 무조건 undefined를 리턴
        - 언제 쓰냐?
        - 완전한 undefined를 얻기 or 즉시 실행함수
            
            ```tsx
            // 오래된 브라우저에선 전역에 undefined를 변수로 선언할 수 있다
            let undefined = 'test';
            let test;
            console.log(Boolean(test === undefined));
            // IE8 이하의 브라우저 false
            // 그 이상의 브라우저 true
            
            // 즉시 실행함수
            (function() {
                var undefined = 'test';
                console.log(undefined);
            })();
             
            void function() {
                var undefined = 'test';
                console.log(undefined);
            }();
            ```
            
        
        [void - JavaScript | MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/void)
        
- 반환값이 없는 경우에 void 타입으로 추론된다.
- 반환되는 값을 막는게 아니라 반환값을 무시하도록 한다.
    - () ⇒ void    반환 값이 있을수도 있지만 반환값을 무시해줘
        - func
    - () : void   반환 값의 타입만 명시하는 경우 실제 반환값이 void 여야만 함.
        - func2
    - () ⇒ void | undefined 유니온 타입이면 반환값을 무시하지 않는다.
        - func3

```tsx
const func : () => void = () => 3; // 리턴값이 void는 아니지만 void 할당은 가능
const value= func() // 할당받은 값은 있지만 type은 void임.

// 반환값의 타입만 지정한 경우에는 반환값을 무시하지 않는다.
const func2 = ():void => 3;  // error void가 들어와야하는데 number가 들어왔어!

const func3 = () => void | undefined = () => 3; // error
```

- void의 등장배경과 실제 사용 목적
    - 반환값을 무시하기 때문에 콜백함수에서 주로 사용됨!
    - 🍀 사용자가 함수의 반환값을 사용하지 못 하도록
    - 🍀 반환값을 사용하지 않는 콜백 함수를 타이핑할 때

### 2.7.4 `{}, Object`

- { } 타입은 객체가 아니라 null과 undefined를 제외한 모든 타입의 유니온 타입이다
- { } 타입은 Object 타입과 정확히 같으며 객체만 대입할 수 있는 것은 아니다
- { }  | null | any = unknown
    - null과 any를 제외한 모든 값들을 대입할 수 있지만 실제로 사용할 순 없다.
    
    ```tsx
    const obj : {} = {name:'유석'}
    obj.name // error 실제로 사용하려고 하면 error를 뱉는다.
    ```
    

### 2.7.5 `never`

- never 타입에는 어떠한 타입도 대입할 수 없다
- 함수 선언문과 함수 표현식의 차이
    
    ```tsx
    // 함수 선언문
    function 선언함수(){throw new Error("에러")} // return void
    
    // 함수 표현식
    const 표현함수(){throw new Error("에러")} // return never
    
    // 무한 반복하는 함수의 경우
    function neverEndingFuc(){while(1){}}; // return void
    const neverEndingFuc = () => {while(1){}}; // return never
    
    // never를 리턴하게 하고 싶다면 함수 선언문의 리턴 타입을 :never로!
    ```
    
- 파라메타가 never로 추론되는 경우
    
    ```tsx
    function strOrNum(value: string|number){
    	if(typeof value ==='string'){
    	else if(typeof value==='number'){
    	} else {
    		param ; // value가 string과 number가 아닌 경우는 없기에
    	}					// value :never로 타입스크립트가 추론해줌. 
    	}         
    }
    ```
    
- noImplicitAny 옵션
    
    ```tsx
    const arr =[] // any[] 였던 것이 noImplicitAny를 키면
    const arr =[] // never[]이 되어서 직접 배열의 타입을 지정해줘야 error가 없어짐!!!!
    ```
    

### 2.7.6 `타입 간 대입 가능표`

- 외우면 바보 ts가 알아서 알려줄텐데..

|  | any | unknown | {} | void | undefined | null | never |
| --- | --- | --- | --- | --- | --- | --- | --- |
| any |  | O | O | O | O | O | X |
| unknown | O |  | X | X | X | X | X |
| {} | O | O |  | X | X | X | X |
| void | O | O | X |  | X | X | X |
| undefined | O | O | X | O |  | X | X |
| null | O | O | X | X | X |  | X |
| never | O | O | O | O | O | O |  |

### 2.8 `타입 별칭으로 타입에 이름을 붙이자`

- type alias로 특정 타입에 이름을 붙여서 사용할 수 있다.
- 객체 타입이나 유니온 타입에 붙여서 사용하자!
- 대문자로 시작

```tsx
type Name = "주은" | "서하" | "송하";
type NewName = "유석";
type TotalFamilyNames  = Name | NewName;
```

### 2.9 `인터페이스로 객체를 타이핑하자`

- 타입 별칭과 마찬가지로 객체에 타입을 붙이는 방법 중 하나.
- 대문자로 시작
- `,` `;` `\n`로 구별할 수 있지만, 한 가지 방법을 채택해서 일광성 있게 작성하자.

```tsx
interface Member {
	name: string;
	age: number;
}

type MemberArr =  Member[];
```

- ts에서 객체의 속성 키는 string, symbol, number(string으로 변환 )
- 튜플의 길이를 명시하지 않고 사용했던 것 처럼 인덱스 시그니처를 사용해서 객체의 key: value를 명시적으로 표현하지 않고도 객체의 타입을 지정할 수 있다.
    - 사실 js에서 배열도 다른 언어의 배열처럼 동작하는 게 아니라 key value 쌍으로된 객체니 당연한 내용임.

```tsx
const Arr = {
	name:string;
	[key:number] :string;
} 
const arr : ['3','4','5']
```

- ****자바스크립트에서 [object Object] 가 대체 뭘까?****
    - "[object Object]"는 JavaScript에서 개체를 문자열로 변환할 때 나타나는 표준 문자열
    - 이 문자열은 개체의 내용을 표시하지 않으며 단순히 개체가 있다는 사실을 나타냄

```tsx
const obj = {
	'****[object Object]****': '너 객체구나!'
}
//obj[{}] => 
// {}는 toString에 의해 '****[object Object]****'가 되고 '****[object Object]****'에 해당하는 
// value 값인  너 객체구나를 얻게 됨!!
```

- 속성이 없는 interface는 {} 처럼 행동한다.

```tsx
interface NoProp{}
const obj:NoProp = {'key':'value'}; // ok
const obj:NoProp=null; // error
```

### 2.9.1 `인터페이스 선언 병합`

- 인터페이스끼리는 서로 합쳐진다.
- 같은 이름으로 선언한 인터페이스를 합쳐짐 (선언병합)
    - 특정 라이브러리 내에 선언된 타입과 내가 선언한 타입의 이름이 같았을 때, 서로의  타입을 오염시키면 큰일이니까 그냥 합쳐버림!
    - 다만 같은 속성을 다르게 정의할 경우에는 에러를 뱉는다!

```tsx
interface Merge {'one':string}; 
interface Merge {'two':string};
interface Merge {'three':string};

const sum = {one:'1',two:'2',three:'3'};
```

### 2.9.2 `네임스페이스`

- 의도치 않은 인터페이스 병합을 방지하기 위해 하나의 타입에 대한 block을 만들어 사용할 수 있다.
- 네임스페이스 내부의 타입을 사용하려면 꼭 export 해야한다.

```tsx
namespace E {
	export interface Inner { test:string}
	export type Test2 = number;
}
```

- 네임스페이스 중첩

```tsx
namespace E {
	export namespace Outer{
		export interface Inner {
			test:string;
		}
		export type Test2 = number
	}
}
const ex1: Example.Outer.Inner = { test :"hell이네", }
const ex2 : Example.Outer.Test2 =123;
```

- 네임스페이스 내부에 실제 값을 선언한 경우에 자바스크립트 값으로 사용가능
    - 내부 타입은 []를 이용해서 접근할 수 없다.

```tsx
namespace Ex { export const a = 'real'; }
const a = Ex;
const b = Ex.a;
const c = Ex["a"]
namespace Ex { export const a = string; }
const d:	Ex['a'] = 123; // error 내부의 타입은 []로 접근 xx 🔥
```

- 네임스페이스에서의 병합
    1. 네임스페이스의 이름도 같고 인터페이스의 이름도 같다면 인터페이스 병합이 이뤄진다.
    2. 네임스페이스의 이름도 같고 타입의 이름도 같다면 Error를 발생 시킨다.
    
    ```tsx
    namespace E {
    	export interface 동일한이름의인터페이스 { test1: string}
    	export type 동일한이름의타입 = number; // 중복 에러
    }  
    
    namespace E {
    	export interface 동일한이름의인터페이스 { test2: number}
    	export type 동일한이름의타입 = number;  // 중복 에러
    }  
    
    const 합쳐진인터페이스의타입 : E.동일한이름의인터페이스 = {test1:'합',test2:'체'}
    ```
    
    - namespace의 병합마저 싫다면 모듈 파일을 이용하자!