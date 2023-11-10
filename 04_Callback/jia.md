# 콜백 함수

- 콜백 함수란?
- 제어권
  - 호출 시점
  - 인자
  - this
- 콜백 함수는 함수다
- 콜백 함수 내부의 this에서 다른 값 바인딩하기
- 콜백 지옥과 비동기 제어

## 콜백 함수란?

콜백 함수(callback function)은 매개변수로 함수를 넘겨줘서 호출 함수 내부에서 실행하는 함수이다.

## 제어권

### 호출 시점

```javascript
var count = 0;
var cbFunc = function () {
  console.log(count);
  if (++count > 4) clearInterval(timer);
};
var timer = setInterval(cbFunc, 300);
```

```javascript
var intervalID = scope.setInterval(func, delay[, param1, param2, ...]);
```

setInterval의 구조와 함께 살펴보면 전달한 첫 번째 인자인 cbFunc 함수가 콜백 함수이고, 두 번째 인자는 delay 300ms로 0.3초마다 자동으로 콜백 함수가 실행된다.

| code                      | 호출 주체   | 제어권      |
| :------------------------ | :---------- | :---------- |
| cbFunc();                 | 사용자      | 사용자      |
| setInterval(cbFunc, 300); | setInterval | setInterval |

이 때, setInterval이라고 하는 '다른 코드'에 첫 번째 인자로서 cbFunc 함수를 넘겨주자 제어권을 넘겨받은 setInterval이 스스로 판단에 따라 적절한 시점(0.3초마다) 함수를 실행한다.

이처럼 콜백 함수의 제어권을 넘겨받은 코드는 콜백 함수 호출 시점에 대한 제어권을 가진다.

### 인자

```javascript
var newArr = [10, 20, 30].map(function (currentValue, index) {
  console.log(currentValue, index);
  return currentValue + 5;
});
console.log(newArr);
```

map 메서드를 사용하면 배열 [10, 20, 30]의 각 요소를 하나씩 가져와 콜백 함수를 실행한다. 여기서 우리는 매개 변수명을 통해 currentValue가 배열의 요소이고, index가 배열의 인덱스인 사실을 알 수 있다.  
하지만 여기서 매개 변수의 순서를 바꿔보아도 **_첫 번째 인자가 배열의 요소_**이고, **_두 번째 인자가 배열의 인덱스_**라는 것은 변함이 없다.

콜백 함수를 호출하는 주체가 사용자가 아닌 map 메서드이므로 map 메서드가 콜백 함수를 호출할 때 인자에 어떤 값들을 어떤 순서로 넘길 것인지는 map 메서드에게 제어권이 있다.

### this

콜백 함수는 기본적으로 `this === window`이다.

```javascript
Array.prototype.map = function (callback, thisArg) {
  var mappedArr = [];
  for (var i = 0; i < this.length; i++) {
    var mappedValue = callback.call(thisArg || window, this[i], i, this);
    mappedArr[i] = mappedValue;
  }
  return mappedArr;
};
```

map 메서드를 직접 구현한 코드를 살펴보면 call/apply 메서드가 있다.  
이처럼 호출하는 함수가 콜백 함수에서 this가 될 대상을 명시적으로 바인딩한다.

```javascript
setTimeout(function () {
  console.log(this);
}, 300); // Window { ... }

document.body.innerHTML += '<button id="a">클릭</button>';
document.body.querySelector("#a").addEventListener(
  "click",
  function (e) {
    console.log(this, e); // <button id="a">클릭</button>
  } // MouseEvent { isTrusted: true, ... }
);
```

- setTimeout은 내부에서 콜백 함수를 호출할 때 call 메서드의 첫 번째 인자에 전역객체를 넘기기 때문에 콜백 함수 내부에서의 this가 전역객체를 가리킨다.
- addEventListener는 내부에서 콜백 함수를 호출할 때 call 메서드의 첫 번째 인자에 addEventListener 메서드의 this를 그대로 넘기도록 정의되어 있기 때문에 HTML 엘리먼트를 가리키게 된다.

## 콜백 함수는 함수다

콜백 함수로 어떤 객체의 메서드를 전달하더라도 그 메서드는 메서드가 아닌 함수로서 호출된다.

```javascript
var obj = {
  vals: [1, 2, 3],
  logValues: function (v, i) {
    console.log(this, v, i);
  },
};
obj.logValues(1, 2); // { vals: [1, 2, 3], logValues: f} 1 2
[4, 5, 6].forEach(obj.logValues); // Window { ... } 4 0
// Window { ... } 5 1
// Window { ... } 6 2
```

메서드로 호출 할 때는 this가 obj를 가리키고, 인자로 넘어온 1, 2가 출력된다.  
하지만 이 메서드를 forEach의 콜백 함수로 전달하면 메서드가 아닌 함수로서 호출되고 별도로 this를 지정하지 않아 전역객체를 바라보게 된다.

## 콜백 함수 내부의 this에 다른 값 바인딩하기

```javascript
// 전통적인 방식
var obj1 = {
  name: "obj1",
  func: function () {
    var self = this;
    return function () {
      console.log(self.name);
    };
  },
};
var callback = obj.func();
setTimeout(callback, 1000);
```

```javascript
// bind 메서드 활용
var obj1 = {
  name: "obj1",
  func: function () {
    return function () {
      console.log(this.name);
    };
  },
};
setTimeout(obj1.func.bind(obj1), 1000);

var obj2 = { name: "obj2" };
setTimeout(obj1.func.bind(obj2), 1500);
```

## 콜백 지옥과 비동기 제어

- 콜백 지옥(callback hell): 콜백 함수를 익명 함수로 전달하는 과정이 반복되어 코드의 들여쓰기 수준이 감당하기 힘들 정도로 깊어지는 현상

- 동기(synchronous)인 코드: 현재 실행 중인 코드가 완료된 후에야 다음 코드를 실행하는 방식 => **_즉시_** 처리가 가능한 대부분의 코드

- 비동기(asyncchronous)적인 코드: 현재 실행 중인 코드의 완료 여부와 무관하게 즉시 다음 코드로 넘어가는 방식 => **_별도의 요청, 실행 대기, 보류_** 등과 관련된 코드

```javascript
// 콜백 지옥 예시
setTimeout(function (name) {
    var coffeeList = name;
    console.log(coffeeList);

    setTimeout(function (name) {
        coffeeList + = ', ' + name;
        console.log(coffeeList);

        setTimeout(function (name) {
            coffeeList + = ', ' + name;
            console.log(coffeeList);


            setTimeout(function (name) {
                coffeeList + = ', ' + name;
                console.log(coffeeList);


            }, 500, '카페라떼');
        }, 500, '카페라떼');
    }, 500, '아메리카노');
}, 500, '에스프레소');
```

```javascript
// 콜백 지옥 해결 - 기명 함수
var coffeeList = "";

var addEspresso = function (name) {
  coffeeList = name;
  console.log(coffeeList);
  setTimeout(addAmericano, 500, "아메리카노");
};
var addAmericano = function (name) {
  coffeeList += ", " + name;
  console.log(addMocha, 500, "카페모카");
};
var addMocha = function (name) {
  coffeeList += ", " + name;
  console.log(coffeeList);
  setTimeout(addLatte, 500, "카페라떼");
};
var addLatte = function (name) {
  coffeeList += ", " + name;
  console.log(coffeeList);
};

setTimeout(addEspresso, 500, "에스프레소");
```

기명 함수를 통해 콜백 지옥을 해결할 수 있다.  
하지만 코드명을 일일이 따라다녀야 하므로 헷갈릴 소지가 있다.  
그래서 ES6에서는 **_Promise, Generator_**,  
ES2017에서는 **_async/await_** 이 도입되었다.

```javascript
// 비동기 작업의 동기적 표현 - Promise
new Promise(function (resolve) {
  setTimeout(function () {
    var name = "에스프레소";
    console.log(name);
    resolve(name);
  }, 500);
})
  .then(function (prevName) {
    return new Promise(function (resolve) {
      setTimeout(function () {
        var name = prevName + ", 아메리카노";
        console.log(name);
        resolve(name);
      }, 500);
    });
  })
  .then(function (prevName) {
    return new Promise(function (resolve) {
      setTimeout(function () {
        var name = prevName + ", 카페모카";
        console.log(name);
        resolve(name);
      }, 500);
    });
  })
  .then(function (prevName) {
    return new Promise(function (resolve) {
      setTimeout(function () {
        var name = prevName + ", 카페라떼";
        console.log(name);
        resolve(name);
      }, 500);
    });
  });
```

```javascript
// 비동기 작업의 동기적 표현 - Generator
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
  var mocha = yield adddCoffee(americano, "카페모카");
  console.log(mocha);
  var latte = yield addCoffee(mocha, "카페라떼");
  console.log(latte);
};
var coffeeMaker = coffeeGenerator();
coffeeMaker.next();
```

'\*'이 붙은 함수가 Generator 함수이다.  
Generator 함수를 실행하면 Iterator가 반환되는데, Iterator는 next라는 메서드를 가지고 있다. 이 next 메서드를 호출하면 Generator 함수 내부에서 가장 먼저 등장하는 yield에서 함수의 실행르 멈춘다. 이후 다시 next 메서드를 호출하면 앞서 멈췄던 부분부터 시작해서 그 다음에 등장하는 yield에서 함수의 실행을 멈춘다.

즉, 비동기 작업이 완료되는 시점마다 next 메서드를 호출해준다면 Generator 함수 내부의 소스가 위에서부터 아래로 순차적으로 진행된다.

```javascript
// 비동기 작업의 동기적 표현 - Promise + Async/await
var addCoffee = function (name) {
    return new Promise(function (resolve) {
        setTimeout(function () {
            resolve(name);
        }, 500);
    });
};
var coffeeMaker = aysnc function () {
    var coffeeList = '';
    var _addCoffee = async function (name) {
        coffeeList += (coffeeList ? ',' : '') + await addCoffee(name);
    };
    await _addCoffee('에스프레소');
    console.log(coffeeList);
    await _addCoffee('아메리카노');
    console.log(coffeeList);
    await _addCoffee('카페모카');
    console.log(coffeeList);
    await _addCoffee('카페라떼');
    console.log(coffeeList);
};

coffeeMakre();
```

비동기 작업을 수행하고자 하는 함수 앞에 async를 표기하고, 함수 내부에서 실직적인 비동기 작업이 필요한 위치마다 await을 표기하는 것만으로도 뒤의 내용을 Promise로 자동 전환하고, 해당 내용이 resolve된 이후에야 다음으로 진행한다.

즉, Promise의 then과 흡사한 효과를 얻을 수 있다.
