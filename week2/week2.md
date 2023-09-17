# 2주차 스터디 내용 정리

> #### info::출처
>
> 이 내용은 [타입스크립트 교과서](https://product.kyobobook.co.kr/detail/S000208416779) 책을 읽고 스터디한 내용을 바탕으로 정리했습니다.

<!-- TOC -->
<!-- TOC -->
- [2주차 스터디 내용 정리](#2주차-스터디-내용-정리)
  - [2.10 객체의 속성과 메서드에 적용되는 특징을 알자](#210-객체의-속성과-메서드에-적용되는-특징을-알자)
    - [구조 분해 할당 타이핑시 주의점 - 수민](#구조-분해-할당-타이핑시-주의점---수민)
    - [2.10.1 인덱스 접근 타입](#2101-인덱스-접근-타입)
    - [아래처럼 표기하는게 더 보기 좋지 않을까요?? - 수민](#아래처럼-표기하는게-더-보기-좋지-않을까요---수민)
    - [2.10.2 매핑된 객체 타입](#2102-매핑된-객체-타입)
  - [2.11 타입을 집합으로 생각하자(유니언, 인터섹션)](#211-타입을-집합으로-생각하자유니언-인터섹션)
    - [type H - 수민](#type-h---수민)
  - [2.12 타입도 상속이 가능하다](#212-타입도-상속이-가능하다)
    - [상속 정리 - 수민](#상속-정리---수민)
  - [2.13 객체 간에 대입할 수 있는지 확인하는 법을 배우자](#213-객체-간에-대입할-수-있는지-확인하는-법을-배우자)
    - [수민 🌻](#수민-)
    - [2.13.1 구조적 타이핑](#2131-구조적-타이핑)
  - [2.14 제네릭으로 타입을 함수처럼 사용하자](#214-제네릭으로-타입을-함수처럼-사용하자)
    - [2.14.1 제네릭에 제약 걸기](#2141-제네릭에-제약-걸기)
  - [2.15 조건문과 비슷한 컨디셔널 타입이 있다.](#215-조건문과-비슷한-컨디셔널-타입이-있다)
    - [2.15.1 컨디셔널 타입 분배법칙](#2151-컨디셔널-타입-분배법칙)

## 2.10 객체의 속성과 메서드에 적용되는 특징을 알자

---

객체의 속성에도 옵셔널이나 readonly 수식어가 사용 가능하다.

```typescript
interface Example {
	hello: string;
	world?: number;
	readonly wow: boolean;
	readonly multiple?: symbol;
}
const example: Example = {
	hello: 'hi',
	wow: false,
};
example.no; // 이런건 존재 x
example.wow = true; // wow는 readonly이기 때문에 새로 할당 불가능
```

```typescript
const example: Example = {
	hello: 'hi',
	world: undefined, // world는 옵셔널이 붙어있기 때문에 가능함
	wow: false,
};
```

```typescript
interface Example {
	hello: string;
}
const example: Example {
	hello: 'hi',
	why: '나만 에러', // 에러 (interface Example에 why 속성 없음)
}
const obj = {
	hello: 'hi',
	why: '나만 에러',
}
const example2: Example = obj; // 에러 안남
```

- example과 example2의 차이
    - example: Example {}
    - example2: Example = obj;
        - 객체 리터럴을 대입했느냐 obj 변수를 대입했느냐의 차이 발생
        

```typescript
interface Money {
	amount: number;
	unit: string;
}

const money = { amount: 1000, unit: 'won', error: '에러 아님' };

function addMoney(money1: Money, money2: Money): Money {
	return (
		amount: money1.amount + money2.amount,
		unit: 'won',
	}
}
addMoney(money, { amount: 3000, unit: 'money', error: '에러' });
```

함수에서도 동일한 현상이 발생한다.

인수 자리에 변수로 값을 대입하면 에러가 발생하지 않지만, 객체 리터럴을 대입하면 에러가 발생한다.

이러한 현상이 발생하는 이유는 객체 리터럴을 대입할 때와 변수를 대입할 때 타입스크립트가 다르게 처리하기 때문이다.

객체 리터럴을 대입하면 잉여 속성 검사가 실행되는데 이것은 타입 선언에서 선언하지 않은 속성을 사용할 때 에러를 표시하는 것을 의미한다.

변수를 대입하면 객체 간 대입 가능성을 비교한다. (2.13절에서 자세히 알아본다고 함)

→ 여기까지 읽었을 때는 잉여 속성 검사는 선언하지 않은 속성을 잡아내기 때문에 에러를 잡아내고
변수를 대입할 때는 대입 가능성을 비교하기 때문에 여부 유무가 있을 수 있어서 에러를 나타내지는 않는 것 같다.

이 뒤의 내용은 코드를 읽어도 잘 들어오지 않아서 생략

사용하기에도 벅찰 것 같아서 우선 넘긴다.

<br/ >

### 구조 분해 할당 타이핑시 주의점 - 수민

```typescript
const { prop: { nested: string } } = {
	prop: { nested: 'hi' },
}
console.log(nested); // error
console.log(string); // hi
console.log(prop); // error
```

구조분해 할당의 타입은 위와같이 하면 안됩니다. 위 코드는 nested의 타입을 string으로 표기한 것이 아니라 nested의 속성 값을 string이라는 변수 이름에 대입한 것입니다.

```typescript
const userId = req.body.id
const [id: userId] = req.body
```

> [!info]
> 
> 
> ```typescript
> const obj = { id: 'sumin', age: 22 };
> const { id, age } = obj;
> const { id: username, age: year } = obj;
> console.log(username)
> 
> ```
> 
> 3번째 줄에 보이는 것 처럼 구조분해 할당한 속성의 이름을 다른 변수명으로 치환할 수 있습니다.
> 때문에  `{ prop: { nested: string } }` 으로 타입을 표기하게 된다면 nested로 구조분해 할당된 속성의
> 변수명을 string에 대입하는 꼴이 됩니다.
> 

구조분해 할당된 객체의 타입을 올바르게 표기 하려면 아래와 같이 해야합니다.

```typescript
const { prop: { nested } }: { prop: { nested: string } } = {
	prop: { nested: 'hi' },
}

console.log(nested);

```
<br/ >

### 2.10.1 인덱스 접근 타입

특정 속성의 타입을 별도 타입으로 만들고 싶으면 어떻게 해야할까?

```typescript
type Animal = {
	name: number; // 여기를 변경하면
}

type N = number; // 여기도 변경해야 함
```

이 두 개를 연동하려면 다음과 같이 작성

```typescript
type Animal = {
	name: string;
}

type N1 = Animal['name'];
type N2 = Animal["name"];
type N3 = Animal.name; // error
```

따옴표는 상관이 없다.

다만 객체의 속성에 접근하듯 . 으로 접근할 수 없다. (객체.속성)

<br/ >

### 아래처럼 표기하는게 더 보기 좋지 않을까요?? - 수민

```typescript
type Animal = {
name: Name;
}
type Name = string;
```

이렇게 표기하는게 더 가독성도 높고 편하지 않을까요??

```typescript
const obj = {
	hello: 'world',
	name: 'zero',
	age: 28,
};
```

이러한 객체가 존재한다.

키(key)의 타입: ‘hello’ | ‘name’ | ‘age’

값의 타입은 string | number이다. (접미사에 as const를 붙였다면 값의 타입은 ‘world’ | ‘zero’ | 28이 된다.

```typescript
const obj = {
	hello: 'world',
	name: 'zero',
	age: 28,
};

type Keys = keyof typeof obj; // Keys에 마우스 오버헤드를 하면 type Keys = "hello" | "name" | "age"
type Values = typeof obj[Keys]; // Values에 하면 type Values = string | number
```

- Keys = 키의 타입: ‘keyof 객체_타입
- Values = 값의 타입: ‘객체_타입[키의_타입]’
    - 객체_타입: obj는 값이기 때문에 앞에 typeof를 붙여서 객체_타입을 만들어준다.
    - 키의_타입: type Keys로 Keys가 키의 타입

**keyof를 적용하면 ‘number | 배열_속성_이름_유니온 | 배열_인덱스_문자열_유니온’이 된다.**

P.76 현재 이해 못함 다시 읽어야함 (2023-09-10) (아래에 적음)

- 이 코드의 정체를….

```typescript
type Keys = keyof any;
type ArrayKeys = keyof [1, 2, 3];
let a: ArrayKeys = 'lastIndexOf';
a = 'length';
a = 'pop';
a = '2';
a = '3'; // error
a = 3;
```

1. 이 코드에서 keyof any가 붙으면 그 타입은 string | number | symbol이 고정 값이 되는것인가?
    
    ⇒ 유석 : keyof 연산자는 객체 타입의 key를 리턴해주는데, any면 들어갈 수 있는 모든 객체 타입의 키 타입이라 string | symbol + number 
    
2. 배열에다가 keyof를 적용하면 ‘number | 배열_속성_이름_유니언 | 배열_’인덱스_문자열_유니언’ 이라고 적혀있고
let a: ArrayKeys = 'lastIndexOf'; 지금 이 해당 코드가 ArrayKeys를 타입으로 가지는 변수 a는 ‘lastIndexOf’라고 되어있는데
forEach, length도 들어갈 수 있다는데 이게 무엇을 말하는건지 잘 모르겠다.
3. 설명상 keyof [1, 2, 3]은 배열에 keyof가 적용되어있기 때문에 number | lastIndexOf | ‘0’ | ‘1’ | ‘2”를 의미한다는 것인가?
4. 그렇기 때문에 인덱스 ‘0’, ‘1’, ‘2’는 되는데 a = ‘3’은 안되고 a = ‘2’는 되는걸까?
    
    ⇒ J088해민: 발표할 때 해보죠?
    
- `keyof` 배열 - 태림
    
    배열에 대해 `keyof` 를 사용하면 결과로?
    
    1. 모든 number
        - 배열의 key로는 Number 타입을 사용하기 때문
    2. 배열의 인덱스
        - 배열의 길이가 n이라면 ‘0’ | ‘1’ | … | ‘n-1’
    3. 배열이 가지고 있는 메소드 이름
        - 배열의 공통 속성 (length, forEach, pop, push 등)
        
        ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/f6644d52-1280-432a-a28f-cec1ce79e570/6d389b73-69ae-4e18-b749-50d8c5a49ef2/Untitled.jpeg)
        
    
    [타입스크립트 교과서: 2.10.1 인덱스 접근 타입 - 4](https://thebook.io/080369/0107/)
    

→ 인덱스 시그니처, 객체의 속성 값을 전부 특정 타입으로 만들 수 있다.

인덱스 시그니처:  이것은 객체의 속성 이름이 문자열이고, 속성 값은 어떤 타입이든 될 수 있다는 것을 의미

```typescript
interface MyObject {
  [key: string]: any;
}

const obj: MyObject = {
  name: "John",
  age: 30,
  city: "New York"
};

console.log(obj.name); // "John"
console.log(obj.age); // 30
console.log(obj.city); // "New York"
```

<br/ >

### 2.10.2 매핑된 객체 타입

속성 전부에 타입을 지정하는 대신 일부 속성에만 타입을 부여할 수도 있다.

인덱스 시그니처를 사용해서 hello와 hi라는 속성 이름을 가진 객체를 타이핑 해보자.

```typescript
type HelloAndHi = {
	[key: 'hello' | 'hi']: string;
}
```

An index signature parameter type cannot be a literal type or generic type. Consider using a mapped object type instead.(1337)
(parameter) key: "hello" | "hi"

이라는 에러가 발생한다.

인덱스 시그니처에서 해당 타입을 사용할 수 없다는 것을 뜻한다.

그리고 mapped object type을 대신 사용하라는 것을 고려하라고 친절히 에러 메세지로 알려준다.

위의 인덱스 시그니처는 정적으로 고정된 속성 이름을 가진 객체를 정의하는 데 사용된다.

이 객체의 속성 이름은 "hello" 또는 "hi"로 고정되어 있으며 다른 속성 이름을 동적으로 추가할 수 없다.

<aside>
🥀 인덱스 시그니처에서 사용할 수 있는 타입은 string, number, symbol, 템플릿 리터럴 타입과 이들의 유니온 뿐
</aside>

- 매핑된 객체 타입
이란 기존의 다른 타입으로부터 새로운 객체 속성을 만들어내는 타입을 의미
    - 인터페이스에서는 사용 불가
    - 타입 별칭에서만 사용 가능

```typescript
type HelloAndHi = {
	[key in 'hello' | 'hi']: string;
}
```

type HelloAndHi = {

hello: string;

hi: string;

}

`in` 연산자를 사용해서 인덱스 시그니처가 표현하지 못하는 타입을 표현한다.

그리고 **in 연산자 오른쪽에는 유니온 타입**이 와야 한다. (’hello’ | ‘hi’)

순서대로 hello: string, hi: string이 속성이 되어 최종적으로  { hello: string, hi: string } 객체가 된다.

- 첫 번째 예제는 정적으로 고정된 속성을 가진 객체를 나타내며, "hello" 또는 "hi" 외의 다른 속성은 허용하지 않는다.
- 두 번째 예제는 "hello" 또는 "hi" 값을 가질 수 있는 모든 속성을 허용하며, 다른 속성도 동적으로 추가할 수 있다.

매핑된 객체 타입을 주로 언제 사용할까?

기존 객체 타입을 복사하는 코드를 봐보자.

```typescript
interface Original {
    name: string;
    age: number;
    married: boolean;
}
type Copy = {
    [key in keyof Original]: Original[key];
}
```

- 질문 - 태림
    
    여기서 `Original[Key]` 안의 `Key` 는 왼쪽의 `Key in keyof Original`  에서 가져오는 건가? ⇒ ⭕️
    

아까 위에서 말했듯이 
**keyof를 적용하면 ‘number | 배열_속성_이름_유니온 | 배열_인덱스_문자열_유니온’이 된다.**

아까 위에서와 다르게 in 옆에 바로 유니온이 오지 않았다.

유니온을 keyof 연산자를 사용해서 Original의 속성 이름만 추린다. (’name’ | ‘age’ | ‘married’)

그리고 인덱스 접근 타입(Original[key])을 사용해서 원래 객체의 타입을 가져온다.

결과적으로 Copy에 마우스 오버헤드를 해보면

type Copy = {

name: string;

age: number;

married: boolean;

}

이렇게 잘 나온다.

‘name’: Original[’name’]에서 ‘name’: string 속성이 되고 위에서처럼 순서대로 흘러간다.

튜플, 배열도 객체이므로 매핑된 객체 타입 적용이 가능하다.

예제가 너무 복잡해서 정리는 패스,, (읽어도 이해가 아직 안된 상태)

```typescript
interface Original {
    name: string;
    age: number;
    married: boolean;
}
type Copy = {
    readonly [key in keyof Original]?: Original[key];
}
```

```typescript
interface Original {
    readonly name?: string;
    readonly age: number?;
    readonly married: boolean?;
}
type Copy = {
    -readonly [key in keyof Original]-?: Original[key];
}
```

readonly, ? 수식어를 붙일수도, 그리고 제거할수도 있다.

```typescript
interface Original {
    name: string;
    age: number;
    married: boolean;
}
type Copy = {
    [key in keyof Original as Capitalize<key>]: Original[key];
}

// Capitalize는 문 자열의 첫 자리를 대문자화 하는 타입이다.

//type Copy = {
//    Name: string;
//    Age: number;
//    Married: boolean;
//}
```

(2023-09-10)

<br/ >

## 2.11 타입을 집합으로 생각하자(유니언, 인터섹션)

![태림 추가](https://prod-files-secure.s3.us-west-2.amazonaws.com/f6644d52-1280-432a-a28f-cec1ce79e570/cf2aff9e-282b-4a94-9adf-7318bc04892b/img.png)

태림 추가

유니언은 | 타입으로 or을 나타내고 합집합을 의미한다.

&이라는 and 연산과 같이 동작하는 교집합 연산자도 있다. 이는 인터섹션 연산자라고도 부른다.

string | number에서 겹치는 부분은 교집합 영역이다.
그런데 string과 number를 둘다 성립하는 값은 존재하지않으며 이를 공집합이라고 부르고 never가 이를 표시한다.

**타입스크립트는 타입을 집합으로 볼 수 있다. 타입스크립트에서 공집합이 `never`라면 전체 집합은 `unknwon`이다.**

any 타입은 이런 집합 관계마저도 무시한다. 정말 TS의 타입을 아예 제껴버릴 수 있는 타입이다.

```typescript
type A = string | boolean;
type B = boolean | number;
type C = A & B; // type C = boolean
type D = {} & (string | null) // type D = string
type E = string & boolean; // type E = never
type F = unknown | {}; // type F = unknown
type G = never & {}; // type G = never
```

type A, B, C는 생략하고 D부터 봐보자.

{}는 string이 포함되지만 null은 포함되지 않기 때문에 교집합 상 string타입으로 결정된다.

E는 위에서 말했듯이 string과 boolean을 모두 성립하는 값은 존재하지않는다. 공집합으로 never타입으로 결정된다.

F는 unknown은 전체집합을 의미한다고 했다. 전체집합의 합집합은 항상 전체집합이다.

G는 전체집합과 공집합의 교집합은 항상 공집합이다. never타입과 & 연산이 나오면 무조건 never타입이다.

<br/ >

### type H - 수민

```typescript
type H = { a: 'b' } & number;
// type H = { a: 'b' } & number;

```

null/undefined를 제외한 원시 자료형과 비어 있지 않은 객체를 & 연산할 때는 교집합이 never가 되지 않습니다.
즉, 타입 H는 never가 되지 않습니다. 객체의 타입과 number는 겹치는 타입이 없어서 never가 되야하는데 그렇지 않습니다. 이는 타입스크립트의 예외 사항입니다. 자세한 것은 2.28절에서~

<br/ >

## 2.12 타입도 상속이 가능하다

자바스크립트와 달리 타입스크립트는 타입을 명시해주어야 한다.

자바스크립트의 경우 타입이 없기 때문에 그냥 객체를 상속 받아서 그대로 사용하면 된다.
하지만 타입스크립트의 경우 클래스를 상속하고 타입 속성을 또 지정해주어야한다.

```typescript
interface Animal {
	name: string;
}
interface Dog extends Animal {
	bark(): void;
}
interface Cat extends Animal {
	meow(): void;
}
```

extends를 사용하여 기존 타입을 상속할 수 있다.
이렇게 하면 Dog와 Cat에서 name이라는 string 타입 값을 사용할 수 있다.
중복적으로 저 인터페이스 안에 일일히 name: string을 선언해줄 필요가 없어진다.

```typescript
type Animal {
	name: string;
}
type Dog Animal & {
	bark(): void;
}
type Cat Animal & {
	meow(): void;
}
type Name = Cat['name'];
```

이렇게 타입별칭과 인터섹션(&) 연산자를 사용해서도 상속을 나타낼 수 있다.

```typescript
interface Merge{
    one: string;
    two: string;
}

interface Merge2 extends Merge {
    one: 'h' | 'w';
    two: 123;
}
```

타입을 상속받으면서 변경할수도 있으나 아예 다른 타입으로 변경할 시 에러가 발생한다.

현재 상속받은 부분에서 two를 number 타입으로 아예 바꿔버렸기 때문에 에러가 발생한다.

<br/ >

### 상속 정리 - 수민

- 타입 별칭이 인터페이스를 상속할 수도 있고, 그 반대도 가능합니다.
- 여러개 타입을 상속할 수도 있습니다.
- 상속할 때 부모의 타입을 변경할 수도 있습니다. 단, 더 좁은 타입으로만 변경 가능합니다.
    - ex) string -> 'hello'

<br/ >

## 2.13 객체 간에 대입할 수 있는지 확인하는 법을 배우자

```typescript
interface A {
    name: string;
}
interface B {
    name: string;
    age: number;
}

const aObj = {
    name: 'zero',
}
const bObj = {
    name: "zero",
    age: 32,
}

const aToA: A = aObj;
const bToA: A = bObj;
const aToB: B = aObj; // error
const bToB: B = bObj;

// J088
const abLiteral: A = {
  name: "Park",
  age: 20,
}

// 이건 또에러입니다 왜일까요?? hint(2.10장)
```

나도 아이러니하였다.

느낌상 인터페이스 A에는 age가 정의되어있지않기 때문에 bToA에서 오류가 날것이라고 생각했었다.
하지만 그 반대로 인터페이스 B를 받는 aObj에서 에러가 발생한다.

좀 생각을 해보니까 타입스크립트에서는 변수를 선언해놓고 그 변수 안쓰기만해도 에러가 난다.

그런 느낌으로 보니까 지정해놓은 타입을 사용하지 않아서 오류가 난다.

> **좁은 타입은 넓은 타입에 대입할 수 있지만, 넓은 타입은 좁은 타입에 대입할 수 없다.
각 타입 속성 값이 필수적으로 필요하다. 라고 정리하면 될 것 같다.
무엇이 더 구체적으로 적혀있는 인터페이스인지 파악하는 것이 중요하다.**
> 

![태림 추가](https://prod-files-secure.s3.us-west-2.amazonaws.com/f6644d52-1280-432a-a28f-cec1ce79e570/8ed986f8-e0cb-4082-8355-4cf169877f41/Group_1.png)

태림 추가

여기서 넓은 타입이란 interface A이고 좁은 타입은 interface B이다.
A타입이 좀더 추상적, B타입이 더 구체적이다.

예시를 한 개 더 보자.

readonly 수식어가 붙은 배열이 더 넓은 타입이다.

```typescript
let a: readonly string[] = ['hi', 'readonly'];
let b: string[] = ['hi', 'normal'];

a = b;
b = a; // error
```

string[]이 readonly string[]보다 더 좁은 타입이므로 b에 a를 대입할 수 없다.

또한 배열이 튜플보다 넓은 타입이다.

하지만 readonly 수식어가 붙는 다면 일반 배열보다 튜플이 넓은 타입으로 변한다.

- 좁은 타입, 넓은 타입 순서
    - **readonly > 배열 > 튜플**
    - **옵셔널 객체 > 옵셔널이 아닌 객체**

<br />

### 수민 🌻

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/f6644d52-1280-432a-a28f-cec1ce79e570/05201f76-a7fa-4dc1-a27f-0787257e62f3/Untitled.png)

옵셔널이란 기존 타입에 undefined가 유니언 된 것과 같으며 기존 타입 | undefined가 기존 타입보다 넓은 타입이므로 옵셔널인 객체가 더 넓은 타입이다.

하지만 배열과 다르게 객체에서는 속성에 readonly가 붙으면 또 서로 대입이 가능하다고 한다…

> **넓은 타입, 좁은 타입을 구분할 수 있다면 누가 누구에게 대입할 수 있는지 없는지를 알 수 있다.**

<br />

### 2.13.1 구조적 타이핑

객체가 어떻게 만들어졌든 구조가 같으면 같은 객체로 인식한다.

```typescript
interface A {
    name: string;
}
interface B {
    name: string;
    age: number;
}

const aObj = {
    name: 'zero',
}
const bObj = {
    name: "zero",
    age: 32,
}

const aToA: A = aObj;
const bToA: A = bObj;
const aToB: B = aObj; // error
const bToB: B = bObj;
```

위에서 보았던 코드이다.

interface A, B가 완전히 동일하지는 않지만 interface B는 A에 존재하는 name 속성을 가지고 있다.
B는 A이기 위한 모든 조건을 충족하고 있다.

따라서 B 인터페이스는 구조적 타이핑 관점에서 A 인터페이스라고 볼 수 있다.
반대로 A ≠ B 이다.

서로 대입하지 못하게 하려면 어떻게 해야할까?

서로를 구분하기 위한 속성을 추가해야한다.

```typescript
interface Money {
    __type: 'money';
    amout: number;
    unit: string;
}
interface Liter {
    __type: 'liter';
    amout: number;
    unit: string;
}
```

```typescript
// 인덱스 시그니처를 사용한 타입 복사
interface a {
    amout: number;
    unit: string;
}

type Money = {
    [key in keyof a]: a[key];
}

type Liter = {
    [key in keyof a]: a[key];
}
// 인덱스 시그니처에는 다른 속상값이 들어올 수 없음

//상속을 이용한 방법
interface a {
    amout: number;
    unit: string;
}

interface Money extends a {
	__type: 'money';
}

interface Money extends a {
	__type: '';
}
```

인터페이스도 관습이 있을까.

SRP(비유) 딱 그 용도에 맞는 인터페이스는 하나 딱 생성

```typescript
interface MoneyLiter {
    amout: number;
    unit: string;
}
```

- 스키마로 생각을 해보자.
- 지금 같더라도 결국 테이블을 나누는 것처럼 일단 우선적으로 나눠서 interface를 구성하는 것이 좋아보인다.
(유지보수 가능성 를 위해)

- [예시](https://cheri.tistory.com/268) (환율)

<br />

## 2.14 제네릭으로 타입을 함수처럼 사용하자

제네릭에 들어가기에 앞서 굉장히 중요하다.
지금까지 타입스크립트를 사용하면서 타입덩어리를 선언할 때 <>을 통하여 많이 사용하였다.
정작 제네릭에 대한 개념이 어렵다고해서 일단 사용하기는 했는데 한번 파보겠다.

```typescript
interface Zero {
    type: 'human',
    race: 'yellow',
    name: 'zero',
    age: 28,
}
interface Nero {
    type: 'human',
    race: 'yellow',
    name: 'nero',
    age: 32,
}
```

다음과 같이 name, age만 다른 Zero, Nero 인터페이스가 존재한다.

이때 중복을 제거하기 위해서 제네릭을 사용할 수 있다.

```typescript
interface Zero {
    type: 'human',
    race: 'yellow',
    name: 'zero',
    age: 28,
}
interface Nero {
    type: 'human',
    race: 'yellow',
    name: 'nero',
    age: 32,
}

interface Person<N, A> {
    type: 'human',
    race: 'yellow',
    name: N,
    age: A,
}

interface Zero extends Person<'zero', 28> {}
interface Nero extends Person<'nero', 32> {}
```

제네릭 표기는 <>로 하며 인터페이스 이름 바로 뒤에 위치한다.

<> 안에 타입 매개변수를 넣으면 된다.

여기서 제네릭을 쓸 때 <> 안에 타입 파라미터 개수가 일치하지않으면 에러가 발생한다.
정확히 값을 집어넣어야한다.

함수에서는 함수 선언문(Function)이냐 함수 표현식(Arrow Function)이냐에 따라 표기 위치가 다르다.

```typescript
const personFactoryE = <N, A>(name: N, age: A) => ({
    type: 'human',
    race: 'yellow',
    name,
    age,
})

function personFactoryD<N, A>(name: N, age: A) {
    return ({
        type: 'human',
        race: 'yellow',
        name,
        age,
    })
}
```

함수 표현식을 제외하고 interface, type, class, function 모두 이름<타입 파라미터>로 사용할 수 있다.
함수 표현식의 경우 const 함수 이름 = <타입 파라미터> 로 사용한다.

```typescript
interface Person<N = string, A = number> {
    type: 'human',
    race: 'yellow',
    name: N,
    age: A,
}
type Person1 = Person;
type Person2 = Person<number>;
type Person3 = Person<number, boolean>;
```

이런식으로 사용할 때 지정 타입을 변경할수도있고 지정을 안한다면 기본 default 타입이 지정된다.
이 코드의 경우 Person1은 NPerson<string, number>를 가지게 된다.

그리고 제네릭은 직접 타입을 넣지 않아도 추론을 통해 타입을 알아낼 수 있다.
기본값으로 unknwon이 들어있지만 좀 더 구체적인 좁은 타입이 있다면 그 타입으로 추론한다.

<br />

### 2.14.1 제네릭에 제약 걸기

extends 예약어를 통해서 상속뿐만 아니라 제약을 걸어줄 수도 있다.

```typescript
interface Example<A extends number, B = string> {
    a: A,
    b: B,
}

type Usecase1 = Example<string, boolean>; // error
type Usecase2 = Example<1, boolean>;
type Usecase3 = Example<number>;
```

Usecase1을 보자.
위의 인터페이스 Example에서 A의 값은 number 타입으로 제약을 걸어주었다.
그렇기 때문에 Usecase1에서 Example의 첫 매개변수의 타입을 string으로 바꿀 수 없다. 고로 에러가 발생한다.

Usecase2의 경우 제약조건 number를 잘 따랐고 B의 타입은 boolean으로 변경하였다.

Usecase3를 보면 전부 number 타입으로 사용하겠다고하며 제약 조건에 성립하여서 가능하다.

> **기본값으로 지정한 타입과 다른 값은 제공할 수 있지만 제약 조건에 어긋난 타입은 제공할 수 없다.**
> 

```typescript
interface Example<A, B extends A> {
    a: A,
    b: B,
}

type Usecase1 = Example<string, number>; // error
type Usecase2 = Example<string, 'hello'>;
type Usecase3 = Example<number, 123>;
```

솔직히 이런 경우가 존재할지는 모르겠지만 A, B extends A란 생각나는 그대로 

두 타입이 모두 A와 같으면 된다.
Usecase1에서는 두번 째 파라미터 number가 B에 해당하지만 A와 타입이 다르기 때문에 에러가 발생한다.

이것을 쓸 일이 있을까..?

<aside>
⚔️ 자주 쓰이는 제약조건
<T extends object> // 모든 객체
<T extends any[]> // 모든 배열
<T extends (…args: any) ⇒ any> // 모든 함수
<T extends abstract new (…args: any) ⇒ any> // 생성자 타입
<T extends keyof any> // string | number | symbol

</aside>

- 수민
    
    
    | 의미 | 코드 |
    | --- | --- |
    | 모든 객체 | <T extends object> |
    | 모든 배열 | <T extends any[]> |
    | 모든 함수 | <T extends (...args: any)> |
    | 생성자 타입 | <T extends abstract new (...args: any) => any> |
    | 모든 타입 | <T extends keyof any> |

P.105쪽 부분은 이해가 되지 않아서 같이 확인하면 좋을 것 같다. 

- J088 105페이지 개인적으로 정리한 노트 첨부합니다.
    
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
    
    // 유석
    
    function onlyBoolean<T extends boolean>(arg:T = false): T {
    	return arg;
    } // error
    
    // 나쁜 코드
    function onlyBoolean<T extends boolean>(arg:T = false as any): T {
    	return arg;
    } // ok
    
    function onlyBoolean<T extends boolean>(arg:T = false as never): T{
    	return arg;
    } // ok
    
    // T extends boolean에서
    // T는 true, false, never가 될 수 있다.
    // 만약에 never가 들어오면  
    
    ```
    
    똑같이 boolean 이외에 never가 들어올 수 있기 때문에 T를 false로 지정하지 못합니다. never의 arg가 세상에 어디있겠습니까
    
    저자에는 이에 대한 해결방법으로 제네릭을 쓰지말자를 내놓았습니다.
    

- J022 저도 첨부할게요~
    
    ```typescript
    interface V0 {
    	value: any;
    }
    
    const returnV0 = <T extends V0>(): T => {
    	return { value: 'test' };
    }
    ```
    
    위 예시처럼 타입을 선언할 수 없습니다. 왜냐하면 현재 제네릭 타입 매개변수는 V0라는 제약이 걸려있기 때문에 V0 뿐만 아니라 V0보다 더 좁은 타입도 대입이 가능합니다. 예를들어 { value: any, key: string } 타입도 V0보다 좁은 의미의 타입이기 때문에 대입 가능합니다. 하지만 returnV0 함수의 반환값은 V0만을 만족하기 때문에 에러가 발생합니다.
    
    ```typescript
    function onlyBoolean<T extends boolean>(arg: T = false): T{
    	return arg;
    }
    
    ```
    
    위와 같은 경우에 arg의 기본값으로 false를 넣으면 에러가 발생합니다. 왜냐하면 타입 매개변수 T에 boolean 제약이 걸려있기 때문입니다. boolean 타입 제약이면never 타입도 대입이 가능합니다. 따라서 T가 boolean이라고 확정된 것이 아니기 때문에 기본값을 설정할 수 없습니다.
    
<br />


## 2.15 조건문과 비슷한 컨디셔널 타입이 있다.

여기에서도 extends가 사용된다.
여태 상속에서만 쓰다가 아주 여러 곳에서 사용되는 예약어이다.

삼항연산자를 알 것이다.
타입을 변경할때 사용할 수 있는데

> **특정 타입 extends 다른 타입 ? 참일 때 타입 : 거짓일 때 타입**
> 

타입 검사로도 사용할 수 있다.
해당 타입이 맞는지 아닌지를 삼항 연산자를 사용해서 확인이 가능하다.

<br />

### 2.15.1 컨디셔널 타입 분배법칙

string | number 타입으로 string[] 을 얻고 싶은 상황

```typescript
type Start = string | number;
type Result = Start extends string? Start[] : never;
```

이 코드의 경우 string | number가 string을 extends 할 수 없기 때문에 never 타입을 반환한다.

```typescript
type Start = string | number;
type Result<Key> = Key extends string ? Key[] : never;
let n: Result<Start> = ['hi'];
```

타입을 제네릭으로 바꾸었다.
검사하려는 타입이 제네릭이면서 유니언이면 분배법칙이 실행된다.

Result<string | number>는 Result<string> | Result<number>

**type Result<Key> = Key extends string  ? Key[] : never;** 를 거치면 string[] | never가 되고, 
never는 사라져서 최종적으로 string[]이 된다.

(P.110쪽 오타 발견) 
교재에서는 **type Result<Key> = Key extends string | boolean ? Key[] : never;으로 나와있음**

여기의 경우 Start는 string이나 number 타입을 지닌다.

Resulte 제네릭에 Start를 넣으면

Key extends string? 에서 string | number인데 string | number은  string을 지니고 있기 때문에 성립이 된다.
고로 Key[]를 반환하고 해당 Key[]는 string[] 타입을 지닌다.

```typescript
type Start = string | number | boolean;
type Result<Key> = Key extends string | boolean ? Key[] : never;
let n: Result<Start> = ['hi'];
n = [true];
```

Key에 Start가 들어가서

Result<string | number | boolean>는 Result<string> | Result<number> | Result<boolean>

- Start (string | number | boolean) extends string | boolean ? key[] : never;
    - (string extends string | boolean ? key[] : never) | (number extends string | boolean ? key[] : never) | (boolean extends string | boolean ? key[] : never)
        - key[] | never | key[]
        - string[] | never | boolean[]
        - string[] | boolean[]이 될 것 같지만
        - string[] | false[] | true[]가 됨 → Why? 
        boolean을 true | false로 인식

Start가 string이나 boolean을 포함하고 있으므로 Key[]를 반환할 것이다.

Key[]는 string[] | false[] | true[]를 반환한다.

분배법칙이 일어났으며 boolean이 분배법칙에 적용되었기 때문에 

string, number, boolean에 대하여 string, boolean은 지니고 있어서 반환하고 number는 없어서 false로 반환한 것 같다.

```typescript
type IsString<T> = T extends string ? true : false;
type Result = IsString<'hi' | 3>; // type Result = boolean
```

IsString<’hi’> | IsString<3> 이고

(’hi’ extends string ? true : false) | (3 extends string. true : false)

true | false이므로 최종적으로 boolean이 된다.

이럴 때 분배법칙이 일어나지 않게 해야함

- **배열로 제네릭을 감싸면 분배법칙이 일어나지 않음**

```typescript
type IsString<T> = [T] extends [string] ? true : false;
type Result = IsString<'hi' | 3>; // type Result = false
```

[’hi’ | 3]이 [string]을 extends하는지 검사하므로 false가 된다.

알아두면 좋은 지식 중 하나는 never도 분배법칙의 대상이 된다.
컨디셔널 타입에서 제네릭과 never가 만나면 무조건 never가 된다고 생각하자.

그렇기 때문에 never을 타입 파라미터로 사용하려면 분배법칙이 일어나는 것을 막아야한다.

분배법칙을 막는 방법? 위에서 적었듯이 배열로 제네릭을 감싸주자. 그러면 막을 수 있다.