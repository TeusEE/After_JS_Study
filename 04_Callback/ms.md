# 콜백함수

## 콜백 함수란?

> ***매개변수로 함수 객체를 전달하는 함수***이며 콜백 함수를 넘겨받은 코드는 이 콜백 함수를 필요에 따라 호출 할 수 있음. => ***제어권***도 같이 넘어감.

### 제어권?

- 콜백함수는 다른 코드(함수 도는 메서드)에게 인자로 넘겨줌으로써 그 제어권도 함께 위임한 함수
- 어떤 함수 x를 호출하여 '특정 조건일때 함수 Y를 실행해서 알려줘!'는 요청을 보내면, 함수 X입장에선 해당 조건인지 확인하고 스스로 함수Y를 호출함 => 함수 Y의 제어권이 함수 X로 넘겨짐
- 콜백함수를 넘겨받은 함수는  콜백 함수의 ***호출 시점, 인자, this에 대한 제어권***을  가지게 됨

#### 호출 시점
- 콜백 함수의 제어권을 넘겨받은 코드는 콜백 함수 호출 시점에 대한 제어권을 가짐


```js
var count = 0;
var callbackFunc = function () {
  console.log(count);
  if (++count > 4) clearInterval(timer);
};

var timer = setInterval(callbackFunc, 300);
```                                        

- setInterval 함수가 0.3초마다 callbackFunc을 호출함.
- setInterval 입장에선 해당 조건(0.3초마다)인지 확인하고 스스로 callbackFunc을 호출함.


### 인자
- 콜백함수의 제어권을 넘겨 받은 코드는 콜백 함수를 호출할 때 인자에 어떤 값들을 어떤 순서로 넘길 것인지에 대한 제어권을 가짐

- 간단한 예제로 map을 활용할때, 내 마음대로 map의 인자의 순서를 바꾸지 못함

```js
[10,20,30].map((cur, idx) => console.log(cur)) // 10 20 30

// index가 나오길 바랬지만, 여전히 현재값이 출력됨
[10,20,30].map((idx, cur) => console.log(idx)) // 10 20 30
```

### this
- 기본으로 this가 전역객체를 참조
- 제어권을 넘겨받을 코드에서 콜백 함수에 별도로 this가 될 대상을 지정한 경우 그 대상을 참조함.

## 콜백 함수는 함수?
- 메서드를 콜백함수로 전달된 경우라도 이는 메서드가 아니고, 결국은 함수

```js
var obj = {
  a : [1,2,3],
  aFunc : function(v,i){
    console.log(this, v, i)
    }
}

// 메서드로 호출
obj.aFunc(1,2)  // {a: Array(3), aFunc: ƒ} 1 2

// 메서드를 forEach함수의 콜백 함수로 전달
// 여기서 this는 obj와의 연관이 없어지게 되고, 전역객체를 바라봄
[4,5,6].forEach(obj.aFunc)
// window {...} 4 0
// window {...} 5 1
// window {...} 6 2
```

➕ 만약 콜백 함수 내부에서 this가 객체를 바라보게 하려면?
- this를 다른 변수에 담아 콜백 함수로 활용할 함수에서는 this 대신 그 변수를 사용, 이를 클로저로 만드는 전통적인 방식
- ES5에서 등장한 bind 메서드 이용


## 콜백 지옥
- 비동기 처리 로직을 위해 콜백 함수를 연속으로 중첩하여 사용할 때 발생하는 문제
- 해결 방안
&emsp; - 익명의 콜백 함수를 모두 기명함수로 전환
&emsp; - ES6에선 Promise, Generator 
&emsp; - ES2017에선 async/await


➕ Promise와 async/await의 차이점
- Promise는 .catch()를 통해 에러 핸들링이 가능하지만 async/await은 try-catch()문 사용
- Promise도 .then() 지옥이 발생할 수 있기 때문에 async/await을 활용한 코드가 코드 흐름을 이해 하기 쉬움