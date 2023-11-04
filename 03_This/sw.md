# this
보통의 객체 지향 언어에서 this 는 클래스에서 생성한 인스턴스 객체를 가리킨다
자바스크립트에서의 this 는 어디서든 사용할 수 있고
상황에 따라 가리키는 대상이 다르다

함수와 객체의 구분이 느슨한 자바스크립트에서 this 는 이 둘을 구분하는 거의 유일한 기능이다

## 상황에 따라 달라지는 this
자바스크립트에서의 this 는 실행 컨텍스트가 생성될 때 함께 결정된다
실행 컨텍스트는 함수 호출 시 생성되므로
this 는 함수를 호출할 때 결정된다고 할 수 있다
함수를 어떤 방식으로 호출하느냐에 따라 값이 달라지는 것이다

### 전역 공간에서의 this
전역 공간에서 this 는 전역 객체를 가리킨다
브라우저에서는 window, node 환경에서는 global 이다

#### 전역 변수
전역 변수는 전역 객체의 프로퍼티로 할당한다
자바스크립트의 변수는 특정 객체의 프로퍼티로 동작한다

변수를 선언하면 자바스크립트 엔진은 특정 객체의 프로퍼티로 인식한다
여기서 특정 객체란 실행 컨텍스트의 LexicalEnviroment 다
실행 컨텍스트는 변수를 수집하여 L.E 의 프로퍼티
이후 변수를 호출하면 L.E 를 조회해서 일치하는 프로퍼티가 있을 경우 그 값을 반환한다
전역 컨텍스트의 경우 L.E 는 GlobalEnviroment 를 참조하고 G.E 가 전역 객체를 참조한다

전역 변수를 선언하면 자바스크립트 엔진이 전역번수를 전역 객체의 프로퍼티로 할당한다
스코프 체인에서 가장 가까운 곳 부터 가장 마지막 전역 스코프의 L.E 인 전역 객체까지 변수 참조가 이루어져
변수가 있으면 값을 참조하고 없다면 undefined 를 반환한다

대체로는 window 프로퍼티에 값을 직접 할당하는 것과 전역 변수는 같게 동작하지만
delete 만 다르게 동작한다
전역 변수는 삭제되지 않고 프로퍼티는 삭제 된다

### 메서드 내부에서의 this
함수와 메서드의 차이는
함수는 독립적이고 객체에 대한 동작을 수행한다

객체의 프로퍼티로 할당된 함수가 모두 메서드로 동작하는 것은 아니다
es6 에서는 메서드 축약표현으로 작성된 것만이 메서드로서 동작한다 

아래는 함수로서 호출과 메서드로서 호출을 나타내는 코드

```js
const func = function (x) {
	console.log(this);
}

func(1); // this -> Winodw

const obj = {
	method: func
}

obj.method(2); // this -> obj
```
호출하는 방법에 따라 this 가 달라진다

함수의 this 는 전역 객체를 가리키고
메서드의 this 는 인스턴스를 가리킨다

객체 내에서 정의된 함수 또한 전역 객체를 가리킨다

es5 까지는 this 를 내부 함수에 상속할 방법이 없지만
변수를 통해 우회할 수는 있다

객체 내부에 변수를 선언하고 변수에 this 를 넣은 후
다른 곳에서 변수를 참조하면 된다

es6 의 화살표 함수는 this 바인딩을 하지 않기 때문에
함수가 전역 객체를 가리키지 않고 상위 스코프의 this 를 활용한다

### 콜백 함수의 this
콜백 함수의 this 또한 기본적으로는 전역 객체를 가리키지만
제어권을 받은 함수에서 콜백 함수에 this 가 될 대상을 지정하면
그 대상을 참조하게 된다

함수와 메서드의 차이와 비슷하게
콜백 함수가 객체 내에서 메서드로 실행된다면 해당 객체를 가리키게 된다

### 생성자 함수의 this
생성자 함수의 this 는 생성되는 인스턴스를 가리키게 된다
new 키워드로 호출되는 함수는 생성자 함수로 동작한다
생성자 함수를 호출하면 생성자의 prototype 프로퍼티를 참조하는 `__proto__` 프로퍼티를 가진
인스턴스를 만들고 미리 준비된 공통 속성 및 개성을 해당 인스턴스에 부여한다

## this 바인딩
call, apply, bind 메서드를 통해 this 를 원하는 객체에 바인딩 할 수 있다

call 은 유사배열객체에 배열 메서드를 적용할 수 있도록 하기도 한다


## 화살표 함수
es6 의 화살표 함수는 this 바인딩이 없고 상위 스코프의 this 를 사용하게 된다

### 별도 인자로 this 를 받는 경우
콜백 함수 내에서의 this 

콜백 함수를 인자로 받는 메서드 중 일부는 추가로 
this 로 지정할 객체를 인자로 지정할 수 있는 경우가 있다
this 값을 지정하면 콜백 함수 내부에서 this 값을 변경할 수 있다
주로 배열 메서드에 많이 있다