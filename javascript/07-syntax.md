# Syntax

리액트에서 자주 사용하는 자바스크립트 문법을 정리한다.

여기서 다루는 문법 대부분은 ES6 이후에 추가된 것이고, **왜 리액트에서 이 문법이 필요한가**는 결국 [Equality](02-equality.md)에서 정리한 **참조 비교 / 불변성**과 이어진다.

---

## 구조 분해 할당 (Destructuring Assignment)

배열 또는 객체의 값을 분해해 개별 변수에 즉시 할당하는 문법이다.

### 배열 구조 분해 할당

배열은 **인덱스 순서**를 기준으로 값을 꺼낸다.

```ts
const array = [1, 2, 3, 4, 5];

const [first, second, third, ...arrayRest] = array;
// first: 1, second: 2, third: 3, arrayRest: [4, 5]
```

#### 기본값을 선언할 수 있다

```ts
const array = [1, 2];
const [a = 10, b = 10, c = 10] = array;

console.log(a); // 1  (값이 존재하므로 기본값 무시)
console.log(b); // 2  (값이 존재하므로 기본값 무시)
console.log(c); // 10 (값이 없으므로 기본값 사용)
```

#### 기본값은 `undefined`일 때만 적용된다

```ts
const arrayWithUndefined = [1, undefined, null];
const [x = 10, y = 10, z = 10] = arrayWithUndefined;

console.log(x); // 1
console.log(y); // 10   (undefined → 기본값 사용)
console.log(z); // null (null은 undefined가 아니므로 기본값 미적용)
```

`null`, `0`, `''`, `false`는 falsy지만 `undefined`가 아니므로 기본값이 적용되지 않는다.

#### 건너뛰기와 위치의 제약

```ts
const [, , third] = [1, 2, 3]; // 쉼표로 인덱스를 건너뛴다

console.log(third); // 3

const [head, ...rest] = [1, 2, 3];

console.log(head); // 1
console.log(rest); // [2, 3]

// const [...rest, tail] = [1, 2, 3]  ❌ SyntaxError
```

rest 요소는 반드시 **마지막**에 와야 한다. 나머지를 모아 담는 문법이므로 뒤에 무언가가 더 올 수 없다.

#### 배열 구조 분해는 배열에만 쓸 수 있는가?

아니다. **값을 하나씩 순서대로 꺼낼 수 있는 값(= 이터러블)이면 모두 가능하다.**

```ts
const [c1, c2] = "hi"; // 문자열

console.log(c1, c2); // 'h' 'i'

const [s1] = new Set([10, 20]); // Set

console.log(s1); // 10
```

반면 객체는 값에 순서가 없고 이름으로만 접근할 수 있어서 배열 구조 분해가 되지 않는다.

```ts
const user = { name: "kim", age: 20 };

const [a, b] = user; // ❌ TypeError: user is not iterable
const { name, age } = user; // ✅ 객체는 이름으로 꺼낸다
```

> **이터러블(iterable)** : 값을 하나씩 순서대로 꺼내는 방법(`Symbol.iterator` 메서드)을 가진 값을 말한다. 배열, 문자열, `Set`, `Map`, `arguments`, `NodeList`가 이터러블이고 일반 객체는 아니다. 배열 구조 분해뿐 아니라 `for...of`, 전개 구문(`...`)도 이 규칙을 따르기 때문에, **이터러블이면 셋 다 되고 아니면 셋 다 안 된다.**

### React의 `useState`가 배열을 반환하는 이유

```ts
const [state, setState] = useState(initialState);
```

- **이름을 자유롭게 지을 수 있다.** 배열은 순서로 꺼내므로 변수명을 마음대로 정할 수 있다. 한 컴포넌트에서 `useState`를 여러 번 써도 이름이 충돌하지 않는다.

```ts
// 배열 → 이름 자유
const [count, setCount] = useState(0);
const [name, setName] = useState("");

// 객체였다면 → 키 이름이 고정되어 매번 이름을 바꿔줘야 한다
const { state: count, setState: setCount } = useState(0);
const { state: name, setState: setName } = useState("");
```

- **반환 순서가 항상 고정된다.** 값이 첫 번째, 갱신 함수가 두 번째다. 규칙이 단순해서 외우기 쉽다.

> 반대로 `useQuery`처럼 반환값이 많고(`data`, `isLoading`, `error`, `refetch` …) 그중 일부만 골라 쓰는 API는 객체를 반환한다. **꺼내 쓸 값이 적고 이름을 바꿀 일이 많으면 배열, 값이 많고 선택적으로 꺼내 쓰면 객체**가 적합하다.

### 객체 구조 분해 할당

객체는 **속성 이름(키)**을 기준으로 값을 꺼낸다. 배열처럼 순서로 꺼내는 것이 아니라서, 키와 **같은 이름의 변수**를 선언해야 값이 들어온다.

```ts
const user = {
  name: "kim",
  age: 20,
  city: "seoul",
};

const { name, age, city } = user;

console.log(name); // 'kim'
console.log(age); // 20
console.log(city); // 'seoul'
```

#### 새로운 변수 이름으로 할당하기

```ts
const { name: fullName, age: years, city: location } = user;

console.log(fullName); // 'kim'
console.log(years); // 20
console.log(location); // 'seoul'
```

#### 기본값과 이름 변경을 함께 쓰기

```ts
const { name: fullName = "anonymous", nickname = "none" } = user;
```

#### 중첩 구조 분해와 rest

```ts
const props = {
  id: 1,
  profile: { email: "a@b.com" },
  theme: "dark",
};

const {
  profile: { email },
  ...restProps
} = props;

console.log(email); // 'a@b.com'
console.log(restProps); // { id: 1, theme: 'dark' }
```

React에서 특정 prop만 빼고 나머지를 그대로 넘길 때 자주 쓰는 패턴이다.

```tsx
const Button = ({ variant, ...props }) => {
  return <button className={variant} {...props} />;
};
```

#### 함수 매개변수에서의 구조 분해

매개변수 자리에서도 구조 분해를 할 수 있다. React 컴포넌트가 props를 받는 방식이 바로 이것이다.

```tsx
const Profile = ({ name = "anonymous", age = 0 }) => { ... }

Profile({ name: "kim" }); // name: 'kim', age: 0
```

여기서 `= "anonymous"`는 **객체 안에 그 키가 없을 때** 쓰이는 기본값이다.

#### 인자를 아예 넘기지 않으면 에러가 난다

```ts
const init = ({ debug = false }) => {
  console.log(debug);
};

init({}); // false
init(); // ❌ TypeError: Cannot destructure property 'debug' of undefined
```

인자를 생략하면 매개변수가 `undefined`가 되는데, 구조 분해는 그 값의 속성에 접근하는 문법이라 `undefined.debug`를 읽으려다 에러가 난다. (`null`도 마찬가지다.)

그래서 **매개변수 자리에도 기본값**을 준다. 구조 분해를 걷어내고 보면 평범한 매개변수 기본값과 같은 형태다.

```ts
const init = (options = {}) => { ... };       // 흔히 보는 매개변수 기본값
const init = ({ debug = false } = {}) => { ... }; // options 자리를 구조 분해로 바꾼 것
```

```ts
const init = ({ debug = false } = {}) => {
  console.log(debug);
};

init(); // false ← 인자가 없으면 {} 를 대신 사용한다
```

- `{ debug = false }` : 넘어온 객체 **안에 `debug` 키가 없을 때**의 기본값
- `= {}` : **인자로 넘어온 객체가 없을 때**(`undefined`) 대신 쓸 매개변수의 기본값

> 함수의 반환값과는 무관하다. `= {}`는 매개변수가 비었을 때 채워 넣는 값일 뿐이다.

> React 컴포넌트는 props가 항상 객체로 전달되므로 `= {}`가 필요 없지만, 직접 만든 유틸 함수라면 붙여두는 것이 안전하다.

### 계산된 속성 이름 (Computed Property Names)

속성 이름을 코드에 고정하지 않고, **변수 값으로 실행 시점에 결정**하는 방법이다. 대괄호(`[]`) 안에 표현식을 쓴다.

```ts
// 정적: 코드에 'name'이라고 박혀 있어 항상 name만 꺼낸다
const { name } = user;

// 동적: key에 무엇이 들어있느냐에 따라 꺼내는 대상이 달라진다
const { [key]: value } = user;
```

```ts
const key = "name";
const user = { name: "kim", age: 20, city: "seoul" };

const { [key]: userName, age, city } = user;

console.log(userName); // 'kim'
```

#### 계산된 속성 이름에는 왜 변수 이름을 반드시 붙여야 하는가?

`const { [key] } = user`는 문법 에러다. `[key]`는 **어떤 속성을 꺼낼지**만 정할 뿐이고, 그 값을 담을 변수 이름을 알 수 없기 때문이다. `key`의 값은 런타임에야 정해지므로 엔진이 변수명을 추론할 수 없다.

```ts
const { [key]: userName } = user; // ✅ 꺼낼 키 [key], 담을 변수 userName
```

#### 객체를 만들 때도 쓴다

React에서 여러 입력값을 하나의 state로 관리할 때 자주 등장하는 패턴이다.

```tsx
const [form, setForm] = useState({ name: "", email: "" });

const handleChange = (e) => {
  const { name, value } = e.target;
  setForm((prev) => ({ ...prev, [name]: value })); // 계산된 속성 이름
};

return (
  <>
    <input name="name" value={form.name} onChange={handleChange} />
    <input name="email" value={form.email} onChange={handleChange} />
  </>
);
```

`[name]`의 `name`은 입력이 발생한 input의 `name` 속성(`e.target.name`)이다. email 칸에 입력하면 `'email'`이 들어오므로 `email` 키만 갱신된다.

```ts
{ ...prev, name: value }   // { name: 'a', email: '' } ❌ 'name'이라는 글자가 키가 된다
{ ...prev, [name]: value } // { name: '', email: 'a' } ✅ name 변수의 값 'email'이 키가 된다
```

`[name]` 덕분에 **어떤 칸이 바뀌었는지 실행 시점에 알아내서 그 키만 갱신**할 수 있다. 대괄호를 쓰지 않으면 입력창마다 핸들러를 따로 만들어야 한다.

### 구조 분해 할당은 어떻게 트랜스파일되는가?

바벨로 ES5까지 낮추면 배열 구조 분해는 `slicedToArray` 같은 헬퍼 함수로, 객체 구조 분해는 `objectWithoutProperties` 같은 헬퍼로 변환된다. 특히 **객체 rest(`...rest`)는 코드량이 눈에 띄게 늘어난다.**

> 다만 요즘 브라우저 대부분이 ES6+를 지원하므로, 지원 대상(browserslist)을 최신으로 잡으면 그대로 남는다. **"구조 분해는 무겁다"가 아니라 "낮은 타깃으로 트랜스파일하면 무거워진다"**가 정확한 표현이다.

---

## 전개 구문 (Spread Syntax)

배열, 객체, 문자열처럼 순회 가능한 값을 펼쳐서 사용하는 구문이다. 구조 분해가 **꺼내는** 문법이라면, 전개는 **펼쳐 넣는** 문법이다.

### 배열 전개

```ts
const arr1 = ["a", "b"];
const arr2 = arr1;

arr1 === arr2; // true : 값이 아니라 참조를 복사했다

const arr3 = [...arr1];

arr1 === arr3; // false : 새로운 배열이 만들어졌다
```

```ts
const merged = [...arr1, "c", ...["d", "e"]]; // ['a','b','c','d','e']
```

배열에서는 전개 위치가 자유롭다. 다만 `push`, `splice` 같은 원본 변경 메서드 대신 전개를 쓰는 이유가 바로 **새 참조를 만들기 위해서**다.

### 객체 전개

```ts
const obj1 = { a: 1, b: 2 };

const obj2 = { c: 3, d: 4, ...obj1 };
console.log(obj2); // { c: 3, d: 4, a: 1, b: 2 }

const obj3 = { a: 3, ...obj1 };
console.log(obj3); // { a: 1, b: 2 }  ← 뒤에 온 ...obj1의 a가 덮어씀

const obj4 = { ...obj1, a: 3 };
console.log(obj4); // { a: 3, b: 2 }  ← 뒤에 온 a: 3이 덮어씀
```

객체는 **같은 키가 있으면 나중에 온 값이 이긴다.** 그래서 "기본값을 먼저 두고 덮어쓸 값을 뒤에" 두는 것이 일반적인 순서다.

```tsx
<Button {...defaultProps} {...userProps} /> // userProps가 우선
```

#### 전개 연산자는 얕은 복사(shallow copy)다

```ts
const origin = { a: 1, nested: { b: 2 } };
const copy = { ...origin };

copy.nested === origin.nested; // true : 중첩 객체는 참조를 그대로 공유
copy.nested.b = 99;
console.log(origin.nested.b); // 99 ← 원본이 함께 바뀐다
```

1 depth만 새로 만들고, 그 안의 객체는 참조를 그대로 복사한다. React에서 중첩 state를 업데이트할 때 각 depth마다 전개해야 하는 이유가 이것이다.

```ts
setState((prev) => ({
  ...prev,
  nested: { ...prev.nested, b: 99 }, // 안쪽도 새 참조로
}));
```

#### 깊은 복사(deep copy)하는 방법

중첩된 객체까지 전부 새로 만들어야 한다면 다음 방법을 쓴다.

| 방법                             | 특징                                                                     |
| -------------------------------- | ------------------------------------------------------------------------ |
| `structuredClone(obj)`           | 브라우저/Node 내장. `Date`, `Map`, `Set`, 순환 참조까지 처리한다          |
| `JSON.parse(JSON.stringify(obj))`| 간단하지만 `undefined`, 함수, `Symbol`은 사라지고 `Date`는 문자열이 된다 |
| `_.cloneDeep(obj)` (Lodash)      | 함수까지 대부분 처리하지만 라이브러리 의존이 생긴다                      |

```ts
const origin = { a: 1, nested: { b: 2 } };
const deep = structuredClone(origin);

deep.nested === origin.nested; // false : 안쪽까지 새 객체
deep.nested.b = 99;
console.log(origin.nested.b); // 2 ← 원본이 그대로다
```

> React state에는 깊은 복사가 거의 필요 없다. 바뀌는 경로만 새 참조로 만들면 되고(**구조적 공유**), 안 바뀐 부분은 참조를 그대로 두는 편이 오히려 메모리와 비교 비용 면에서 유리하다.

#### 얕은/깊은 **복사**와 얕은/깊은 **비교**를 혼동하지 않기

이름이 비슷해서 헷갈리지만 목적이 다르다.

| 구분     | 복사 (copy)                            | 비교 (compare)                          |
| -------- | -------------------------------------- | --------------------------------------- |
| 하는 일  | 새로운 객체를 **만든다**               | 두 객체가 같은지 **판단한다**           |
| 얕은 것  | 1 depth만 새 객체, 안쪽은 참조 공유    | 1 depth 값만 `Object.is`로 비교         |
| 깊은 것  | 중첩된 객체까지 전부 새로 만든다       | 중첩된 값까지 재귀적으로 비교           |
| 도구     | `{...obj}`, `structuredClone`          | `shallowEqual`, `_.isEqual`             |

둘은 이렇게 맞물린다. **전개 구문으로 얕은 복사를 해서 새 참조를 만들어 두면, React가 얕은 비교만으로도 변경을 감지할 수 있다.** 전개 구문이 리액트에서 그토록 자주 쓰이는 이유다.

> 비교가 어떻게 리렌더링으로 이어지는지는 [Equality — React에서의 활용](02-equality.md)에 정리했다.

#### 배열 전개와 객체 전개는 조건이 다르다

- **배열 전개**는 이터러블이어야 한다. → `[...'abc']`, `[...new Set()]` 가능, `[...{a:1}]`은 `TypeError`
- **객체 전개**는 이터러블이 아니어도 되고, **자기 자신의 열거 가능한(own enumerable) 속성**만 복사한다. → `{...'ab'}`는 `{0:'a', 1:'b'}`

그래서 프로토타입 메서드, `getter`의 정의 자체, non-enumerable 속성은 복사되지 않는다. 클래스 인스턴스를 전개하면 메서드가 사라지는 이유다.

```ts
class User {
  constructor(name) {
    this.name = name;
  }
  greet() {
    return `hi ${this.name}`;
  }
}
const u = new User("kim");
const spread = { ...u };
spread.greet; // undefined : greet는 프로토타입에 있다
```

#### 트랜스파일 관점

배열 전개는 단순히 값을 이어 붙이는 형태로 변환되지만, 객체 전개는 속성 설명자 확인·심벌 키 처리 등이 필요해 헬퍼 코드가 커진다. 구조 분해와 마찬가지로 **낮은 타깃으로 트랜스파일할 때 번들 크기에 영향**을 준다.

---

## 객체 초기자 (Object Shorthand)

키와 값을 담은 변수가 이미 있다면 축약해 넣을 수 있다.

```ts
const a = 1;
const b = 2;

const obj = { a, b }; // { a: 1, b: 2 }
```

메서드도 축약할 수 있다.

```ts
const obj = {
  greet() {
    return "hi";
  }, // greet: function () {...} 와 동일
};
```

> 단, 축약 메서드와 `function` 표현식은 **화살표 함수와 달리 자신의 `this`를 가진다.** 객체 리터럴 안에서 화살표 함수를 쓰면 `this`가 객체를 가리키지 않는다. ([Function — 화살표 함수](03-function.md) 참고)

---

## Array 프로토타입의 메서드

React에서 자주 쓰는 배열 메서드는 대부분 **원본을 바꾸지 않고 새 배열을 반환**한다. 불변성을 유지해야 리렌더링이 정상 동작하기 때문이다.

### Array.prototype.map

인수로 전달받은 배열과 **똑같은 길이**의 새로운 배열을 반환한다.

```ts
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((num) => num * 2);

console.log(doubled); // [2, 4, 6, 8, 10]
```

React에서는 배열을 JSX 요소 배열로 변환할 때 쓴다.

```tsx
{
  todos.map((todo) => <li key={todo.id}>{todo.text}</li>);
}
```

#### `key`에 인덱스를 쓰면 왜 안 되는가?

React는 `key`로 이전 렌더링의 요소와 현재 요소를 매칭한다. 인덱스를 키로 쓰면 목록의 앞쪽에 항목이 추가·삭제될 때 **같은 인덱스에 다른 데이터가 오게 되어** React가 "같은 요소인데 내용만 바뀌었다"고 판단한다. 그 결과 입력값이나 포커스 같은 컴포넌트 내부 상태가 엉뚱한 행에 남는다.

정렬·삽입·삭제가 없는 정적 목록이라면 인덱스도 문제되지 않지만, 기본적으로는 **데이터 고유의 id**를 쓴다.

### Array.prototype.filter

콜백이 truthy를 반환하는 원소만 모아 새 배열을 반환한다. 결과는 원본 길이 **이하**다.

```ts
const numbers = [1, 2, 3, 4, 5];
const evenNumbers = numbers.filter((num) => num % 2 === 0);

console.log(evenNumbers); // [2, 4]
```

#### React에서 filter로 항목 삭제하기

```tsx
import { useState } from "react";

const TodoList = () => {
  const [todos, setTodos] = useState([
    { id: 1, text: "밥 먹기" },
    { id: 2, text: "운동하기" },
    { id: 3, text: "책 읽기" },
  ]);

  const deleteTodo = (id) => {
    // splice로 원본을 자르지 않고, filter로 새 배열을 만든다
    setTodos((prev) => prev.filter((todo) => todo.id !== id));
  };

  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>
          {todo.text}
          <button onClick={() => deleteTodo(todo.id)}>삭제</button>
        </li>
      ))}
    </ul>
  );
};
```

`todos.splice(index, 1)`처럼 원본을 변경하면 배열의 **참조가 그대로**라서 React가 변경을 감지하지 못하고 리렌더링하지 않는다. `filter`는 항상 새 배열을 반환하므로 참조가 바뀐다.

> 정렬도 마찬가지다. `arr.sort()`는 원본을 바꾸므로 `[...arr].sort()` 또는 `arr.toSorted()`를 쓴다.

### Array.prototype.reduce

콜백과 초깃값을 받아 배열을 하나의 값으로 줄인다. 반환 타입이 자유로워서 숫자, 객체, 배열 무엇이든 만들 수 있다.

```ts
const numbers = [1, 2, 3, 4, 5];
const sum = numbers.reduce((acc, cur) => acc + cur, 0);

console.log(sum); // 15
```

배열을 조회용 객체(맵)로 바꿀 때 특히 유용하다.

```ts
const users = [
  { id: "a", name: "kim" },
  { id: "b", name: "lee" },
];

const userMap = users.reduce((acc, user) => {
  acc[user.id] = user;
  return acc;
}, {});
// { a: { id: 'a', name: 'kim' }, b: { id: 'b', name: 'lee' } }
```

#### `filter` + `map` 대신 `reduce`를 쓰면 무엇이 좋은가?

`filter`와 `map`을 연달아 쓰면 배열을 **두 번 순회하고 중간 배열도 한 번 더 만든다.** `reduce`는 한 번의 순회로 끝난다.

```ts
// 2번 순회 + 중간 배열 생성
const result1 = numbers.filter((n) => n % 2 === 0).map((n) => n * 2);

// 1번 순회
const result2 = numbers.reduce((acc, n) => {
  if (n % 2 === 0) acc.push(n * 2);
  return acc;
}, []);
```

다만 대부분의 화면 단위 데이터에서는 차이가 체감되지 않는다. **간단한 변환은 가독성이 좋은 `filter`+`map`, 복잡한 집계나 형태 변환은 `reduce`**를 쓰는 것이 좋다.

#### 초깃값을 생략하면 어떻게 되는가?

첫 번째 요소가 초깃값이 되고 두 번째 요소부터 순회한다. 그래서 **빈 배열에서 초깃값 없이 `reduce`를 호출하면 `TypeError`**가 난다. 초깃값은 항상 명시하는 것이 안전하다.

```ts
[].reduce((a, b) => a + b); // ❌ TypeError: Reduce of empty array with no initial value
[].reduce((a, b) => a + b, 0); // ✅ 0
```

### Array.prototype.forEach

배열을 순회하며 콜백을 실행하기만 한다.

```ts
const numbers = [1, 2, 3, 4, 5];
numbers.forEach((num) => console.log(num * 2));
// 2, 4, 6, 8, 10
```

- **반환값이 `undefined`**다. 체이닝할 수 없다.
- **중간에 멈출 수 없다.** `break`, `continue`를 쓸 수 없고 `return`은 그 콜백 한 번만 종료시킨다.

```ts
[1, 2, 3, 4, 5].forEach((num) => {
  console.log(num);
  if (num === 3) {
    console.log("끝인줄 알았지?");
    return; // 이번 콜백만 끝날 뿐, 순회는 계속된다
  }
});
console.log("끝");

// 1 → 2 → 3 → 끝인줄 알았지? → 4 → 5 → 끝
```

에러를 던지거나 프로세스를 종료하지 않는 한 멈출 수 없다. 중간에 빠져나와야 한다면 `for...of`를 쓰거나, 목적에 따라 `some`/`every`/`find`를 쓴다.

```ts
arr.some((n) => n > 3); // 하나라도 만족하면 즉시 중단하고 true
arr.every((n) => n > 0); // 하나라도 실패하면 즉시 중단하고 false
arr.find((n) => n > 3); // 첫 번째로 만족하는 값을 찾고 즉시 중단
```

#### `map`과 `forEach`는 언제 구분해서 쓰는가?

**결과 배열이 필요하면 `map`, 부수 효과만 필요하면 `forEach`**다. 반환값을 쓰지 않으면서 `map`을 쓰면 불필요한 배열을 만들게 되고, 읽는 사람에게 "이 결과를 어딘가 쓰겠구나"라는 잘못된 신호를 준다.

### 원본을 바꾸는 메서드 vs 새 배열을 반환하는 메서드

| 원본 변경 (React state에 직접 쓰면 위험)    | 새 배열 반환 (안전)                                    |
| ------------------------------------------- | ------------------------------------------------------ |
| `push`, `pop`, `shift`, `unshift`, `splice` | `concat`, `slice`, `map`, `filter`, `flat`             |
| `sort`, `reverse`, `fill`, `copyWithin`     | `toSorted`, `toReversed`, `with`, `toSpliced` (ES2023) |

state를 다룰 때는 오른쪽 열을 쓰거나 전개 구문으로 새 배열을 만든다.

---

## 삼항 조건 연산자

자바스크립트에서 유일하게 피연산자를 3개 취하는 연산자다.

```ts
condition ? expr1 : expr2;
```

**문(statement)이 아니라 식(expression)**이라서 값으로 평가된다. JSX 안에서는 `if` 문을 쓸 수 없으므로 조건부 렌더링에 삼항 연산자를 쓴다.

```tsx
{
  isLoading ? <Spinner /> : <Content data={data} />;
}
```

#### 중첩은 피한다

```tsx
// ❌ 읽기 어렵다
{
  isLoading ? <Spinner /> : error ? <Error /> : data ? <List /> : <Empty />;
}

// ✅ 이른 반환(early return)으로 분리
if (isLoading) return <Spinner />;
if (error) return <Error />;
if (!data) return <Empty />;
return <List />;
```

#### `&&`로 조건부 렌더링할 때의 함정

```tsx
{
  items.length && <List />;
} // ❌ 길이가 0이면 화면에 "0"이 그대로 렌더링된다
{
  items.length > 0 && <List />;
} // ✅
```

React는 `false`, `null`, `undefined`는 렌더링하지 않지만 **`0`은 유효한 값이라 그대로 출력한다.** `&&` 왼쪽은 반드시 boolean으로 만든다.

#### `||`와 `??`의 차이

```ts
const count1 = props.count || 10; // count가 0이면 → 10 (의도치 않음)
const count2 = props.count ?? 10; // count가 0이면 → 0  (null/undefined일 때만 10)
```

`||`는 모든 falsy(`0`, `''`, `false`, `NaN`, `null`, `undefined`)를 대체하고, `??`는 `null`과 `undefined`만 대체한다. **기본값을 줄 때는 `??`가 대체로 안전하다.**

#### 옵셔널 체이닝 `?.`

```ts
const email = user?.profile?.email; // 중간이 null/undefined면 undefined 반환
onChange?.(value); // 함수가 없으면 호출 자체를 건너뛴다
list?.[0];
```

`null`, `undefined`일 때만 단축 평가되며, `0`이나 `''`에서는 정상적으로 접근한다.

---

## 요약

- **구조 분해 할당**은 배열은 이터러블(순서), 객체는 속성 이름을 기준으로 값을 꺼낸다. 기본값은 `undefined`일 때만 적용된다.
- `useState`가 배열을 반환하는 이유는 **변수 이름을 자유롭게 지을 수 있고 순서가 고정**되기 때문이다.
- **계산된 속성 이름**(`[key]`)은 담을 변수 이름을 추론할 수 없으므로 `[key]: name` 형태로 반드시 이름을 붙여야 한다.
- **전개 구문**은 새로운 참조를 만든다. 단, **얕은 복사**라서 중첩 객체는 참조를 공유한다. 객체 전개는 같은 키가 있으면 뒤에 온 값이 이긴다.
- 깊은 복사는 `structuredClone`을 쓴다. **복사(새 객체를 만드는 것)와 비교(같은지 판단하는 것)는 다른 개념**이고, 얕은 복사로 새 참조를 만들어야 React의 얕은 비교가 변경을 감지한다.
- 구조 분해와 객체 전개는 **낮은 타깃으로 트랜스파일하면** 헬퍼 코드 때문에 번들이 커진다.
- `map`/`filter`/`reduce`는 새 배열(값)을 반환하고, `push`/`splice`/`sort`는 원본을 바꾼다. React state에는 **참조가 바뀌는 메서드**를 써야 리렌더링된다.
- `reduce`는 초깃값을 생략하면 빈 배열에서 `TypeError`가 난다. 항상 명시한다.
- `forEach`는 반환값이 없고 중간에 멈출 수 없다. 중단이 필요하면 `for...of` / `some` / `every` / `find`를 쓴다.
- 삼항 연산자는 식이라서 JSX에서 조건부 렌더링에 쓴다. 중첩은 피하고 early return으로 푼다.
- `&&` 조건부 렌더링은 왼쪽이 `0`이면 `0`이 렌더링된다. 기본값은 `||`보다 `??`가 안전하다.
