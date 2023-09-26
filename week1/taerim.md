### 2.1 변수, 매개변수, 반환값에 타입을 붙이면 된다

- 그러나 타입을 붙이지 않아도 타입 추론을 사용할 수 있다

### 2.2 타입 추론을 적극 활용하자

- 타입스크립트의 추론을 어떻게 활용할 수 있을까?
  - 타입스크립트가 타입을 제대로 추론하면 그대로 쓰고, 틀리게 추론할 때만 올바른 타입을 표기한다
- `let` , `const` 를 사용할 경우 같은 변수임에도 타입이 다르게 추론될 수 있다
- {} 타입은 객체를 의미하는 것이 아니라 `null` 과 `undefined` 를 제외한 모든 타입을 의미한다
- `null` 과 `undefined` 를 `let` 변수에 대입할 때는 `any` 로 추론한다
- `typeof sym` 은 고유한 symbol, unique symbol을 의미하고 unique symbol끼리는 서로 비교할 수 없다
  - unique symbol과 일반 symbol, 일반 symbol과 일반 symbol은 비교 가능
  - `This comparision appears to be unintentional because th types 'typeof sym1' and typeof sym2' have no overlap`
- 타입 추론과 **타입 넓히기**를 적극 활용해 보자
- 타입스크립트의 에러를 무시하는 방법
  - @ts-ignore
  - @ts-expect-error
  - @ts-expect-error를 사용하는 것이 다음 줄의 코드에 확실히 에러가 있다는 것을 알리기 때문에 더 낫다

### 2.3 값 자체가 타입인 리터럴 타입이 있다

```tsx
let str: 'hello' = 'hello';
let str: string = 'hello'; // 주로 이렇게 자료형 타입으로 사용하는 게 일반적이긴 하지
```

- `as const` 접미사를 사용하면 타입이 고정되어 추론된다
  ```tsx
  const obj = { name: 'zero' } as const;
  // const obj: {
  //     readonly name: "zero";
  // }
  ```
  - 주로 리터럴 타입을 사용하고자 할 때
  - `readonly` 수식어가 붙은 값은 변경할 수 없다

### 2.4 배열 말고 튜플도 있다

- 빈 배열은 `any[]` 로 추론된다
- 배열 추론의 한계
  ```tsx
  const array = [123, 4, 56];
  array[3].toFixed();
  // array[3]은 존재하지 않아 undefined인데도 toFixed 메소드를 사용해도 컴파일 에러 발생하지 않음
  ```
  → 튜플을 사용하여 해결할 수 있음
- 튜플: 각 요소 자리에 타입이 고정되어 있음

  ```tsx
  const tuple: [number, string, boolean] = [1, 'yes', true];
  tuple[3].toFixed();
  // Object is possibly 'undefined'.
  ```

  - 표기하지 않은 세 번째 인덱스부터는 undefined 타입으로 추론됨
  - but `push` , `pop` 등의 메소드로 배열의 요소 추가/제거하는 것이 가능함
    - 방지하려면 `readonly` 키워드 사용

  ```tsx
  const tuple2: [number, ...string[], number] = [1, 'hello', 'hi', 22];
  ```

  - `…타입[]` 표기로 특정 타입이 연달아 나올 수 있다는 것을 타입으로 설정 가능

- 값에도 구조 분해 할당 사용 가능, 여기서도 자동 타입 추론됨
  ```tsx
  const arr = [123, 4, 56, true];
  const arr2 = [...arr, 'hi']; // const arr2: (string | number | boolean)[]
  ```
- 구조분해 할당 사용할 때 나머지 속성(rest property) 문법도 사용할 수 있는데 여기서도 타입 추론
  ```tsx
  const [a, ...rest] = arr;
  // const a: number | boolean
  // const rest: (number | boolean)[]

  const [b, ...rest2] = [1, 2, 3, 'd'];
  // const b: number
  // const rest2: [number, number, string]
  ```
  - 여기서는 arr이 `(number | boolean)[]` 이기 때문에 a가 `number | boolean` 값으로 추론된다
- 튜플에 옵셔널도 사용할 수 있다
  - `[number, boolean?, string?]` 타입에는 `[number, boolean, string]` , `[number, boolean]` , `[number]` 의 값이 올 수 있다
  - `[number, string]` 타입은 쓸 수 없음.

### 2.5 타입으로 쓸 수 있는 것을 구분하자

- 내장 객체는 타입으로 사용할 수 있지만 String, Object, Number, Boolean, Symbol은 string, object, number, boolean, symbol을 사용하는 것을 권장한다
  - Number 간 연산자 사용 불가능
  - string과 String 호환 불가능
  - Object 타입에는 문자열 대입 가능 등
- 함수의 반환값을 타입으로 사용하고 싶을 때는 ReturnType 참고 (3.3절이라 추후 공부..)
- 클래스는 typeof 연산자 없이 타입으로 사용 가능

### 2.6 유니언 타입으로 OR 관계를 표현하자

- **유니언 타입**: 하나의 변수가 여러 타입을 가질 수 있는 것
- `(string | number)[]` 과 `string | number[]` 의 차이에 유의하기
- **타입 좁히기**(Type Narrowing): 유니언 타입으로부터 정확한 타입을 찾아내는 기법
- 특정 타입에만 사용할 수 있는 메소드를 사용할 때는 타입 좁히기를 통해 사용하면 된다
  ```tsx
  function getNumber(str: number | string) {
    if (typeof str === 'number') {
      return str;
    }
    return parseInt(str);
  }
  ```
- 타입 앞에도 | 연산자를 사용할 수 있음 (여러 줄에 걸쳐서 유니언 표기할 때)
  ```tsx
  type Union = string | number | boolean;
  ```

### 2.7 타입스크립트에만 있는 타입을 배우자

- **any**
  - any를 사용하면 타입을 검사하지 못하므로 타입스크립트를 사용하는 의미가 퇴색됨
  - any를 사용하면 그 이후로도 계속 any 타입이 생성된다
  → 지양하자
  - 타입스크립트가 any로 추론하는 타입은 직접 타입을 표기하자
  - 그런데 `any[]` 로 추론된 배열은 요소가 추가될 때마다 추론되는 타입이 바뀐다고 한다!
    - pop으로 요소를 제거할 때 이전 추론으로 돌아가는 것은 아니다
  - 연산 시 타입 추론이 string, number로 변하기도 한다
  - `JSON.parse` 나 `fetch` 는 명시적으로 any를 반환하므로 타입을 명시해 주어야 한다
  -
- **unknown**
  - unknown 타입을 지정하면 어떠한 동작도 수행할 수 없다?!
  - try-catch 문에서 unknown 타입을 확인할 수 있고, catch(e)의 e에 접근하고 싶을 땐 Type Assertion으로 타입 강제 지정
    ```tsx
    try {
    } catch (e) {
      const error = e as Error;
      // 또는
      const error = <Error>e;
    }
    ```
    - <>를 사용할 수 있지만 react jsx 문법과 충돌하므로 as로 타입 주장을 권장
  - 진짜 안 되는 것 강제 주장 방법: unknown으로 주장한 뒤에 원하는 타입으로 다시 주장하기
    `const a: number = '123' as unknown as string`
  - ! 연산자 사용 예시는 `param!.slice(3)` 이 맞는 건가?
- **void**
  - 함수의 반환값을 무시하도록 하는 특수한 타입
  - 반환값 없는 함수의 리턴 타입은 자동으로 void 추론
  - 반환값이 void 타입인 함수가 undefined가 아닌 반환값을 리턴하게 만들 수는 있음, 그렇지만 반환값이 void이기 때문에 사용하지 못함
    - forEach 콜백 함수에 사용됨
- **{}, Object**
  - {}, Object: null과 undefined를 제외한 모든 값
  - if 문 안에서 unknown과 {}로 분류됨
- **never**
  - 어떤 타입도 대입할 수 없음
  - 함수 선언문과 함수 표현식에서 각각 throw 에러를 던지면 반환값의 타입은 void, never
    ```tsx
    function result() {
      throw new Error(); // void type
    }

    const result2 = () => {
      throw new Error(); // never type
    };
    ```
  - 무한 반복문이 들어 있는 함수처럼 값을 반환하지 않는 경우도 반환값이 never
    - 이 경우에도 함수 표현식일 때만 never
  - 함수 선언문에서 void 추론될 때는 never로 표기하여 사용
  - if-else로 타입 좁히기 사용할 경우에도 해당 변수의 모든 타입이 성립하지 않을 경우에는 never 타입
    ```tsx
    const param: number | string = '';

    if (typeof param === 'number') {
    } else if (typeof param === 'string') {
    } else {
      param;
      // 여기서 param은
    }
    ```

### 2.8 타입 별칭으로 타입에 이름을 붙이자

- `type` 키워드를 사용하여 타입 별칭 지정 가능
- 관습은 대문자로 시작하는 파스칼 케이스
- 타입 표기가 길어질 때, 주로 함수/객체/배열을 사용할 때 가독성과 재사용 측면에서 사용하면 좋다

### 2.9 인터페이스로 객체를 타이핑하자

- **인터페이스 선언**: 타입 별칭처럼 객체 타입에 이름을 붙일 수 있는 또 다른 방법
- 타입 별칭과 마찬가지로 관습은 대문자로 시작하는 파스칼 케이스
- 인덱스 시그니처
  `[key: string]`
  ```tsx
  interface Arr {
    length: number;
    name: string;
    address: string;
    email: string;
    introduce: string;
  }

  // 다음과 같이 쓸 수 있음
  interface Arr {
    length: number;
    [key: string]: string;
  }
  ```
- 인터페이스 선언 병합
  - 같은 이름으로 인터페이스 여러 번 선언하는 경우는 하나로 합쳐짐
  - 인터페이스 확장을 위함
  ```tsx
  interface Merge {
    one: string;
  }

  interface Merge {
    two: number;
  }

  // 다음과 같음

  interface Merge {
    one: string;
    two: number;
  }
  ```
  - 속성을 겹치게 설정할 수는 없음 ⇒ 에러 발생
- 네임스페이스
  - 의도치 않은 인터페이스 선언 병합을 방지하여 사용할 수 있음
  - `export` 해야 사용 가능
  - 중첩해서 사용 가능
    ```tsx
    namespace Example {
      export namespace Outer {
        export interface Inner {
          test: string;
        }
        export type test2 = number;
      }
      export const a = 'real';
    }

    const obj: Example.Outer.Inner = { test: 'hello' };
    ```
  - 내부에 값을 선언했다면 값으로도 선언 가능
  - 동일한 이름을 가진다면 인터페이스처럼 병합됨
