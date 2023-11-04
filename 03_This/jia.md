# this

- 상황에 따라 달라지는 this
  - 전역 공간에서의 this
  - 메서드로서 호출할 대 그 메서드 내부에서의 this
  - 함수로서 호출할 때 그 함수 내부에서의 this
  - 콜백 함수 호출 시 그 함수 내부에서의 this
  - 생성자 함수 내부에서의 this
- 명시적으로 this를 바인딩하는 방법
  - call 메서드
  - apply 메서드
  - call / apply 메서드의 활용
  - bind 메서드
  - 화살표 함수의 예외사항
  - 별도의 인자로 this를 받는 경우(콜백 함수 내에서의 this)

## 상황에 따라 달라지는 this

자바스크립트에서 this는 기본적으로 실행 컨텍스트가 생성될 때 함께 결정된다. 실행 컨텍스트는 함수를 호출할 때 생성되므로, this는 **_함수를 호출할 때 결정된다_** 라고 할 수 있다.

### 전역 공간에서의 this

**_전역 공간의 this_** === **_전역 객체_**  
=> 개념상 전역 컨텍스트를 생성하는 주체가 전역 객체이기 때문이다.

전역 객체는 런타임 환경에 따라 브라우저 환경은 **window**, Node.js 환경에서는 **global**이다.

```javascript
// 전역 공간에서의 this(브라우저 환경)
console.log(this); // { alert: f(), atob: f(), blur: f(), btoa: f, ... }
console.log(window); // { alert: f(), atob: f(), blur: f(), btoa: f, ... }
console.log(this === window); // true
```

```javascript
// 전역 공간에서의 this(Node.js 환경)
console.log(this); // { process: { title: 'node', version: 'v10.13.0', ... } }
console.log(window); // { process: { title: 'node', version: 'v10.13.0', ... } }
console.log(this === window); // true
```

---

전역 공간에서만 발생하는 특이한 성질

```javascript
// 전역 변수와 전역 객체(1)
var a = 1;
console.log(a); // 1
console.log(window.a); // 1
console.log(this.a); // 1
```

> 🤔 var로 선언했는데 window, this로 모두 값이 1로 출력되는 이유는?  
> **_자바스크립트의 모든 변수는 특정 객체의 프로퍼티_** 로서 작동하기 때문이다.  
> 특정 객체의 프로퍼티 === 실행 컨텍스트의 LexicalEnvironment

---

```javascript
var a = 1;
delete window.a; // false
console.log(a, window.a, this.a); // 1 1 1

var b = 2;
delete b; // false
console.log(b, window.b, this.b); // 2 2 2

window.c = 3;
delete window.c; // true
console.log(c, window.c, this.c); // Uncaught ReferenceError: c is not defined

window.d = 4;
delete d; // true
console.log(d, window.d, this.d); // Uncaught ReferenceError: d is not defined
```

전역변수를 선언하면 자바스크립트 엔진이 사용자가 의도치 않게 삭제하는 것을 방지하기 위해 자동으로 전역객체의 프로퍼티로 할당하면서 추가적으로 해당 프로퍼티의 configurable 속성(변경 및 삭제 가능성)을 false로 정의하는 것이다.

### 메서드로서 호출할 때 그 메서드 내부에서의 this

#### 함수 vs. 메서드

함수와 메서드의 차이점 => 독립성

- 함수 : 그 자체로 독립적인 기능을 수행,
- 메서드 : 자신을 호출한 대상 객체에 관한 동작을 수행

```javascript
var func = function (x) {
  console.log(this, x);
};
func(1); // Window { ... } 1

var obj = {
  method: func,
};
obj.method(2); // { method: f } 2
```

#### 메서드 내부에서의 this

this에는 호출한 주체에 대한 정보가 담긴다.  
어떤 함수를 메서드로서 호출하는 경우 호출 주체는 바로 함수명(프로퍼티명) 앞의 객체이다.

```javascript
var obj = {
  methodA: function () { console.log(this); },
  inner: {
    methodB: function () { console.log(this); }
  }
};
obj.methodA();  // { methodA: f, inner: { ... } } ( === obj)
obj['methodA']);  // { methodA: f, inner: { ... } } ( === obj)

obj.inner.methodB();  // { methodB: f } ( --- obj.inner)
obj.innder['methodB']();  // { methodB: f } ( --- obj.inner)
obj['innder'].methodB();  // { methodB: f } ( --- obj.inner)
obj['inner']['methodB']();  // { methodB: f } ( --- obj.inner)
```

### 함수로서 호출할 때 그 함수 내부에서의 this

#### 함수 내부에서의 this

어떤 함수를 함수로서 호출할 경우에는 this를 지정하지 않았기 때문에 호출 주체의 정보를 알 수 없다. 하지만 자바스크립트는 **_this가 지정되지 않을 경우 전역 객체_**를 바라본다.

#### 메서드의 내부함수에서의 this

```javascript
var obj1 = {
  outer: function () {
    console.log(this); // (1)
    var innerFunc = function () {
      console.log(this); // (2) (3)
    };
    innerFunc();

    var obj2 = {
      innerMethod: innerFunc,
    };
    obj2.innerMethod();
  },
};
obj1.outer();
```

결과는 (1): obj1, (2): 전역객체(Window), (3): obj2 이다.  
(1)과 (3)은 호출 주체인 객체가 지정되어있지만, (2)는 지정되어 있지 않기 때문에 전역객체가 되는 것이다.

여러 케이스들을 확인해본 결과, this 바인딩은 함수를 실행하는 당시의 주변 환경은 중요하지 않고,  
오직 **_호출 주체인 객체가 지정 유무_**가 중요하다.

#### 메서드의 내부 함수에서의 this를 우회하는 방법

```javascript
// 변수를 활용하는 방법
var obj = {
  outer: function () {
    console.log(this); // (1) { outer: f }
    var innerFunc1 = function () {
      console.log(this); // (2) Window { ... }
    };
    innerFunc1();

    var self = this;
    var innerFunc2 = function () {
      console.log(self); // (3) { outer: f }
    };
    innerFunc2();
  },
};
obj.outer();
```

#### this를 바인딩하지 않는 함수

```javascript
var obj = {
  outer: function () {
    console.log(this); // (1) { outer: f }
    var innerFunc = () => {
      console.log(this); // (2) { outer: f }
    };
    innerFunc();
  },
};
obj.outer();
```

ES6에서는 함수 내부에서 this가 전역객체를 바라보는 문제를 보안하고자, this를 바인딩하지 않는 화살표 함수(arrow function)를 도입했다.

### 콜백 함수 호출 시 그 함수 내부에서의 this

```javascript
setTimeout(function () {
  console.log(this);
}, 300); // (1)

[1, 2, 3, 4, 5].forEach(function (x) {
  // (2)
  console.log(this, x);
});

document.body.innerHTML += '<button id="a">클릭</button>';
document.body.querySelector("#a").addEventListener("click", function (e) {
  // (3)
  console.log(this, e);
});
```

콜백 함수도 위에서 봤던 케이스들과 마찬가지로, 호출 주체인 객체를 지정하지 않으면 전역객체를 바라보게 된다.

### 생성자 함수 내부에서의 this

```javascript
var Cat = function (name, age) {
  this.bark = "야옹";
  this.name = name;
  this.age = age;
};
var choco = new Cat("초코", 7);
var nabi = new Cat("나비", 5);
console.log(choco, nabi);

/* 결과
Cat { bark: '야옹', name: '초코', age: 7 }
Cat { bark: '야옹', name: '나비', age: 5 }
*/
```

생성자 함수를 호출하면 생성자의 prototype 프로퍼티를 참조하는 **proto**라는 프로퍼티가 있는 객체(인스턴스)를 만들고, 미리 준비된 공통 속성 및 개성을 해당 객체(this)에 부여한다.

## 명시적으로 this를 바인딩하는 방법

### call 메서드

```javascript
Function.prototype.call(thisArg[, arg1[, arg2[, ...]]])
```

call 메서드는 메서드의 호출 주체인 함수를 즉시 실행하도록 하는 명령이다.
메서드의 첫번째 인자는 this를 바인딩하고, 나머지 인자는 함수의 매개변수로 한다.

```javascript
var func = function (a, b, c) {
  console.log(this, a, b, c);
};

func(1, 2, 3); // Window{ ... } 1 2 3
func.call({ x: 1 }, 4, 5, 6); // { x: 1 } 4 5 6
```

### apply 메서드

```javascript
Function.prototype.apply(thisArg[, argsArray])
```

apply 메서드는 call 메서드와 기능이 동일하다.  
차이점은 apply 메서드는 두 번째 인자인 매개변수를 배열로 받는 것이다.

```javascript
var func = function (a, b, c) {
  console.log(this, a, b, c);
};
func.apply({ x: 1 }, [4, 5, 6]); // { x: 1 } 4 5 6
```

### call / apply 메서드의 활용
