## 2.29 배운 것 바탕으로 타입을 만들어 보자

- 타입스크립트로 컨디셔널 타입으로 원하는 타입을 제거하거나 추리기 위해 특정 타입이 무슨 타입인지 판단하는 능력 기르기

### IsNever

```tsx
type IsNever<T> = [T] extends [never] ? true : false;
```

- 배열로 감싸는 이유: 분배 법칙 방지
    
    [배열로 감싸지 않았을 경우 다른 예시](https://www.typescriptlang.org/play?#code/C4TwDgpgBAkgzgOQgNwgJwIwB4AqA+KAXihyggA9gIA7AEzimpXSgH4pg0BXaALigBmAQwA2cCAG4AUKEixEzNACZcBYgG0cAXTKUa9KOqao0O9px5R+wsZKkzw0JCZwQ4wDEXnP02Y+jwHOR80V3clL3gQlX80PCA)
    

### IsAny

```tsx
type IsAny2<T> = string extends (number & T) ? true : false;
```

- string과 number는 겹치지 않아서 extends 할 수 없는데
- any 타입은 모든 타입의 상위 호환으로 취급되어 어떤 타입과 `&` (교차 타입) 연산을 하더라도 결과는 항상 `any`가 된다
- 따라서 T가 `any` 일 경우에만 `string extends (number & T)` 조건을 만족하게 된다
- 더 보기
    
    ```tsx
    type IsAny<T> = 0 extends (1 & T) ? true : false;
    ```
    

### IsArray

다음과 작성할 경우에는 세 가지 경우의 반례를 들 수 있음

```tsx
type IsArray<T> = T extends any[] ? true : false;

type Test1 = IsArray<never>; // never
type Test2 = IsArray<any>; // boolean
type Test3 = IsArray<readonly string[]>; // false
```

- 따라서 세 가지의 반례를 통과할 수 있도록 타입을 작성해야 함

```tsx
type IsArray<T> = IsNever<T> extends true ? false : IsAny<T> extends true ? false : T extends readonly unknown[] ? true: false;
```

### IsTuple

- 배열과 튜플의 차이점:
    - 튜플의 length 속성: (property) length: 3 ⇒ **길이 고정**
    - 배열의 length 속성: (property) Array<number>.length: number

IsTuple → number extends T["length"]가 false여야 함

그래서 IsArray 타입에서 `T extends readonly unknown[] ? number extends T['length'] ? false : true : false` 로 걸러 주면 되는데…… 

`any` 타입 값은 어떤 속성이든 가질 수 있다 → 그리고 그 속성의 값도 어떤 것이든 될 수 있다

따라서 `any["length"]`는 `any` → `number extends any`는 `true`라서 `number extends T["length"]` 에서 `any`는 걸러진다

### IsUnion

```tsx
type IsUnion<T, U = T> = IsNever<T> extends true
  ? false
  : T extends T
    ? [U] extends [T]
      ? false
      : true
    : false;
```

T extends T에서 T: `string | number`일 경우 분배 법칙이 적용되는 것을 응용함
`T extends T` ->  `string | number extends string | number` -> `(string extends string | number) | (number extends string | number)`

그러면 `(string extends string | number) | (number extends string | number)` 는 `[U] extends [T]` 로 넘어가게 되는데, 여기서 U는 U = T를 통해 원본 타입인 `string | number` 타입, T는 이전에서 분배 법칙이 일어난 `string` 과 `number` 타입이 해당된다.

따라서 `[U] extends [T]`에는 `string | number extends string` 와 `string | number extends number` 가 되어 false `[U] extends [T] ? false : true` 구문에서 true를 반환한다

→ `IsUnion<string|number> === true`

### Omit으로 차집합 만들기

- Omit: 특정 객체에서 지정한 속성을 제거하는 타입

```tsx
type Diff<A, B> = Omit<A & B, keyof B>;
type R1 = Diff<{ name: string, age: number }, { name: string, married: boolean }>;
```

A & B (A와 B의 합집합) → `{ name: string, age: number, , married: boolean }`에서 keyof B인 `{name, married}` 가 제거되어 `age` 속성만 남게 된다

### 대칭 차집합

```tsx
type SymDiff<A, B> = Omit<A & B, keyof (A | B)>;
type R2 = SymDiff<{ name: string, age: number }, { name: string, married: boolean }>
```

대칭 차집합은 합집합에서 교집합 타입의 속성만만 제거하면 되므로 Diff 속성을 수정하여 만들 수 있다

### Exclude로 유니언의 대칭 차집합 만들기

- Exclude: 어떤 타입에서 다른 타입을 제거하는 타입

유니언에서의 대칭 차집합은 `Exclude` 를 사용해야 한다

```tsx
type SymDiffUnion<A, B> = Exclude<A | B, A & B>;
type R3 = SymDiffUnion<1 | 2 | 3, 2 | 3 | 4>;
```

- 궁금한 점
    
    유니언과 객체의 대칭 차집합은 각기 다르게 사용해야 하는 건가? [링크](https://www.typescriptlang.org/play?#code/C4TwDgpgBAyiC2ARAlgM1QHgIIBooCEA+KAXigHl5lhsoAyAvAawhAHtUoAKLKAHwIBKQgG4AUKEiwEKdAFUAdsjYLseIqSgBRAB4BjADYBXACYRaA-Hl4Mi4ieGgAlAIya4SNJgDeUBQEN4CAAuKABnYAAnZAUAczx-WJC-I3gAIwhIqABfPF8AoNCI6Li8eH9I6IgTULS2NgMIfwUcwgcpJwAmdxkvRWVVfMDk4pj4qETkhVSMrNyoIcLwqLGyiqqaqDqGppbstvbnAGYez3klFQw3AW6BI7xbqBOBABZRIA)
    

### 부분 집합

A가 B 타입에 대입 가능하면 A는 B의 부분집합

```tsx
type IsSubset<A, B> = A extends B ? true : false;
type R1 = IsSubset<string, string | number>; // true
type R2 = IsSubset<{ name: string, age: number }, { name: string }>; // true
type R3 = IsSubset<symbol, unknown>;  // true
```

- 궁금한 점
    
    [분배 법칙 적용되지 않게 []로 감싸야 하는 것은 아닐까?](https://www.typescriptlang.org/play?#code/C4TwDgpgBAkgzgZQK4CM4WAHgIIBooBCAfFALxTZQQAewEAdgCZyFQD8UwATktAFxQAZgEMANugDcAKCmhIFMrESp0WONwCW9AOb51XLdqgAfKPSQBbFBC5Fpc6AUXxkaDJn2GTZy9a57NHTsZAHoQwkAAGqgUAHsY0QhhekAXccAfdqhAEN7ABh6oQFSewE055KhAQAnASrHADBbAAcmoQClRwBiaqABtbABdKloGZiaCNo5uXigBEXEIQB0OqEBE8cBGQcAXVcAfUahADVXAFKaoQGCawAFxqEARUcAFpsAZOrYpIA)
    

### Equal

```tsx
// any와 다른 타입을 구별하지 못하는 Eqaul 타입
type Equal<A, B> = [A] extends [B] ? [B] extends [A] ? true : false : false;

// 인터섹션을 인식하지 못하는 Equal 타입
type Equal2<X, Y>
  = (<T>() => T extends X ? 1 : 2) extends (<T>() => T extends Y ? 1 : 2)
    ? true
    : false
```

### NotEqual

```tsx
type NotEqual<X, Y> = Equal<X, Y> extends true ? false : true;
```

## 2.30 타입스크립트의 에러 코드로 검색하자

```tsx
const arr1: string[] = ['1', '2', '3'];
const arr2: Array<number> = [1, 2, 3];
arr1.push(4);
// Argument of type 'number' is not assignable to parameter of type 'string'. (2345)
```

- “TS + 에러 코드 뒤에 있는 숫자”로 에러 검색하기 (ex. TS2345)
- [타입스크립트의 에러 코드와 해결 방법을 정리한 사이트](https://typescript.tv/errors/)

## 2.31 함수에 기능을 추가하는 데코레이터 함수가 있다

- 데코레이터: 클래스의 기능을 증강하는 함수
    - 공통적으로 수행되는 부분을 데코레이터로 만들어 활용

**Before**

```tsx
class A {
  eat() {
    console.log("start");
    console.log("Eat");
    console.log("end");
  }

  work() {
    console.log("start");
    console.log("Work");
    console.log("end");
  }

  sleap() {
    console.log("start");
    console.log("Sleap");
    console.log("end");
  }
}
```

**After**

```tsx
function startAndEnd<This, Args extends any[], Return>(
  originalMethod: (this: This, ...args: Args) => Return,
  context: ClassMethodDecoratorContext<
    This,
    (this: This, ...args: Args) => Return
  >,
) {
  function replacementMethod(this: This, ...args: Args) {
    console.log("start");
    const result = originalMethod.call(this, ...args);
    console.log("end");
    return result;
  }
  return replacementMethod;
}

class A {
  @startAndEnd
  eat() {
    console.log("Eat");
  }

  @startAndEnd
  work() {
    console.log("Work");
  }

  @startAndEnd
  sleap() {
    console.log("Sleap");
  }
}
```

### 데코레이터 타입

- `ClassDecoratorContext`: 클래스 자체를 장식하는 데코레이터 타입
    - 클래스 데코레이터 예시
    
    ```tsx
    function log<Input extends new (...args: any[]) => any>(
      value: Input,
      context: ClassDecoratorContext,
    ) {
      if (context.kind === "class") {
        return class extends value {
          constructor(...args: any[]) {
            super(args);
          }
          log(msg: string): void {
            console.log(msg);
          }
        };
      }
      return value;
    }
    
    @log
    export class C {
     ...
    }
    ```
    
    - **첫 번째 매개변수**: 클래스 타입
    - **반환값**: 장식 대상 클래스를 상속한 클래스
    - export / export default 앞이나 뒤에 데코레이터를 붙일 수 있음
        
        ```tsx
        @Log export class C {}
        
        export @Log class C {}
        
        @Log 
        export class C {}
        ```
        
- `ClassMethodDecoratorContext`: 클래스 메서드를 장식하는 데코레이터 타입
- `ClassGetterDecoratorContext`: 클래스의 getter를 장식하는 데코레이터 타입
- `ClassSetterDecoratorContext`: 클래스의 setter를 장식하는 데코레이터 타입
- `ClassMemberDecoratorContext`: 클래스 멤버를 장식하는 데코레이터 타입
- `ClassAccessorDecoratorContext`: 클래스 accessor를 장식하는 데코레이터 타입
- `ClassFieldDecoratorContext`: 클래스 필드를 장식하는 데코레이터 타입

### Context 객체

```tsx
type Context = {
  kind: string;
  name: string | symbol;
  access: {
    get?(): unknown;
    set?(value: unknown): void;
    has?(value: unknown): boolean;
  };
  private?: boolean;
  static?: boolean;
  addInitializer?(initializer: () => void): void;
}
```

- `kind`: 데코레이터의 유형
- `name`: 장식 대상의 이름
- `access` : has, get, set 등 접근자를 모아 둔 객체
- `private` : private 여부
- `static`: static 여부
- `addInitializer`: 초기화할 때 실행되는 메서드
    - `addInitializer`에 등록한 함수는 클래스의 인스턴스를 생성할 때(=초기화할 때) 호출된다
        
        ```tsx
        
        function bound(
          originalMethod: unknown,
          context: ClassMethodDecoratorContext<any>,
        ) {
          const methodName = context.name;
          if (context.kind === "method") {
            context.addInitializer(function () {
              this[methodName] = this[methodName].bind(this);
            });
          }
        }
        
        export class C {
          @bound
          eat() {
            console.log("Eat");
          }
        }
        ```
        
        - new C()를 할 때 this.eat = this.eat.bind(this)가 호출된다

### 고차함수를 활용하여 매개변수 사용하기

```tsx
function startAndEnd(start = "start", end = "end") {
  return function RealDecorator<This, Args extends any[], Return>(
    originalMethod: (this: This, ...args: Args) => Return,
    context: ClassMethodDecoratorContext<
      This,
      (this: This, ...args: Args) => Return
    >,
  ) {
    function replacementMethod(this: This, ...args: Args) {
      console.log(context.name, start);
      const result = originalMethod.call(this, ...args);
      console.log(context.name, end);
      return result;
    }
    return replacementMethod;
  };
}

class A {
  @startAndEnd()
  eat() {
    console.log("Eat");
  }

  @startAndEnd()
  work() {
    console.log("Work");
  }

	@startAndEnd('시작', '끝')
  sleap() {
    console.log("Sleap");
  }
}

const a = new A();
a.eat();
// [LOG]: "eat",  "start" 
// [LOG]: "Eat" 
// [LOG]: "eat",  "end" 
a.sleap()
// [LOG]: "sleap",  "시작" 
// [LOG]: "Sleap" 
// [LOG]: "sleap",  "끝"
```

- `startAndEnd` 데코레이터로 인수를 받을 수 있게 작성

## 2.32 앰비언트 선언도 선언 병합이 된다

- `declare` 예약어로 **앰비언트 선언**을 ****사용하여 타입스크립트에서 자바스크립트 라이브러리를 사용할 때 활용할 수 있음
- 값이나 구현부가 없음에도 불구하고 컴파일 에러가 발생하지 않음
- 값이 없기 때문에 런타임 에러는 발생함 → 따라서 값이 실제로 존재하는지 확인해야 하고 사용하기

```tsx
declare namespace NS {
  const v: string;
};
declare enum Enum {
  ADMIN = 1
}
declare function func(param: number): string;
declare const variable: number;
declare class C {
  constructor(p1: string, p2: string);
};

new C(func(variable), NS.v); // 자바스크립트로 변환하면 이 코드만 남음
```

### 네임스페이스/타입/값 정리

| 유형 | 네임스페이스 | 타입 | 값 |
| --- | --- | --- | --- |
| 네임스페이스 | O |  | O |
| 클래스 |  | O | O |
| enum |  | O | O |
| 인터페이스 |  | O |  |
| 타입 별칭 |  | O |  |
| 함수 |  |  | O |
| 변수 |  |  | O |
|  |  |  |  |

### 같은 이름의 다른 선언과의 병합 가능 여부 정리

| 병합 가능 여부 | 네임스페이스 | 클래스 | enum | 인터페이스 | 타입 별칭 | 함수 | 변수 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 네임스페이스 | O | O | O | O | O | O | O |
| 클래스 | O | X | X | O | X | O | X |
| enum | O | X | O | X | X | X | X |
| 인터페이스 | O | O | X | O | X | O | O |
| 타입 별칭 | O | X | X | X | X | O | O |
| 함수 | O | O | X | O | O | O | X |
| 변수 | O | X | X | O | O | X | X |

**인터페이스 병합, 네임스페이스 병합, 함수 오버로딩**

### 선언 병합 활용 케이스

- 클래스 `new` 키워드 없이 사용하기
    
    ```tsx
    declare class A {
      constructor(name: string);
    }
    function A(name: string) {
      return new A(name);
    }
    
    new A('zerocho');
    A('zerocho');
    ```
    
    - 클래스와 함수를 같은 이름으로 병합할 수 있음
    - declare로 앰비언트 선언한 타입도 해당
- 함수에 속성 추가하여 사용하기
    
    ```tsx
    function Ex() { return 'hello'; }
    namespace Ex {
      export const a = 'world';
      export type B = number;
    }
    Ex(); // hello
    Ex.a; // world
    const b: Ex.B = 123;
    ```