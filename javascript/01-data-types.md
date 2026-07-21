# Data Types

JavaScript의 데이터 타입과 특징을 정리한다.

JavaScript의 값은 크게 **원시 타입**과 **객체 타입**으로 나뉜다.

- **원시 타입 (7가지)**: `undefined`, `null`, `boolean`, `number`, `bigint`, `string`, `symbol`
- **객체 타입**: 객체, 배열, 함수 등

---

## 원시 타입 (Primitive Type)

원시 타입은 값을 그대로 저장하며, 대부분 immutable(불변)한 특성을 가진다.

### undefined

선언은 되었지만 값이 할당되지 않은 상태

```ts
let value;

console.log(value); // undefined
```

---

### null

명시적으로 비어 있음을 나타내는 값

```ts
const value = null;
```

#### 왜 `typeof null === "object"` 인가?

JavaScript 초기 구현에서 발생한 버그로, 현재는 명세에 포함되어 유지되고 있다.

초기 JavaScript 엔진은 값의 타입을 하위 비트 태그로 구분했는데, 객체의 태그가 `000`이었고 `null`은 모든 비트가 `0`으로 표현되어 객체와 동일한 태그로 인식되었다. 그 결과 `typeof null`이 `"object"`를 반환하게 되었다.

ES4 시절 이를 `"null"`로 고치자는 제안이 있었으나, 기존 코드와의 호환성이 깨진다는 이유로 거부되어 현재까지 유지되고 있다.

> `typeof null`은 `"object"`를 반환하므로 `null` 여부는 `typeof`가 아닌 `value === null`로 확인한다.

#### `null`과 `undefined`의 차이는?

- `undefined`: 선언은 되었지만 값이 할당되지 않은 상태 (엔진이 자동 부여)
- `null`: 명시적으로 "비어 있음"을 나타내는 값 (개발자가 의도적으로 할당)

```ts
undefined == null; // true (== 는 두 값을 동등하게 취급)
undefined === null; // false (타입이 다름)
```

---

### boolean

참(`true`) 또는 거짓(`false`)을 표현하는 타입

---

### number

- JavaScript의 기본 숫자 타입
- 정수와 실수를 모두 표현

#### 왜 `0.1 + 0.2 === 0.3`은 false인가?

```ts
0.1 + 0.2; // 0.30000000000000004
```

`number`는 내부적으로 실수를 이진수로 저장한다. 그런데 `0.1`, `0.2` 같은 일부 소수는 이진수로 정확하게 표현할 수 없어 근삿값으로 저장되고, 이 오차가 연산 결과에 그대로 남는다.

> 이런 방식을 IEEE 754 부동소수점이라고 부른다. 이름 자체보다 "이진수로 정확히 표현되지 않아 근삿값으로 저장된다"는 원리가 핵심이다.

소수 비교가 필요할 때는 오차를 감안해서 비교한다.

```ts
Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON; // true
```

#### 안전한 정수 범위가 왜 `2^53 - 1`까지인가?

`number`가 정수를 정확히 표현할 수 있는 한계는 `2^53 - 1`(`Number.MAX_SAFE_INTEGER`)이다. 이 값을 넘는 정수는 정확하게 표현하지 못한다. 이 한계를 극복하기 위해 등장한 것이 `bigint`다.

---

### bigint

`number`의 안전한 정수 범위를 넘어서는 큰 정수를 표현하기 위한 타입

```ts
const value1 = BigInt(123);
const value2 = 123n;
```

#### 왜 `number`로 변환하면 데이터 손실이 발생하는가?

`number`는 표현할 수 있는 정수의 범위가 제한되어 있다(`2^53 - 1`). 큰 `bigint` 값을 `number`로 변환하면 일부 숫자를 정확하게 표현하지 못해 값이 변경될 수 있다.

#### 왜 `123n == 123`은 true이고 `123n === 123`은 false인가?

`==`는 비교 전에 타입을 변환하기 때문에 값이 같으면 `true`가 된다. 반면 `===`는 타입까지 비교하므로 `bigint`와 `number`는 서로 다른 타입으로 판단해 `false`가 된다.

---

### string

문자열을 표현하는 타입. 문자열은 immutable(불변)이다.

```ts
const name = "javascript";

name[1] = "b";

console.log(name); // javascript
```

#### immutable(불변)이란?

한 번 생성된 값을 변경할 수 없는 특성이다. 문자열을 수정하는 것처럼 보여도 기존 문자열이 바뀌는 것이 아니라 **새로운 문자열이 생성된다.**

#### 왜 문자열은 변경할 수 없는가?

원시 값은 값 자체로 저장되고 엔진이 이를 불변으로 다룬다. 그래서 `name[1] = "b"`처럼 인덱스로 접근해 값을 바꾸려 해도 조용히 무시된다. (strict mode에서는 이 할당이 `TypeError`를 던진다.)

#### 원시 타입인데 왜 `"abc".length`처럼 속성/메서드를 쓸 수 있는가?

원시 타입 자체는 메서드를 갖지 않는다. 하지만 문자열에 속성이나 메서드로 접근하는 순간, 엔진이 임시로 래퍼 객체(`String`)를 생성해 접근을 처리하고 곧바로 폐기한다. 이를 **오토박싱(autoboxing)** 이라고 한다.

---

### symbol

중복되지 않는 고유한 값을 생성하기 위한 원시 타입

```ts
// 같은 description이어도 Symbol()은 매번 새로운 심볼을 생성
Symbol("key") === Symbol("key"); // false

// Symbol.for()은 key를 기억해뒀다가 같은 key면 같은 심볼을 반환
Symbol.for("key") === Symbol.for("key"); // true
```

#### `Symbol()`은 왜 항상 새로운 값을 생성하는가?

항상 고유한 값을 생성하기 위해서다. 같은 설명(description)을 전달하더라도 서로 다른 심볼이 생성된다.

#### `Symbol.for()`는 무엇인가?

같은 key로 만든 심볼을 앱 전체에서 공유하게 해주는 메서드다. 심볼을 **전역 심볼 레지스트리(Global Symbol Registry)** 라는 보관함에 저장해두고, 같은 key로 부르면 기존 심볼을 꺼내 반환한다.

#### 언제 `Symbol.for()`를 사용하는가?

애플리케이션 전체에서 동일한 심볼을 공유해야 하는 경우 사용한다.

---

## 객체 타입 (Object Type)

객체 타입은 값 자체가 아니라 객체가 저장된 위치(reference)를 저장한다. 그래서 객체 타입을 **참조 타입(Reference Type)** 이라고도 한다.

```ts
const func1 = function () {};
const func2 = func1;

console.log(func1 === func2); // true (같은 참조)
```

```ts
const func1 = function () {};
const func2 = function () {};

console.log(func1 === func2); // false (다른 객체, 다른 참조)
```

#### 객체를 왜 참조 타입이라고 부르는가?

변수에는 객체 자체가 아니라 객체가 저장된 메모리 위치(reference)가 저장되기 때문이다.

#### 객체는 메모리에 어떻게 저장되는가?

객체는 메모리에 생성되고, 변수에는 해당 객체를 가리키는 참조가 저장된다. 여러 변수가 하나의 객체를 함께 참조할 수도 있다.

#### 함수도 객체인가?

그렇다. JavaScript에서 함수는 객체이며, 변수에 저장하거나 다른 함수의 인수로 전달할 수 있다(일급 객체).

#### 배열도 객체인가?

그렇다. 배열은 객체를 기반으로 만들어진 특수한 객체다.

#### 원시 타입과 객체 타입의 저장 방식은 어떻게 다른가?

- **원시 타입**: 값 자체를 저장한다.
- **객체 타입**: 객체가 저장된 메모리 위치(reference)를 저장한다.

---

## 객체 기반 자료구조 (Object / Array / Map / Set)

`Array`, `Map`, `Set`은 모두 객체를 기반으로 만들어진 특수한 객체다. 저장하는 방식과 잘하는 일이 다르다.

- **Object** : 이름(키)으로 값을 찾는다
- **Array** : 순서(인덱스)로 값을 찾는다
- **Map** : 이름으로 값을 찾되, 키에 아무 타입이나 쓸 수 있다
- **Set** : 값만 저장하고 중복을 허용하지 않는다

|             | Object                  | Array           | Map                     | Set             |
| ----------- | ----------------------- | --------------- | ----------------------- | --------------- |
| 무엇을 저장 | 키 - 값                 | 값 (순서 있음)  | 키 - 값                 | 값 (중복 없음)  |
| 키 타입     | 문자열, 심벌만          | 숫자 인덱스     | **모든 타입** (객체도)  | —               |
| 순서        | 보장되지 않음           | 보장            | **삽입 순서 보장**      | 삽입 순서 보장  |
| 크기 확인   | `Object.keys(o).length` | `arr.length`    | `map.size`              | `set.size`      |
| 중복        | 키 중복 불가            | 값 중복 허용    | 키 중복 불가            | **값 중복 불가**|
| 이터러블    | ❌                      | ✅              | ✅                      | ✅              |
| JSON 변환   | 바로 가능               | 바로 가능       | 불가 (변환 필요)        | 불가 (변환 필요)|

```ts
// Object
const obj = { name: "kim" };
obj.age = 20;

// Map
const map = new Map();
map.set("name", "kim");
map.get("name"); // 'kim'

// Set
const set = new Set([1, 2, 2, 3]);
console.log(set); // Set(3) {1, 2, 3} ← 중복 제거
```

#### Object 대신 Map을 쓰면 무엇이 좋은가?

- **키에 아무 타입이나 쓸 수 있다.** 객체는 키가 항상 문자열로 변환되지만, Map은 객체나 함수도 키로 쓸 수 있다.

```ts
const obj = {};
obj[1] = "a";
console.log(Object.keys(obj)); // ['1'] ← 숫자 1이 문자열로 변환됨

const map = new Map();
map.set(1, "a");
map.set("1", "b"); // 숫자 1과 문자열 '1'을 구분한다
```

- **프로토타입 키와 충돌하지 않는다.** 객체는 `toString`, `constructor` 같은 상속받은 키가 이미 존재한다.

```ts
const obj = {};
console.log(obj.toString); // ƒ toString() ← 넣은 적 없는데 값이 있다

const map = new Map();
console.log(map.get("toString")); // undefined
```

- **크기를 바로 알 수 있고**(`size`), **추가·삭제가 잦은 데이터에 유리하다.**

> 반대로 JSON으로 주고받는 데이터, 형태가 고정된 데이터는 Object가 편하다. Map은 `JSON.stringify`가 되지 않는다.

#### Set은 언제 쓰는가?

**중복 제거**와 **포함 여부 확인**에 쓴다.

```ts
const arr = [1, 2, 2, 3, 3];

const unique = [...new Set(arr)]; // [1, 2, 3] ← 가장 흔한 사용법
```

배열의 `includes`는 앞에서부터 전부 훑기 때문에 원소가 많을수록 느려지지만(O(n)), `Set.has`는 개수와 무관하게 빠르다(O(1)).

```ts
arr.includes(3); // 원소 개수에 비례해 느려진다
set.has(3); // 항상 빠르다
```

#### WeakMap과 WeakSet은 무엇인가?

키를 **약하게 참조**하는 Map/Set이다. 다른 곳에서 그 객체를 더 이상 참조하지 않으면, WeakMap에 들어 있어도 가비지 컬렉션의 대상이 된다.

```ts
let user = { name: "kim" };
const map = new Map();
const weak = new WeakMap();

map.set(user, "data"); // Map은 user를 붙잡고 있어 메모리에서 해제되지 않는다
weak.set(user, "data"); // WeakMap은 붙잡지 않는다

user = null; // weak에 있던 항목은 자동으로 정리된다
```

DOM 요소에 부가 정보를 붙여둘 때처럼, **원본이 사라지면 같이 사라져야 하는 데이터**에 쓴다. 키는 객체만 가능하고, 순회할 수 없다.
