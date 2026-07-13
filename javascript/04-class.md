# Class

JavaScript의 클래스 문법과 그 밑에서 동작하는 프로토타입을 정리한다.

---

## 클래스란

특정 객체를 반복적으로 만들기 위한 템플릿. 같은 구조의 객체를 여러 개 찍어내야 할 때 사용한다.

이 템플릿으로 만들어낸 객체 하나하나가 **인스턴스(instance)**다.

- 클래스 → 붕어빵 틀
- 인스턴스 → 그 틀로 구워낸 붕어빵

```ts
const user = new User("김개발"); // user는 User 클래스의 인스턴스
```

`new` 키워드로 클래스를 호출하면 **인스턴스가 생성**된다.

---

## constructor

인스턴스가 만들어질 때 **딱 한 번 자동으로 실행**되는 특수한 메서드. 이때 인스턴스의 초기값을 세팅한다.

`new User("김개발")`의 실행 순서를 보자.

```ts
class User {
  // ② constructor 함수가 자동 실행 (name 자리에 "김개발"이 들어옴)
  constructor(name) {
    this.name = name; // ③ 새로 만들어진 인스턴스에 name = "김개발" 을 채움
  }
}

const user = new User("김개발"); // ① new → 빈 인스턴스를 만들고 constructor 실행
// ④ 다 채워진 인스턴스를 user에 담음
```

`new`가 인스턴스를 만드는 행위라면, `constructor`는 그 인스턴스가 태어나는 순간 끼어들어 내용물(`name` 등)을 채워주는 담당자다. constructor가 없으면 `user.name`은 `undefined`.

- 클래스 안에 하나만 존재할 수 있다 (여러 개 정의하면 에러)
- 생략할 수 있다 (생략하면 빈 constructor가 자동으로 들어간다)

#### constructor는 JavaScript 내장 메서드인가?

아니다. `constructor`는 클래스 문법에서 미리 약속된 예약어 성격의 특수 메서드다.

이 이름으로 정의해두면 `new`로 인스턴스를 만들 때 엔진이 알아서 호출해준다. `myObject.constructor()`처럼 우리가 직접 부르는 게 아니라, `new`가 대신 불러주는 것.

#### "인스턴스 생성 시 constructor 내부에 빈 객체가 할당된다"는 무슨 뜻인가?

앞의 순서를 조금 더 뜯어보면 이렇다.

1. 빈 객체 `{}`를 하나 만든다
2. 그 빈 객체를 constructor 안의 `this`에 연결한다
3. constructor 코드(`this.name = name` 등)를 실행해 빈 객체를 채운다
4. 완성된 객체를 자동으로 반환한다

결국 `this`가 그 "빈 객체"이고, `this.name = name`은 빈 객체에 프로퍼티를 하나씩 채워 넣는 과정.

---

## 프로퍼티

클래스로 인스턴스를 생성할 때 인스턴스 내부에 정의하는 속성값(키-값)

```ts
class User {
  // 클래스 프로퍼티 선언 (기본값)
  name = "";
  level = 1;

  constructor(name, level) {
    this.name = name;
    this.level = level;
  }

  introduce() {
    console.log(`${this.name}, 레벨 ${this.level}`);
  }
}

const user = new User("김개발", 5);
user.introduce(); // 김개발, 레벨 5
```

---

## getter와 setter

- **getter**: 클래스에서 값을 **가져올 때** 동작하는 메서드 (`get`)
- **setter**: 클래스 필드에 값을 **할당할 때** 동작하는 메서드 (`set`)

메서드처럼 정의하지만, 사용할 때는 `()` 없이 **일반 프로퍼티처럼** 접근한다.

```ts
class Temperature {
  constructor(celsius) {
    this._celsius = celsius; // 관례상 내부 값은 _를 붙임
  }

  // 가져올 때: 섭씨를 화씨로 변환해서 반환
  get fahrenheit() {
    return this._celsius * 1.8 + 32;
  }

  // 할당할 때: 화씨로 넣으면 섭씨로 변환해서 저장
  set fahrenheit(value) {
    this._celsius = (value - 32) / 1.8;
  }
}

const t = new Temperature(25);
console.log(t.fahrenheit); // 77   ← get 호출 (괄호 없음)

t.fahrenheit = 212; // ← set 호출
console.log(t._celsius); // 100
```

`_celsius`처럼 내부 값에 `_`를 붙이는 것은 "직접 건드리지 말고 getter/setter로 접근하라"는 관례상의 표시다.

#### getter/setter는 클래스에서만 있는 개념인가?

아니다. 클래스 전용이 아니라 JavaScript 전반의 기능이라, **객체 리터럴에서도** 그대로 쓸 수 있다.

```ts
const person = {
  _name: "김개발",
  get name() {
    return this._name;
  },
  set name(value) {
    this._name = value;
  },
};

person.name; // "김개발"  (get)
person.name = "이코딩"; // (set)
```

`Object.defineProperty()`로도 정의할 수 있다. 클래스는 이 기능을 담아 쓰는 여러 장소 중 하나일 뿐.

---

## 인스턴스 메서드

클래스 내부에 선언한 메서드. JavaScript의 **프로토타입에 등록**되기 때문에 **프로토타입 메서드**라고도 부른다.

```ts
class Person {
  constructor(name) {
    this.name = name;
  }

  // 인스턴스 메서드
  hello() {
    console.log(`내 이름은 ${this.name}`);
  }
}

const kim = new Person("김개발");
kim.hello(); // 내 이름은 김개발
```

`kim` 객체 안에는 `hello`를 직접 선언한 적이 없는데도 `kim.hello()`가 동작한다. 어떻게 가능한 걸까?

#### 인스턴스에 없는 메서드를 어떻게 호출하는가? (프로토타입)

핵심은 **메서드가 인스턴스마다 복사되지 않고, 프로토타입이라는 공용 창고에 딱 한 번만 저장된다**는 것이다.

```ts
const a = new Person("김개발");
const b = new Person("이코딩");
// hello()는 a, b 안에 각각 들어있는 게 아니라
// Person.prototype에 하나만 있고, a와 b가 그것을 공유한다
```

`a.hello()`를 호출하면 엔진은 이렇게 찾는다.

1. `a` 객체 자신에게 `hello`가 있나? → 없다
2. `a`의 프로토타입(`Person.prototype`)에 있나? → 있다 → 실행

인스턴스에 없으면 프로토타입으로 올라가 찾는 구조 덕분에, 직접 선언하지 않은 메서드도 호출할 수 있다.

이때 메서드 안의 `this`는 **호출한 인스턴스**(`a`)를 가리킨다. 그래서 `this.name`으로 각 인스턴스의 값에 접근할 수 있다.

> 정리하면: 메서드는 프로토타입에 한 번만 저장해 메모리를 아끼고, `this`는 호출한 인스턴스를 가리켜 각자의 데이터를 다룬다.

#### `__proto__`는 왜 쓰지 말라고 하는가?

`__proto__`는 프로토타입에 접근하는 옛날 방식이다. `typeof null`이 `"object"`인 것처럼, 과거 브라우저 호환성 때문에 남아있는 기능이다.

표준이 아니었다가 나중에 마지못해 명세에 편입된 것이라, 실무에서는 대신 `Object.getPrototypeOf()` / `Object.setPrototypeOf()`를 쓴다.

---

## 프로토타입 체이닝

인스턴스에 직접 선언하지 않은 메서드라도, 프로토타입을 타고 올라가며 찾아 실행해주는 동작이 **프로토타입 체이닝**이다.

자기 자신 → 프로토타입 → 그 위의 프로토타입 → ... → 최상위 `Object`까지 훑는다. 중간에 찾으면 실행, 끝까지 없으면 `undefined`(또는 에러).

```
kim ──(없음)──▶ Person.prototype ──(hello 발견!)──▶ 실행
                        │
                        ▼ (여기서 못 찾으면)
                 Object.prototype ──▶ 최상위
```

`kim.hello()`가 동작한 것도 이 체이닝을 거쳐 `Person.prototype`의 `hello`를 찾아냈기 때문이다.

---

## 정적 메서드 (static)

인스턴스가 아니라 **클래스 자체에 속하는** 메서드. 그래서 인스턴스가 아닌 **클래스 이름으로 호출**한다.

```ts
class Person {
  constructor(name) {
    this.name = name;
  }

  hello() {
    console.log(`내 이름은 ${this.name}`);
  }

  static hi() {
    console.log("하이");
  }
}

const kim = new Person("김개발");

kim.hello(); // 내 이름은 김개발   (인스턴스 메서드)
Person.hi(); // 하이            (정적 메서드 - 클래스로 호출)
kim.hi(); // TypeError: kim.hi is not a function
```

정적 메서드는 인스턴스에 속하지 않으므로 `kim.hi()`처럼 인스턴스로는 호출할 수 없다.

또 클래스에 매여 있어서, 내부에서 `this`로 인스턴스 프로퍼티(`this.name`)에 접근하는 것도 안 된다.

> 특정 인스턴스와 무관한 유틸성 기능을 묶을 때 쓴다. `Math.random()`, `Array.from()`처럼 우리가 이미 쓰던 것들이 정적 메서드다.

---

## 상속 (extends)

기존 클래스를 물려받아, 자식 클래스에서 기능을 확장하는 것.

```ts
// 부모 클래스
class Person {
  constructor(name) {
    this.name = name;
  }

  introduce() {
    console.log(`안녕하세요, 제 이름은 ${this.name}입니다.`);
  }
}

// 자식 클래스
class Student extends Person {
  constructor(name, grade) {
    super(name); // 부모의 constructor 호출
    this.grade = grade;
  }

  study() {
    console.log(`${this.name}은(는) ${this.grade}학년입니다.`);
  }
}

const student = new Student("김개발", 3);
student.introduce(); // 부모에게 물려받은 메서드
student.study(); // 자식이 새로 추가한 메서드
```

- `extends`: 어떤 클래스를 상속받을지 지정한다
- `super(name)`: 자식의 constructor에서 **부모의 constructor를 먼저 실행**한다. 부모가 세팅하는 값(`this.name`)을 물려받으려면 반드시 호출해야 한다

자식은 부모의 메서드(`introduce`)를 그대로 쓰면서, 자기만의 메서드(`study`)를 추가로 가진다. 부모 메서드를 찾는 과정 역시 앞서 본 프로토타입 체이닝이다.

---

## 클래스는 문법적 설탕이다

원래 JavaScript는 **프로토타입 기반 언어**라 클래스 개념이 없었다. 프로토타입으로 객체를 만들고 상속을 구현했다.

하지만 Java 같은 객체지향 언어에 익숙한 개발자들이 클래스 문법을 원했고, ES6(ECMAScript 2015)에서 클래스가 도입되었다. 덕분에 다른 객체지향 언어와 비슷한 형태로 코드를 쓸 수 있게 됐다.

핵심은, 클래스가 프로토타입을 **대체한 게 아니라는** 점이다. 클래스도 내부적으로는 여전히 프로토타입으로 동작한다. 겉모습만 클래스처럼 감싼 것이라, 이를 **문법적 설탕(syntactic sugar)** 이라 부른다.

> 클래스는 프로토타입에 익숙하지 않은 개발자가 JavaScript에 더 쉽게 접근하도록 도와주는 문법적 설탕이다.

---

## 요약

- **클래스**는 같은 구조의 객체(**인스턴스**)를 찍어내는 템플릿이고, `new`로 인스턴스를 만든다.
- **constructor**는 내장 메서드가 아니라 예약된 특수 메서드이며, `new` 시점에 빈 객체(`this`)를 만들어 프로퍼티로 채운다.
- **인스턴스 메서드**는 각 인스턴스에 복사되지 않고 **프로토타입에 한 번만** 저장되어 공유된다. 인스턴스에 없으면 **프로토타입 체이닝**으로 찾아 올라간다.
- **정적 메서드(static)** 는 클래스 자체에 속해 클래스 이름으로 호출하며, 인스턴스로는 호출할 수 없다.
- **상속(extends)** 은 부모 클래스를 확장하며, `super()`로 부모 constructor를 호출한다.
- 클래스는 프로토타입을 대체하지 않는다. 프로토타입 위에 씌운 **문법적 설탕**이다.
