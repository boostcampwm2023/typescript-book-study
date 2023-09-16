## 2.2, 2.3, 2.4 타입 추론

타입 추론은 타입스크립트가 타입을 자동적으로 추론을 해주는 것을 뜻합니다.

```tsx
const a = 4;
// const a:4
const b:number = 4;
// const b:number
```

다음과 같은 코드가 있을때 개발자가 a에 타입을 붙이지 않아도 타입스크립트가 a에 4라고 붙이는 것을 볼 수 있습니다. 

함수 타입도 마찬가지로 추론을 하고 있는것을 볼 수 있습니다.

```tsx
const func = (a:number,b:number)=> a+b;
// const func = (a:number,b:number)=> number
```

const 외에 let사용에서는 약간 다릅니다.

```tsx
let a = 4;
// let a:number
```

const를 사용했을때는 추론이 4가 되었지만 let을 사용했을 때는 추론이 number로 바뀌었습니다. const는 불변값으로 값이 고정인 반면 let은 다른 값이 들어올 수 있기 때문에 타입이 더 넓혀 진것이죠

비슷하게 객체의 타입도 다릅니다.

```tsx
const a ={
  first:1,
  second:'second'
}
/* 
const a ={
  first:number,
  second:string
}
*/
```

1과 ‘second’ 가 아닌 number와 string으로 추론되고 있는것을 볼 수 있습니다. JS에서는 const여도 객체에 값을 변경할 수 있기 때문에 let과 같이 타입이 넓혀진 상태로 추론되고 있습니다.

그러면 배열이나 튜플은 어떨까요?

```tsx
const a = [1,2,3,4]
// const a:number[]
const b = ['a','b','c','d'];
// const b:string[]
const c = [1,'b', 2,'c', 3];
// const c:(number|string)[]
const d = [...a,...b]
// const d: (string | number)[]
```

배열의 속성을 추론하여 해당 추론의 값을 type으로 사용하고 있는 것을 볼 수 있습니다. 

이런식으로 타입을 추론한다면 다음과 같은 타입 에러를 발생시킬 가능성이 있습니다.

```tsx
c[5].toString();
// c[5]는 존재하지 않는데???
```

따라서 const처럼 타입을 고정 시킬 필요가 있습니다.

```tsx
const a ={
  first:1,
  second:'second'
} as const;
/*
 const a: {
    readonly first: 1;
    readonly second: "second";
}
*/

const c = [1,'b', 2,'c', 3] as const;
// const c: readonly [1,'b', 2,'c', 3];
```

as const라는 접미사를 붙여서 해결할 수 있습니다.

## 2.5 String? string?

ts를 쓰면서 의문이 들었던 부분이다 타입을 쓸때 string같이 소문자로 시작한것을 볼 수 있는데 왜 String이라 쓰면 안되지? 라는 생각을 하곤 했습니다.

지금 내 생각으로는 String이라는 타입은 타입이 아닌 참조 객체죠. 즉 JS에서 Object에서 파생된 객체인 것입니다.

```tsx
console.log(typeof Number)
// 'function'
```

Number이외의 String,Object 객체의 반환 값도 function이었습니다. 즉 우리가 string이 아닌 String으로 가져다 사용하면 string 타입 리터럴이 아닌 class 즉 객체 리터럴을 사용하는 것이고 우리가 예상하지 못하는 문제가 생기는 것인거죠

```tsx
const func = (a:Number, b:Number)=> a+b;
//Error: Operator '+' cannot be applied to types 'Number' and 'Number'.(2365)
```

## 2.7 타입스크립트에만 있는 타입을 배우자

### 1. any

any는 모든 값을 넣을 수 있는 타입입니다.

다음과 같은 상황에 만날 수 있습니다

```tsx
const func = (a, b)=> a+b;
// 1. 파라미터의 타입을 명시하지 않을시
// const func: (a: any, b: any) => any

const a =[];
// 2. 배열의 초기값을 주지 않았을시
// const a:any[];

fetch('url').then((res)=>res.json()).then((result)=>result);
// 3. fetch로 데이터를 받아온 결과값
// result:anty

const str = JSON.parse("{'hello':'hello'}")
// 4. JSON.parse로 객체를 복호화 시킬때
// str:any
```

여기서 재미있는점은 any인 배열에서 push를 하면 push한 타입으로 추론이 된다는 점입니다.

```tsx
const a = [];
a.push(1);
a;
// a:number[]
a.push('a');
a;
// a:(number|string)[]
```

하지만 pop을 한다고 해서 추론이 바뀌지 않습니다.

```tsx
const a = [];
a.push(1);
a;
a.push('a');
a;
a.pop();
a;
// a:(number|string)[]
```

### 2. unknown

any처럼 모든 타입에 대입할 수 있지만 그후 사용할 수는 없습니다.

에러 처리를 할때 만날 수 있습니다.

```tsx
try{
} catch(e){
  e
// e:unknown
}
```

### 3. void

어떤 값을 대입할 수 있으면서 개발자가 해당 반환 값을 사용하지 못할때 사용합니다. 보통은 함수의 return값으로 들어가죠

특히 callback 함수에서 자주 사용하고는 하는데요 생각을 해보면 callback함수의 return값이 뭐가 될지 모르기 떄문에 사용하고는 합니다. 값을 사용하지는 않을건데 입력할때마다 return값이 달라지기 때문에 사용하는 것이죠

```tsx
const func: ()=>void = ()=>3;
```

### 4. Object {}

null과 undefined를 제외한 모든 값을 의미합니다. 

```tsx
const str:{} ="str"
const num:{}=3

const n:{} = null;
const u:{} = undefined;
// 아래 두개는 assigned 할수 없다고 에러가 뜬다

// object로 타이핑 해도 결과는 같다
```

타입을 직접 사용할 일도 없고 해본적도 없지만 unknown타입을 넣을떄 볼 수 있습니다.

```tsx
const unk:unknown = 'hello';

if(unk){
  unk;
// unk:{}
}
else{
  unk;
// unk:unknown
}
```

### 4. never

어떠한 값도 대입할 수 없는 타입입니다.

잘 쓰지는 않지만 해당 사항을 만나면 관련된 코드는 실행이 되지 않겠죠

## 2.8 타입 별칭

타입의 별칭은 PascalCase를 따릅니다.

## 2.9 Interface

기본적으로 interface는 타입을 별칭할때 종종 사용이 됩니다. 또한 기본적으로 같은 이름의 interface끼리는 병합이 되는 특징을 가지고 있죠

```jsx
interface test{
    name:string,
    description:string;
    bool:boolean,
}
```

interface에서는 `,`와 `;` 을 이용해서 구분할 수 있습니다. 다만 사용할때는 한가지로 통일해서 사용하도록 해야할것 같습니다 저는 `;` 을 사용할것 같네요

interface에는 index signature라는 문법이 있습니다.

```jsx
interface test{
	[key: number]:number
}
```

타입을 하나 하나 입력하는것이 아니라 key의 타입이 name인것들의 value값은 number라고 전체다 지정을 해놓는것이죠

이렇게 지정을 해놓으면 동적으로 해당 객체에 값을 추가하게 할 수 있습니다.

### 네임스페이스

interface가 멋대로 결합되다 보니 남이 만든거에 멋대로 침범해 버리는 수가 있었습니다. 따라서 네임스페이스를 이용해서 interface를 관리합니다.

```jsx
namespace Ex{
    export type name=string;
    interface obj{
        test:number;
    }
    export const a = 3;
}

const a:Ex.name = 'wow'
const b = Ex.a
```

js로 컴파일 후

```jsx
var Ex;
(function (Ex) {
    Ex.a = 3;
})(Ex || (Ex = {}));
const a = 'wow';
const b = Ex.a;
```

각각의 type들을 묶어서 하나의 namespace로 활용 할 수 있고 값을 넣어줄수도 있습니다. 하나의 class라고 생각하면 편할거 같습니다.

네임스페이스 끼리도 병합이 가능합니다. 다만 interface와 마찬가지로 같은 속성값에 다른 타입이 들어가는 경우 에러가 발생하게 되죠
