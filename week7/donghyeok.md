# 7주차 스터디 내용 정리

> #### info::출처
>
> 이 내용은 [타입스크립트 교과서](https://product.kyobobook.co.kr/detail/S000208416779) 책을 읽고 스터디한 내용을 바탕으로 정리했습니다.

<!-- TOC -->
- [7주차 스터디 내용 정리](#7주차-스터디-내용-정리)
- [Express.js](#expressjs)
  - [9.1 req, res, next 타입 분석 및 타이핑하기](#91-req-res-next-타입-분석-및-타이핑하기)
  - [9.2 Express 직접 타이핑하기](#92-express-직접-타이핑하기)
  - [제네릭 추가 학습](#제네릭-추가-학습)


<!-- TOC -->

<br />

# Express.js

> 해당 교재에서는 Express.js Version 4.17을 사용하고 있음을 알립니다.
> 

- test.ts

```typescript
import cookieParser from 'cookie-parser';
import express, { RequestHandler, ErrorRequestHandler } from 'express';
import session from 'express-session';
import passport from 'passport';
import flash from 'connect-flash';

const app = express();

app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use('/', express.static('./public'));
app.use(cookieParser('SECRET'));
app.use(session({
  secret: 'SECRET',
}));
app.use(passport.initialize());
app.use(passport.session());
app.use(flash());

// 미들웨어는 RequestHandler 타입이다.
const middleware: RequestHandler = (req, res, next) => {
  req.params.paramType;
  req.body.bodyType;
  req.query.queryType;
  res.locals.localType;
  res.json({
    message: 'hello',
  });

  req.flash('플래시메시지');
  req.flash('1회성', '플래시메시지');
  req.flash();

  req.session;
  req.user?.zerocho;
};
app.get('/', middleware);

const errorMiddleware: ErrorRequestHandler = (err, req, res, next) => {
  console.log(err.status);
};
app.use(errorMiddleware);

app.listen(8080, () => {
    console.log('8080 포트에서 서버 실행 중');
});
```

express를 TypeScript로 install 하고 난 후 cookie-parser, express-session, passport, connect-flash를 설치해줍니다.

주의해야할 점은 그냥 설치할 시 js 버전으로 설치가 되기 때문에 잘 알아보고 @types가 붙은 타입스크립트 버전으로 설치하여야합니다.

여기서 req.user?zerocho 부분은 오류가 계속 발생합니다.

우선적으로 express()의 해당 타입 정의로 이동해보겠습니다.

```typescript
// node_module/@types.express.index.d.ts

...
declare function e(): core.Express;

delcar namespace e {
	...
}

export = e;
```

function e()로 이동하게됩니다.

그래서 express()로 호출할 수 있었던 것입니다.

express.json()의 경우 코드를 살펴보면 json은 namespace e 내부의 json이고, 이것은 다시 bodyParser 인터페이스의 json입니다.

express 함수 자체는 declare function으로 선언되어 있고, express의 속성과 메서드들은 delcare namespace e로 선언되어있음을 확인할 수 있습니다.

app.use의 타입은 상세하게 적지 않겠습니다.

간단하게 설명하자면 express()의 반환 값인 app의 타입은 `core.Express` 타입이었습니다.

Express의 타입 정의로 이동하면

```typescript
export interface Express extends Application {
	request: Request;
	response: Response;
}
```

다음과 같이 되어있는데 Application을 보면 http 모듈을 사용한 httpRequest, httpResponse의 함수들을 생성하고 해당 콜백들을 부르는 코드들이 작성되어있습니다.

이번 백엔드 두 번째 프로젝트가 http 모듈도 사용하지 않고 net 모듈로 완전 기반 없이 WAS를 구현해보았는데 

net을 통해 만드는 것이 아닌 기본적으로 제공되는 http 모듈로 요청, 응답에 의한 코드들이 생성되어있습니다.

이 코드들로 사용자들이 req, res를 바로바로 사용할 수 있게됩니다.

해당 Application에는 use, listen 메서들을 모두 찾을 수 있습니다.

이를 타고 들어가면 `ApplicationRequestHandler`가 있고 그 안에 `IRouterHandler, IrouterMatcher, ((…handlers: RequestHandlerParams[]) ⇒ T`로 구성되어있습니다.

IRouterHandler 코드에 대한 설명은 생략합니다. 구조적인 코드이기 때문에 이해할 필요는 없다고 판단하였습니다.

IRouterHandler 안을 타고 들어가면 다음과 같은 점을 확인할 수 있습니다.

app.use는 `ApplicationRequestHandler`이고, `ApplicationRequestHandler`는 `IRouterHandler`라는 것을 알 수 있습니다.

app.use에는 `RequestHandler`나 `RequestHandler Params`가 들어갈 수 있습니다.

`RequestHandler`는 `(req, res, next) ⇒ {}` 꼴의 함수입니다.
이것이 익스프레스 미들웨어의 전형적인 형태이고, 익스프레스 미들웨어 타입이 `RequestHandler`임을 알 수 있습니다.

<br />

## 9.1 req, res, next 타입 분석 및 타이핑하기

미들웨어의 구성요소인 req, res, next는 각각 Request, Response, NextFunction 타입입니다.

내부 타입을 node_moduels/@types/express-serve-static-core/index.d.ts에서 분석합니다.
여기에서 Request, Response, NextFunction에 대한 타입 정의를 모두 찾을 수 있습니다.

코드 정의가 굉장히 복잡하고 해당 프레임워크에 대한 구조적인 코드이므로 까보고 싶은 경우 직접 분석해보면 좋을 것 같습니다.

현재 위의 test.ts에서 미들웨어 부분을 보겠습니다.

```typescript
// 미들웨어는 RequestHandler 타입이다.
const middleware: RequestHandler = (req, res, next) => {
  req.params.paramType;
  req.body.bodyType; // (property) bodyType: any
  req.query.queryType;
  res.locals.localType; // (property) localType: any
  res.json({
    message: 'hello',
  });

  req.flash('플래시메시지');
  req.flash('1회성', '플래시메시지');
  req.flash();

  req.session;
  req.user?.zerocho; // Property 'zerocho' does not exist on type 'User'.
};
```

주석으로 표시해놓았듯이 bodyType, localType이 any 타입으로 추론되는 것을 확인할 수 있습니다.

req 객체 안에 데이터를 넣을 수 있는 공간은 기본적으로 세 가지 입니다.
req.params, req.body, req.query입니다.

req.flash, req.session이나 req.user는 Express가 아니라 각각 설치한 모듈에서 추가된 객체입니다.

res 객체 안에 데이터를 넣을 수 있는 공간은 res.locals입니다.
클라이언트로 응답을 보낼 데이터도 res.send, res.json 같은 res 객체의 메서드를 통해 보내는데 이때 보내는 데이터도 타이핑할 수 있습니다.

```typescript
// 미들웨어는 RequestHandler 타입이다.
const middleware: RequestHandler = (req: Request<{ paramType: string }, { message: string }, { bodyType: symbol }, 
{ queryType: boolean }, { localType: number }>, 
res: Response<{ message: string }, { localType: number }>, 
next: NextFunction) => {
  req.params.paramType; // (property) paramType: string
  req.body.bodyType; // (property) bodyType: symbol
  req.query.queryType; // (property) queryType: boolean
  res.locals.localType; // (property) localType:: number
  res.json({
    message: 'hello',
  });

  req.flash('플래시메시지');
  req.flash('1회성', '플래시메시지');
  req.flash();

  req.session;
  req.user?.zerocho; // Property 'zerocho' does not exist on type 'User'.
};
```

이렇게 각각 타이핑을 하면 any가 아닌 타입으로 직접 매핑이 가능합니다.

하지만 저렇게 적는 것보다 RequestHandler를 활용하는 것이 더 좋은 방법입니다.

```typescript
export interface RequestHandler<
	P = ParamsDictionary, 
	ResBody = any, 
	ReqBody = any, 
	ReqQuery = ParsedQs, 
	LocalsObj extends Record<string, any> = Record<string, any> {
    ..
}

```

```typescript
// 미들웨어는 RequestHandler 타입이다.
const middleware: RequestHandler<{ paramType: string }, { message: string }, { bodyType: symbol }, 
{ queryType: boolean }, { localType: number }>
= (req, res, next) => {
  req.params.paramType; // (property) paramType: string
  req.body.bodyType; // (property) bodyType: symbol
  req.query.queryType; // (property) queryType: boolean
  res.locals.localType; // (property) localType:: number
  res.json({
    message: 'hello',
  });

  req.flash('플래시메시지');
  req.flash('1회성', '플래시메시지');
  req.flash();

  req.session;
  req.user?.zerocho; // Property 'zerocho' does not exist on type 'User'.
};
```

이정도까지만 알아보겠습니다.

내부 express 프레임워크를 뜯는 구조이다보니 상당히 이해하기 버겁고 복잡한 코드들인 탓에 
훑고 넘어가는 정도로 우선적으로 마무리하겠습니다.

- Express 타입 정의
    
    ```typescript
    // Type definitions for Express 4.17
    // Project: http://expressjs.com
    // Definitions by: Boris Yankov <https://github.com/borisyankov>
    //                 China Medical University Hospital <https://github.com/CMUH>
    //                 Puneet Arora <https://github.com/puneetar>
    //                 Dylan Frankland <https://github.com/dfrankland>
    // Definitions: https://github.com/DefinitelyTyped/DefinitelyTyped
    
    /* =================== USAGE ===================
    
        import express = require("express");
        var app = express();
    
     =============================================== */
    
    /// <reference types="express-serve-static-core" />
    /// <reference types="serve-static" />
    
    import * as bodyParser from 'body-parser';
    import * as serveStatic from 'serve-static';
    import * as core from 'express-serve-static-core';
    import * as qs from 'qs';
    
    /**
     * Creates an Express application. The express() function is a top-level function exported by the express module.
     */
    declare function e(): core.Express;
    
    declare namespace e {
        /**
         * This is a built-in middleware function in Express. It parses incoming requests with JSON payloads and is based on body-parser.
         * @since 4.16.0
         */
        var json: typeof bodyParser.json;
    
        /**
         * This is a built-in middleware function in Express. It parses incoming requests with Buffer payloads and is based on body-parser.
         * @since 4.17.0
         */
        var raw: typeof bodyParser.raw;
    
        /**
         * This is a built-in middleware function in Express. It parses incoming requests with text payloads and is based on body-parser.
         * @since 4.17.0
         */
        var text: typeof bodyParser.text;
    
        /**
         * These are the exposed prototypes.
         */
        var application: Application;
        var request: Request;
        var response: Response;
    
        /**
         * This is a built-in middleware function in Express. It serves static files and is based on serve-static.
         */
        var static: serveStatic.RequestHandlerConstructor<Response>;
    
        /**
         * This is a built-in middleware function in Express. It parses incoming requests with urlencoded payloads and is based on body-parser.
         * @since 4.16.0
         */
        var urlencoded: typeof bodyParser.urlencoded;
    
        /**
         * This is a built-in middleware function in Express. It parses incoming request query parameters.
         */
        export function query(options: qs.IParseOptions | typeof qs.parse): Handler;
    
        export function Router(options?: RouterOptions): core.Router;
    
        interface RouterOptions {
            /**
             * Enable case sensitivity.
             */
            caseSensitive?: boolean | undefined;
    
            /**
             * Preserve the req.params values from the parent router.
             * If the parent and the child have conflicting param names, the child’s value take precedence.
             *
             * @default false
             * @since 4.5.0
             */
            mergeParams?: boolean | undefined;
    
            /**
             * Enable strict routing.
             */
            strict?: boolean | undefined;
        }
    
        interface Application extends core.Application {}
        interface CookieOptions extends core.CookieOptions {}
        interface Errback extends core.Errback {}
        interface ErrorRequestHandler<
            P = core.ParamsDictionary,
            ResBody = any,
            ReqBody = any,
            ReqQuery = core.Query,
            Locals extends Record<string, any> = Record<string, any>
        > extends core.ErrorRequestHandler<P, ResBody, ReqBody, ReqQuery, Locals> {}
        interface Express extends core.Express {}
        interface Handler extends core.Handler {}
        interface IRoute extends core.IRoute {}
        interface IRouter extends core.IRouter {}
        interface IRouterHandler<T> extends core.IRouterHandler<T> {}
        interface IRouterMatcher<T> extends core.IRouterMatcher<T> {}
        interface MediaType extends core.MediaType {}
        interface NextFunction extends core.NextFunction {}
        interface Locals extends core.Locals {}
        interface Request<
            P = core.ParamsDictionary,
            ResBody = any,
            ReqBody = any,
            ReqQuery = core.Query,
            Locals extends Record<string, any> = Record<string, any>
        > extends core.Request<P, ResBody, ReqBody, ReqQuery, Locals> {}
        interface RequestHandler<
            P = core.ParamsDictionary,
            ResBody = any,
            ReqBody = any,
            ReqQuery = core.Query,
            Locals extends Record<string, any> = Record<string, any>
        > extends core.RequestHandler<P, ResBody, ReqBody, ReqQuery, Locals> {}
        interface RequestParamHandler extends core.RequestParamHandler {}
        interface Response<
            ResBody = any,
            Locals extends Record<string, any> = Record<string, any>
        > extends core.Response<ResBody, Locals> {}
        interface Router extends core.Router {}
        interface Send extends core.Send {}
    }
    
    export = e;
    ```
    

<br />

## 9.2 Express 직접 타이핑하기

Pass

<br />

## 제네릭 추가 학습

이정도만 하기는 그래서 제네릭에 대해서 조금 더 학습했습니다.

제네릭을 정작 잘 사용하지 않아서 사용하는 경우에 대해서 예시를 들어보겠습니다.

우리는 add라는 함수를 만들어서 x + y를 작성하려고 합니다.

```typescript
function add(x, y) {
	return x + y;
}
```

다음과 같은 add 함수에 타입을 넣어보려고 합니다.

여기서 파라미터는 number가 아닌 string 값이 오는 경우도 있습니다.

```typescript
function add(x: string | number, y: string | number): string | number {
	return x + y;
}
```

다음과 같이 string 혹은 number가 올 수 있도록 작성합니다.

하지만 이의 경우 string + number 꼴이 들어올 수 있어 TS 자체에서 에러를 반환합니다.

여기서 방법이 두 가지가 있습니다.

1. 함수 오버로딩

```typescript
function add(x: string, y: string): string;
function add(x: number, y: number): number;
function add(x: any, y: any) {
	return x + y;
}
```

다음과 같이 작성할 수 있습니다.

그런데 여기서 이런 string, number뿐 아니라 타입이 굉장히 많아진다면 가독성이 굉장히 나빠질 것입니다.

이럴때 사용할 수 있는 것이 바로 제네릭입니다.

```typescript
function add<T>(x: T, y: T): T {
	return x + y;
}

add<number>(1, 2); // 3
add<string>('1', '2'); // 12
```

그런데 사실 코드를 작성할 때 호출하는 부분에서 제네릭으로 타입을 지정하면서 사용해본적은 없습니다.

사실 컴파일러는 전달하는 인수의 타입을 보고 스스로 추론하기 때문에 함수 호출할때 제네릭을 안써줘도 알아서 추론합니다.
하지만 가끔 추론을 잘못하는 경우에는 위와 같이 해주면 해결할 수 있습니다.

<aside>
✨ 꺾쇠 <> 기호를 변수명, 함수명 앞에다 쓰면 '타입 단언' 이 됩니다.

따라서 제네릭을 구현하려면 변수명, 함수명 **뒤에다가** 꺾쇠 괄호를 적어주어야 합니다.

그리고 제네릭명은 꼭 T가 아니어도 됩니다.

내마음대로 붙일수는 있지만 관습적으로 대문자 알파벳 한글자로 처리하는 편이다. (T, U, K ...)

</aside>

가장 많이 사용하는 예시는 아무래도 DTO(인터페이스, 클래스)와 가장 많이 사용할 것입니다.

```typescript
// 제네릭 인터페이스
interface Mobile<T> { 
   name: string;
   price: number;
   option: T; // 제네릭 타입 - option 속성에는 다양한 데이터 자료가 들어온다고 가정
}

// 제네릭 자체에 리터럴 객테 타입도 할당 할 수 있다.
const m1: Mobile<{ color: string; coupon: boolean }> = {
   name: 's21',
   price: 1000,
   option: { color: 'read', coupon: false }, // 제네릭 타입의 의해서 option 속성이 유연하게 타입이 할당됨
};

const m2: Mobile<string> = {
   name: 's20',
   price: 900,
   option: 'good', // 제네릭 타입의 의해서 option 속성이 유연하게 타입이 할당됨
};
```

참고: [https://inpa.tistory.com/entry/TS-📘-타입스크립트-Generic-타입-정복하기](https://inpa.tistory.com/entry/TS-%F0%9F%93%98-%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-Generic-%ED%83%80%EC%9E%85-%EC%A0%95%EB%B3%B5%ED%95%98%EA%B8%B0)