---
title:  "자바스크립트 프로토타입 #1"
excerpt: "자바스크립트에서 프로토타입이 무엇인지 알아본다. 그런데 일급 객체도 곁들인"
categories:
  - javascript
tags:
  - javascript
---

## 일급 객체
프로토타입에 대해서 알아보기 전에 일급 객체가 무엇인지 잠깐 보고 가자. 아래와 같은 조건을 만족하면 **일급 객체**라고 부른다.
1. 무명의 리터럴로 생성할 수 있다.(런타임에 생성이 가능하다)
2. 변수나 자료구조에 저장할 수 있다.
3. 함수의 매개변수에 전달할 수 있다.
4. 함수의 반환값으로 사용할 수 있다.

자바스크립트에서 함수는 위 조건을 모두 만족한다. 자바스크립트에서 함수는 일급 객체다. 즉, 함수는 객체로서 프로퍼티를 가질 수 있다는 뜻으로, 브라우저에서 `console.dir()` 로 함수 객체 프로퍼티를 확인해보면 (내부 슬롯을 제외하고) 아래와 같이 할 수 있다.

- `arguments` : 함수 객체 고유의 데이터 프로퍼티로 함수 호출 시 전달된 인자들의 정보를 담고 있는 순회 가능한 유사 배열이다. 함수 내부에서  `arguments` 라는 이름으로 지역 변수처럼 사용할 수 있다. ES6에서 Rest 파라미터를 사용할 수 있게 되면서 `argument`의 활용도가 줄었다.

  ```js
  function sum() {
    let sum = 0;
    // arguments는 배열이 아니므로 배열 메서드를 사용할 수 없지만 for문 순회는 가능
    for(let i=0; i<arguments.length; i++) {
      sum += arguments[i];
    }
    return sum
  }
  
  sum(1,2,3,4) // 10
  
  function sumES6(...args) {
    // Rest 파라미터를 사용하면 arguments 객체를 배열로 바꾸는 과정이 필요없이 배열 메서드 사용 가능
    return args.reduce((pre, cur) => pre+cur, 0)
  }
  
  sumES6(1,2,3,4) // 10
  ```

- `caller` : 함수 객체 고유의 데이터 프로퍼티로 함수 자신을 호출한 함수를 가리킨다. 비표준 프로퍼티이다.

- `length` : 함수 객체 고유의 데이터 프로퍼티로 함수를 정의할 때 선얺나 매개변수의 개수이다.

- `name` : 함수 객체 고유의 데이터 프로퍼티로 함수 이름을 나타낸다. 다만, ES6 이후부터는 익명함수 표현식이라도 함수 객체를 가리키는 변수 이름을 값으로 가진다.

  ```js
  var foo = function() {};
  console.log(foo.name) // foo ES6 이전에는 빈 문자열로 나옴
  ```

- `prototype` : 함수 객체 고유의 데이터 프로퍼티로 생성자 함수로 호출할 수 있는 객체(constructor)만이 가지는 프로퍼티이다. 프로토타입에 대해서는 아래에서 다 자세히 다룰 것이다.

- `__proto__` : **함수 객체의 고유 프로퍼티가 아니라**, Object.prototype 객체의 프로퍼티를 상속받은 접근자 프로퍼티이다. 따라서 `foo.hasOwnProperty('__proto__') `의 값은 false가 된다. `__proto__` 프로퍼티는 모든 객체가 가지고 있는 [[Prototype]] 내부 슬롯이 가리키는 프로토타입 객체에 접근하기 위해 사용하는 접근자 프로퍼티다. 마찬가지로 자세한 내용은 아래에서 다룰 것이다.



## 프로토타입

자바스크립트에서 원시 타입을 제외한 값들은 모두 객체다. 객체는 데이터와 동작을 하나의 논리적인 단위로 묶은 자료구조이다. 객체의 상태 데이터를 프로퍼티, 동작을 메서드라고 부른다. 각 객체는 독립적인 부품으로서 기능을 수행하면서 다른 객체와 관계 속에서 데이터를 주고 받을 수 있다. 그리고 다른 객체의 상태나 동작을 상속받아 사용하기도 한다.

### 상속과 프로토타입

아래 커피 생성자 함수에서 `getVentiPrice` 메서드는 인스턴스가 생성될 때마다 동일한 메서드가 계속해서 생겨나며 메모리를 불필요하게 낭비하게 된다. 이런 문제에서 상속을 구현할 때 자바스크립트는 프로토타입을 기반으로 구현한다.

```js
function Coffee(name, price) {
  this.name = name;
  this.price = price;
  
  // 일반 메서드(정적 메서드)
  this.getVentiPrice = function () {
    return this.price*1.15-this.price*1.15%100;
  }
  
  // prototype 메서드
  // 생성자 함수의 prototype 프로퍼티에 바인딩된 경우
  Coffee.prototype.getVentiPriceProto = function () {
    return this.price*1.15-this.price*1.15%100;
  }
};
let latte = new Coffee('latte', 3300);
let americano = new Coffee('americano', 3000);

latte.getVentiPrice === americano.getVentiPrice // false
latte.getVentiPriceProto === americano.getVentiPriceProto // true
```

모든 객체는 [[Prototype]]이라는 내부 슬롯을 가진다고 했다. 여기에 저장되는 프로토타입 객체(줄여서 프로토타입)는 객체가 생성될 때 객체 생성 방식에 따라 결정된다. 모든 객체는 하나의 프로토타입을 가지며 모든 프로토타입은 생성자 함수와 연결되어 있다. 즉, 프로토타입prototype은 어떤 객체의 부모 객체의 역할을 하는 객체로 다른 객체에 메서드를 포함한 공유 프로퍼티를 제공한다.

다른 언어에서 먼저 클래스를 다루어 보았을 경우 ES6이후 자바스크립트에 도입된 class 문법이 사실 더 익숙하다. 아래에서 간단하게 javascript에서 class 구현이 어떻게 되었는지 비교하였다. 추후 class 단원 글에서 더 자세히 다룰 것이다.

```js
class Coffee {
  // 생성자
  constructor(name, price) {
    this.name = name;
    this.price = price;
  }
  
  // 프로토타입 메서드
  getVentiPriceProto = function () {
    return this.price*1.15-this.price*1.15%100;
  }

	// 정적 메서드
	static getVentiPrice = function() {
    return this.price*1.15-this.price*1.15%100;
  }
}
```



### 프로토타입과 `__proto__`

모든 객체는 `__proto__` 접근자 프로퍼티를 통해 자신의 프로토타입 [[Prototype]] 내부 슬롯에 간접적으로 접근할 수 있다. 접근자 프로퍼티로 [[Get]], [[Set]] 프로퍼티 어트리뷰트로 구성된 프로퍼티다. `__proto__` 는 상속을 통해 사용된다. 아래 코드를 천천히 살펴보자. 

```js
// 위 코드로부터 이어진다.
Coffee.prototype === latte.__proto__    // true
latte.__proto__ === americano.__proto__ // true
Function.prototype === Coffee.__proto__ // true

// 상위 객체(부모 객체)의 prototype에 매소드 추가
Coffee.prototype.sayCoffeeName = function() {
  console.log("i am " + this.name)
}

// 하위 객체는 부모 객체의 prototype에 접근 가능.

latte.sayCoffeeName()     // 'i am latte'
americano.sayCoffeeName() // 'i am americano'
latte.sayCoffeeName === latte.__proto__.sayCoffeeName // true, __proto__는 접근자 프로퍼티
latte.__proto__.sayCoffeeName(); // 'i am undefined'

// 생성자 함수 Coffee는 자신의 prototype을 가지고 있으며 
// 그 prototype은 Object.prototype을 상속받고 있다.
Coffee.prototype.__proto__ === Object.prototype // true

// 모든 객체는 프로토타입 계층구조, 프로토타입 체인에 묶여 있다. 
// 프로토타입 체인의 종점에 Object.prototype
Object.prototype === latte.__proto__.__proto__  // true
```

프로토타입 체인은 단방향 링스크 리스트로 구현되어야 한다. 서로를 포로토타입으로 설정하면 순환 참조하는 프로토타입 체인이 만들어져 체인 종점이 존재하지 않기 때문에 에러가 발생한다.

 `__proto__` 접근자 프로퍼티를 사용할 수 없도록 객체를 생성하는 방법이 있어 `__proto__` 를 코드 내에서 사용하는 것은 권장되진 않는다. 따라서 `__proto__` 접근자 프로퍼티 대신 프로토타입의 참조를 얻거나 프로토타입을 교체하고 싶은 경우에는 `Object.getPrototypeOf` 혹은 `Object.setPrototypeOf` 메서드를 사용하면 된다.

오직 함수 객체만 prototype 프로퍼티를 소유할 수 있으며, 생성자 함수로부터 생성되는 인스턴스의 prototype을 가리키게 된다. 이전 글에서 언급한 것처럼 화살표 함수나, ES6의 메서드 축약 표현으로 정의한 메서드와 같이 non-constructor의 경우 prototype을 생성하지 않는다.

```js
(function () {}).hasownProperty('prototype'); // true
(()=>{}).hasOwnProperty('prototype') // false
```



### 리터럴로 생성한 객체

모든 프로토타입은 `constructor` 프로퍼티를 갖는다. 생성자 함수에 의해 생성된 인스턴스가 가진 `constructor` 프로퍼티는 자신을 생성한 생성자 함수를 가리킨다.

```js
const a = new Object();
a.constructor === Object // true
const latte = new Coffee();
latte.constructor === Coffee // true
```

하지만 생성자 함수를 호출하지 않고 객체를 생성하는 경우도 많다. 이렇게 생성된 객체의 프로토타입의 constructor가 가리키는 생성자 함수가 애매해지지만(가지긴 하지만 바로 단정할 순 없으므로), 이렇게 객체를 생성하더라도 프로토타입은 존재한다. 프로토타입과 생성자 함수는 언제나 함께 존재한다. 객체 리터럴, 함수 리터럴에 의한 생성과 `Object` 생성자 함수 `Function` 생성자 함수 생성에는 차이가 있긴 하다. 그러나 프로토타입 단원에서 보았을 때 기본적으로 리터럴 표기법에 의해 생성된 객체도 생성자 함수와 연결된다는 점에서 같다.

```js
const obj = {} 
const arr = [] 
// Function 생성자로 생성한 함수와
const foo = function () {} 
const regexp = /is/ig; 
obj.__proto__ === Object.prototype // true
arr.__proto__ === Array.prototype // true
foo.__proto__ === Function.prototype // true
regexp.__proto__ === RegExp.prototype // true
```



### 프로토타입 생성 시점

이전 글에서 함수 생성 시점은 함수 선언문이 런타임 이전에 먼저 평가된다고 하였다. 프로토타입은 constructor 함수 정의가 평가되어 함수 객체를 생성하는 시점에서 같이 생성되어 생성자 함수의 prototype 프로퍼티에 바인딩된다. 여기서 생성자 함수라고 하면 `Object`, `Number`, `Array` 등의 빌트인 생성자와 일반 생성자 함수 모두 포함한다. 단, 이 때 생성된 프로토타입 객체는 오직 constructor 프로퍼티만을 가지고 있다. 즉, 객체가 생성되기 이전에 함수와 프로토타입이 이미 객체형태로 존재한다는 뜻이다. 이후에 생성자 함수든 리터럴 방식이든 객체를 생성하면 프로토타입은 생성된 객체의 [[Prototype]] 내부 슬롯에 할당됨으로서 객체가 프로토타입을 상속받게 된다.

코드가 실행되기 이전에 자바스크립트 엔진에 의해서 생성되는 특수한 객체를 전역 객체Global Object라고 한다. 브라우저에서 window 노드에서 global 객체가 그것이다. 그 중에 `Object`, `Number`, `String`과 같은 빌트인 생성자 함수(빌트인 객체)도 존재한다.