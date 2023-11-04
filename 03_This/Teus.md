# 0\. JS의 함수와 method

Python을 예로보면, method는 항상 cls 혹은 self를 매개변수로 받습니다.

때문에 해당 method가 사용하는 self(JS의 this)가 무엇인지 명확합니다.

```python
class mycls:
    #매개변수 첫번째에서 self를 받으므로써 method임을 명시해줌
    def my_method(self, a):
        ...
```

하지만 JS의 경우 함수와 method의 구분이 느슨한 편 입니다.

그럼 JS에서 this가 어떻게 동작하는지 알아보겠습니다.

# 1\. this 그때는 맞고 지금은 틀린다\.

#### 1\_1. 전역공간에서의 this

전역공간(global)영역에서 JS는 this를 사용할 수 있습니다.

```js
console.log(this)
//>>Window{...}
console.log(this ==== window)
//>>true
```

이때 window Object가, JS에서 사용되는 window와 완전히 일치합니다.

this는binding된 object를 가리키게 되는데

전역공간이란 사실 window Object의 영역 이라고 할 수가 있습니다.
(전역공간 = 전역Object의 Scope = windowObject의 Scope)

그래서 window Object에 property를 추가할 경우 변수처럼 사용할 수가 있습니다.

```js
window.a = 20
console.log(a)
//>>20
```

위 부분도

전역공간이 window Object의 scope이기 때문에

변수a가 사실 `window Object`의 `property a`인것을 예상할 수 있습니다.
(대신 완전 같지는 않고, delete의 상호작용이 차이가 납니다)

#### 1\_2. method로서 호출할 때 그 method 내부에서의 this

함수는 누군가와 무관하게 동작하고(독립적이다)

method는 어떤 Object와 연계되어 동작합니다(종속적이다)

위 때문인지, this역시 함수에서 동작할때와 method에서 동작할때 다른동작을 하게 됩니다.

```js
var func = function(x){
    console.log(this, x);
}
func(1);     // Window {...} 1

var obj = {method : func}
obj.method(2)// {method:f } 2
//method를 부르는 시점에서 가장 가까운 object를 this로 가지고간다.
//obj1.obj2.obj3.obj4.obj5.method(2) 라면
//위 상황에서 this는 obj5가됨.
```

같은 익명함수여도

변수에 할당되어 불리울때(함수)와

Object의 Property에 할당되어 불리울때(Method)가 다른 결과를 보여주게 됩니다.

즉

###### `JS에서 this는 함수를 호출하는 순간에 결정됩니다.`

위와같은 특징 때문에 다른 언어를 사용하던 개발자들에게 JS의 this는 악몽처럼 작동합니다.
(다행히 ES6 이후에 화살표함수를 사용하면 해당 문제를 마주할 일이 없지만, 패키지를 쓰다보면 결국 다시 만납니다😫)

#### 1\_3. 함수로서 호출할 때 그 함수 내부에서의 this

함수 내부에서의 this : 함수의 호출 주체를 알 수 없기 때문에 this는 전역 객체를 의미하게됨.

```js
var fucn = function(){
    var inner = function(){
        console.log(this)
    }
}
func() // Window{...}

var inner2 = function(){
    console.log(this)
}
inner2() // Winodw{...}
```

#### 1\_4. method 내부함수에서의 this

이제 위 두가지를 혼합해 보겠습니다.

```js
var obj1 = {
    outer : function() {
        console.log(this);            //1️⃣
        var innerFunc = function() {
            console.log(this);        //2️⃣
        }
        innerFunc()
        
        var obj2 = {
            innerMethod : innerFunc    //3️⃣
        };
        obj2.innerMethod();
    }
}
/*
1️⃣ {outer:f} == obj1
2️⃣ Window{...}
3️⃣ {innerMethod:f} ==obj2
*/
```

1️⃣ : 익명의 함수이고, 해당 함수는 obj1의 `outer` property에 할당되어 있으므로 `this = obj1`
2️⃣ : 함수 내부에서 불리는 this이므로`this = window`
3️⃣ : obj1.outer함수 내부의 obj2 Object의 `innerMethod` proeprty에 할당되어 있으므로 `this = obj2`

정말 같은 문장이 언제는 맞고, 언제는 틀리게 되는 특성을 갖고 있습니다.

#### 1\_5. method 내부함수에서 this를 의미있게 쓰는법

함수 내부에서 this가 항상 window를 지목하는것은 설계상 오류입니다.

이럴때 함수 내부에서 method내부에서 사용하던 this를 쓰고싶을 수 있습니다.

이때는 `.bind`를 통해서 this를 이어주거나, 아니면 단순하게 this를 다른 변수에 저장하는 방법이 있습니다.

```js
//ES5까지의 Legacy 함수를 사용할때 자주 사용하게됨.
var obj1 = {
    outer : function() {
        console.log(this);            
        var innerFunc = function() {
            console.log(this);        
        }.bind(this)
        innerFunc()
    }
}

//자주사용안함
var obj1 = {
    outer : function() {
        console.log(this);            
        var _self = this
        var innerFunc = function() {
            console.log(_self);
        }.bind(this)
        innerFunc()
    }
}
```

두 방법 모두 함수 innerFunc 내부에서 정상적으로 obj1 Scope에 접근할 수가 있습니다.

#### 1\_6. 화살표 함수에서의 this 바인딩

`정리 : 화살표함수는 this바인딩을 하지 않습니다!`

때문에 가장 가까운 Scope에서 존재하는 this를 가지고와서 사용하게 됩니다.

```js
const a = () => {
    console.log(this)
}
//>> Window{...}
var obj1 = {
    outer : function() {
        console.log(this);            
        //기존의 함수표현식이 화실표함수로 변경
        const innerFunc = () => {
            console.log(this);        
        }
        innerFunc()
    }
}
//>> obj1
//>> obj1
```

#### 1\_7. 생성자 함수 내부에서의 this

JS에서도 Class가 존재하지만

Class 이전에 함수앞에 new를 붙여줄 경우 Class처럼 동작하게 됩니다.

```js
var Cat = function(name, age) {
    this.name = name;
    this.age = age;
    this.loc = "서울"
}
var nabi = new Cat("나비", 5);
// 새로운 Array를 만들 때 사용되는
// new Array(50)과 Array(50)의 차이를 아시겠나요?
```

new 키워드와 함께 사용하면 매개변수를 넣고 함수를 실행시키는것이 아니라

해당 값을 저장하는 Instance 형태로 저장되는것을 볼 수가 있습니다.

이때의 this는 함수 자기 자신(혹은 Class)를 가리키게 됩니다.

# 2\. 명시적\(강제로\) this를 바인딩하는 방법

위와같이 함수를 쓸 때, method를 쓸 때 this가 바뀌는 문제점이 존재합니다.

이때 JS에서는 일부 method를 통해서 this를 특정한 값으로 강제할 수가 있습니다.

이런 방법을 `this binding`이라고 합니다.

this binding을 하는 `call method` `apply method` `binding method` 에 대해서 알아보겠습니다.

#### 2\_1. call method

`Function.prototype.call(thisArg[, arg1[, arg2[...]]])`
call method는 method의 this binding을 함과 동시에 바로 실행시키는 방법 입니다.

아래를 보실까요?

```js
var func = function(a, b, c) {
    console.log(this, a, b, c)
}

func(1, 2, 3)                //Window{...}    , 1, 2, 3
func.call("hello", 1, 2, 3)  //String("hello"), 1, 2, 3
```

call을 사용하면 가장 처음 받는 매개변수를 this에 binding 시킵니다.

때문에 보는것처럼 this가 `String Hello`로 변경된 것을 볼 수 있습니다.

#### 2\_2. apply method

`Function.prototype.apply(thisArg[, argsArray])`
call method와 동일하게 첫번째 매개변수를 this에 binding 시킵니다.

하지만 차이점은 함수 자체의 매개변수를 Array 형태로 입력하게 됩니다.

```js
var func = function(a, b, c) {
    console.log(this, a, b, c)
}

func(1, 2, 3)                  //Window{...}    , 1, 2, 3
func.apply("hello", [1, 2, 3]) //String("hello"), 1, 2, 3
```

#### 2\_3. call/apply method 응용

#### 2\_4. bind method

`Function.prototype.bind(thisArg[, arg1[, arg2[...]]])`

```js
var func = function(a, b, c) {
    console.log(this, a, b, c)
}

func.bind({"x" : 1})
func(1, 2, 3)         //{"x" : 1}, 1, 2, 3
```

bind의 경우 function의 this를 영구적으로 바꿔주는 역할을 합니다.

그러기 때문에 위 코드를 보면`func.bind` 실행 이후에 func를 실행할 경우

this가 `{"x" : 1}`로 바뀌는것을 볼 수 있습니다.

#### 2\_5. Callback 함수의 binding
ES6에서 추가된 배열 method(Array.map, filter, reduce....)는

배열마다 적용할 함수를 콜백형태로 집어넣습니다.

이때 콜백함수 내부에서 this를 어떤 것으로 할지를 두번째 매개변수로 받습니다.
```js
my_obj = {
    b : 0,
    func : function(){[...Array(10).keys()].forEach(function(val, idx){
        this.b += 1
        console.log(this)
    }, this)
    }
}
my_obj.func()
conosle.log(my_obj)
>>{b : 10, func()}...
```
근데 사실 위처럼 매개변수로 this를 넣지않고

bind함수를 써도 동일한 결과를 얻을수는 있습니다.
```js
my_obj = {
    b : 0,
    func : function(){[...Array(10).keys()].forEach(function(val, idx){
        this.b += 1
        console.log(this)
    }.bind(this))
    }
}
```