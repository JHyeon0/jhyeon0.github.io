---
title:  "프로퍼티 어트리뷰트와 생성자 함수"
excerpt: "프로퍼티 어트리뷰트와 자바스크립트에서 함수의 두 가지 호출방법에 대해서 알아본다."
categories:
  - javascript
tags:
  - javascript
---

## 프로퍼티 어트리뷰트

내부 슬롯internal slot과 내부 메서드internal method는 자바스크립트 엔진의 내부 로직으로 실제로 동작하지만 개발자 직접 접근할 수 있는 객체의 프로퍼티는 아니다. 자바스크립트 엔진은 프로퍼티를 생성할 대 프로퍼티의 상태를 나타내는 프로퍼티 어트리뷰트를 기본값으로 자동 정의한다. Property Attribute란 자바스크립트 엔진이 관리하는 내부 상태 값meta-property인 내부 슬롯 [[Value]], [[Writable]], [[Enumerable]], [[Configurable]]이다. 이중 대괄호는 ECMAScript에서 등장하는 내부 슬롯과 내부 메소드를 뜻한다. 직접 접근할 수는 없지만, `Object.getOwnPropertyDescriptor` 혹은 `Object.getOwnPropertyDescriptors` 메서드로 프로퍼티 디스크립터 객체를 반환받아 간접적으로 확인할 수 있다.

```js
const person = {
	name: 'Lee'
  age: 20
}

console.log(Object.getOwnPropertyDescriptors(person));
```

### 데이터 프로퍼티와 접근자 프로퍼티

프로퍼티는 데이터 프로퍼티와 접근자 프로퍼티로 구분할 수 있다. 

- 데이터 프로퍼티data property : key-value 형태의 일반적인 프로퍼티이다.
  - [[Value]] : 프로퍼티 키에 접근했을 때 반환되는 값
  - [[Writable]] : [[Value]] 값 변경 가능 여부(boolean)
  - [[Enumerable]] : 프로퍼티 열거 가능 여부(boolean)
  - [[Configurable]] : 프로퍼티 재정의 가능 여부(boolean)
- 접근자 프로퍼티accessor property : 자체적인 값은 없고 다른 데이터 프로퍼티 값을 읽거나 저장할 때 호출되는 접근자 함수accessor function로 구성된 프로퍼티이다.
  - [[Get]] : 접근자 프로퍼티로 데이터 프로퍼티 값을 읽을 때 호출되는 접근자 함수
  - [[Set]] : 접근자 프로퍼티로 데이터 프로퍼티 값을 저장할 때 호출되는 접근자 함수
  - [[Enumerable]], [[Configurable]] : 데이터 프로퍼티와 동일

```js
const person = {
  // data property
  firstName: "gildong",
  lastName: "hong",
  
  // accessor property
  // getter
  get fullName() {
    return `${this.firstName} ${this.lastName}`
  }
  //setter
  set fullName(name){
    [this.firstName, this.lastName] = name.split(' ');
  }
}

person.fullName = "ha donghun"
console.log(person) // { firstName: "ha", lastName="donghun" }
console.log(person.fullName) // ha donghun
```

`Object.defineProperty` 메서드를 사용하면 프로퍼티의 어트리뷰트를 개발자가 직접 명시적으로 정의하거나 기존 프로퍼티 어트리뷰트를 재정의할 수 있다. 즉, 개발자가 객체 프로퍼티의 동작을 더 명확하게 정의할 수 있다는 것이다.

### 객체 변경 방지

자바스크립트 객체는 mutable하지만 자바스크립트에서 객체의 변경을 방지하는 다양한 메서드들이 제공된다.

- `Object.preventExtionsions` : 새로운 프로퍼티 추가만 금지되면 기존 객체와 동일하게 동작
- `Object.seal` : 프로퍼티 추가, 삭제, 프로퍼티 어트리뷰트 재정의가 금지된다.
- `Object.freeze` : 프로퍼티 값을 읽는 것을 제외하고 객체의 모든 변경을 금지한다.

다만, `Object.freeze`로 중첩 객체까지 동결할 수 없으므로 중첩 객체까지 동결하려면 재귀적으로 객체의 객체를 `Object.freez` 메소드로 모두 동결시켜야 한다.



## 생성자 함수

### 생성자 함수

자바스크립트에서 객체 리터럴에 의한 객체 생성 방식이 가장 간단하고 일반적이다. 자바스크립트에서는 생성자 함수를 사용하여 객체를 만들 수도 있다.

```js
//객체 리터럴에 의한 객체 생성
const person1 = {
  name: "jhyeon",
  age: 2,
}

//생성자 함수에 의한 객체 생성
const person2 = new Object();
person2.name = 'lee'

//생성자 함수에 의한 객체 생성
function Person(name) {
  this.name = name;
  this.sayHello = function () {
    console.log(`hello ${name}!`)
  }
}

// 생성자 함수로서 함수 호출
const person3 = new Person('gildong');
// 일반 함수 호출
Person('tony')
```

객체 리터럴이 아닌 생성자 함수로 객체를 생성할 경우 클래스에서 인스턴스를 생성하는 것과 비슷해 프로퍼티 구조가 동일한 여러 객체를 만드는데 유용하다. 자바스크립트에서 **함수가 `new` 연산자와 함께 호출되면 해당 함수는 생성자 함수로 동작**한다. `new` 연산자 없이 함수ㅜ가 호출될 경우 생성자 함수가 아니라 일반 함수로서 동작한다.

함수가 생성자 함수로서 호출될 때(일반 함수로 호출될 때는 해당 되지 않음) 자바스크립트 엔진은 아래와 같은 순서로 인스턴스를 생성하여 반환한다.

1. 런타임 이전에 암묵적으로 완성되지 않은 빈 객체가 생성되며 이 인스턴스는 this에 바인딩된다. 바인딩이란 식별자와 값을 연결하는 과정을 의미한다. 따라서 함수 내부의 this가 생성된 인스턴스를 가리키게 된다.
2. 생성자 함수의 코드가 한 줄씩 실행되며 this에 바인딩되어 있는 인스턴스를 초기화한다. 즉, 작성한 코드대로 생성자 함수가 인수로 전달받은 초깃값을 인스턴스 프로퍼티에 할당하거나 인스턴스에 프로퍼티 메서드를 추가하는 작업이 이루어지는 과정이다.
3. 생성자 함수 내부에서 모든 처리가 끝나면 (`return this`가 없더라도) 바인딩된 `this`가 암묵적으로 반환된다. 만약 개발자가 임의로 명시한 객체를 반환한다면 this가 아니라 명시한 객체가 반환된다.하지만 원시 값의 반환은 무시되고 `this`가 반환된다. `this`가 아닌 다른 값을 반환하는 것은 생성자 함수의 목적에 어긋나므로, 만약 생성자 함수로 사용할 것이라면 `return`문은 생략해야 한다.

```js
function Person(name) {
  // 1. 암묵적으로 빈 객체 생성 후 this에 바인딩
  // 2. this에 바인딩되어 있는 인스턴스를 초기화한다.
  this.name = name;
  this.sayHello = function () {
    console.log(`hello ${name}!`)
  };
  
  // 3. 암묵적으로 this 반환
}

// 생성자 함수로서 함수 호출(인스턴스 생성)
const person = new Person('kim');
```

생성자 함수로 함수를 호출할 때는 this는 생성자 함수가 생성할 인스턴스를 가리킨다. 하지만 일반 함수로서 호출할 경우 함수 내부의 this 전역 객체를 가리키게 된다. 생성자 함수로 사용될 함수는 클래스와 같이 파스칼 케이스로 명명하여 일반 함수와 구분할 수 있게 한다. 또한 ES6의 `new.target` 을 사용하여 생성자 함수가 new 연산자 없이 호출되는 것을 방지할 수 있다. 



### constructor와 non-constructor

자바스크립트에서 함수는 일반 객체가 가지고 있는 내부 슬롯과 메서드를 모두 가지고 있기 때문에 일반 객체와 동일하게 동작할 수 있다. 아래와 같이 함수는 프로퍼티와 메서드를 모두 소유할 수 있지만 일반 객체와 다르게 **호출할 수 있다**.

```js
function foo() {}
foo.myDataProperty = 10;
foo.myMethod = function() { console.log("hi"); };
```

이는 함수 객체는 일반 객체가 가진 내부 슬롯과 메서드를 포함하여 함수로서 동작하기 위한 함수 객체만을 위한 내부 슬롯과 내부 메서드를 추가로 가지고 있기 때문이다. 앞서 자바스크립트에서 함수 선언문 또는 함수 표현식으로 정의한 함수를 호출할 때 `new` 연산자의 사용 여부에 따라 생성자 함수로 호출하여 객체를 생성할지 일반 함수로서 호출할 지 결정할 수 있다는 것을 살펴보았다. 이는 함수가 일반 함수로서 호출되면 함수 객체 내부 메서드 [[Call]]이 호출되고 생성자 함수로 호출되면 내부 메서드 [[Construct]]가 호출되기 때문이다.

자바스크립트에서 모든 함수 객체는 내부 메서드 [[Call]]을 가지고 있어 일반함수로 호출할 수 있다. 그러나 모든 함수 객체가 [[Construct]]를 갖는 것은 아니다. 내부 메서드 [[Construct]]를 가지는 함수를 constructor라고 부르며 [[Construct]]가 없으면 non-constructor라고 부른다. 즉, 생성자 함수로서 호출할 수 없는 함수 객체가 있다는 뜻이다.

- constructor: 함수 선언문, 함수 표현식, 클래스(클래스도 함수임)
- non-constructor: 메서드(ES6 메서드 축약 표현), 화살표 함수

여기서 메서드는 ECMAScript 사양에서 인정하는 좁은 의미의 메서드이며 ES6 메서드 축약 표현만을 의미한다.

```js
// 메서드 : ES6 축약 표현, non-constructor
const obj = {
  foo() {}
}
new obj.foo(); // 가능

// 메서드 아님 : foo 값으로 할당된 것은 일반 함수로 취급, constructor
const obj2 {
  foo: function() {}
}
new obj2.foo(); // 불가능 TypeError: obj2.foo is not a constructor
```