# 콜백 함수

### 콜백 함수란?

> 자바스크립트에서 콜백함수는 **매개 변수로 함수 객체를 전달**해서 **호출 함수 내에서 매개변수 함수를 실행하는 것**을 말한다.

예를 들어 설명하자면,

```javascript
function sayGoodBye(name, callback) {
  const words = "안녕" + name + "조만간 또 보자";

  callback(words); // 매개 변수의 함수(콜백 함수)를 호출
}

sayGoodBye("철수", function whatWords(words) {
  console.log(words); // 안녕 철수 조만간 또 보자
});
```

여기서 나온 것으로 보아 콜백 함수는 파라미터로 값이나 변수로 값을 전달하는 것이 아닌 **함수 그 자체를 전달**하는 것으로 볼 수 있다.

### 콜백 함수의 제어권

> 다른 의미의 정의로, 콜백 함수란 **제어권을 다른 함수에게 넘겨준 함수**라고 한다.
> 정확하게는 함수 A의 제어권을 다른 함수 B에게 넘겨주었을 때 이 함수 A를 콜백 함수라고 한다.

이처럼 콜백 함수는 다른 코드에게 인자를 넘겨줌으로써 그 제어권도 함께 위임한 함수다.

함수의 제어권을 줬기 때문에 함수 B는 여러가지를 할 수 있다.

- 콜백 함수 A의 호출 시점은 함수 B가 결정한다.
- 콜백 함수 A의 인자로 넘겨줄 값과 그 순서도 함수 B가 결정한다.
- 콜백 함수 A의 this 또한 함수 B가 결정할 수 있다.

### 콜백 함수도 함수다

콜백 함수는 `함수`다. 이 부분을 강조하는 이유는 this 때문이다. 3장에서 객체의 메소드를 실행할 때와 함수로 실행할 때 this가 달라지는 것을 저번 장에서 공부했었다. 때문에 콜백 함수로 객체의 메소드를 넘긴다면 this가 여전히 그 객체라고 오해할 수 있다.

결과적으로 말하자면 아니다. 콜백 함수는 함수라는 말을 증명하듯 콜백 함수로 어떤 객체의 메소드를 전달하더라도 그 메소드는 메소드가 아닌 함수로서 호출된다.

즉, this를 따로 명시적 바인딩을 해주지 않는 이상 this는 전역 객체를 가리키게 된다는 것이다.

### 콜백 함수 내부의 this에 다른 값 바인딩하기

객체의 메소드를 콜백 함수로 전달하면 해당 객체를 this로 바라볼 수 없게 된다는 점을 이해했을 것이다.

그럼에도 콜백 함수 내부에서 this가 객체를 바라보게 하려면

- 전통적 방식

  - 함수 안에서 this를 변수에 담아 그 변수를 this로 이용하는 것
  - 불필요하고 복잡하고 번거롭다.

- 현재 방식
  - bind 메소드로 쉽게 객체를 바인딩한다.

```javascript
btn.addEventListener("click", test.bind(obj));
```

### 콜백 지옥과 비동기 제어

자바스크립트는 기본적으로 동기적으로 작동하지만 싱글 스레드 언어이기 때문에 수많은 비동기적 핸들링이 존재한다. 자바스크립트에서 비동기와의 관계는 뗄래야 뗄 수 없는 관계인데 가장 대표적으로 다루는 부분을 말하자면 비동기를 동기적으로 다루는 부분이라고 할 수 있다.

비동기 함수들을 동기적으로 다루기 위해 한마디로 실행 순서를 보장하기 위해 개발자는 원하는 순서에 비동기 함수를 콜백 함수 형식으로 사용했다. 하지만 콜백 함수로 익명 함수를 계속 전달하다보니 `끝도 없이 늘어지는 형태의 코드로 구현되면서 가독성이 엉망`으로 되어버렸다. 또한, `가장 아래(깊은 곳)부터 실행되기 때문에 보통 위에서 아래로 값을 읽는 것에 익숙`해진 인간들은 최악의 가독성으로 꼽혔다.

#### 비동기 핸들링 방법

이러한 콜백 지옥을 해결하기 위한 전통적인 해결 방법은 기명 함수를 쓰는 것이다.

```javascript
var coffeeList = "";

var addEspresso = function (name) {
  coffeList = name;
  console.log(coffeeList);
  setTimeout(addAmericano, 500, "아메리카노");
};

var addAmericano = function (name) {
  coffeList = name;
  console.log(coffeeList);
  setTimeout(addMocha, 500, "카메모카");
};

...

```

솔직히 말하면 복잡하다. 그래서 현재의 ES6, ES2017에서 나온게 Promise, Generator, async/await 이 도입됐다.

#### Promise

프로미스는 비동기 동작을 값으로 다룰 수 있게 도와준다.
비동기 작업의 처리가 완료된 상태에서 resolve/reject 함수를 호출하는 구문이 있을 경우에만 then 또는 catch 구문으로 넘어간다.

```javascript
new Promise(function (resolve) {
  setTimeout(function () {
    var name = "에스프레소";
    console.log(name);
    resolve(name);
  }, 500);
}).then(function (prevName) {
  return new Promise(function (resolve) {
    setTimeout(function () {
      var name = "에스프레소";
      console.log(name);
      resolve(name);
    }, 500);
  });
}).then(function (prevName) {
  ...
});
```

이런 식으로 resolve를 호출하는 구문이 있을 경우 둘중 하나를 실행되기 전까진 then 구문으로 넘어가지 않는다.

#### 제네레이터

제네레이터는 이터러블을 쉽게 만들 수 있도록 도와주는 함수이자, 이터레이터이다.

```javascript
var addCoffee = function (prevName, name) {
  setTimeout(function () {
    coffeeMaker.next(prevName ? prevName + ", " + name : name);
  }, 500);
};

var coffeeGenerator = function* () {
  var espresso = yield addCoffee("", "에스프레소");
  console.log(espresso);
  var americano = yield addCoffee(espresso, "아메리카노");
  console.log(americano);
  var mocha = yield addCoffee(americano, "카페모카");
  console.log(mocha);
};

var coffeeMaker = coffeeGenerator();
coffeeMaker.next();
```

함수에 "\*"이 붙은 함수가 바로 제네레이터 함수다. 제네레이터 함수를 실행하면 이터레이터가 반환되는데 이터레이터는 next라는 메서드를 가지고 있다. 이 next 메서드를 호출하면 제네레이터 함수 내부에서 가장 먼저 등장하는 yield에서 함수의 실행을 멈춘다. 이후 next 메서드를 호출하면 멈춘 부분부터 시작해 그 다음 yield까지 실행을 멈추는 방식이다. 이런 방식으로 비동기 작업의 완료 시점을 제어할 수 있다.

#### async/await

가장 최근에 나왔고 정말 가독성이 뛰어난 기능이라고 할 수 있다. 나도 실무에서 가장 많이 쓰고 애용하고 있는 부분이다.

더 설명이 필요없다 코드를 보면 바로 알 것이다.

```javascript
var addCoffee = function (name) {
  return new Promise(function (resolve) {
    setTimeout(function () {
      var name = "에스프레소";
      console.log(name);
      resolve(name);
    }, 500);
  });
};

var coffeeMaker = async function () {
  var coffeeList = "";
  var _addCoffee = async function (name) {
    coffeeList += (coffeeList ? "," : "") + (await addCoffee(name));
  };
  await _addCoffee("에스프레소");
  console.log(coffeeList);
  await _addCoffee("아메리카노");
  console.log(coffeeList);
  await _addCoffee("카페모카");
  console.log(coffeeList);
};

coffeeMaker();
```

간략히 설명하자면 비동기 작업을 수행해야하는 함수에 async를 표기하고, 함수 내부에서 실직적인 비동기 작업이 필요한 위치마다 await을 표기하는 것만으로 Promise와 같은 기능을 한다.

추가로 작업하자면 try catch 구문으로 감싸 error를 관리할 수 있다.
