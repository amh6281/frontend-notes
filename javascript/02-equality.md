# Equality

JavaScript의 동등 비교와 React가 이를 활용하는 방식을 정리한다.

React의 가상 DOM 비교, 컴포넌트 리렌더링 여부 판단, 변수와 함수의 메모이제이션 등은 모두 JavaScript의 동등 비교를 기반으로 한다.

## 개념 관계

동등 비교와 얕은/깊은 비교는 같은 층위의 개념이 아니다.

- **동등 비교(`==`, `===`, `Object.is`)**: JavaScript가 기본 제공하는 비교. 값 하나 대 값 하나를 비교하는 기본 도구다. 객체는 **참조(주소)만** 비교한다.
- **얕은 비교 / 깊은 비교**: 객체의 **내용**을 비교하기 위한 전략. 프로퍼티를 순회하며 각 값을 동등 비교로 확인한다. 즉 동등 비교를 **부품으로 사용**한다.

```
동등 비교 (JS 기본 연산)          ← 값 1개 비교 (기본 도구)
├─ ==        타입 변환 후 값 비교
├─ ===       타입 + 값 비교
└─ Object.is ===와 유사, NaN/-0 보정
        │
        │  (객체는 참조만 비교 → 내용 비교 불가)
        ▼
객체 내용을 비교하는 전략           ← 동등 비교를 반복해서 조립
├─ 얕은 비교(shallow)  1 depth 프로퍼티까지
└─ 깊은 비교(deep)      중첩 객체까지 재귀
```

---

## 1. 동등 비교 (Equality)

두 값이 같은지 판단하는 것을 의미한다. JavaScript에는 대표적으로 세 가지 방식이 있다.

### == (Abstract Equality)

값을 비교하기 전에 필요하면 타입을 변환한 후 비교한다.

```ts
1 == "1"; // true
false == 0; // true
null == undefined; // true
```

#### 왜 예상과 다른 결과가 나오는가?

`==`는 두 값의 타입이 다르면 JavaScript 엔진이 내부 규칙에 따라 타입을 변환한 뒤 비교하기 때문이다. 예측하기 어려운 결과가 발생할 수 있으므로 일반적으로 사용을 권장하지 않는다.

### === (Strict Equality)

타입 변환 없이 값과 타입을 모두 비교한다.

```ts
1 === "1"; // false
1 === 1; // true
```

#### 그럼 `===`만 쓰면 되는 것 아닌가?

대부분의 경우 그렇다. 하지만 `===`도 두 가지 특별한 값은 직관과 다르게 비교한다.

- `NaN === NaN`은 `false` — NaN은 자기 자신과도 같지 않다
- `-0 === +0`은 `true` — 부호가 다른 0을 같다고 본다

이 두 경우를 의도대로 비교하려면 `Object.is()`를 사용한다. (아래에서 자세히 다룬다)

### Object.is()

대부분 `===`와 동일하게 동작하지만 `NaN`과 `-0`을 올바르게 비교한다.

#### 이름이 `Object.is`인데 원시값도 비교할 수 있는가?

가능하다. 여기서 `Object`는 "객체를 비교한다"는 뜻이 아니라, 이 함수가 `Object` 전역 객체에 속해 있다는 위치 표시일 뿐이다. (`Object.keys`, `Object.assign`과 같은 맥락)

오히려 `Object.is`는 원시값 비교에서 진가를 발휘한다.

```ts
Object.is(1, 1); // true  (숫자)
Object.is("Lee", "Lee"); // true  (문자열)
Object.is(NaN, NaN); // true
Object.is(-0, +0); // false
```

- 원시값 → 값 자체를 비교
- 객체 → 참조를 비교

```ts
Object.is(NaN, NaN); // true
NaN === NaN; // false
```

#### 왜 `NaN === NaN`은 false인가?

`NaN`은 "숫자가 아님(Not-a-Number)"을 나타내는 특수한 값이다. IEEE 754 규격에서 모든 NaN은 자기 자신과도 같지 않다고 정의되어 있다. `Object.is()`는 이 예외를 보완하여 `true`를 반환한다.

#### 왜 `Object.is(-0, +0)`은 false인가?

`===`는 `+0`과 `-0`을 같은 값으로 판단하지만, `Object.is()`는 두 값을 서로 다른 값으로 구분한다.

```ts
-0 === +0; // true
Object.is(-0, +0); // false
```

> `Object.is()`도 객체는 참조만 비교한다. `===`보다 특별한 값 처리만 정확할 뿐, 객체 내용 비교는 하지 못한다.

---

## 2. 참조 비교의 한계

세 방식 모두 **객체에 대해서는 참조(주소)만 비교**한다. 즉 내용이 같아도 서로 다른 객체면 다르다고 판단한다.

```ts
const obj1 = { a: 1 };
const obj2 = { a: 1 };

Object.is(obj1, obj2); // false (내용은 같지만 다른 객체)
```

```ts
const obj1 = { a: 1 };
const obj2 = obj1;

Object.is(obj1, obj2); // true (같은 객체를 참조)
```

객체의 내용이 같은지 확인하려면 동등 비교만으로는 부족하고, 별도의 비교 전략(얕은 비교 / 깊은 비교)이 필요하다.

---

## 3. 얕은 비교 vs 깊은 비교 (내용 비교 전략)

객체의 프로퍼티를 순회하며 값을 비교하는 방식이다. 어디까지 파고드는지에 따라 나뉜다.

- **얕은 비교(shallow)**: 최상위(1 depth) 프로퍼티까지만 비교
- **깊은 비교(deep)**: 중첩된 객체까지 재귀적으로 비교

> 얕은 비교와 깊은 비교는 별개의 비교 방법이 아니라, 프로퍼티마다 동등 비교(`Object.is`)를 반복하는 것이다.

### 얕은 비교 — 1 depth

```ts
const obj1 = { name: "Lee" };
const obj2 = { name: "Lee" };

Object.is(obj1, obj2); // false
shallowEqual(obj1, obj2); // true
```

`Object.is()`는 참조만 비교하므로 서로 다른 객체로 판단해 `false`를 반환한다. 반면 `shallowEqual()`은 최상위 프로퍼티를 순회하며 값을 비교하고, `obj1.name`과 `obj2.name`이 모두 `"Lee"`이므로 `true`를 반환한다.

### 얕은 비교 — 2 depth

```ts
const obj1 = { user: { name: "Lee" } };
const obj2 = { user: { name: "Lee" } };

shallowEqual(obj1, obj2); // false
```

최상위 프로퍼티는 `user`다. 얕은 비교는 `user.name`까지 파고들지 않고 `user` 객체의 참조만 비교한다. `obj1.user`와 `obj2.user`는 서로 다른 객체이므로 `false`를 반환한다.

### 얕은 비교는 어떻게 동작하는가?

얕은 비교는 새로운 비교 도구가 아니라, **각 프로퍼티 값을 `Object.is`로 하나씩 비교**하는 것이다. React 내부에는 이를 구현한 `shallowEqual` 함수가 있다.

- `Object.is` → 값 하나 대 값 하나를 비교하는 기본 도구
- `shallowEqual` → 객체의 1 depth 프로퍼티를 순회하며 각 값을 `Object.is`로 비교하는 함수 (React 내부 함수)

키 개수를 먼저 확인하고, 각 프로퍼티 값을 `Object.is`로 비교한다.

```ts
function shallowEqual(objA, objB) {
  const keysA = Object.keys(objA);
  const keysB = Object.keys(objB);

  // 1. 키 개수가 다르면 다른 객체
  if (keysA.length !== keysB.length) return false;

  // 2. 각 프로퍼티 값을 Object.is로 비교 (1 depth만)
  for (const key of keysA) {
    if (!Object.is(objA[key], objB[key])) return false;
  }

  return true;
}
```

값이 원시값이면 내용까지 비교되지만, 값이 객체면 `Object.is`가 참조만 비교하므로 중첩 내용은 잡지 못한다.

### 깊은 비교는 어떻게 동작하는가?

얕은 비교와 같지만, 값이 또 객체면 그 안으로 **재귀**해서 다시 비교한다.

```ts
function deepEqual(objA, objB) {
  const keysA = Object.keys(objA);
  const keysB = Object.keys(objB);

  if (keysA.length !== keysB.length) return false;

  for (const key of keysA) {
    const valA = objA[key];
    const valB = objB[key];

    // 값이 또 객체면 → 안으로 재귀해서 비교
    if (typeof valA === "object" && typeof valB === "object") {
      if (!deepEqual(valA, valB)) return false;
    } else {
      // 원시값이면 그대로 비교
      if (!Object.is(valA, valB)) return false;
    }
  }

  return true;
}
```

### 실무에서는 무엇을 쓰는가?

보통 직접 구현하지 않고 이미 만들어진 것을 사용한다.

- **얕은 비교**: `React.memo`, `PureComponent`가 내부적으로 `shallowEqual`을 사용한다. 직접 감싸기만 하면 얕은 비교가 적용된다.
- **깊은 비교**: Lodash의 `_.isEqual()`을 가장 많이 사용한다. `JSON.stringify()`로 비교하는 방법도 있으나 키 순서·함수·`undefined` 등을 제대로 다루지 못해 한계가 있다.

### 왜 깊은 비교를 기본으로 쓰지 않는가?

깊은 비교는 객체 전체를 재귀적으로 탐색해야 하므로, 객체가 크고 깊을수록 비용이 커진다. 최상위 값만 비교해도 대부분의 변경을 감지할 수 있기 때문에, 성능을 위해 얕은 비교를 기본으로 사용한다.

---

## 4. React에서의 활용

React는 state와 props의 변경 여부를 판단할 때 이 비교 방식을 사용한다.

기본적으로 `Object.is()`로 비교하되, 객체는 참조만 비교해서는 내용 변경을 알 수 없으므로 얕은 비교를 한 번 더 수행한다.

### 비교 과정

1. `Object.is()`로 먼저 비교
2. 객체인 경우 얕은 비교(`shallowEqual`)
3. 변경 여부를 판단해 리렌더링 여부 결정

이 방식은 `React.memo`, `PureComponent` 등에서 사용된다.

### 결국 `Object.is`를 반복하는 것이다

두 단계로 나뉘어 보이지만, 사용하는 도구는 처음부터 끝까지 `Object.is` 하나다. 적용하는 **대상**만 "전체 → 각 프로퍼티"로 바뀔 뿐이다.

```ts
// 1단계: 전체를 Object.is로 비교 (원시값이면 여기서 끝)
Object.is(prev, next);

// 2단계: 객체면 shallowEqual → 그 안에서 프로퍼티마다 다시 Object.is
function shallowEqual(prev, next) {
  for (const key of Object.keys(prev)) {
    if (!Object.is(prev[key], next[key])) return false; // 또 Object.is
  }
  return true;
}
```

| 단계              | 비교 대상          | 도구                |
| ----------------- | ------------------ | ------------------- |
| 1단계             | 값 / 객체 전체     | `Object.is` 한 번   |
| 2단계 (얕은 비교) | 객체의 각 프로퍼티 | `Object.is` 여러 번 |

### 비교 결과와 리렌더링 방향

이전 값과 새 값을 비교해서 **다르면** 변경으로 판단하고 리렌더링한다. **같으면** 변경이 없다고 보고 건너뛴다.

- 비교 결과 **다름** → 리렌더링 O
- 비교 결과 **같음** → 리렌더링 X (건너뜀)

> `React.memo`는 "props가 같으면 건너뛴다"고 설명되지만, 결국 "다르면 리렌더링"과 같은 말이다.

### 불변성이 중요한 이유

React가 얕은 비교(참조 비교)에 의존하기 때문에, 상태를 변경할 때는 기존 객체를 직접 수정하지 않고 **새 객체를 만들어야** 변경이 감지된다.

기존 객체를 직접 수정하면 값은 바뀌어도 **참조가 그대로**라, React가 "같다"고 판단해 리렌더링이 일어나지 않는다.

```ts
// ❌ 참조가 그대로 → "같다"고 판단 → 리렌더링 X
state.count = 1;

// ✅ 새 객체 → 참조가 달라짐 → "다르다"고 판단 → 리렌더링 O
setState({ ...state, count: 1 });
```

### 왜 참조가 그대로인가 (자료구조 관점)

JavaScript는 값을 저장하는 위치가 타입에 따라 다르다.

- **원시값**: 스택(stack)에 값 자체를 저장
- **객체**: 힙(heap)에 실제 데이터를 저장하고, 변수(스택)에는 그 주소(참조)만 저장

```
원시값:  변수 → [스택] 값
객체:    변수 → [스택] 주소 → [힙] 실제 객체
```

React는 변경 감지를 위해 이 **참조(주소)** 를 `Object.is`로 비교한다.

- 객체를 직접 수정 → 힙 내용만 바뀌고 **주소는 그대로** → "같다" 판단 → 리렌더링 X
- 새 객체 생성 → 힙에 새로 만들어져 **주소가 바뀜** → "다르다" 판단 → 리렌더링 O

```ts
// ❌ 힙 내용만 변경, 주소 동일 → 감지 못 함
state.todos.push(newTodo);

// ✅ 새 배열 → 새 주소 → 감지됨
setTodos([...state.todos, newTodo]);
```

정리하면, React가 성능을 위해 **얕은 비교(참조 비교)** 를 선택했기 때문에, 개발자는 상태를 바꿀 때 참조를 새로 만들어(불변성 유지) 변경을 감지시켜야 한다.

> 스택/힙은 엔진의 메모리 관리를 단순화한 개념 모델이다. 실제 엔진(V8 등)은 최적화에 따라 다르게 동작할 수 있다.

> 참고: React는 기본적으로 부모가 리렌더링되면 자식도 리렌더링된다. `React.memo`로 감쌌을 때 비로소 props를 얕은 비교해 리렌더링을 건너뛸지 판단한다. (자세한 렌더링 동작은 rendering 노트 참고)
