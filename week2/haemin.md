> #### info::출처
>
> 이 내용은 [타입스크립트 교과서](https://product.kyobobook.co.kr/detail/S000208416779) 책을 읽고 스터디한 내용을 바탕으로 정리했습니다.


## 2.10 객체의 속성과 메서드에 적용되는 특징을 알자

기본적으로 타입스크립트는 객체를 리터럴로 작성했냐 변수를 대입했냐에 따라서 검사 방법이 달라집니다.

객체 리터럴을 대입하면 잉여속성 검사가 실행이 되고 변수를 대입하면 실행하지 않습니다 이 잉여속성 검사는 타입에서 선언하지 않는 속성을 사용했는지 확인하는 역할을 합니다.

타입스크립트에서는 인덱스 시그니처라는게 있습니다

```typescript
type test ={
  [key:string]:string
}
```

객체의 특정 타입을 미리 매핑해서 특정 타입으로 만들수 있는거죠 지금은 key가 string이고 value값이 string인 객체 타입을 미리 래핑 해 두었습니다.

이때 이 key로 설정할 수 있는 타입은 string,number,symbol밖에 없는데요 실제 만들어서 사용하려면 더 많은 타입들이 들어가기 위해서 해당 타입의 확장이 필요합니다.

따라서 in 키워드를 통해 이를 매핑 시킬 수 있습니다.

```typescript
interface original {
  name:string;
  age:number;
}

type copy = {
  [key in keyof original] : original[key];
}
```

기본적으로 key의 값이 3가지가 필요하는 모두 `|` 합집합 기호를 통해 key값에 넣어 주어야 합니다 따라서 original의 key값을 뽑게 되면 `name | age` 가 나오게 되고 key값에 들어가면서 copy는 original과 똑같은 타입이 됩니다.

## 2.11 타입은 집합이다

타입을 사용할때는 집합이랑 비슷하게 생각을 하면됩니다. 집합과 똑같이 합집합 `|` 교집합 `&` 전체 집합 `unknown` 공집합 `never` 로 생각할 수 있습니다. 이런 특성때문에 타입은 항**상 좁은 타입에서 넓은 타입으로 대입할 수 있다**는 특징을 가지고 있습니다.

다만 여기에서 조심해야 할 부분이 있는데요 바로 빈객체는 null 과 undefined를 제외한 모든 값을 의미하는 타입입니따 따라서 I는 `never` 가 나오게 되는거죠

```jsx
type H = {a:'b'} & number;
type G = null & {b:'c'};
type I = {} & null;
```

## 2.13 대입 방법을 배우자

그러면 좁은 타입에서 넓은 타입으로 대입할 수 있다고 했는데 조금 더 자세하게 보겠습니다.

**객체**

```jsx
interface A{
  name: string
}

interface B{
  name: string;
  age:number;
}

const aObj = {
  name:'Kim'
}

const bObj = {
  name:"Park",
  age:20
}

const a:A = aObj;
const b:B = bObj;

const abReference:A = bObj;
const baReference:B = aObj;
//error

const abLiteral:A = {
  name:"Park",
  age:20,
}
// 리터럴을 입력했기 때문에 잉여 속성 검사가 펼쳐진다.
```

A타입으로 선언된 객체에 B타입의 객체를 넣을 수 있지만 B타입의 객체에는 A타입의 객체를 넣을 수 없습니다. A타입은 B타입 보다 더 큰 타입입니다 큰 타입이라는 것은 타입이 강제 되어 있는 값이 작다 라는 것입니다. 따라서 타이핑이 덜 되어 있는 A타입이 더 큰 타입이라고 할 수 있는것이죠

**합집합 교집합**

```jsx
interface A{
  name: string
}

interface B{
  age:number;
}

function testOR(): A|B{
  if(Math.random()>0.5){
    return {
      age:28
    }
  }
  return {
    name:'Kim'
  }
}

function testAND(): A &B{
  return {
    name:'Kim',
    age:20,
  }
}

const test1:A|B = testAND();
const test2:A&B = testOR();
// error
```

A | B는 각각의 개별 타입인 A, B A&B보다 넓은 타입 입니다. A | B가 나마지에 비해서 더 강제되어 있는게 작죠 따라서 느슨한 넓은 타입입니다. 즉 A | B에 A, B A&B는 추가할 수 있지만 반대인 경우에는 불가능 합니다.

**튜플**

튜플을 보겠습니다 num은 튜플이고 numArr는 배열입니다.

```typescript
let num:[1,2]=[1,2];
let numArr:number[] = [1,2,3,4,5,6,7,8];

num = numArr;
//error
numArr = num;

let a:['hi',1]= ['hi',1];
let numStrArr:(number|string)[] =  [1,"hi",2,3];

numStrArr = a;
a= numStrArr
//error
```

똑같이 생각하면 됩니다. num은 타입이 더 강제되어 있으므로 좁은타입이고 numArr 같은 경우에는 타입이 상대적으로 더 느슨합니다. 따라서 num에는 numArr를 대입할 수 없지만 numArr에는 num을 대입할 수 있습니다.

**readonly**

하지만 조금 외워야 하는 부분이 있습니다 바로 readonly가 들어가면 인데요.

```typescript
let num:readonly [1,2]=[1,2];
let numArr: number[] = [1,2,3,4,5,6,7,8];

num = numArr;
//error
numArr = num;
//error
```

이번에는 둘다 error가 발생 합니다. readonly가 붙인 수식어는 기본적으로 더 넓은 타입이 됩니다. 구정 할 수 없으니 강제가 줄은 것이죠. 따라서 num에 numArr를 넣으면 튜플이 강제된게 더 많기 때문에 위에서 봤던것 같이 에러가 발생하고 반대인 경우에는 위에서는 되었으나 readonly 때문이 더 넓은 타입이 되어서 대입할 수 없습니다.

객체에서는 상황이 달라지는데요

객체에서는 readonly가 붙어도 서로 대입할 수 있습니다.

```typescript
interface ReadOnly{
  readonly a: string;
  readonly b: string;
}

interface Original{
  a:string;
  b:string;
}

const org:Original ={
  a:'a',
  b:'b',
}

const ronly:ReadOnly = {
  a:'a',
  b:'b',
}

const org2:ReadOnly = org
const ronly2:Original = ronly
```

느낌에 JS에서 처럼 타입이 강제되는지 아닌지는 참조자에만 관여하기 때문에 일어나는 일인것 같습니다. (확실하지 않음)

**옵셔널**

객체에서 옵셔널도 잘 생각해보면 타입 | undefined이랑 같습니다. 따라서 위에서 합집합 교집합에서 했던것 처럼 적용하면 유니온이 더 넓은 타입 된다는 결론을 내릴 수 있습니다.

**결론**

즉 타입이 많을 수록 제한되어 있는게 많아 더 좁은 타입이고 타입이 적을수록 제한되어 있는게 적어 더 넓은 타입입니다. 그리고 타입은 항상 좁은 타입에서 넓은 타입으로 대입할 수 있습니다.

## 2.13 구조적 타입

타입은 모든 속성이 동일하면 타입의 이름이 다르더라도 동일한 타입으로 취급합니다. 

```typescript
type Arr = number[];
type CopyArr = {
  [key in keyof Arr]:Arr[key]
}

const copyArr:CopyArr = [1,3,9];
```

CopyArr는 객체지만 number[]와 같습니다. 즉 구조적 타이핑의 특성을 가지고 있어 둘은 같은 것이죠

```typescript
type SimpleArr ={[key:number]:number, length:number};
const simpleArr:SimpleArr = [1,2,3];
```

다음과 같은 재미있는 코드도 가능합니다. arr의 요소가 전부 타이핑이 되어있으니 arr와 같은거죠

그러면 구조적 타입의 특성을 가지고 있지 않게 하려면 어떻게 할까요? 간단합니다. 서로 구분이 되는 속성을 하나 추가해 주기만 하면됩니다. 

## ⭐ 2.14 제네릭

타입스크립트도 타입간의 중복을 방지하기 위해서 제네릭 이라는 것을 사용합니다. 

```typescript
function values<T>(inital:T[]){
  return {
    hasValue(value:T) {return inital.includes(value)}
  }
}

const savedValues = values(['a','b','c']);
savedValues.hasValue('x')
```

inital의 타입을 T로 잡아서 해당 타입을 기반으로 계산하게 해줍니다.

다만 여기서는 조금 의도치 않게 타입이 추론이 되는데요. a,b,c 문자열 안에 있는지 확인하려는게 목적이었지만 여기서는 T는 string으로 추론이 됩니다.

이를 해결하기 위해서는 들어오는 인자 값이 상수라고 명시를 해줘야 하는데요 방법은 2가지가 있습니다.

1. typescript 5.0이전

```typescript
function values<T>(inital: readonly T[]){
  return {
    hasValue(value:T) {return inital.includes(value)}
  }
}

const savedValues = values(['a','b','c'] as const);
savedValues.hasValue('x')
```

as const를 사용해서 상수로 명시해주고 readonly가 추가된 만큼 inital도 readonly를 추가해 줍니다.

1. typescript 5.0이후

```typescript
function values<const T>(inital: T[]){
  return {
    hasValue(value:T) {return inital.includes(value)}
  }
}

const savedValues = values(['a','b','c']);
savedValues.hasValue('x')
```

const라는 매개변수가 도입되어 다음과 같이 작성하면 T를 추론할때 as const 붙인 값으로 추론합니다.

**제약**

제네릭을 사용해서 여러타입을 정의할 수 있으니 당연히 제약을 걸 수도 있습니다. extends라는 키워드를 사용해서 제약을 걸 수 있습니다.

```typescript
function values<T extends string>(inital: T[]){
  return {
    hasValue(value:T) {return inital.includes(value)}
  }
}

const savedValues = values(['a','b','c']);
const savedNumberValues = values([1,2,3]);
// error
```

이러면 T제네릭은 string에 대입할수 있는 타입만 정의할 수 있도록 제한이 됩니다 따라서 1,2,3과 같은 숫자는 넣을 수 없죠

```typescript
<T extends object> // 모든 객체
<T extends any[]> // 모든 배열
<T extends (...args:any)=>any> // 모든 함수
<T extends abstract new (...args:any)=>any> // 생성자 타입
<T extends keyof any> // 모든 타입 string | number | symbol
```

이를 응용해서 다음과 같이 제네릭에 들어올 인자를 제한할 수 있습니다.

다만 여기서 중요한것이 extends를 이용해서 제한을 하면 ⭐**해당 타입에 대입할 수 있는 타입이 제한**⭐이 되는것입니다.

다음을 보면 당연하게 되어야 한다고 생각하죠?

```typescript
const returnV1 = <T extends boolean>():T =>{
  return false;
}

// Type 'boolean' is not assignable to type 'T'.
```

하지만 실상은 에러입니다. 왜그럴까요?

T는 boolean의 타입으로 제한하기도 하지만 제한은 해당 타입에 대입할 수 있는 타입이 제한이 된다고 했습니다 즉 boolean 타입에 대입할 수 있는 never가 존재합니다. 따라서 T는 never가 될 수 있으므로 return false가 불가능한것입니다.

이번에는 객체로 보겠습니다.

```typescript
interface V0{
  value :any;
}

const returnV0 = <T extends V0>():T =>{
  return {value:'test'};
}
// error
```

똑같이 V0로 제약을 걸어 두었고 T를 반환하고 있습니다. 여기서도 V0에 대입할 수 있는 타입들로 제한이 걸리는데 `{ value : ‘test’ , …args}` 해당 타입도 가능하기 때문에 불가능 한 것이죠

비슷하게 하나 더 보겠습니다.

```typescript
function onlyBoolean<T extends boolean>(arg:T = false){
  return arg;
}
```

똑같이 boolean 이외에 never가 들어올 수 있기 때문에 T를 false로 지정하지 못합니다. never의 arg가 세상에 어디있겠습니까

저자에는 이에 대한 해결방법으로 제네릭을 쓰지말자를 내놓았습니다.

**조건문**

그러면 타입에 제한을 알아봤으니 조건문을 보겠습니다. 똑같이 extends라는 키워드를 사용합니다.

제가 생각하기에는 제네릭을 정의하는 곳에서는 정의 이기때문에 조건문과 제약을 걸 수 있지만 실제로 타입을 정의하는 곳에서는 제네릭을 제약 거는게 이상하므로 extends는 조건문 키워드가 된다라고 구분지어봤습니다 (확실하지 않음)

```typescript
type Result<T> = T extends String ? true : false
```

T가 String이 맞다면 true를 반환하고 아니라면 false를 반환합니다.보통은 타입 검사를 할때 자주 사용합니다.

여기서 새로운 특징은 never인데요 객체나 배열에서 만약 never로 지정된다면 타입스크립트에서는 해당 속성을 지워줍니다.

```typescript
type OmitByType <T,K>={
  [O in keyof T as T[O] extends K ? never: O ]: T[O];
}

// 같은 기능을 하는 다른 방법도 만들어 보자
```

다음과 같이 never를 사용해서 key값을 지울 수 있습니다.

또 신기한 특성중 하나가 분배법칙이 적용된다는 것입니다.

```typescript
type IsString<T> = T extends string ? true : false;

type Result = IsString<'hi'| 3>;
```

직관적으로 생각 했을때는 IsString안에 string과 number가 들어가있고 이것은 string보다 넓은 타입이므로 string에 대입할 수 없습니다. 따라서 false가 나올것이라 예상을 하지만 실제로 보면 boolean타입으로 잡아주고 있습니다.

이는 분배법칙 때문입니다. 실제로 실행을 하면 `IsString<’hi>` `IsString<3>` 이렇게 실행이 되기 때문에 true와 false가 둘다 나와 boolean 타입이 되는것이죠

그러면 의도와 같이 두 타입을 같이 넘기려면 어떻게 해야 할까요? 분배법칙이 실행되지 않게 해야 합니다.

```typescript
type IsString<T> = [T] extends string ? true : false;

type Result = IsString<'hi'| 3>;
```

제네릭에 []를 붙여 줌으로써 hi와 3을 같이 넘겨줄 수 있습니다.

여기서도 마찬가지로 never가 말썽을 피우는데요. never또한 분배법칙이 일어나서 나오는 문제들이 있습니다. 기본적으로 never는 공집합을 의미하기 때문에 해당 never를 분배법칙하면 결국 아무것도 실행되지 않습니다. 공집합을 어떻게 나눌까요?? 나눠사 나오는게 없으니까요

그래서 제네릭에 never가 들어가면 타입의 값은 never가 되는것이죠

```typescript
type IsString<T> = [T] extends [never] ? true : false;

type Result = IsString<never>;
// true
```

> ???: 내가 이해한 분배 법칙이랑 다른데?? 왜 never에도 []를 명시해 주어야 할까?
> 

**제네릭 판단**

제네릭은 기본적으로 판단을 뒤로 미루는 특징이 있습니다. 따라서 다음과 같은 코드는 타입을 읽을 수 없습니다.

```typescript
function test<T>(a:T){
  type R<T> = T extends string ? T : T;
  const b:R<T> = a;
}
```

우리가 봤을때는 R<T>의 반환값이 T라는것을 알지만 타입스트립트는 제네릭의 판단을 뒤로 미루기 때문에 새로운 타입인 R<T>의 타입이 T라는 것을 알지 못합니다. 이런 경우는 보통 infer를 쓴다고 합니다?
