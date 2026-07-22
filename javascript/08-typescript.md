# TypeScript

타입스크립트의 동작 원리와 리액트에서 자주 사용하는 개념을 정리한다.

---

## 타입스크립트란

기존 자바스크립트 문법에 **타입**을 가미한 것이다.

동적 언어인 자바스크립트에서 런타임에만 타입을 체크할 수 있는 한계를 극복해, 코드를 더 안전하게 작성하면서 잠재적인 버그도 줄일 수 있게 해준다.

자바스크립트는 대부분의 에러를 코드를 실행했을 때만 확인할 수 있다.

```js
const add = (a, b) => {
  if (typeof a !== "number" || typeof b !== "number") {
    throw new TypeError("두 인수는 모두 숫자여야 합니다");
  }

  return a + b;
};

console.log(add(2, 3)); // 출력: 5
console.log(add("2", 3)); // TypeError: 두 인수는 모두 숫자여야 합니다
```

물론 자바스크립트에서도 타입을 체크해 문제를 방지할 수 있다. 하지만 모든 함수와 변수에 `typeof` 같은 타입 확인 연산자를 적용하는 것은 너무 번거롭고 코드 크기를 과도하게 키운다.

타입스크립트는 이러한 자바스크립트의 한계를 벗어나, 타입 체크를 런타임이 아닌 **컴파일 타임에 정적으로** 수행할 수 있게 해준다.

정확히는 `tsc`가 소스를 자바스크립트로 변환하는 과정 중 Checker 단계에서 타입을 검사한다.

#### 컴파일 타임과 빌드 타임은 다른가?

빌드 타임은 컴파일에 번들링과 최적화까지 포함한 더 넓은 개념이라 흔히 혼용되지만, 타입 검사가 일어나는 지점은 컴파일 타임이다. esbuild나 SWC로 빌드하는 프로젝트는 빌드 타임에 타입 검사를 아예 하지 않기도 한다.

---

## 타입스크립트는 어떻게 동작하는가

브라우저나 Node.js는 타입스크립트를 직접 실행하지 못한다. 반드시 자바스크립트로 변환(컴파일)되어야 실행된다.

`tsc`가 하는 일을 단계로 나누면 다음과 같다.

```
  .ts 소스 코드
       │
       ▼
┌──────────────┐
│   Scanner    │  문자열을 의미 단위(토큰)로 쪼갠다
│  (Lexer)     │  "const a: number = 1" → [const][a][:][number][=][1]
└──────┬───────┘
       ▼
┌──────────────┐
│    Parser    │  토큰을 문법 규칙에 맞게 트리로 조립한다
└──────┬───────┘  문법이 틀리면 여기서 Syntax Error
       ▼
┌──────────────┐
│     AST      │  추상 구문 트리 (Abstract Syntax Tree)
└──────┬───────┘  코드의 구조를 표현한 객체 트리
       ▼
┌──────────────┐
│    Binder    │  스코프와 심벌(Symbol) 테이블을 만든다
└──────┬───────┘  "이 이름이 가리키는 선언이 무엇인가"를 연결
       ▼
┌──────────────┐
│   Checker    │  타입 추론 + 타입 검사 (타입스크립트의 핵심)
└──────┬───────┘  타입이 안 맞으면 여기서 Type Error
       ▼
┌──────────────┐
│   Emitter    │  타입을 지우고 target 버전에 맞는 JS를 출력
└──────┬───────┘  + .d.ts, source map 생성
       ▼
  .js 결과물 → 엔진이 실행
```

### AST (추상 구문 트리)

소스 코드의 문법 구조를 **트리 형태의 객체**로 표현한 것이다. 컴파일러가 코드를 "문자열"이 아니라 "구조"로 다루기 위해 만든다.

```ts
const a: number = 1;
```

위 코드는 대략 이런 트리가 된다.

```
VariableStatement
└─ VariableDeclarationList (const)
   └─ VariableDeclaration
      ├─ Identifier          : a
      ├─ TypeReference       : number   ← 타입 정보 (JS로 나갈 때 제거됨)
      └─ NumericLiteral      : 1
```

세미콜론, 괄호, 공백처럼 의미에 영향을 주지 않는 토큰은 트리에 담지 않기 때문에 **추상**(Abstract)이라고 부른다. 이런 토큰까지 모두 담은 것은 구체 구문 트리(CST)다.

AST는 타입스크립트만의 개념이 아니다. ESLint, Prettier, Babel, 코드모드 같은 도구도 전부 AST를 만들어 코드를 분석하고 변형한다. 자바스크립트 엔진(V8)도 실행 전에 파싱해서 AST를 만들고 이를 바이트코드로 바꿔 실행한다.

> AST는 TS를 컴파일하는 단계와 JS를 실행하는 단계 양쪽에 모두 등장한다.

### 에디터의 빨간 밑줄

저장도 하기 전에 그어지는 빨간 밑줄은 **tsserver**가 그린 것이다. `tsc`와 같은 파이프라인을 메모리 위에서 돌리는 별도 프로세스다.

- **저장이 필요 없다** : 에디터 버퍼를 그대로 넘겨받아 타이핑 중에도 검사한다.
- **JS를 만들지 않는다** : Emitter까지 가지 않고 Checker 결과만 돌려준다.
- **증분 검사다** : 바뀐 파일과 영향 범위만 다시 본다.

에디터의 에러와 `tsc --noEmit`의 에러는 같은 Checker의 결과다. 다만 에디터는 열린 파일 위주로, `tsc`는 프로젝트 전체를 검사해서 에디터는 멀쩡한데 CI가 깨지기도 한다.

### 타입 소거 (Type Erasure)

컴파일 결과물에는 타입이 하나도 남지 않는다.

```ts
// 입력 (.ts)
interface User {
  name: string;
}
const greet = (user: User): string => `hi ${user.name}`;
```

```js
// 출력 (.js)
const greet = (user) => `hi ${user.name}`;
// interface User 는 흔적조차 남지 않는다
```

- **런타임 성능에 영향이 없다** : 실행되는 코드에 타입이 없다.
- **런타임 안전도 보장하지 않는다** : API 응답이나 사용자 입력은 선언한 타입과 다를 수 있다. 검증이 필요하면 zod 같은 라이브러리를 쓴다.
- **타입 전용 문법은 확인할 수 없다** : `interface`, `type`, `as`는 `typeof`나 `instanceof`로 잡히지 않는다.

> `enum`, `class`, `namespace`는 예외로, 실제 자바스크립트 코드를 만들어낸다.

### enum이 남기는 코드와 트리 셰이킹

`enum`은 타입이 아니라 값이기 때문에 컴파일 후에도 코드가 남는다.

```ts
// 입력 (.ts)
enum Direction {
  Up,
  Down,
}
```

```js
// 출력 (.js)
var Direction;
(function (Direction) {
  Direction[(Direction["Up"] = 0)] = "Up";
  Direction[(Direction["Down"] = 1)] = "Down";
})(Direction || (Direction = {}));
```

이름과 값을 서로 찾을 수 있도록 역방향 매핑까지 만드느라 IIFE가 통째로 들어간다.

문제는 이 IIFE가 **트리 셰이킹되지 않는다**는 점이다. 번들러는 즉시 실행 함수에 부수 효과가 없다고 단정할 수 없어서, 하나만 쓰거나 아예 안 써도 전체를 번들에 남긴다.

`const enum`은 값을 인라인해 이 문제를 없애지만, 파일 하나만 보고 변환하는 esbuild나 SWC와 함께 쓸 수 없다. 다른 파일의 값을 알아야 인라인이 가능하기 때문이다.

그래서 `as const` 객체와 유니온 타입으로 대체한다.

```ts
const Direction = {
  Up: "UP",
  Down: "DOWN",
} as const;

type Direction = (typeof Direction)[keyof typeof Direction]; // "UP" | "DOWN"
```

평범한 객체라서 트리 셰이킹이 되고, 값은 `Direction.Up`, 타입은 `Direction`으로 enum처럼 쓸 수 있다.

### 타입 검사와 트랜스파일은 별개다

esbuild, SWC, Babel은 타입 검사를 하지 않고 타입만 지워서 빠르게 자바스크립트를 뽑아낸다.

| 도구                  | 타입 검사 | 트랜스파일 | 특징                             |
| --------------------- | --------- | ---------- | -------------------------------- |
| `tsc`                 | O         | O          | 느리지만 정확, `.d.ts` 생성 가능 |
| `tsc --noEmit`        | O         | X          | CI에서 타입 검사만 돌릴 때       |
| esbuild / SWC / Babel | X         | O          | 파일 단위로 타입만 제거해 빠름   |

그래서 빌드는 esbuild나 SWC가 하고 타입 검사는 `tsc --noEmit`이 맡는 구성이 흔하다. 빌드가 성공했다고 타입 에러가 없는 것은 아니라는 뜻이다.

---

## 리액트 코드를 효과적으로 작성하기 위한 타입스크립트 활용법

### interface와 type의 차이

객체의 형태를 선언한다는 점에서는 거의 같지만, 할 수 있는 일이 다르다.

|                            | `interface` | `type` |
| -------------------------- | ----------- | ------ |
| 객체 형태 선언             | O           | O      |
| 확장                       | `extends`   | `&`    |
| 유니온, 튜플, 조건부 타입  | X           | O      |
| 원시 타입에 별칭 붙이기    | X           | O      |
| 같은 이름으로 다시 선언    | O (병합)    | X      |

가장 큰 차이는 **선언 병합**(declaration merging)이다. `interface`는 같은 이름으로 여러 번 선언하면 하나로 합쳐진다.

```ts
interface User {
  name: string;
}
interface User {
  age: number;
}
// User = { name: string; age: number }

type Point = { x: number };
type Point = { y: number }; // Error: 중복된 식별자
```

이 성질 덕분에 외부 라이브러리의 타입을 확장할 수 있다. `styled-components`의 테마 타입을 넓히거나 `window`에 속성을 추가하는 것이 대표적이다.

```ts
declare global {
  interface Window {
    dataLayer: unknown[];
  }
}

window.dataLayer.push({ event: "click" }); // 이제 타입 에러가 나지 않는다
```

타입스크립트가 이미 정의해둔 `Window` 인터페이스에 내 속성을 끼워 넣는 것이다. `type`으로는 재선언이 안 되기 때문에 이런 확장이 불가능하다.

#### declare는 무엇인가

값을 만들지 않고 "이건 어딘가에 이미 존재하니 타입만 알아둬라"라고 컴파일러에 알려주는 키워드다. **앰비언트(ambient) 선언**이라고 부른다.

```ts
declare const API_URL: string; // 실제 값은 다른 곳에 있다고 가정한다

console.log(API_URL);
```

타입 정보일 뿐이라 컴파일 결과에는 아무 코드도 남지 않는다. 주로 쓰이는 자리는 세 군데다.

- `.d.ts` 파일 : 타입 선언만 모아둔 파일. `@types/*` 패키지가 전부 이것이다.
- `declare global` : 모듈 파일 안에서 전역 스코프에 타입을 추가할 때
- `declare module "..."` : 타입 정의가 없는 라이브러리나 `.svg` 같은 파일 import에 타입을 붙일 때

반대로 유니온이나 튜플, 조건부 타입처럼 객체가 아닌 타입은 `type`으로만 만들 수 있다.

```ts
type Status = "idle" | "loading" | "done"; // 유니온
type Handler = (e: MouseEvent) => void; // 함수
type Pair = [number, number]; // 튜플
```

정리하면, 컴포넌트 props나 객체 형태처럼 확장 가능성이 있는 것은 `interface`, 유니온이나 유틸리티 타입 조합은 `type`을 쓰면 된다. 다만 팀 안에서 한 가지 기준으로 통일하는 편이 더 중요하다.

> 예상치 못한 선언 병합을 막고 싶어서 `type`으로 통일하는 팀도 많다.

### any 대신 unknown을 사용하자

- `any`는 정말로 불가피할 때만 사용해야 하는 타입이다. (any 쓸 거면 왜 타입스크립트를 쓰는가?)
- `any`는 JS에서 TS로 넘어가는 과도기 같은 예외적인 경우에만 쓰자.
- 불가피하게 아직 타입을 단정할 수 없다면 `unknown`을 사용하자.

```ts
const printString = (value: unknown) => {
  if (typeof value === "string") {
    console.log(value);
  } else {
    console.log("입력값이 문자열이 아닙니다");
  }
};

printString("Hello, TypeScript!"); // 출력: Hello, TypeScript!
printString(42); // 출력: 입력값이 문자열이 아닙니다
```

`unknown`은 사용하기 전에 반드시 타입을 좁히도록 강제하기 때문에 `any`보다 안전하다.

### never

어떠한 타입도 들어올 수 없는, 코드상으로 존재가 불가능한 타입을 나타낸다.

→ 클래스 컴포넌트를 선언할 때 props는 없지만 state는 존재하는 상황에서, "어떠한 props도 받지 않는다"는 의미로 사용할 수 있다.

```tsx
import React, { Component } from "react";

// Props가 존재하지 않음을 나타내는 never 타입
type EmptyProps = never;

interface MyComponentState {
  count: number;
}

class MyComponent extends Component<EmptyProps, MyComponentState> {
  state: MyComponentState = {
    count: 0,
  };

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
      </div>
    );
  }
}

export default MyComponent;
```

> 다만 props 타입이 `never`면 `<MyComponent />`처럼 사용할 때도 오류가 난다. 빈 객체조차 `never`에 할당할 수 없기 때문이다. props를 받지 않는다는 의미로는 `Record<string, never>`가 더 정확하다.

### 타입 가드를 적극 활용하자

```ts
const func = (value: number | string) => {
  if (typeof value === "number") {
    console.log(value.toFixed());
  } else if (typeof value === "string") {
    console.log(value.toUpperCase());
  }
};
```

조건문을 이용해 조건문 내부에서 변수가 특정 타입임을 보장하면, 그 내부에서는 변수의 타입이 보장된 타입으로 좁혀진다.

`if (typeof === …)` 처럼 조건문과 함께 사용해 타입을 좁히는 표현들을 **타입 가드**라고 부른다.

#### instanceof

지정한 인스턴스가 특정 클래스의 인스턴스인지 확인하는 연산자다. 내장 클래스 또는 직접 만든 클래스에만 사용할 수 있다.

#### in

`property in object` 형태로, 어떤 객체에 키가 존재하는지 확인하는 용도다.

```ts
type Person = {
  name: string;
  age: number;
};

const func = (value: number | string | Date | null | Person) => {
  if (typeof value === "number") {
    console.log(value.toFixed());
  } else if (typeof value === "string") {
    console.log(value.toUpperCase());
  } else if (value instanceof Date) {
    console.log(value.getTime());
  } else if (value && "age" in value) {
    console.log(`${value.name}은 ${value.age}살 입니다`);
  }
};
```

### 제네릭

함수나 클래스 내부에서 단일 타입이 아닌 **다양한 타입에 대응**할 수 있도록 도와주는 도구다. 비슷한 작업을 하는 컴포넌트를 단일 제네릭 컴포넌트로 선언해 간결하게 작성할 수 있다.

```ts
const func = <T>(value: T): T => value;

let num = func(10); // number 타입
```

- `T`에 어떤 타입이 할당될지는 **함수가 호출될 때** 결정된다.
- `func(10)`처럼 number 타입의 값을 인수로 전달하면 매개변수 `value`에 number 값이 저장되면서 `T`가 number로 추론된다.

> `.tsx` 파일에서는 `<T>`를 JSX 태그로 오해하기 때문에 `<T,>` 또는 `<T extends unknown>`으로 써야 한다.

### 인덱스 시그니처

객체의 키를 정의하는 방식이다.

```ts
interface StringArray {
  [index: number]: string;
}

const myArray: StringArray = ["hello", "world"];
const firstItem: string = myArray[0];
```

인덱스 시그니처를 사용할 때, 키의 범위가 넓어서 해당 키에 대응하는 값이 항상 존재하지 않는 경우에는 `undefined`가 반환된다. 객체의 키는 동적으로 선언되는 경우를 최대한 지양해야 하고, 객체의 타입도 필요에 따라 좁혀야 한다.

```ts
interface NumberToString {
  [index: number]: string;
}

const myObject: NumberToString = {
  0: "zero",
  1: "one",
  2: "two",
};

console.log(myObject[3]); // 출력: undefined
```

→ `Record`를 사용하여 객체를 생성하는 방법

```ts
const myRecord: Record<string, number> = {
  a: 1,
  b: 2,
  c: 3,
};
```

→ 타입을 사용한 인덱스 시그니처

```ts
type StringToNumber = {
  [key: string]: number;
};

const myObject: StringToNumber = {
  a: 1,
  b: 2,
  c: 3,
};
```

---

## Object.keys(hello)의 반환 타입

```ts
Object.keys(hello).map((key) => {
  const value = hello[key]; // Error
  return value;
});
```

`Object.keys()` 메서드는 주어진 객체의 속성 이름들을 문자열 배열로 반환한다. → `string[]`

이 `string`은 `hello`의 인덱스 키로 접근할 수 없다.

### 해결하기

**1. `as`로 타입을 단언하는 방법**

```ts
// Object.keys의 반환 타입을 string[] 대신 개발자가 단언한 타입으로 강제
(Object.keys(hello) as Array<keyof Hello>).map((key) => {
  const value = hello[key];
  return value;
});
```

**2. 타입 가드 함수를 만드는 방법**

```ts
// Object.keys를 대신할 keysOf 함수를 만들어
// 키를 가져오면서 동시에 타입 단언까지 처리한다
const keysOf = <T extends object>(obj: T): Array<keyof T> =>
  Array.from(Object.keys(obj)) as Array<keyof T>;

keysOf(hello).map((key) => {
  const value = hello[key];
  return value;
});
```

**3. 가져온 key를 단언하는 방법**

```ts
Object.keys(hello).map((key) => {
  const value = hello[key as keyof Hello];
  return value;
});
```

### 왜 string[]인가

**구조적 타이핑** 때문이다. 타입이 맞기만 하면 되므로, `Hello`로 선언한 변수에 선언하지 않은 키가 더 들어 있을 수 있다.

```ts
type Hello = { a: number };

const extra = { a: 1, b: 2 };
const hello: Hello = extra; // 통과. 하지만 런타임의 키는 a, b 두 개다
```

런타임에 어떤 키가 들어 있을지 알 수 없으니 `keyof Hello`로 좁히면 실제 결과와 어긋난다. 그래서 어느 경우에도 안전한 `string[]`을 반환하는 것이다.

### 구조적 타이핑 = 덕 타이핑

객체의 타입이 클래스 상속, 인터페이스 구현 등으로 결정되는 것이 아니라, 어떤 객체가 필요한 변수와 메서드만 지니고 있다면 그냥 해당 타입에 속하도록 인정해주는 것이다. Java나 C# 같은 명목적 타이핑과 가장 크게 다른 지점이다.

```ts
interface Point {
  x: number;
  y: number;
}

const draw = (p: Point) => {};

const vector = { x: 1, y: 2, z: 3 };
draw(vector); // Point를 구현한다고 선언하지 않았지만 형태가 맞으므로 통과
```
