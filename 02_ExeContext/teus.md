# Execution Context
# 1. Execution Context란?
Execution Context(이하 exe_context)의 경우

코드 실행을 위한 정보를 모아놓은 Object type의 object입니다.

이때 해당 정보를 하나가아닌 둘 이상의 exe_context를 관리해야 하는데

이때 `Queue`나 `Stack`을 사용하게 됩니다.
1.Queue : ..
2.Stack : ..

이때 javascript의 경우 `call stack` 을 통해서 가장 최근에 실행한 exe_context를 Stack 최상단에 유지시켜놓습니다.

아래 실행을 예시로 보겠습니다.
```js
var a = 1;
function outer() {
    function inner() {
        1️⃣
        console.log(a); //undefined
        var a = 3;
    }
    inner()
    console.log(a); //1
}
outer()
console.log(a); //1
```
위 코드를 실행하면 1️⃣위치에서 아래처럼 Context가 변하게 됩니다.

inner() Context
----------------
outer() Context
----------------
Global Context

이때 Stack 최 상단에는 inner에 해당하는 Context가 존재하고

inner Context내부에는 a에 대한 정보가 없이 때문에 undefined를 표시해 줍니다.
> 이때 특이사항으로, 변수 Context내부에서는 전체 Code를 훑고, 변수가 있을 경우
> 변수 선언 전 위치에서는 undefined를 반환해주는것을 볼 수 있습니다.
> 이 내용은 변수 hoisting부분에서 더 다룹니다.

이때 해당 Context가 가지고 있는 정보들을 간단히 실펴보면
1.VariableEnvironment : 
2.LexicalEnvironment :
3.ThisBinding : 

# 2. VariableEnvironment란?
LexicalEnv 같은경우 코드가 실행되면서 그 내부값이 변경되게 됩니다.

VariableEnv 같은 경우 LexicalEnv가 생성될 때의 초기값을 스냅샷 형태로 저장하고 있습니다.

# 3. LexicalEnvironment란?
위에서 말 했듯 Code가 실행됨에 따라 지속적으로 변경이 발생하는 Context로

Code실행을 위한 정보를 담고 있습니다.

## 3_1. environmentRecord와 호이스팅
envRecord는 현재 Context와 관련된 모든 식별자 정보를 저장하게 됩니다.

이때 해당 Context가 모든 식별자를 관리하는 과정에서 JavaScript의 악몽과도같은 변수 호이스팅이 발생합니다.

**JavaScript엔진은 함수 내부의 모든 변수를 처음부터 끝까지 읽고, 순서대로 수집하게 됩니다.**

즉 JavaScript엔진은 코드를 실행하기 이전에 해당 코드에 존재하는 식별자의 정보를 모두 가지고 있습니다.

때문에 아래와 같은 코드에서 변수가 없어야 하는 위치에서

변수가 없다는 에러가 아닌, undefined를 띄워주는것을 볼 수 있습니다
```js
console.log(a);
var a = 30;
```

이때 아래와 같이 동작을 했다고 생각을 해 볼 수 있습니다.
```js
var a;
console.log(a);
a = 30;
```
이처럼 변수가 위로 올라가는 효과가 있다하여, `변수 hoisting`이라고 불립니다.

### 3_1_1. 호이스팅 규칙
1. 변수 선언과 동시에 값이 할당되는 경우, 변수 선언이 최상단에서 동작하게 됨

그래서 아래 코드는
```js
console.log(x);
var x;
console.log(x);
x = 20;
console.log(x);
```

실제는 아래처럼 동작해서
```js
var x
console.log(x);
console.log(x);
x = 20;
console.log(x);
```

아래의 결과를 보여주게 됩니다.
```js
>>undefined, undefined, 20
```
2. 함수의 경우 변수와는 약간 다른 동작을 하게 됩니다.

함주를 선언할 수 있는 방법을 아래처럼 3가지가 있습니다
```js
function a() {/*...*/}          //함수 선언문, 함수명 a가 곧 변수명 
var b = function() {/*...*/}    //(익명)함수 표현식, 변수명 b가 곧 함수명
var c = function d () {/*...*/} //기명 함수 표현식, 잘 안쓰임
```
이때 함수 선언문과 함수표현식 두가지를 모두 사용한 함수에서

호이스팅이 동작하는것을 보겠습니다.
```js
console.log(sum(1, 2))
console.log(multiply(3, 4))

function sum(a, b){
    return a+b
}

var multiply = function(a, b){
    return a*b
}
```
위 상태일때 일반 변수였다면 sum과 multiply가 모두 호이스팅되고

모두 error 혹은 undefined가 나왔을 겁니다.

하지만 호이스팅이 되었을때는 아래처럼 동작하게 됩니다.
```js
var sum = function sum(a, b){// 함수 선언문은 전체를 호이스팅함
    return a + b
}
var multiply;//함수 표현식은 변수 선언부만 호이스팅함
console.log(sum(1, 2))
console.log(multiply(3, 4))

multiply = function(a, b){
    return a * b
}
```
변수에 대입하는 방식인 함수표현식은 변수와 동일하게 행동하여

변수의 선언부만 호이스팅되고, 실제 변수에 함수를 대입하는 과정을 위치가 고정되게 됩니다.

이러한 문제가 있어서 의도한 것이 아니면 호이스팅에서 안전한 함수 표현식을 사용하는것이 옳은 방법 입니다.
(ES6 이후에 const함수가 함수표현식을 쓰는것을 통해서도 유추해볼 수 있습니다.)

## 3_2. 스코프, 스코프 체인, outerEnvironmentReference
스코프는 많은 프로그래밍 언어에서 사용되는 용어 입니다.

Scope는 식별자가 유효한 범위를 의미합니다.

그래서 일반적으로 프로그래밍언어 에서는 {} 대괄호를 사용해서 Scope를 구분 합니다.
(python은 indent를 사용하죠)

특이하게도 JavaScript의 경우 ES5까지는 함수만이 Scope를 만들 수 있었고

ES6 이후에서야 let, const, class, strict mode의 함수를 통해서 다른 프로그래밍언어들과 비슷하게 Scope를 만들 수 있게 되었습니다.

이러한 Scope는 안에서 밖으로 검색해 나가게 되며, 이러한 방식은 Scope Chain이라고 합니다.

그리고 Scope Chain을 가능하게 만들어주는게 LexicalEnv를 구성하는 두번째 자료인 outerEnvironmentReference(outerEnvRef)입니다.

### 3_2_1. 스코프 체인
LexicalEnv가 생성될 때, 이전 LexicalEnv를 outerEnvRef가 포인팅 합니다.

예를들어 보겠습니다.

```js
var outer = function(){
    var inner = function(){
        1️⃣
        console.log(a);
        var a = 3;
    }
    inner();
    conosle.log(a);
}
outer();
conosle.log(a);
```
위처럼 main -> outer -> inner 형태로 함수가 구성된 경우
`main's LexEnv`  <- `outer's outerEnvRef` 
`outer's LexEnv` <- `inner's outerEnvRef` 
형태로 연결리스트처럼 이어지게 됩니다.

그래서 1️⃣ 위치에서 a를 찾기위해서는
1. `inner의 LexEnv`에서 식별자a를 찾음
2. 없으면 `inner의 outerEnvRef`로 이동(=outer의 LexEnv)
3. `outer의 LexEnv`에서 식별자a를 찾음
... main까지 반복

하게 됩니다.

이때 가장 가까운 위치에서 찾은 식별자의 데이터를 가지고 오고, 탐색을 멈춥니다.
> 이때 식별자를 찾고, 멈추는것을 응용해서
> 변수를 은닉화 할 수 있습니다.
> ex.scope 밖에있는 값을 보여주지 않기 위해서 일부러 scope안에 같은 이름의 변수 만들기.

JavaScript는 browser상에서 `debugger`라는 명령어는 통해서

스코프체인 내에 변수와 스코프정보를

dubugger를 실행한 위치에서 중단점을 만들어 확인할 수가 있습니다.