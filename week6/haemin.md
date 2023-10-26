## 유틸리티 타입

```tsx
type MyReadonly<T> = {
  readonly [S in keyof T]: T[S]
}
```

```tsx
type MyPick<T, K extends keyof T> = {
  [S in K]: T[S]
}
```

```tsx
type MyExclude<T, U> = T extends U ? never : T;
```

```tsx
type MyOmit<T, K extends keyof T> = {
  [S in keyof T as S extends K ? never : S] : T[S]
}
```

```tsx
type MyParameters<T extends (...args: any[]) => any> = T extends (...args:infer A)=>any? A: never
```

```tsx
type MyReturnType<T> = T extends (...args:any)=> infer R ? R:never
```

```tsx
type Intersection<T> = {
  [S in keyof T]: T[S]
}

type PartialByKeys<T, K extends keyof T = keyof T> = Intersection<{
  [S in keyof T as S extends K ? S : never]?:T[S]
} &{
  [S in keyof T as S extends K ? never: S] : T[S]
}>
```

```tsx
type InterSection<T>={
  [S in keyof T]: T[S]
}

type RequiredByKeys<T, K extends keyof T = keyof T> = InterSection<{
  [S in keyof T as S extends K ? S:never]-?:T[S]
}&{
  [S in keyof T as S extends K ? never : S]:T[S]
}>
```

## 고차함수 분석하기

```tsx
interface Array<T>{
  myForEach(cb:(v:T,i:number,a:T[])=>void):void,
  myMap<U>(cb:(v:T,i:number,a:T[])=>U):U[],
  myFilter<S extends T>(cb:(v:T,i:number,a:T[])=> boolean):S[];
  // 틀림
  myReducer<S>(cb:(a:S,c:T,i:number,arr:T[])=>S,init?:S):S;

}
```

## 오답노트

```tsx
  myFilter:<S extends T>(cb:(v:T,i:number,a:T[])=> unknown)=>S[],
```

callback의 return이 flasly가 아니면 해당 T를 return하므로 unkown 이라고 생각하였습니다. 하지만 하나 빼먹은것이 있었는데요.

바로 타입 좁히기를 사용할 시 입니다.

```tsx
const result = [1,"2",3,"4"].myFilter((v,i,a): v is string =>{return typeof v ==="string"});
```

다음과 같은 타입을 고려할 때 result의 값으로는 string이 와야 합니다. 제가 작성한 코드에서는 이를 추론하지 않고 있습니다.

```tsx
// 추가
myFilter:<S extends T>(cb:(v:T,i:number,a:T[])=> v is S)=>S[],
```

다음 타입도 추가해 주어야 합니다.

이렇게 하면 완성이지만 불필요한 타입을 하나 줄일 수 있습니다.

```tsx
myFilter:<S extends T>(cb:(v:T,i:number,a:T[])=> unknown)=>S[],
```

이 타입에서 S 제네릭을 없앨 수 있는데요. 이 경우에는 filter가 타입을 바꾸는 것이 아닌 속성의 개수만 변하기 때문에 기존 T타입과 똑같습니다.

결론

```tsx
myFilter:<S extends T>(cb:(v:T,i:number,a:T[])=> v is S)=>S[],
myFilter:(cb:(v:T,i:number,a:T[])=> unknown)=>T[],
```

## 3.10 Promise Await 분석하기

promise는 js의 구현체이기 때문에 타입만 따로 delcare를 이용하여 정의하면 됩니다.

```tsx
interface PromiseConstructor {
    readonly prototype: Promise<any>;
    new <T>(executor: (resolve: (value: T | PromiseLike<T>) => void, reject: (reason?: any) => void) => void): Promise<T>;
    all<T extends readonly unknown[] | []>(values: T): Promise<{ -readonly [P in keyof T]: Awaited<T[P]> }>;
    race<T extends readonly unknown[] | []>(values: T): Promise<Awaited<T[number]>>;
    reject<T = never>(reason?: any): Promise<T>;
    resolve(): Promise<void>;
    resolve<T>(value: T): Promise<Awaited<T>>;
    resolve<T>(value: T | PromiseLike<T>): Promise<Awaited<T>>;
}

declare var Promise: PromiseConstructor;
```

여기서 Awaited라는 타입을 볼 수 있는데요 

Promise의 return 타입을 꺼내오기 위해서 존재합니다.

```tsx
type Awaited<T> = T extends null | undefined ? T : // special case for `null | undefined` when not in `--strictNullChecks` mode
    T extends object & { then(onfulfilled: infer F, ...args: infer _): any; } ? // `await` only unwraps object types with a callable `then`. Non-object types are not unwrapped
        F extends ((value: infer V, ...args: infer _) => any) ? // if the argument to `then` is callable, extracts the first argument
            Awaited<V> : // recursively unwrap the value
        never : // the argument to `then` was not callable
    T; // non-object or non-thenable
```

하나씩 살펴 보겠습니다. 먼저 첫줄은 null과 undefined면 그 타입을 그대로 return해줍니다.

```tsx
T extends object & { then(onfulfilled: infer F, ...args: infer _): any; }
```

다음을 보겠습니다. 다음 타입은 Promise 인스턴스 타입을 extends하는가에 대한 여부입니다.

Promise 인스턴스 타입과 Promise 객체는 다릅니다. Promise 인스턴스 타입은 new Promise()나 Promise.resolve()의 반환값을 의미합니다.

즉 이 인스턴스 타입에는 then이라는 매서드가 있기 때문에 해당 타입을 만족합니다.

```tsx
F extends ((value: infer V, ...args: infer _) => any)
```

다음은 then의 onfulfilled로 추론된 F타입에 대한 조건문인데요. 해당 F는 다음과 같이 callback을 받고 있는데 저희가 봐야 할것은 이 callback의 타입 자체에 대해서 접근을하고 return을 해주어야 하기 떄문에 한단계 더 infer를 작성한 것입니다.

```tsx
Promise.resolve('hi').then((data)=>{data})
```

따라서 Await 내부에 Promise가 들어가면 Await<T>가 되는걸 알 수 있습니다.