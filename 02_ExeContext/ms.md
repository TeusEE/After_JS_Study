# 실행 컨텍스트

## 1. 실행 컨텍스트란?

- 실행 컨텍스트 : 실행할 코드에 제공할 환경 정보들을 모아놓은 객체
- 자바스크립트는 `call stack`에 컨텍스트를 쌓아 올림 => 가장 위에 쌓여있는 컨텍스트와 관련 있는 코드를 실행하면서 전체 코드의 환경과 순서를 보장함.

```js
var a = 1;
function outer() {
  function inner() {
    console.log(a); //undefined
    var a = 3;
  }
  inner();
  console.log(a); //1
}
outer();
console.log(a); //1
```

- [`가장 최상단 컨텍스트`]

1. [`전역 컨텍스트`] 자바스크립트 파일이 열리는 순간 전역 컨텍스트가 활성화됨.

2. [`outer`] outer함수를 호출 시 js엔진은 outer 함수의 환경 정보를 수집해 outer실행 컨텍스트를 생성 후 call stack에 담음 => outer 함수 내부 순차적 실행

3. [`inner`] inner 함수 내부 순차적 실행

- a 변수에 3 할당

4. [`outer`] inner 컨텍스트 제거, 2단계에서 3단계 넘어가면서 중단됐던 outer 함수 내부 순차적 실행

- a 변수 출력

5. [`전역 컨텍스트`] 1단계에서 2단계로 넘어가면서 중단됐던 코드 실행

- a 변수 출력

6. [` `] 빈 콜스택 => 종료

> 실행 컨텍스트가 콜 스택의 맨 위에 쌓이는 순간이 곧 현재 실행할 코드에 관여하게 되는 시점

## 2. 실행 컨텍스트에 담긴 정보는?

- Variable Environment
  - environmentRecord(snapshot)
  - outerEnvironmentReference(snapshot)
- Lexical Environment
  - environmentRecord
  - outerEnvironmentReference
- This Binding

### 2-1 Variable Environment

- 최초 실행 시의 스냅샷을 유지
- 실행 컨텍스트 생성 시 Variable Environment 정보를 먼저 담은 후 복제 하여 Lexical Environment을 만듦.
  - Variable Environment의 내부는 environmentRecord, outer-Environment Reference로 구성
  - => Lexical Environment의 내부도 동일

### 2-2 Lexical Environment

- 컨텍스트를 구성하는 환경 정보들을 사전에서 접하는 느낌으로 모아놓은 것

#### environmentRecord이란?

- 현재 컨텍스트와 관련된 코드의 식별자 정보가 저장
- js엔진은 함수 내부의 모든 변수를 순차적으로 읽고 수집함.

=> **_호이스팅_** 이란 가상의 개념으로 변수 정보를 수집하는 과정을 수월하게 하기위한 추상화 개념

- 변수 선언과 값 할당이 동시에 이뤄진 문장은 **_선언_**만 호이스팅
- 함수 선언문은 전체를 호이스팅하지만 함수 표현식은 변수와 동일하게 행동하여 변수 선언부만 호이스팅이 됨.
  => 문제점: 동일한 변수명에 서로 다른 함수를 할당할 경우 가장 마지막에 할당된 값이 이전 값을 덮어 씌우기 때문에 에러가 발생하지 않고 찾기도 어려움.

#### outerEnvironmentReference이란?

- outerEnvironmentReference는 해당 함수가 선언된 위치의 LexicalEnvironment를 참조함.
  => 가장 가까운 요소부터 차례대로만 접근, 다른 순서로 접근 불가능
- 만약 전역 컨텍스트의 LexicalEnvironment까지 탐색해도 해당 변수를 찾지 못한다면 undefined 반환함.
- 스코프 체인 상에 있는 변수라고 해서 무조건 접근 가능한 것은 아님 => 클로저 변수 은닉화 개념

##### 전역변수와 지역변수

- 전역 변수 : 전역 컨텍스트의 Lexical Environment에 담은 변수
- 지역 변수 : 그 밖의 함수에 의해 생성된 실행 컨텍스트의 변수들

=> 안전한 코드 구성을 위해 가급적 전역 변수 지양해야 함.

## this

- 실행 컨텍스트를 활성하는 당시에 지정된 this가 저장됨.
- 함수 호출 방법에 따라 값이 달라지고, 호출 방법이 지정되지 않는다면 전역 객체가 저장됨
