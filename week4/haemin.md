## 2.22 infer

한줄로 설명하면 타입을 추론하게 해주는 문법입니다. 제네릭과 굉장히 햇갈렸는데 제네릭과 다른점은 제네릭은 인자로 받는 것이지만 infer는 받은 인자의 값을 추론하게 해주는 것입니다.

아마 제네릭도 arg에 다음과 같이 쓰면 타입을 추론해 주기 때문에 햇갈리는 것 같습니다.

```tsx
type A<T> = (arg:T)=>void
```

이런식으로 쓰게 되면 arg에 따라서 T를 추론해 주는데요 이는 파라미터로 받는 순간에 추론이 되는 것입니다.

반면에 infer키워드는 타입이 실행될때 추론이 됩니다. 

```tsx
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
```

1. T의 타입을 모든 함수로 제한하는 제네릭을 만듭니다.
2. 그다음 타입의 리턴값을 추론하는데요 함수의 return의 값은 알 수 없기에 infer를 이용해서 추론하는 모습을 볼 수 있습니다. 
3. 추론한 infer값을 결과 값으로 반환합니다.

제가 햇갈렸던 부분은 제네릭과 infer의 혼용인데요 다음을 보면서 정리해 봤습니다.

```tsx
type YourReturnType<T,G> = T extends (...args: any) => G ? G : any;
type MyReturnType<T> = T extends (...args: any) => infer R ? R : any;

function fn(num : number) {
  return num.toString(); 
}

const a : MyReturnType<typeof fn> = "Hello";   // ReturnType<T> 이용
const b : YourReturnType<typeof fn,string> = "Hello";   // ReturnType<T> 이용

console.log(a); //Hello
```

둘다 똑같이 리턴값을 반환해주는 유틸 타입인데 infer를 사용하지 않는다면 다음과 같이 제네릭 두개로 사용을 해야합니다. 이럴 때는 타입 하나만 사용하고 return 값을 추론하는게 더 정확하겠죠?

책 내부에서 좀 어려운 타입을 한번 분석해 보겠습니다.

```tsx
type UnionToIntersection<U> = (U extends any ?  (p:U) => void: never) extends (p:infer I) =>void ? I :never;

type Result = UnionToIntersection<{a:number}|{b:string}>;
```

해당 타입은 인자로 받은 객체를 유니언을 인터섹션으로 만드는 타입입니다.

```tsx
(U extends any ? (p:U) => void : never)
// {a:number}가 분배 법칙으로 실행이 되고
// 이 코드는 (p:{a:number}) => void로 추론이 됨
```

```tsx
(p:{a:number})=>void extends (p:infer I) => void ? I : never
// 해당 함수 꼴이 맞으므로 I가 {a:number}로 추론이 됩니다. 
// 따라서 I가 리턴이 되고 분배법칙으로 나머지 b도 리턴이 되어 
// {a:number}&{b:string}이 최종 타입으로 나오게 됩니다.
```

## 2.23 타입 좁히기

타입좁히기는 자바스크립트 문법을 사용해야 합니다. 타입스크립트도 자바스크립트이기 때문에 자바스크립트에서 사용가능한 코드이기 때문입니다.

- typeof 를 이용한 타입 좁히기

```tsx
const strOrNum = (arg:number|string)=>{
    if(typeof arg === 'string'){
        arg;
// string
    }
    else if(typeof arg ==='number'){
        arg;
// number
    }
    else{
        arg;
// never
    }
}
```

- instanceof를 이용한 타입 좁히기 (interface는 instanceof로 사용할 수 없음)

```tsx
class A{}
class B{}

const strOrNum = (arg:A|B)=>{
    if(arg instanceof A){
        arg;
// A
    }
    else if(arg instanceof B){
        arg;
// B
    }
    else{
        arg;
// never
    }
}
```

- is 키워드를 이용한 타입 좁히기

```tsx
// 만약 해당 타입좁히기를 함수로 만들경우 return에 따라서 해당 타입이 추론이 되어야 합니다.
// is를 통해 해당 반환 타입을 좁힐 수 있습니다.

interface Cat {
    type: 'cat';
    purrs: boolean;
}

interface Dog {
    type: 'dog';
    barks: boolean;
}

const func = (arg: Cat|Dog)=>{
    if(isCat(arg)){
        console.log(arg);
    }    
}

// is 키워드를 통해서 true가 반환된다면 Cat을 반환합니다.
function isCat(animal: Cat | Dog): animal is Cat{
    return animal.type === 'cat';
}
```

```tsx
// 왜 안될까???
class A{}
class B{}

const isA = (arg:A|B): arg is A=>{
    if(arg instanceof A){
        return true;    
    }
    else{
        return false;
    }
}

const func = (arg:A|B)=>{
    if(isA(arg)){
        console.log(arg);
    }    
}
```

## 2.24 2.25 탬플릿 리터럴과 재귀를 사용해 보자

재귀 그냥 짜보기

```tsx
type Reverse<T> = T extends [...infer R, infer L] ? [L,...Reverse<R>]:[];
```

템플릿 리터럴을 이용하면 타입을 합성해서 만들 수 있습니다. 문자열의 템플릿 리터럴과 비슷합니다.

```tsx
type City = 'seoul'| 'suwon' | 'busan';
type Vehicle = 'car'| 'bike' | 'walk';

type ID = `${City}:${Vehicle}`
```

백틱 안에 타입을 입력해서 사용할 수 있습니다.

재귀 + 템플릿 리터럴

```tsx
type RemoveSpace<T extends string> = T extends ` ${infer W}`? RemoveSpace<W>: T extends `${infer R} ` ? RemoveSpace<R>: T;
```

## 2.26 satisfies

> 타입스크립트 4.9문법
> 

https://mycodings.fly.dev/blog/2023-07-14-understanding-typescript-satisfies-operator

타입스크립트는 해당 코드를 실행할때 타입을 추론합니다. 

```tsx
type MyState = StateName | StateCordinates

type StateName = 'Seoul' | 'Busan' | 'Daegu'

type StateCordinates = { x: number, y: number }

type User = {
  birthState: MyState,
  currentState: MyState,
}

const user: User = {
  birthState: 'Seoul',
  currentState: { x: 3, y: 4 },
}

user.birthState.toUpperCase()
//error
```

즉 여기서는 user가 실행할때 user의 타입인 `User` 로 인식하고 해당 값인 birthState의 타입인 MyState로 인식하게 됩니다 MyState는 string과 obejct이기 때문에 .toUpperCase가 실행되지 않습니다.

이를 해결해 주기 위해서 기존에는 타입 좁히기를 사용했는데요 4.9문법의 satisfies를 이용해서 이를 해결 할 수 있습니다. 

타입을 채크할 때 satisfies를 이용해서 검사해서 추론을 해 줍니다. 따라서 조금더 타입을 좁혀서 사용할 수 있는것이죠. as const 방법보다는 좀 느슨하게 좁혀주는것 같습니다.

```tsx
type MyState = StateName | StateCordinates

type StateName = 'Seoul' | 'Busan' | 'Daegu'

type StateCordinates = { x: number, y: number }

type User = {
  birthState: MyState,
  currentState: MyState,
}

const user = {
  birthState: 'Seoul',
  currentState: { x: 3, y: 4 },
} satisfies User;

user.birthState.toUpperCase()
```

## 2.28 원시 자료형의 브랜딩 기법

> with gpt (굉장히 유용하게 써볼수 있을거 같은 기법)
> 

```tsx
type UserID = string & { readonly brand: unique symbol };
type OrderID = string & { readonly brand: unique symbol };

function createUserID(id: string): UserID {
    return id as UserID;
}

function createOrderID(id: string): OrderID {
    return id as OrderID;
}

function queryUser(id: UserID) {
    // 데이터베이스 또는 API 쿼리 코드
}

function queryOrder(id: OrderID) {
    // 데이터베이스 또는 API 쿼리 코드
}

const myUserID = createUserID("user1234");
	// 타입이 UserId타입으로 지정이 된다.
const myOrderID = createOrderID("order5678");
// 타입이 OrderId타입으로 지정이 된다.
// 이러면 해당 변수만 보고서 원시 타입이 아닌 어떤 값을 받는것인지 더 명확해 진다.

queryUser(myUserID);
queryOrder(myOrderID);

// 다음 코드는 컴파일 에러를 발생시킵니다. 
// 각 함수는 브랜딩된 ID만 받아들입니다.
// queryUser(myOrderID);
// queryOrder(myUserID);
```

 userId와 OrderId는 모두 문자열로 원래는 타입이 string으로 같아야 합니다. 하지만 해당 타입을 내가 원하는 타입으로 줄 수 있습니다. 이를 통해 해당 값이 특별한 값이라고 지정해 줄 수 있는것이죠
