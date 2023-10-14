## 2.22 infer로 타입스크립트의 추론을 직접 활용하자

- `infer`: 타입 추론 극한 활용 기능
- 제네릭 타입에서 유용하게 사용될 수 있다
  - 제네릭 타입 파라미터를 추론하고 다른 타입과 관련된 타입을 추출할 수 있음

```tsx
type El<T> = T extends (infer E)[] ? E : never;

type Str = El<string[]>;
type NumOrBool = El<(Number | boolean)[]>;
```

- 추론을 맡길 부분을 `infer E` 로 처리

```tsx
type MyParameters<T> = T extends (...args: infer P) => any ? P : never;
// (...args: infer P) => any 는 임의의 함수 타이핑하는 부분

type MyConstructorParameters<T> = T extends abstract new (...args: infer P) => any ? P : never;
// abstract new (...args: any) => any는 임의의 생성자를 타이핑하는 방법

type MyReturnType<T> = T extends (...args: any) => infer R ? R : any;

type MyInstanceType<T> = T extends abstract new (...args: any) => infer R ? R : any;

type P = MyParameters<(a: string, b: number) => string>;
// type P = [a: string, b: number]

type R = MyReturnType<(a: string, b: number) => string>;
// type R = string

type CP = MyConstructorParameters<new (a: string, b: number) => {}>;
// type CP = [a: string, b: number]

type I = MyInstanceType<new (a: string, b: number) => {}>;
// type I = {}
```

- `MyParameters`: 함수의 매개변수의 타입을 추론
  - 제네릭이 함수 타입이면 매개변수를 리턴
- `MyConstructorParameters` : 생성자의 매개변수 타입을 추론

  - 제네릭이 생성자 타입이면 매개변수를 리턴

- 여러 개의 타입 변수 (infer) 사용
  ```tsx
  type MyPAndR<T> = T extends (...args: infer P) => infer R ? [P, R] : never;
  type PR = MyPAndR<(a: string, b: number) => string>;
  // type PR = [[a: number, b: string], string]
  ```
- 같은 타입 변수 (infer)을 여러 개 사용할 수도 있음

  ```tsx
  type Union<T> = T extends { a: infer U; b: infer U } ? U : never;
  type Result1 = Union<{ a: 1 | 2; b: 2 | 3 }>;
  // type Result1 = 1 | 2 | 3

  type Intersection<T> = T extends {
    a: (pa: infer U) => void;
    b: (pb: infer U) => void;
  }
    ? U
    : never;
  type Result2 = Intersection<{ a(pa: 1 | 2): void; b(pb: 2 | 3): void }>;
  // type Result2 = 2
  ```

  - 이 경우에는 기본적으로 유니언, 매개변수일 경우에는 인터섹션

## 2.23 타입을 좁혀 정확한 타입을 얻어내자

타입 좁히기는 자바스크립트 문법을 사용해서 진행해야 함

- `null`과 `undefined` 구분
  ```tsx
  function example(param: string | null | undefined) {
    if (param === undefined) {
      param;
    } else if (param === null) {
      param;
    } else {
      param;
    }
  }
  ```
  - 굳이 `typeof`를 사용하여 구분할 필요는 없음
- boolean 구분
  ```tsx
  function example(param: boolean) {
    if (param) {
      param; // true;
    } else {
      param; // false;
    }
  }
  ```
- 배열 구분
  ```tsx
  function example(param: string | number[]) {
    if (Array.isArray(param)) {
      param;
    } else {
      param;
    }
  }
  ```
- 클래스 구분

  ```tsx
  class A {}
  class B {}

  function example(param: A | B) {
    if (param instanceof A) {
      param; // A
    } else {
      param; // B
    }
  }
  ```

- 두 종류의 객체 구분

  ```tsx
  interface X {
    width: number;
    height: number;
  }
  interface Y {
    length: number;
    center: number;
  }

  function objXorY(param: X | Y) {
    if ('width' in param) {
      param; // X
    } else {
      param; // Y
    }
  }
  ```

  - 객체가 가지고 있는 프로퍼티와 `in` 연산자 사용하여 구분
  - 브랜드 속성으로 객체 구분하기
    공통으로 사용하는 `__type` 프로퍼티가 있으므로 `in` 연산자를 사용하지 않고도 바로 속성에 접근할 수 있다

    ```tsx
    interface Money {
      __type: 'money';
      amount: number;
      unit: string;
    }

    interface Liter {
      __type: 'liter';
      amount: number;
      unit: string;
    }

    function moneyOrLiter(param: Money | Liter) {
      if (param.__type === 'money') {
        param;
      } else {
        param;
      }
    }
    ```

    ```tsx
    function isMoney(param: Money | Liter): param is Money {
      if (param.__type === 'money') {
        return true; // true 리턴할 경우 Money 값으로 좁혀짐
      } else {
        return false;
      }
    }
    function moneyOrLiter(param: Money | Liter) {
      if (isMoney(param)) {
        param;
      } else {
        param;
      }
    }
    ```

    - 다음과 같이 타입 좁히기 함수를 따로 만들 때는 **타입 서술 함수(Type Predicate)**로 `param is Money` 라고 표기해 주어야 한다

## 2.24 자기 자신을 타입으로 사용하는 재귀 타입이 있다

- 자기 자신을 타입으로 다시 사용하는 타입

```tsx
type Recursive = {
  name: string;
  children: Recursive[];
};

const recur1: Recursive = {
  name: 'recur',
  children: [{ name: 're-recur', children: [] }],
};
```

- 컨디셔널 타입에도 사용 가능

  ```tsx
  type ElementType<T> = T extends any[] ? ElementType<T[number]> : T;
  ```

  - ElementType이 어떻게 사용되는 것일까??
    from ChatGPT

    ```tsx
    type ElementType<T> = T extends any[] ? ElementType<T[number]> : T;

    type P = ElementType<number[][]>; // number
    ```

    여기서 **`[number]`**는 인덱스 접근 타입입니다. 만약 **`T`**가 배열이라면 **`T[number]`**는 배열의 모든 요소 타입을 나타냅니다. 예를 들어, 만약 **`T`**가 **`number[]`**라면 **`T[number]`**는 **`number`**가 됩니다. 그리고 **`ElementType<T[number]>`** 부분은 다시 그 배열의 요소 타입을 찾아내는 재귀적인 조건부 타입입니다.
    이것을 더 자세히 설명하자면:

    - **`T[number]`**는 배열 **`T`**의 모든 요소를 나타냅니다.
    - **`ElementType<T[number]>`**는 **`T[number]`**의 요소 타입을 나타냅니다.
    - 따라서, **`T`**가 배열일 때 **`ElementType<T>`**는 배열의 요소 타입을 찾아냅니다.
      이를 통해 중첩된 배열에서도 올바른 요소 타입을 추출할 수 있습니다. 예를 들어, **`number[][]`**와 같은 중첩된 배열에 대해서도 제대로 작동하여 **`number`**를 찾아낼 수 있습니다.

- 재귀 타입 사용할 때 infinite 에러를 조심하자
  ```tsx
  type InfiniteRecur<T> = { item: InfiniteRecur<T> };
  type Unwrap<T> = T extends { item: infer U } ? Unwrap<U> : T;
  type Result = Unwrap<InfiniteRecur<number>>;
  // Type instantiation is excessively deep and possibly infinite.
  ```
- JSON 타입을 재귀 타입 사용해서 선언해 보기

  ```tsx
  type JSONType = string | boolean | number | null | JSONType[] | { [key: string]: JSONType };

  const a: JSONType = 'string';
  const b: JSONType = [1, false, { hi: 'json' }];
  const c: JSONType = {
    prop: null,
    arr: ['string', 123, true],
    jsons: {
      name: 'this is json',
      address: '어딘가',
      keys: [{ 1: true }, { 2: false }],
    },
  };
  ```

  - 1. 재귀 타입으로 배열 타입 거꾸로 뒤집는 타입 만들어 보기
    ```tsx
    type Origin = [number, string, boolean];
    type Reverse<T> = T extends [...infer L, infer R] ? [R, ...Reverse<L>] : [];
    ```
  - 2. 1 타입을 응용해서 함수의 매개변수 순서를 바꾸는 타입 만들어 보기

    ```tsx
    type Reverse<T> = T extends [...infer L, infer R] ? [R, ...Reverse<L>] : [];

    type FilpArgument<T> = T extends (...args: infer A) => infer R ? (...args: Reverse<A>) => R : never;

    type P = FilpArgument<(a: string, b: number) => string>;
    ```

  - 이것처럼 타입 응용해 보는 연습 도움이 될 것 같다! → 타입 챌린지
  - 그렇지만 아직 어렵다 😂

## 2.25 정교한 문자열 조작을 위해 템플릿 리터럴 타입을 사용하자

- 템플릿 리터럴도 타입으로 사용할 수 있다
- 타입과 타입을 조합하여 쓸 때 유용
  ```tsx
  type City = 'seoul' | 'busan' | 'suwon';
  type Vihicle = 'car' | 'bike' | 'walk';
  type CityVihicle = `${City}:${Vihicle}`;
  ```
  - City 타입에 도시를 추가하거나 Vihicle 타입에 탈 것을 추가하면 자동으로 CityVihicle 타입도 조합된다
- 템플릿 리터럴로 좌우 공백(문자열) 제거하기

  ```tsx
  type RemoveX<Str> = Str extends `x${infer Rest}` ? RemoveX<Rest> : Str extends `${infer Rest}x` ? RemoveX<Rest> : Str;
  type P = RemoveX<'xxtestxx'>;

  type RemoveEmpty<Str> = Str extends ` ${infer Rest}`
    ? RemoveEmpty<Rest>
    : Str extends `${infer Rest} `
    ? RemoveEmpty<Rest>
    : Str;
  type R = RemoveEmpty<'  test   '>;
  ```

  `P` 는 재귀적으로 수행되면서 xxtestxx ⇒ xtestxx ⇒ testxx ⇒ testx ⇒ testx

## 2.26 추가적인 타입 검사에는 satisfies 연산자를 사용하자

```tsx
type Universe = {
  [key in 'sun' | 'sirius' | 'earth']: { type: string; parent: string } | string;
};

const universe: Universe = {
  sun: 'star',
  sirius: 'star', // sirius 오타
  earth: { type: 'planet', parent: 'sun' },
};

universe.earth.type;
// Property 'type' does not exist on type 'string | { type: string; parent: string; }'.
// Property 'type' does not exist on type 'string'.
```

- satisfies 사용

  ```tsx
  type Universe = {
    [key in 'sun' | 'sirius' | 'earth']: { type: string; parent: string } | string;
  };

  const universe = {
    sun: 'star',
    sirius: 'star',
    earth: { type: 'planet', parent: 'sun' },
  } satisfies Universe;

  universe.earth.type;
  ```

  ![image1](./images/week4/image1.png)
  ![image2](./images/week4/image2.png)

  - type을 사용하지 않았을 때의 객체 내부 값을 그대로 추론하되, 속성 값에 오타가 있는지를 검사할 수 있다

## 2.27 타입스크립트는 건망증이 심하다

```tsx
try {
} catch (error) {
  if (error as Error) {
    error.message;
    // 'error' is of type 'unknown'
  }
}
```

- as 타입 강제 주장이 일시적이기 때문에 변수에 적용해야 타입이 유지된다

```tsx
try {
} catch (err) {
  const error = err as Error;
  if (error) {
    error.message;
  }
}

// 클래스의 인스턴스일 경우 instanceof 사용
try {
} catch (error) {
  if (error instanceof Error) {
    error.message;
  }
}
```

## 2.28 원시 자료형에도 브랜딩 기법을 사용할 수 있다

- 원시 자료형과 브랜딩 기법을 묶어 사용할 수 있다
- 원시 자료형 타입을 세분화하여 사용할 수 있다
- 원래 존재하는 타입이 아니기 때문에 as로 강제 변환

```tsx
type Brand<T, B> = T & { __brand: B };
type KM = Brand<number, 'km'>;
type Mile = Brand<number, 'mile'>;

function kmToMile(km: KM) {
  return (km * 0.62) as Mile;
}

const km = 3 as KM;
const mile = kmToMile(km);
// const mile: Mile
const mile2 = 5 as Mile;
// Argument of type 'Mile' is not assignable to parameter of type 'KM'.
kmToMile(mile2);

//
```
