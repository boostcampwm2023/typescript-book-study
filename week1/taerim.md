### 2.10 객체의 속성과 메서드에 적용되는 특징을 알자

- 객체의 속성을 옵셔널로 쓰거나 `readonly` 로 만들 수 있다

  ```tsx
  interface World {
    name: string;
    people: number;
    readonly sky: boolean;
    pet?: boolean;
    // World.pet?: boolean | undefined
  }

  const world: World = {
    name: 'Wonderland',
    people: 10,
    sky: true,
  };

  world.sky = false;
  // Cannot assign to 'sky' because it is a read-only property.
  ```

- 객체 리터럴을 대입할 때와 변수를 대입할 때의 타입 검사 방식이 달라진다

  ```tsx
  interface World {
    name: string;
  }

  const world: World = {
    name: 'Wonderland',
    people: 10,
    // Type '{ name: string; people: number; }' is not assignable to type 'World'.
    // Object literal may only specify known properties, and 'people' does not exist in type 'World'.
  };

  const obj = {
    name: 'Wonderland',
    people: 10,
  };

  const world2: World = obj;

  function unityWorld(world1: World, world2: World): World {
    return { name: world1.name + world2.name };
  }

  unityWorld(obj, { name: 'hi', people: 10 });
  // Argument of type '{ name: string; people: number; }' is not assignable to parameter of type 'World'.
  // Object literal may only specify known properties, and 'people' does not exist in type 'World'.
  ```

  - 객체 리터럴: 잉여 속성 검사(Excess Property Checking)
    - 타입 선언에서 선언하지 않은 속성을 사용할 때 에러를 표시
  - 변수: 객체 간 대입 가능성을 비교

- 구조 분해 할당 시 명시적 타입 타이핑에 유의하기

  ```tsx
  // ❌
  const {
    props: { nested: string },
  } = { props: { nested: 'hi' } };

  // ⭕️
  const {
    props: { nested },
  }: { props: { nested: string } } = { props: { nested: 'hi' } };
  ```

- 특정 속성의 타입을 별도의 타입으로 만들기

  - 인덱스 접근 타입(Indexed Access Type)

    ```tsx
    type Animal = {
      name: 'zebra' | 'lion';
    };

    type AnimalName = Animal['name'];
    // type AnimalName = "zebra" | "lion"
    ```

- 객체의 키의 타입과 값의 타입 구하기

  - `keyof` 연산자 사용하기

    ```tsx
    const obj = {
      hello: 'world',
      name: 'ttaerrim',
      age: 24,
    };

    type Keys = keyof typeof obj;
    type Values = (typeof obj)[Keys];
    ```

- `keyof`
  - 타입스크립트에서는 배열을 위해 객체의 키에 string, symbol + number 타입 허용
    ```tsx
    type Keys = keyof any;
    // type Keys = string | number | symbol
    ```
  - keyof 배열
    - 모든 number → 이해가 잘 안 됨..
    - 배열 속성 이름
      - `length`, `forEach`, `lastIndexOf` 등 배열의 공통 속성
    - 배열 인덱스 문자열
      - `[1, 2, 3]` 이라면 인덱스인 `'0' | '1' | '2'`
- 인덱스 접근 타입으로 특정 키들의 값 타입 추리기

  ```tsx
  const obj = {
    hello: 'world',
    name: 'ttaerrim',
    age: 24,
  };

  type Values = (typeof obj)['hello' | 'name'];
  // type Values = string
  ```

- 매핑된 객체 타입 별칭의 키로 사용하기

  ```tsx
  // ❌
  type HelloAndHi = {
    [key: 'hello' | 'hi']: string;
    // An index signature parameter type cannot be a literal type or generic type. Consider using a mapped object type instead
  };

  // ⭕️
  type HelloAndHi = {
    [key in 'hello' | 'hi']: string;
  };
  ```

  ```tsx
  interface Original {
    name: string;
    age: number;
    married: boolean;
  }

  type Copy = {
    [key in keyof Original]: Original[key];
  };

  /**
  type Copy = {
  	name: string;
  	age: number;
  	married: boolean;
  }
  */
  ```

  - `keyof` 연산자를 사용해 `Original` 의 속성 이름만 추릴 수 있다
  - 인덱스 접근 타입으로 원래 객체의 타입을 가져온다
    1. `'name': Original['name']` 에서 `string` 을 가져옴
    2. `'age': Original['age']` 에서 `number` 을 가져옴
    3. `'married': Original['married']` 에서 `boolean` 을 가져옴

- 튜플에 매핑된 객체 타입 적용하기

  ```tsx
  type Tuple = [1, 2, 3];
  type CopyTuple = {
    [Key in keyof Tuple]: Tuple[Key];
  };
  ```

  - 질문
    여기서 `Tuple[Key]` 안의 `Key` 는 왼쪽의 `Key in keyof Tuple` 에서 가져오는 건가?
  - 읽기 전용

    ```tsx
    interface Original {
      name: string;
      age: number;
      married: boolean;
    }

    type Copy = {
      readonly [key in keyof Original]: Original[key];
    };
    ```

  - 옵셔널
    ```tsx
    type Copy = {
      readonly [key in keyof Original]?: Original[key];
    };
    ```
  - 타입 제거
    ```tsx
    type Copy = {
      -readonly [key in keyof Original]-?: Original[key];
    };
    ```
    - `-readonly`: readonly 수식어 제거
    - `-?`: ? 수식어 제거
  - 속성 이름 바꾸기
    ```tsx
    type Copy = {
      [key in keyof Original as Capitalize<key>]: Original[key];
    };
    ```

### 2.11 타입을 집합으로 생각하자(유니언, 인터섹션)

- 타입스크립트를 집합 관계로 보기
  ![img.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/84ad46d5-32ec-4902-aa6d-09f1c52af182/cf2aff9e-282b-4a94-9adf-7318bc04892b/img.png)
  - 가장 넓은 타입(전체 집합)은 `unknown`
  - 가장 좁은 타입(공집합)은 `never`
  - 유니언 연산자(`|`)는 합집합 역할
  - 인터섹션 연산자(`&`)는 교집합 역할
    ```tsx
    type nev = string & number;
    // type nev = never;
    ```

✔️ 항상 좁은 타입에서 넓은 타입으로 대입해야 한다

- `any` 타입은 집합 관계를 무시하므로 `&` , `|` 연산을 하지 않는 것이 좋다

- `null/undefined` 를 제외한 자료형 & 비어 있지 않은 객체 ⇒ `never` 타입이 되지 않음

  ```tsx
  type H = { a: 'b' } & number;
  // { a: 'b' } & number

  type I = null & { a: 'b' };
  // never

  type J = {} & string;
  // string
  ```

  - {}는 `null`과 `undefined`를 제외한 모든 값을 의미하는 타입이다

### 2.12 타입도 상속이 가능하다

- 상속을 사용하면 부모 객체에 존재하는 속성을 다시 입력하지 않아도 되므로 중복을 제거할 수 있다
- & 연산자로 타입 상속하기

  ```tsx
  type Animal = {
    name: string;
  };

  type Dog = Animal & {
    bark(): void;
  };

  type Cat = Animal & {
    meow(): void;
  };

  type Name = Cat['name'];
  ```

- 타입 별칭이 인터페이스를 상속할 수 있고 인터페이스가 타입 별칭을 상속할 수도 있다
  ```tsx
  interface Animal {
    name: string;
  }
  type Dog = Animal & {
    bark(): void;
  };
  type Cat = Animal & {
    meow(): void;
  };
  type Name = Cat['name'];
  ```
  - 타입 별칭으로 선언한 객체 타입과 인터페이스로 선언한 객체 타입이 호환된다
- 한 번에 여러 타입 상속하기

  ```tsx
  type Animal = {
    name: string;
  };
  interface Dog extends Animal {
    bark(): void;
  }
  interface Cat extends Animal {
    meow(): void;
  }

  interface DogCat extends Dog, Cat {}
  ```

- 상속할 때 부모 속성의 타입 변경하기
  ```tsx
  interface Merge {
    one: string;
    two: string;
  }
  interface Merge2 extends Merge {
    one: 'h' | 'w';
    two: '123';
  }
  ```
  - 부모에 대입할 수 있는 타입으로 좁히는 것은 가능하지만 완전히 다른 타입으로 변경하면 에러 발생
    ```tsx
    interface Merge2 extends Merge {
      one: 'h' | 'w';
      two: 123;
    }
    ```

### 2.13 객체 간에 대입할 수 있는지 확인하는 법을 배우자

```tsx
interface A {
  name: string;
}
interface B {
  name: string;
  age: number;
}

const aObj = {
  name: 'zero',
};
const bObj = {
  name: 'nero',
  age: 32,
};

const aToA: A = aObj;
const bToA: A = bObj;
const aToB: B = aObj;
// Property 'age' is missing in type '{ name: string; }' but required in type 'B'.
const bToB: B = bObj;
```

- B 타입에 A 타입 객체를 대입하는 것은 실패
- A 타입에 B 타입을 대입하는 것은 좁은 타입을 넓은 타입에 대입하는 것이기 때문에 가능하다
  - 여기서는 A 타입이 B 타입보다 넓은/추상적인 타입
  - B 타입이 A 타입보다 좁은/구체적인 타입
- `A | B` 합집합은 `A`, `B`, `A | B` 각각의 집합과 교집합을 전부 대입할 수 없다

  - 합집합은 각각의 집합보다 교집합보다 넓다

    ![Untitled](./images/week2_taerim_diagram.png)

- 튜플은 배열보다 좁은 타입

  - 튜플은 배열에 대입할 수 있으나 배열은 튜플에 대입할 수 없음

  ```tsx
  let a: ['hi', 'readonly'] = ['hi', 'readonly'];
  let b: string[] = ['hi', 'normal'];

  a = b;
  // Type 'string[]' is not assignable to type "['hi', 'readonly']". Target requires 2 element(s) but source may have fewer
  ```

- readonly 수식어가 붙은 배열이 더 넓은 타입

  ```tsx
  let a: readonly string[] = ['hi', 'readonly'];
  let b: string[] = ['hi', 'normal'];

  a = b;
  b = a;
  // Type type 'readonly ['hi', 'readonly'] is 'readonly' and cannot be assigned to the mutable type 'string[]'.
  ```

  - 좁은 타입을 넓은 타입에 대입해야 하기 때문에~

- `readonly` 튜플과 일반 배열 대입하기

  ```tsx
  let a: readonly ['hi', 'readonly'] = ['hi', 'readonly'];
  let b: string[] = ['hi', 'normal'];

  a = b;
  // Type 'string[]' is not assignable to type 'readonly ["hi", "readonly"]'.
  // Target requires 2 element(s) but source may have fewer.
  b = a;
  // The type 'readonly string[]' is 'readonly' and cannot be assigned to the mutable type 'string[]'.
  ```

  - `a = b` 배열이 튜플보다 넓은 타입
  - `b = a`
    - 튜플이 배열보다 좁은 타입이지만
    - readonly 수식어가 붙으면 일반 배열보다 넓은 타입

- 옵셔널인 객체가 옵셔널이지 않은 객체보다 더 넓은 타입

  ```tsx
  type Optional = {
    a?: string;
    b?: string;
  };

  type Mandatory = {
    a: string;
    b: string;
  };
  ```

  - `Optional` 타입보다 `Mandatory` 타입보다 넓은 타입이다
  - 옵셔널은 `기존 타입 | undefined`

- 객체에서는 `readonly` 속성이 붙어도 서로 대입할 수 있다
  - 객체의 `readonly` 속성은 타입의 범위에 영향을 끼치지 않음
  ```tsx
  type ReadOnly = {
    readonly a: string;
    readonly b: string;
  };
  type Mandatory = {
    a: string;
    b: string;
  };
  const o: ReadOnly = {
    a: 'hi',
    b: 'world',
  };
  const m: Mandatory = {
    a: 'hello',
    b: 'world',
  };
  const o2: ReadOnly = m;
  const m2: Mandatory = o;
  // 모두 가능
  ```
- 모든 속성이 동일하면 객체 타입 이름이 달라도 동일한 타입으로 취급됨
  → 구조가 같으면 같은 객체로 인식하는 것을 **구조적 타이핑**(structural typing)이라고 함
  ```tsx
  interface A {
    name: string;
  }
  interface B {
    name: string;
    age: number;
  }
  ```
  - B 인터페이스는 A 인터페이스에 존재하는 모든 속성을 가지고 있기 때문에 **구조적 타이핑 관점에서 A인터페이스**라고 볼 수 있다
  - 구조가 같아야만 동일한 것도 아니고, B가 A라고 해서 A도 B인 것도 아니다
- 배열에 매핑된 객체 타입

  ```tsx
  type Arr = number[];
  type CopyArr = {
    [Key in keyof Arr]: Arr[Key];
  };

  const copyArr: CopyArr = [1, 3, 9];
  ```

  - `CopyArr` 타입에 존재하는 모든 속성을 숫자 배열이 가지고 있으므로 구조적으로 동일하다

  ```tsx
  type SimpleArr = { [key: number]: number; length: number };
  const simpleArr: SimpleArr = [1, 2, 3];
  ```

  - 숫자 배열은 `SimpleArr` 객체 타입에 있는 모든 속성을 가지고 있다 ⇒ 왜?
  - 숫자 배열은 구조적으로 `SimpleArr` 이라고 볼 수 있기 때문에 대입할 수 있다

- 구조적으로 동일하지 않으면 서로 대입할 수 없다

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

  const liter: Liter = { amount: 1, unit: 'liter', __type: 'liter' };
  const circle: Money = liter;
  // Type 'Liter' is not assignable to type 'Money'.
  // Types of property '__type' are incompatible.
  // Type '"liter"' is not assignable to type '"money"'.
  ```

  - 객체를 구별할 수 있는 속성을 추가하여 서로 대입되지 않도록 할 수 있다
  - `__type` 같은 속성을 브랜드 속성이라고 하고
  - 브랜드 속성을 사용하는 것을 브랜딩한다고 표현한다

### 2.14 제네릭으로 타입을 함수처럼 사용하자

- 제네릭 표기는 `<>` 로 하고 인터페이스 이름 바로 뒤에 위치한다
- 제네릭을 사용해서 중복 제거하기

  ```tsx
  interface Person<N, A> {
    type: 'human';
    race: 'yellow';
    name: N;
    age: A;
  }

  interface Zero extends Person<'zero', 28> {}
  interface Nero extends Person<'nero', 32> {}
  ```

  - `<>` 안에 들어가는 N과 A가 각각 `name`, `age` 에 대입된다

- 인터페이스, 클래스, 타입 별칭, 함수에도 적용 가능

  ```tsx
  type Person<N, A> = {
    type: 'human';
    race: 'yellow';
    name: N;
    age: A;
  };

  type Zero = Person<'zero', 28>;
  type Nero = Person<'nero', 32>;
  ```

  ```tsx
  class Person<N, A> {
    name: N;
    age: A;
    constructor(name: N, age: A) {
      this.name = name;
      this.age = age;
    }
  }

  type Zero = Person<'zero', 28>;
  type Nero = Person<'nero', 32>;
  ```

  ```tsx
  const personFactoryE = <N, A>(name: N, age: A) => ({
    type: 'human',
    race: 'yellow',
    name,
    age,
  });

  function personFactoryD<N, A>(name: N, age: A) {
    return {
      type: 'human',
      race: 'yellow',
      name,
      age,
    };
  }
  ```

  🌟 함수는 함수 선언문과 함수 표현식의 제네릭 표기 위치가 다름

- Array도 제네릭 타입
  - `Array<number>`
  - `Array` 타입은 다음과 같은 꼴로 선언되어 있다
    ```tsx
    interface Array<T> {
      [key: number]: T;
      length: number;
    }
    ```
    **→ 제네릭으로 하나의 타입을 여러 방법으로 재사용할 수 있다**
- `interface` 와 `type` 간 교차 사용도 가능

  ```tsx
  interface IPerson<N, A> {
    type: 'human';
    race: 'yellow';
    name: N;
    age: A;
  }

  type TPerson<N, A> = {
    type: 'human';
    race: 'yellow';
    name: N;
    age: A;
  };

  type Zero = IPerson<'zero', 28>;
  interface Nero extends TPerson<'nero', 32> {}
  ```

- 객체나 클래스의 메서드에 제네릭을 표기할 수도 있다

  ```tsx
  class Person<N, A> {
    name: N;
    age: A;
    constructor(name: N, age: A) {
      this.name = name;
      this.age = age;
    }
    method<B>(param: B) {}
  }

  interface IPerson<N, A> {
    type: 'human';
    race: 'yellow';
    name: N;
    age: A;
    method: <B>(param: B) => void;
  }
  ```

- 타입 매개변수에 default 값을 지정할 수 있다

  ```tsx
  interface Person<N = string, A = number> {
    type: 'human';
    race: 'yellow';
    name: N;
    age: A;
  }

  type Person1 = Person;
  type Person2 = Person<number>;
  type Person3 = Person<number, boolean>;
  ```

  - 타입 인수를 옵셔널처럼 사용할 수 있고, 지정하지 않은 타입은 기본값 타입이 됨

- 제네릭도 추론을 통해 타입을 알아낼 수 있다

  ```tsx
  interface Person<N, A> {
    type: 'human';
    race: 'yellow';
    name: N;
    age: A;
  }

  const personFactoryE = <N, A = unknown>(name: N, age: A): Person<N, A> => ({
    type: 'human',
    race: 'yellow',
    name,
    age,
  });

  const zero = personFactoryE('zero', 28);
  ```

  - ‘zero’는 `string`, 28은 `number` 타입으로 추론된다
  - zero의 타입은 `Person<string, number>`
  - 추론을 통해 타입을 알아낼 수 있는 경우 직접 타입을 넣지 않는 경우가 많다

- 배열을 튜플로 타입 추론되게 하고 싶은 경우

  - v4.9 이하

  ```tsx
  function values<T>(initial: readonly T[]) {
    return {
      hasValue(value: T) {
        return initial.includes(value);
      },
    };
  }

  const savedValues = values(['a', 'b', 'c'] as const);
  ```

  - v5.0 이상
    - 타입 매개변수 앞에 `const` 수식어를 추가

  ```tsx
  function values<const T>(initial: T[]) {
    return {
      hasValue(value: T) {
        return initial.includes(value);
      },
    };
  }

  const savedValues = values(['a', 'b', 'c']);
  ```

- 제네릭에 `extends` 키워드로 제약 걸기

  ```tsx
  interface Example<A extends number, B = string> {
    a: A;
    b: B;
  }

  type UseCase1 = Example<string, boolean>;
  type UseCase2 = Example<1, boolean>;
  type Usecase3 = Example<number>;
  ```

  - 타입 매개변수 A는 number 타입이어야 한다는 의미
  - `UseCase1` 은 A 타입에 string 타입을 대입할 수 없으므로 에러 발생
  - `UseCase2` 처럼 더 구체적인 타입은 대입 가능
    📌 다음과 같이 하나의 타입 매개변수를 다른 타입 매개변수의 제약으로 사용할 수도 있다

  ```tsx
  interface Example<A, B extend A> {
  	a: A,
  	b: B,
  }
  ```

  ```tsx
  <T extends object> // 모든 객체
  <T extends any[]> // 모든 배열
  <T extends (...args: any) => any> // 모든 함수
  <T extends abstract new (...args: any) => any> // 생성자타입
  <T extends keyof any> // string | number | symbol
  ```

- 타입 매개변수와 제약을 동일하게 생각하지 않기

  ```tsx
  interface V0 {
    value: any;
  }

  const returnV0 = <T extends V0>(): T => {
    return { value: 'test' };
  };
  ```

  - T는 V0에 대입할 수 있는 모든 타입을 의미하므로 에러가 발생함
    - `{ value: string, another: string}` 도 대입할 수 있기 때문
    - 다음과 같이 해결 가능
      ```tsx
      const returnV0 = (): V0 => {
        return { value: 'test' };
      };
      ```
      왜 에러가 난다는 거지

  ```tsx
  function onlyBoolean<T extends boolean>(arg: T = false): T {}
  ```

  - false 기본값 지정에서 에러가 발생하는데
    - `never` 를 모든 타입에 대입할 수 있다
    - 따라서 T가 `never` 타입일 수도 있기 때문에 false 기본값이 불가능하다
  - 다음과 같이 제네릭 쓰지 않고 해결 가능
    ```tsx
    function onlyBoolean(arg: true | false = false): T {}
    ```

### 2.15 조건문과 비슷한 컨디셔널 타입이 있다

- 조건에 따라 타입을 다르게 지정할 수 있다
  ```tsx
  type A1 = string;
  type B1 = A1 extends string ? number : boolean;
  ```
- 타입 검사를 위해 사용하기
  ```tsx
  type Result = 'hi' extends string ? true : false;
  type Result = [1] extends [string] ? true : false;
  ```
  ```tsx
  type ChooseArray<A> = A extends string ? string[] : never;
  type StringArray = ChooseArray<string>;
  type Never = ChooseArray<number>;
  ```
  - 제네릭과 `never` 같이 사용하여 필터링 조건에 부합한 경우만 사용하고자 하는 타입 지정하기
- 컨디셔널 타입과 함께 사용하기

  ```tsx
  type OmitByType<O, T> = {
    [K in keyof O as O[K] extends T ? never : K]: O[K];
  };

  type Result = OmitByType<
    {
      name: string;
      age: number;
      married: boolean;
      rich: boolean;
    },
    boolean
  >;

  /*
  type Result = {
  	name: string;
  	age: number;
  }
  */
  ```

  - boolean인 속성을 제거하는 예시

- 검사하려는 타입이 제네릭이면서 유니언이면 분배법칙이 실행된다
  ```tsx
  type Start = string | number;
  type Result<Key> = Key extends string ? Key[] : never;
  let n: Result<Start> = ['hi'];
  ```
  - `Result<string | number>` → `Result<string> | Result<number>` → `string extends string ? string[] : never |  number extends string ? number[] : never`→ `string[] | never`→ `string[]`
  - 단, `boolean` 에 분배 법칙이 적용될 때에는 `string[] | boolean[]` 같은 형태가 아닌 `string[] | true[] | false[]` 가 됨
- 분배 법칙 막기
  ```tsx
  type IsString<T> = T extends string ? true : false;
  type Result = IsString<'hi', 3>;
  ```
  - Result 타입이 `true | false` 이기 때문에 `boolean` 타입이 됨
  - 다음과 같이 분배 법칙을 막아 해결하자
    ```tsx
    type IsString<T> = [T] extends [string] ? true : false;
    type Result = IsString<'hi', 3>;
    ```
    - 배열로 제네릭을 감싸면 분배 법칙이 일어나지 않는다
- `never` 타입도 분배 법칙의 대상이 된다
  - `never` 는 공집합과 같으므로 공집합에서 분배 법칙을 실행하는 것은 아무것도 실행하지 않는 것과 같다
    ```tsx
    type R<T> = T extends string ? true : false;
    type RR = R<never>;
    // type RR = never
    ```
  - 컨디셔널 타입에서 제네릭과 `never`가 만나면 `never` 가 된다
  - `never` 사용할 때 분배 법칙 막기
    ```tsx
    type IsNever<T> = [T] extends [never] ? true : false;
    type T = IsNever<never>;
    // type T = true
    type F = IsNever<string>;
    // type F = false
    ```
- 제네릭이 들어 있는 컨디셔널 타입은 값의 판단이 미루어진다

  ```tsx
  function test<T>(a: T) {
  	type R<T> = T extneds string ? T : T;
  	const b: R<T> = a;
  }

  // Type 'T' is not assignable to type 'R<T>'
  ```

  - 변수 b에 a를 대입할 때는 R<T> 타입을 모르는 상태이다
  - 해결

    ```tsx
    function test<T extends ([T] extends [string] ? string : never)>(a: T) {
    	type R<T> = [T] extneds [string] ? T : T;
    	const b: R<T> = a;
    }

    // Type 'T' is not assignable to type 'R<T>'
    ```

    - 타입 매개변수를 선언할 때 `[T] extneds [string]` 하는 것이 불가능하다?
      - 그래서 컨디셔널 타입으로 선언
