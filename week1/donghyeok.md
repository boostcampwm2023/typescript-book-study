# 1주차 스터디 내용 정리

> #### info::출처
>
> 이 내용은 [타입스크립트 교과서](https://product.kyobobook.co.kr/detail/S000208416779) 책을 읽고 스터디한 내용을 바탕으로 정리했습니다.

<!-- TOC -->

- [1주차 스터디 내용 정리](#1주차-스터디-내용-정리)
  - [2.1 변수, 매개변수, 반환값에 타입을 붙이면 된다](#21-변수-매개변수-반환값에-타입을-붙이면-된다)
    - [주의할 점 (bigint, symbol)](#주의할-점-bigint-symbol)
  - [2.5 타입으로 쓸 수 있는 것을 구분하자](#25-타입으로-쓸-수-있는-것을-구분하자)
    - [타입 선택 주의할 점](#타입-선택-주의할-점)
  - [2.2 타입 추론을 적극 활용하자](#22-타입-추론을-적극-활용하자)
    - [타입 표기 방법](#타입-표기-방법)
    - [타입 추론은 항상 정확할까?](#타입-추론은-항상-정확할까)
    - [위의 내용에 대해서 어떻게 생각하는지 물어볼 것!](#위의-내용에-대해서-어떻게-생각하는지-물어볼-것)
    - [기억할 점](#기억할-점)
  - [모든 내용을 전부 정리하기엔 낭비인 것 같아서 핵심 요소만 정리](#모든-내용을-전부-정리하기엔-낭비인-것-같아서-핵심-요소만-정리)
  - [2.3 값 자체가 티입인 리터럴 타입이 있다](#23-값-자체가-티입인-리터럴-타입이-있다)
  - [2.4 배열 말고 튜플도 있다](#24-배열-말고-튜플도-있다)
    - [옵셔널 (?)](#옵셔널-)
  - [2.7 타입스크립트에만 있는 타입을 배우자](#27-타입스크립트에만-있는-타입을-배우자)
    - [2.7.1 any](#271-any)
    - [타입스크립트가 명시적으로 any를 반환하는 경우](#타입스크립트가-명시적으로-any를-반환하는-경우)
    - [2.7.2 unknown](#272-unknown)
    - [2.7.3 void](#273-void)
    - [2.7.4 {}, Object](#274--object)
    - [2.7.5 never](#275-never)
    - [함수 표현식 vs 함수 선언문](#함수-표현식-vs-함수-선언문)
  - [2.8 타입 별칭으로 타입에 이름을 붙이자](#28-타입-별칭으로-타입에-이름을-붙이자)
  - [2.9 인터페이스로 객체를 타이핑하자](#29-인터페이스로-객체를-타이핑하자)
    - [2.9.1 인터페이스 선언 병합](#291-인터페이스-선언-병합)
    - [2.9.2 네임스페이스](#292-네임스페이스)

<br />

## 2.1 변수, 매개변수, 반환값에 타입을 붙이면 된다

---

TypeScript의 경우 JavaScript의 자료형과 대응되며 함수와 배열도 객체이므로 Object 타입을 가지게 된다.

- string(문자열)
- number(숫자)
- noolean(부울 값)
- null
- undefined
- symbol
- nigint
- object

- bigint의 경우 JS에도 추가된 데이터 타입이지만 처음 보게되어 조사를 해보았다.
- number는 2의 53승까지의 정수를 표현할 수 있지만 bigint를 이용하면 이보다 큰 수도 표현할 수 있게 된다.

### 주의할 점 (bigint, symbol)

bigint, symbol 둘 다 한번도 타이핑해서 사용해본 적이 없다.

bigint의 경우 ES2020에 추가된 타입

symbol의 경우 ES2015에 추가된 타입이기 때문에 사용하기 위해선 ts config에서 설정을 해주어야 사용할 수 있다.

→ 사실상 개발을 할 때 사용이 될까? 싶다.

## 2.5 타입으로 쓸 수 있는 것을 구분하자

### 타입 선택 주의할 점

타입을 적을 때 string과 String, number와 Number는 다른 것을 주의할 것!

이 값들은 Date, Math, Error, String, Object, Number, Boolean과 같은 내장 객체는 타입으로 이용할 수 있다.

이 중 Date, Math, Error는 어쩔 수 없이 사용하게 되나 String, Object, Number, Boolean을 타입으로 사용하는 것은 권장하지 않는다.

1. number끼리는 x + y 같은 연산이 가능하지만 Number는 연산이 불가능
2. string 타입에 String 타입을 대입할 수 없음

이런 다양한 사유가 있기 때문에 소문자로 통일해서 사용하는 것이 정신건강에 좋다.

## 2.2 타입 추론을 적극 활용하자

---

### 타입 표기 방법

```tsx
function plus(x: number, y: number): number {
  return x + y;
}

const minus = (x: number, y: number): number => x - y;

// function을 사용할 수도 있고 arrow function을 사용할 수 있음
// 위 두 개의 함수는 보이는 형식만 다를뿐 우리가 알고 있는 +, - 연산 함수
```

함수를 사용할 때 매개변수(파라미터)에는 타입을 지정해주어야 한다.

왜냐면 타입스크립트는 스스로 타입 추론을 할 수 있는 기능이 내장되어있기 때문이다.

그렇기 때문에 위의 함수를

```tsx
function plus(x: number, y: number): number {
	return x + y;
}

const result1: number = plus(1, 2);
const result2: plus(1, 2);

// 둘다 원활히 동작함 (result2의 plus의 타입을 추론하기 때문)

function plus(x: number, y: number) {
	return x + y;
}

// 이렇게 매개변수에만 타입을 넣어도 추론을 알아서 해줌
```

### 타입 추론은 항상 정확할까?

정답은 X

타입 추론을 우리가 생각하는 것과 다르게 할 수도 있음

교재에서 세운 타입스크립트 사용 원칙

```tsx
const str: string = "hello";
const str2: "hello" = "hello";
const str3 = "hello";

// 이 3개에서 가장 부정확한 것은 의외로 str1임
// const로 정의되어있기 때문에 hello 외의 다른 문자열이 될 수 없으나 string으로 정의가 되어있음
// 오히려 str3은 타입을 붙여주지 않았지만 오히려 타입 추론으로 정확하게 나오는 것을 확인 가능
```

위의 같은 이유로 다음과 같은 원칙을 권장

> 타입스크립트가 타입을 제대로 추론하면 그대로 쓰고, 틀리게 추론할 때만 올바른 타입을 표기한다.

### 위의 내용에 대해서 어떻게 생각하는지 물어볼 것!

진짜 상수로 사용하는 것 예를 들어 절대 안바뀌는 값들을 주로 사용할 때

```tsx
const KAKAO_AUTH: string = “ASFKLNANSFLK";
const KAKAO_AUTH: "KAKAO_AUTH" = “ASFKLNANSFLK";

// 이런 절대 안바뀔 값의 경우에는 이렇게 진짜 전용 타입을 만들어주면 되고
// 이런게 아닌 이상은 그냥 string을 사용할 것 같은데 어떻게 생각하는지
```

### 기억할 점

null, undefined를 let 변수에 대입할 때에는 any 타입으로 추론된다.

symbol 또한 const 변수에 대입할 때에는 typeof sym, let 변수에 대입할 때에는 symbol
typeof sym의 경우 고유한 심볼을 뜻하기 떄문에 unique symbol이라고 하며 비교를 할 수 없는 말 그대로 unique한 값이 된다.
타입스크립트에서는 Unique symbole 끼리의 비교를 금지한다.

## 모든 내용을 전부 정리하기엔 낭비인 것 같아서 핵심 요소만 정리

## 2.3 값 자체가 티입인 리터럴 타입이 있다

---

as const; 접미사를 붙이면 타입을 고정시킬 수 있다.

마우스 오버헤드를 해보면 readonly라는 수식어가 붙어있음을 확인할 수 있고 이렇게 되면 해당 값은 변경할 수 없다.

## 2.4 배열 말고 튜플도 있다

```tsx
const arr1: number[] = [1, 2, 3];
const arr2: Array<number> = [1, 2, 3];

// 위의 두 개 모두 같은 것을 의미함
// <> 제네릭인데 사실상 사용해보면서 제네릭에 대한 이해도가 거의 전혀 없음
// 상당히 어려운 개념이기도 한데 2.14절에서 학습하는 것으로
```

정교한 타입 검사를 원한다면 튜플을 사용할 것

```tsx
const array = [123, 4, 56];
array[3].toFixed();

// 인덱스 3은 값이 없는데 toFixed()를 붙일 수 있는 문제가 생김
// number 타입으로 추론되어서 그렇다고 함

// 튜플 사용
const array: [number, number, number] = [123, 4, 56];
array[3].toFixed(); // Obeject is possibly 'undefined'
```

이렇게 하면 일일히 타입을 넣는 것이 매우 불편하다.

그래서 잘 사용하지 않을 것 같다.

본래 배열에는 보통 하나의 타입만 넣기 때문에 사용하는 경우는 드물 것 같다.

```tsx
const strNumBools: [string, nuber, ...bolean[]] = [
  "hi",
  123,
  false,
  true,
  false,
];

// 이렇게 전개연산자를 사용해서 그 구간은 일일히 지정할 필요가 없다곤 하지만
// 인덱스 당 타입이 다 다르면 타입 노가다를 해야할 것 같음
```

### 옵셔널 (?)

타입스크립트를 사용하면서 흔하게 볼 수 있는 ? 는 옵셔널을 뜻한다.

옵셔널 체이닝이라고 예전 ios를 잠깐 해볼 때 했던 경험으로는 ‘있던지 말던지’ 라는 식으로 이해를 했었다.

교재에서도 ?는 옵셔널 수식어로 해당 자리에 값이 있어도 그만, 없어도 그만 이라는 의미라고 한다.

옵셔널 자리에는 undefined가 들어갈 수 있으며 순서는 지켜져야 한다.

→ 순서는 지켜져야한다?

```tsx
let tuple: [number, boolean?, string?] = [1, false, "hi"];
tuple = [3, true]; // string 없어도 됨
tuple = [5]; // boolean, string 없어도 됨
tuple = [7, "no"]; //제일 마지막 줄의 'no'는 저 자리는 boolean 자리이므로 못 들어감
```

## 2.7 타입스크립트에만 있는 타입을 배우자

---

### 2.7.1 any

any는 타입스크립트에서 지양해야할 타입

모두가 알다시피 JS에서 TS로 전환한다고 하면서 any가 떡칠되어있는 짤 같은 것들을 모두 봤을 것이다.
이는 TS로 전환한 이유가 모두 사라지고 오히려 TS 세팅으로 인한 불편함만 증폭시킬 수 있는 미친 짓이다.
(물론 나도 가끔 모르겠거나 도피를 할 때 이런 미친 판단으로 any를 박고 돌아가니까 넘어간 적이 몇 번 있다.
후에 gpt를 사용해서 고쳤던 기억이,,)

any 타입을 사용할 경우 js와 같이 전혀 쌩뚱맞은 짓을 해도 오류를 표시하지 않는다.
any 타입은 모든 동작을 허용하며 타입을 검사할 수 없다.

아까 첫 번째에 이어 두번 째 원칙이 나온다.

> any 타입은 타입 검사를 포기한다는 선언과 같다. 타입스크립트가 any로 추론하는 타입이 있다면 타입을 직접 표기해야한다.

any를 사용하지말자.

```tsx
const arr = []; // const arr: any[]
arr.push("1");
arr; // const arr: string[]
arr.push(3);
arr; // const arr: (string | number)[]
```

any[]로 추론된 배열은 신기하게도 값을 넣으면 any가 아닌 타입 추론이 자동으로 된다.

배열을 제외한 다른 것들의 경우 any 타입에서 파생된 값들은 전부 any 타입으로 지정이 된다.

그리고 사용하면서 어떠한 배열에는 보통 동일한 타입을 넣는 경우가 많아서
지금 현재 위에서 본 배열에 관한 타입은 외우진 못할 것 같고 그때 사용할 일이 생기면 구글링을 해서 바로바로 사용하도록 하자.

### 타입스크립트가 명시적으로 any를 반환하는 경우

```tsx
fetch("url")
  .then((response) => {
    return response.json();
  })
  .them((result) => {
    // (parameter) result: any
  });

const result = JSON.parse('{"hello":"json"}'); // const result: any
```

자주 사용하게되는 fetch에도 타입이 any로 나온다고 한다.
이 구절을 읽고 아 정말 하나하나 다 설정해줘야하구나.. 라는 생각이 들었다.

```tsx
fetch("url")
  .then<{ data: string }>((response) => {
    return response.json();
  })
  .them((result) => {
    // (parameter) result: { data: string; }
  });

const result: { hello: string } = JSON.parse('{"hello":"json"}');
```

→ 책에서 마지막 구절 } 빠져서 issue 올림

https://github.com/ZeroCho/ts-book/issues/31

### 2.7.2 unknown

unknown은 any와 비슷하게 모든 타입을 대입할 수 있지만 그 후 어떠한 동작도 수행할 수 없다.

직접 unknown 타입을 타이핑할 일은 없을 것 같다.

흔히 우리가 사용하는 try-catch 문에서 catch(e)에서의 error가 unknown으로 추론된다고 한다.

### 2.7.3 void

void는 TS에서 타입으로 사용된다.

함수의 반환 값이 없는 경우 반환 값이 void 타입으로 추론된다.
JS에서 반환 값이 없을 경우 자동으로 undefined가 반환되는 것을 많이 봤을 것이다.
TS에서도 마찬가지지만 타입은 void가 된다.

```tsx
import { Injectable, UnauthorizedException } from "@nestjs/common";
import { InjectRepository } from "@nestjs/typeorm";
import { UserRepository } from "./repositories/user.repository";
import { AuthCredentialsDto } from "./dto/auth-credential.dto";
import * as bcrypt from "bcryptjs";
import { JwtService } from "@nestjs/jwt";

@Injectable()
export class AuthService {
  constructor(
    @InjectRepository(UserRepository)
    private userRepository: UserRepository,
    private jwtservice: JwtService
  ) {}

  async signUp(authCredentialsDto: AuthCredentialsDto): Promise<void> {
    return this.userRepository.createUsers(authCredentialsDto);
  }

  // 해당 로직 유저를 추가하는 기능이며, 이미 등록되어있는 유저를 거르는 함수
  async createUsers(authCredentialsDto: AuthCredentialsDto): Promise<void> {
    const { username, password } = authCredentialsDto;

    const salt = await bcrypt.genSalt();
    const hashedPassword = await bcrypt.hash(password, salt);

    const user = this.create({ username, password: hashedPassword });

    try {
      await this.save(user);
    } catch (error) {
      if (error.code == "23505") {
        throw new ConflictException("Existing username");
      } else {
        throw new InternalServerErrorException();
      }
    }
  }
}
```

이 지문을 읽고 궁금해서 찾아보았다.

지금 생각해보면 그때 당시에도 이해가 안됐는데 찾지를 않고 지나간 부분이 생각나서 가지고 왔다.

이 코드에서

```tsx
  async signUp(authCredentialsDto: AuthCredentialsDto): Promise<void>
```

이 구절에서 왜 반환값이 Promise<void>일 것이라고 생각하실까요?

이 메서드는 비동기 작업을 수행하지만 실제로는 다른 값을 반환하는 것이 아니라, 성공하면 아무것도 반환하지 않고 작업이 성공적으로 완료될 때까지 대기하기만 한다.

그래서 이 교재에서 나왔듯이 반환값이 따로 없기 때문에 타입을 void로 지정한 것 같다.

**void는 두 가지 목적을 위해 사용**

1. 사용자가 함수의 반환값을 사용하지 못하도록 제한
2. 반환값을 사용하지 않는 콜백 함수를 타이핑할 때 사용

### 2.7.4 {}, Object

{} 타입은 객체로 오해할 수 있으나, null과 undefined를 제외한 모든 값을 의미

{} 타입은 Object 타입과 같고 앞서 말했듯이 object 타입과는 다르다.

이름은 객체이지만 객체만 대입할 수 있는 타입은 아님

```tsx
const obj: object = { hello: "world" };
const obj2: {} = { hello: "world" };

// object

// 이 2개의 값은 할당할 수 없음
const n: Object: null;
const u: Object = undefined;
```

교재에서도 나왔듯이 사실상 이 타입도 잘 사용할 일이 없을 것 같다.

object vs Object vs {}

- object(객체)

  - 함수와 배열도 객체이므로 object에 포함
  - 원시값이 아닌 객체를 의미함

- Object, {}
  - 둘이 동일함
  - 객체가 아님, null, undefined를 할당할 수 없음, 이 두 개 제외 모든 값을 의미 가능
  - 실제로 사용할 수가 없음

원시 값은 `문자(string), 숫자(number), bigint, 불린(boolean), 심볼(symbol), null, undefined`

결국 object도 위의 7가지 타입을 사용할 수 없고 Object, {}도 실제로 사용할 수 없기 때문에 그냥 쓰레기인 것 같다.

왜 쓰는지 모르겠다. 의견 공유를 해보면 좋을 것 같다!

### 2.7.5 never

never 타입에는 어떠한 타입도 대입할 수 없음

함수 선언문과 함수 표현식에 따라 다름

- 함수 선언문
  ```tsx
  function neverFunc1() {
    throw new Error("에러");
  }
  ```
- 함수 표현식
  ```tsx
  const neverFunc2 = () => {
    throw new Error("에러");
  };
  ```

나는 function 보다는 Arrow function을 사용하고 있다.
딱히 둘의 차이점이 뭔지 모르고 리액트를 사용할 때 대부분 Arrow Function으로 사용했던 방식이 익숙해져서
단지 이유를 모르고 사용하고 있다.

저번 마스터 클래스 때도 질문을 남겼었는데 크롱님께서는 그냥 둘다 써도 된다고 하셨는데
TS에서 never 파트를 읽으며 타입 반환이 달라지는 점을 확인하고 찾아보게 되었다.

우선 never는 함수 선언문에서 반환값의 타입이 void, 함수 표현식에서 반환값의 타입이 never로 추론된다.

### 함수 표현식 vs 함수 선언문

- **함수 선언식은 호이스팅에 영향을 받지만, 함수 표현식은 호이스팅에 영향을 받지 않는다.**
- **함수 선언문은 var와 같이 함수 스코프(function scope)를 가지고 let과 const 변수는 블록 스코프(block scope)를 갖는다.**

함수 선언식은 코드를 구현한 위치와 관계없이 자바스크립트의 특징인 **호이스팅**에 따라 브라우저가 자바스크립트를 해석할 때 맨 위로 끌어 올려진다.

그래서 JS에서 코드를 타이핑할 때 보면 function으로 선언한 함수의 경우 함수가 아래에 있어도 위에서 사용할 수 있다.
이것은 호이스팅에 따라 선언한 function이 코드상 제일 위로 끌어올려지기 때문에 사용할 수 있다.

그런데 함수 표현식인 Arrow Function을 사용할 경우 아래에서 선언할 경우 위에서 사용할 수가 없다.

이외에도 클로저, 인자에 따른 장단점이 존재하는데 현재 클로저에 대한 이해도가 높지 않은 상태이기 때문에 후에 학습을 진행할 때 궁금증이 생기면 그때 학습해서 익힐 예정
(지금 블로그 포스트를 읽어보아도 와닿지가 않음, 필요할 때가 되면 이해하기 쉬울 것으로 판단)

## 2.8 타입 별칭으로 타입에 이름을 붙이자

---

```tsx
type A = string;
const str: A = "hello"; // string 타입을 가짐
```

- 타입 별칭은 대문자로 시작하는 단어로 만들 것

```tsx
type Person = {
  name: string;
  age: number;
  married: boolean;
};

const person2: Person = {
  name: "zero",
  age: 28,
  married: false,
};

const person3: Person = {
  name: "nero",
  age: 32,
  married: true,
};
```

여기까지 학습하고 나는 평소에 type보다는 interface를 사용해서 구글링으로 차이점을 보려고 했더니

## 2.9 인터페이스로 객체를 타이핑하자

---

객체 타입에 이름을 붙이는 방법

나는 타입을 붙일 때 interface를 사용한다.

- 인터페이스도 마찬가지로 첫 자를 대문자로 선언하는 것이 좋다. (관습)

```tsx
interface Person {
  name: string;
  age: number;
  married: boolean;
}

const person2: Person = {
  name: "zero",
  age: 28,
  married: false,
};

const person3: Person = {
  name: "nero",
  age: 32,
  married: true,
};
```

type과의 차이점을 확인할 수 있겠는가?

```tsx
interface Person {
  name: string; // 콤마
  age: number; // 세미콜론
  married: boolean; // 아무것도 안붙임
}
```

인터페이스의 속성 마지막에는 콤마, 세미콜론, 줄바꿈으로 구분할 수가 있다.
그런데 사실 이렇게 하지말고 일관성 있게 ,로 구분하는 것이 좋다고 생각한다. 오히려 헷갈린다.

![스크린샷 2023-09-09 오후 5.04.33.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/78f25085-53dd-4580-a6e4-cc29cfae714d/bb76fdde-6c8b-4621-9512-c63d6f95996a/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-09_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_5.04.33.png)

과거 나의 기록을 보니 나는 전부 ; 로 구분을 했었다.

![스크린샷 2023-09-09 오후 5.04.56.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/78f25085-53dd-4580-a6e4-cc29cfae714d/b64bb17f-ed95-4321-801c-24e5ff9c1819/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-09-09_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_5.04.56.png)

### 2.9.1 인터페이스 선언 병합

이 내용에 대해서 서칭을 하고 있었는데 바로 교재에 나왔다.

type과 interface의 차이점 중 하나는 선언적 확장이 가능하다는 것이다.

type의 경우 처음 위의 코드에서 type Person으로 등록을 하면 다시 추가할 수 없다.

하지만 interface의 경우 한번 더 선언해서 내용을 합쳐서 늘릴 수 있다.

하지만 이 방법도 같은 변수명의 interface가 늘어나기 때문에 가급적 interface 정의 구문에 가서 늘려주는 것이 좋다고 판단한다. 아마 interface 전용 파일을 하나 새로 파서 거기에 타입을 지정했던 것으로 기억하기 때문에 그렇게 하는게 좋아보인다.

### 2.9.2 네임스페이스

인터페이스 병합의 단점 중 하나는 이미 선언을 해놓은 인터페이스가 다른 사람에 의해 의도치않게 병합될 수 있다는 점이다.

나 혼자 개발을 할 때에는 이런 일이 없을 수 있겠지만 여러 명이서 작업을 한다면 같은 명의 interface가 존재하는지 모르고 작업을 진행할 수 있다.

네임스페이스는 이러한 경우를 방지하기 위해 생겼다.

```tsx
namespace Example {
  interface Inner {
    test: string;
  }
  type test2 = number;
}

const ex1: Example.Inner = {
  test: "hello",
};

const ex2: Example.test2 = 123;
```

Example 네임스페이스를 선언했고 이 안에는 Inner 인터페이스와 test2 타입 별칭이 존재한다.
각각 Example.Inner, Example.test2로 접근이 가능하다.

다만 외부에서 사용하려면 interface, type 앞에 export를 붙여야한다.

네임스페이스 기능은 한번도 사용해보지 않았는데 알고 있어야 할 내용 중 하나라고 생각된다.
