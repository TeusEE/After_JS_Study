# 실행 컨텍스트

- 실행 컨텍스트란?
- VariableEnvironment
- LexicalEnvironment
  - environmentRecord와 호이스팅(hoisting)
  - 스코프, 스코프 체인, outerEnvironmentReference
- this

## 실행 컨텍스트란?

실행 컨텍스트(execution context)는 실행할 코드에 제공할 환경 정보들을 모아놓은 객체로, 자바스크립트의 동적 언어로서의 성격을 가장 잘 파악할 수 있는 개념이다.

- 스택(stack) : 출입구가 하나뿐인 깊은 우물 같은 데이터 구조로 마지막에 저장된 값을 먼저 꺼내는 구조 w
  많은 프로그래밍 언어들이 스택이 넘치면 **콜스택 초과 에러**가 발생한다.  
  (100개만 저장할 수 있는 우물에 100개 이상의 데이터를 넣으려고 하는 경우)
  > a, b, c, d -> d, c, b, a
- 큐(quecue) : 양쪽이 모두 열려있는 파이프 같은 데이터 구조로 먼저 저장된 값을 먼저 꺼내는 구조

  > a, b, c, d -> a, b, c, d

동일한 환경에 있는 코드들을 실행할 때 필요한 환경 정보들을 모아 컨텍스트를 구성하고, 이를 콜 스택(call stack)에 쌓아올렸다가, 가장 위에 쌓여있는 컨텍스트와 관련 있는 코드들을 실행하는 식으로 전체 코드의 환경과 순서를 보장한다.

> 여기서 '동일한 환경', 즉 하나의 실행 컨텍스트를 구성할 수 있는 방법으로는 eval() 함수, **함수 실행**있다.
>
> > ES6에서는 블록[]에 의해서도 새로운 실행 컨텍스트가 생성된다.

**실행 컨텍스트가 콜 스택에 쌓이는 순서**

```javascript
var a = 1;
function outer() {
  function inner() {
    console.log(a); // undefined
    var a = 3;
  }
  inner();
  console.log(a); // 1
}
outer();
console.log(a); // 1
```

[　　　　　 ] 　　[　　　　　　 ]　　　[　　　　　 　] 　　 [　　　　　 　] 　　　[　　　　　 　　] 　　[　　　　　 　] 　　　[　　　　　 ]  
[　　　　　 ] ->　[　　　　　　 ] ->　[　　　　　　 ] ->　 [　　 inner 　 ] ->　[　　　　　　　 ] -> 　[　　　　　　 ] 　->　[　　　　 　]  
[　　　　　 ] 　　[　　　　　　 ]　　　[　　 outer 　 ] 　　 [　　 outer 　 ] 　　 [　　 outer 　 ] 　　[　　　　　　 ] 　　　[　　　　　 ]  
[　　　　　 ] 　　[전역 컨텍스트]　　　[전역 컨텍스트]　　　[전역 컨텍스트] 　　 [전역 컨텍스트 ] 　　[전역 컨텍스트]　　　 [　　　　　 ]

> **전역 컨텍스트와 실행 컨텍스트의 차이**  
> 굳이 차이점을 찾자면 전역 컨텍스트가 관여하는 대상은 함수가 아닌 전역 공간이기 때문에 arguments가 없습니다. 전역 공간을 둘러싼 외부 스코프란 존재할 수 없기 때문에 스코프체인 상에는 전역 스코프 하나만 존재한다. 이런 성질들은 구조상 당연히 그럴 수 밖에 없는 것이다.

## VariableEnvironment

VariableEnvironment는 현재 컨텍스트 내의 식별자들에 대한 정보(environmentRecord) + 외부 환경 정보(outer-EnvironmentReference), 선언 시점의 LexicalEnvironment의 스냅샷(snapshot)들의 정보들을 가지고 있으며, 변경 사항은 반영되지 않는다.

## LexicalEnvironment

### environmentRecord와 호이스팅

environmentRecord에는 현재 컨텍스트와 관련된 코드의 식별자 정보들이 저장된다.  
컨텍스트 내부 전체를 처음부터 끝까지 쭉 훑어나가며 **순서대로** 수집한다.

이 때, **_호이스팅(hoisting)_** 이라는 개념이 등장한다.  
호이스팅이란 '끌어올리다'라는 의미의 hoist에 ing를 붙여 만든 동명사로, 변수 정보를 수집하는 과정을 더욱 이해하기 쉬운 방법으로 대체한 가상의 개념이다. 자바스크립트 엔진이 실제로 식별자들을 최상단으로 끌어올리지는 않지만 편의상 끌어올린 것으로 간주하는 것이다.

#### 호이스팅 규칙

```javascript
function a(x) {
  // 수집 대상 1(매개변수)
  console.log(x); // (1)
  var x; // 수집 대상 2(변수 선언)
  console.log(x); // (2)
  var x = 2; // 수집 대상 3(변수 선언)
  console.log(x); // (3)
}
a(1);
```

호이스팅이 되지 않을 때 (1), (2), (3) 순서대로, 1, undefined, 2가 출력될 것이라고 예상된다.

```javascript
function a() {
  var x = 1; // 수집 대상 1(매개변수 선언)
  console.log(x); // (1)
  var x; // 수집 대상 2(변수 선언)
  console.log(x); // (2)
  var x = 2; // 수집 대상 3(변수 선언)
  console.log(x); // (3)
}
```

첫 번째 코드와 두 번째 코드는 LexicalEnvironment의 입장에서는 완전히 같다.  
그래서 매개변수를 함수 내부의 다른 코드보다 먼저 선언 및 할당을 이루어진 것으로 간주할 수 있다고 볼 수 있다.

```javascript
function a() {
  var x; // 수집 대상 1의 변수 선언 부분
  var x; // 수집 대상 2의 변수 선언 부분
  var x; // 수집 대상 3의 변수 선언 부분

  x = 1; // 수집 대상 1의 할당 부분
  console.log(x); // (1)
  console.log(x); // (2)
  x = 2; // 수집 대상 3의 할당 부분
  console.log(x); // (3)
}
```

세 번째 코드는 변수 정보를 수집하는 과정, 즉 호이스팅을 처리한 코드이다.  
이렇게 보면 처음 예상한 결과와 다르게, 1, 1, 2 라는 결과가 나오게 된다.

environmentRecord는 현재 실행될 컨텍스트의 대상 코드 내에 어떤 식별자들이 있는지에만 관심이 있고, 각 실별자에 어떤 값이 할당될 것인지는 관심이 없다. 그래서 호이스팅할 때 **_변수명만 끌어올리고_** 할당 과정은 원래 자리에 그대로 남겨둔다.

#### 함수 선언문과 함수 표현식

- 함수 선언문(function declaration) : function 정의부만 존재하고 별도의 할당 명령이 없는 것.
  > 반드시 함수명이 정의
- 함수 표현식(function expression) : 정의한 function을 별도의 변수에 할당하는 것
  > 함수명이 정의 없어도 된다.  
  > 함수명을 정의한 함수 표현식은 '기명 함수 표현식', 정의하지 않은 것은 '익명 함수 표현식'
  >
  > > 일반적으로 함수 표현식은 익명 함수 표현식을 말한다.

```javascript
function a() {
  /* ... */
} // 함수 선언문. 함수명 a가 곧 변수명.
a(); // 실행 OK.

var b = function () {
  /* ... */
}; // (익명) 함수 표현식. 변수명 b가 곧 함수명.
b(); // 실행 OK.

var c = function d() {
  /* ... */
}; // 기명 함수 표현식. 변수명은 c, 함수명은 d.
c(); // 실행 OK.
d(); // 에러!
```

**_함수 선언문과 함수 표현식_**

```javascript
console.log(sum(1, 2));
console.log(multiply(3, 4));

function sum(a, b) {
  // 함수 선언문 sum
  return a + b;
}

var multiply = function (a, b) {
  // 함수 표현식 multiply
  return a * b;
};
```

```javascript
var sum = function sum(a, b) {
  // 함수 선언문은 전체를 호이스팅합니다.
  return a + b;
};
var multiply; // 변수는 선언부만 끌어올립니다.
console.log(sum(1, 2));
console.log(multiply(3, 4));

multiply = function (a, b) {
  // 변수의 할당부는 원래 자리에 남겨둡니다.
  return a * b;
};
```

### 스코프, 스코프 체인, outerEnvironmentReference

- 스코프(scope) : 식별자에 대한 유효범위
- 스코프 체인(scope chain) : '식별자의 유효범위'를 안에서부터 바깥으로 차례로 검색해나가는 것

그리고 이를 가능케 하는 것이 LexicalEnvironment의 두 번째 수집 자료인 outerEnvironmentReference이다.

> ES5까지의 자바스크립트는 특이하게도 전역공간을 제외하면 오직 함수에 의해서만 스코프가 생성된다. 하지만 ES6에서는 let, const, class, strict mode를 통해 블록에 의해서도 스코프를 생성할 수 있다.

#### 스코프

outerEnvironmentReference는 현재 호출된 함수가 선언될 당시의 LexicalEnvironment를 참조한다.

A 함수 내부에 B 함수를 선언하고 다시 B 함수 내부에 C 함수를 선언한 경우, 함수 C의 outerEnvironmentReference는 함수 B의 LexicalEnvironment를 참조한다.  
함수 B의 LexicalEnvironment에 있는 outerEnvironmentReference는 다시 함수 B가 선언되던 때 (A)의 LexicalEnvironment를 참조한다.

이처럼 outerEnvironmentReference는 _연결리스트(linked list)_ 형태를 띤다.  
이런 구조적 특성 덕분에 여러 스코프에서 동일한 식별자를 선언한 경우에는 **_무조건 스코프 체인 상에서 가장 먼저 발견된 식별자에만 접근 가능_** 하게 된다.

```javascript
var a = 1;
var outer = function () {
  var inner = function () {
    console.log(a);
    var a = 3;
  };
  inner();
  console.log(a);
};
outer();
console.log(a);
```

<table>
<thead>
  <tr>
    <th colspan="4">[0] 전역 컨텍스트 활성화 - LexicalEnvironment, VariableEnvironment, thisBinding</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td rowspan="10">
      전역 컨텍스트<br/>
      <table>
        <thead>
          <tr>
            <th colspan="2">L.E</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>e</td>
            <td>a, outer</td>
          </tr>
        </tbody>
      </table>
      (this: 전역 객체)
      </td>
    <td colspan="3">[1], [2] a에 1, outter에 함수 할당</td>
  </tr>
  <tr>
    <td colspan="3">[10] outer 함수 호출. 전역 컨텍스트 비활성화<br>[2] outer 실행 컨텍스트 활성화</td>
  </tr>
  <tr>
    <td rowspan="6">
      outer 컨텍스트<br />
      <table>
        <thead>
          <tr>
            <th colspan="2">L.E</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>e</td>
            <td>a, outer</td>
          </tr>
          <tr>
            <td>o</td>
            <td>GLOBAL L.E</td>
          </tr>
        </tbody>
      </table>
      (this: 전역 객체)
    </td>
    <td colspan="2">[3] inner에 함수 할당</td>
  </tr>
  <tr>
    <td colspan="2">[7] inner 함수 호출. outer 실행 컨텍스트 비활성화<br>[3] inner 실행 컨텍스트 활성화</td>
  </tr>
  <tr>
    <td rowspan="2">
    inner 컨텍스트<br />
    <table>
      <thead>
        <tr>
          <th colspan="2">L.E</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>e</td>
          <td>a</td>
        </tr>
        <tr>
          <td>o</td>
          <td>outer L.E</td>
        </tr>
      </tbody>
    </table>
    (this: 전역 객체)
    </td>
    <td>[4] inner의 L.E에서 a 탐색<br>&nbsp;&nbsp;&nbsp;&nbsp;-&gt; undefined 출력</td>
  </tr>
  <tr>
    <td>[5] a에 3 할당</td>
  </tr>
  <tr>
    <td colspan="2">[6] inner 함수 종료. inner 실행 컨텍스트 제거<br>[7] outer 실행 컨텍스트 재활성화</td>
  </tr>
  <tr>
    <td colspan="2">[8] outer의 L.E에서 a 탐색<br>&nbsp;&nbsp;&nbsp;&nbsp;-&gt; GLOBAL의 L.E에서 a 탐색 -&gt; 1 출력</td>
  </tr>
  <tr>
    <td colspan="3">[9] outer 함수 종료 outer 실행 컨텍스트 제거<br>[10] 전역 컨텍스트 재활성화</td>
  </tr>
  <tr>
    <td colspan="3">[11] GLOBAL의 L.E에서 a 탐색 -&gt; 1 출력</td>
  </tr>
  <tr>
    <td colspan="4">[] 전체 종료. 전역 컨텍스트 제거</td>
  </tr>
</tbody>
</table>

inner 함수 내부에서 a 변수를 선언했기 때문에 전역 공간에서 선언한 동일한 이름의 a 변수에 접근할 수 없다.  
이를 **_변수 은닉화(variable shadowing)_** 라고 한다.

### 전역변수와 지역변수

- 전역변수 : 전역 공간에서 선언한 변수
- 지역변수 : 함수 내부에서 선언한 변수

## this

실행 컨텍스트의 thisBinding에는 this로 지정된 객체가 저장된다. 실행 컨텍스트 활성화 당시에 this가 지정되지 않을 경우 this에는 전역 객체가 저장된다. 그 밖에는 함수를 호출하는 방법에 따라 this에 저장되는 대상이 다르다.
