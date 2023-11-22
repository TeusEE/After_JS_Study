안녕하세요. Teus입니다.

이번주는 5주차로, JavaScript의 Closure와 Closure를 응용한 기술들에 대해서 다룹니다.

# 1\. What is Closure?

클로저는 함수가 존재하는 다양한 프로그래밍 언어에서 사용됩니다.

클로저는

`함수가 종료되었음에도, 함수 내부의 변수들이 살아있는 것을 이용하는 프로그래밍`

정도로 정리할 수가 있을것 같습니다.

아래 예제 코드를 보겠습니다.

```js
var outer = function () {
    var a = 1;
    var inner = function() {
        console.log(++a)
    }
    return inner
}
temp = outer()
temp() //>>2
temp() //>>3
```

JS의 Context에서 배웠듯

`outer func의 LexEnv` ⬅ `inner의 outerEnv`

이기 때문에 inner에서 `var a`에 접근할 수 있습니다.

중요한점은

**outer내부의 `inner`에 대한 참조가 함수 외부에서 생깁니다.**

그리고 inner의 outerEnv는 outer의 LexEnv이기 때문에

outer의 LexEnv는 GC대상에서 제외되게 됩니다.

정리하면

> -1. inner의 외부 참조가 생김
> -2. inner를 실행하기 위해서는 outer의 정보가 유지되어야됨
> -3. outer의 정보(LexEnv)가 GC대상에서 제외됨

이러한 논리로 클로저가 작동합니다.

이때 `inner`의 참조 때문에 이런일이 일어났죠?

그래서 return으로 함수를 반환하는것 이외에 다른 방법으로도 closure가 만들어질 수 있습니다.

아래 예시를 보실까요

```js
(function() {
    var a = 0;
    var intervalId = null;
    var inner = function() {
        if (++a >10) {
            clearInterval(intervalId)
        }
        console.log(a);
    };
    intervalId = setInterval(inner, 1000)
})();
```

함수를 실행하지만, `setInterval`을 통해서 `inner`가 window에 참조되기 때문에

함수를 실행한 이후에서 함수 내부 변수값들이 살아남게 됩니다.

# 2\. Closure와 Memoery

이전에 JS는 Python과 유사하게 변수의 reference count를 측정하고

이것을 기반으로 GC를 진행한다고 했습니다.

근데 Closure는 `inner Func`에 Reference를 유지시켜서 GC의 대상에서 벗어나게 합니다.

이말즉슨 Closure를 쓰면 Memory를 지속적으로 점유하게 됩니다.

그러기 때문에 Closure의 경우 참조를 끊어줌으로써 메모리할당을 해제할 수 있습니다.

```js
var outer = (function() {
    var a = 1;
    var inner = function() {
        return ++a;
    }
    return inner
}

outer2 = outer()
console.log(outer2())
console.log(outer2())
outer2 = null //outer() Object(=inner func)에 대한 Reference를 해제시켜서 메모리 누수 방지
```

# 3\. 그래서 이걸 어따쓰죠?

#### 3\_1. 콜백 함수 내부에서 외부데이터를 사용하고자 할 때

```js
var fruits = ["apple", "banana", "peach"]
var $ul = document.createElement("ul");

fruits.forEach(function(fruit){ 2️⃣
    var $li = document.createElement('li');
    $li.innerText = fruit;
    #li.addEventListener('click', function(){ 1️⃣
        alert('your choice is ' + fruit);
    })    
    $ul.appendChild($li);
})
```

위 코드를 보면 `function(){alert...}`의 실행의 주체는

사용자가 아니라 EventListener에게 넘어갔습니다.

1️⃣에서 2️⃣의 변수 fruit을 참조하고 있기 때문에

Loop문이 끝난 후에도 2️⃣에 대한 LexEnv정보가 남게 됩니다.

이때 익명함수가 아니라 기명함수로 사용하기 위해서는 고차함수를 사용해 주면 됩니다.

> 고차함수는 함수의 결과값으로 함수를 주는 경우를 말합니다.

```js
var fruits = ["apple", "banana", "peach"]
var $ul = document.createElement("ul");

var alertFruitBuilder = function(fruit) {
    return function(){
        alert('your choice is ' + fruit);
    }
}

fruits.forEach(function(fruit){
    var $li = document.createElement('li');
    $li.innerText = fruit;
    #li.addEventListener('click', alertFruitBuilder(fruit));
    $ul.appendChild($li);
})
```

#### 3\_2. 접근 권한 제어

접근권한 제어의 경우 다른 프로그래밍 언어에서 Class의 public, private, protected로 유명합니다.

JS에도 Class가 존재하나, **Closure를 사용해서 충분히 위와같은 효과를 낼 수 있습니다.**

아래 예시를 보실까요?

```js
var car = {
    fuel : Math.ceil(Math.random() * 10 + 10),
    power : Math.ceil(Math.random() * 3 + 2),
    moved : 0,
    run : function() {
        var km = Math.ceil(Math.random()*6);
        var wasteFuel = km / this.power;
        this.fuel = this.fuel - wasteFuel;
        this.moved = this.moved + km;
        console.log(km + 'km 이동 (총 ' + this.moved + 'km)');        
    }
}
car()
...
```

위 같은경우 아예 Object 형태로 property와 function을 반환 했습니다.

문제는 위 코드에서`car.fuel = 1000` 같이 유저가 Object 내부에 바로 접근할 수가 있습니다.

이때 Closure를 사용하면 외부의 접근을 통제할 수가 있게 됩니다.

```js
var createCar = function(){
    var fuel = Math.ceil(Math.random() * 10 + 10)
    var power = Math.ceil(Math.random() * 3 + 2)
    var moved = 0
    var ret = {
        get moved(){
            return moved
        };
        run : function() {
            var km = Math.ceil(Math.random()*6);
            var wasteFuel = km / this.power;
            this.fuel = this.fuel - wasteFuel;
            this.moved = this.moved + km;
            console.log(km + 'km 이동 (총 ' + this.moved + 'km)');        
        }
    }
    Object.freeze(ret)
    return ret;
}
```

> 근데 보면 알겠지만, 사실 Class를 쓰면 손쉽게 해결될 부분이긴 합니다.
> 문제가 JS에서는 Class를 자주 사용을 안한다는 점이죠.

#### 3\_3. 부분 적용 함수

부분함수는 `함수의 매개변수 일부를 고정시킨 함수`를 의미합니다.

예를들어

N개의 매개변수를 받는 함수가 있다면, M개의 매개변수를 고정한 부분함수를 만들어내고

이후에는 `N-M개의 매개변수만 사용하는 함수`를 만들어내는 것이지요.

이때 bind를 사용하면 기본적인 부분함수를 적용할 수 있습니다.

```js
var add = function() {
    var result = 0;
    for (var i = 0; i < arguments.length; i++) {
        result += arguments[i]
    }
    return result
}
var addP_func = add.bind(null, 1, 2, 3, 4, 5)
console.log(addP_func(6, 7, 8, 9, 10))
//>> 55
```

하지만 bind를 사용할 경우 함수 내부의 `this를 강제로 다른 값으로 치환`해 주어야 합니다.

이때 별도의 고차함수를 사용해서 this를 binding하지않고 부분함수를 만들 수 있습니다.

```js
var p_func = function() {
    var ori_Partialargs = arguments;
    var func = ori_Partialargs[0];
    if (typeof func !== "function"){
        throw new Error("첫번째 인자가 함수가 아닙니다");
    }
    return function() {
        //func외에 매개변수를 저장해둠
        var partialArgs = Array.prototype.slice.call(ori_Partialargs, 1);
        //이후 부분함수가 받아들일 매개변수
        var restArgs = Array.prototype.slice.call(arguments);        
        //두 매개변수를 concat해서 함수의 arguments로 집어넣고 실행시킴
        //여기서 함수 실행 시점의 this를 사용하기 때문에 this binding이 문제가 되지 않음
        return func.apply(this, partialArgs.concat(restArgs))
    }
}

var add = function() {
    var result = 0;
    for (var i = 0; i < arguments.length; i++) {
        result += arguments[i]
    }
    return result
}

var p_add = p_func(add, 1, 2, 3, 4, 5)
console.log(p_add(6, 7, 8, 9, 10))
//>>55
```

> Python같은 경우는 functools패키지에서 partial함수를 만들수 있는 [functools.partial](https://docs.python.org/ko/3/library/functools.html?highlight=partial#functools.partial) 함수를 지원해 주기 때문에, 위처럼 복잡하게 만들 필요가 없기는 합니다.

위와같은 partial함수를 사용하는 예시로, JS의 디바운스가 있습니다.

> 디바운스는 짧은시간동안 동일한 다수의 이벤트가 발생했을 경우 마지막 이벤트만 처리하는 기법 입니다.

이때 Closure의 개념과 부분함수가 사용되게 됩니다.

```js
var debounce = function(eventName, func, wait){
    var timeoutId = null;
    return function(event){
        var self = this
        //외부함수가 받은 매개변수 eventName, func, wait에 대한 reference를 발생시킴
        console.log(eventName, 'evnet발생');
        //기존 setTimeOut된 timeoutId가 있다면 초기화시킴
        clearTimeout(timeoutId)        
        //새롭게 timeoutId를 설정해줌
        timeoutId = setTimeout(func.bind(self, event), wait)
    }
}

var moveHandler = function(e){console.log("move event 처리")};

//500ms내에 마우스 move가 있다면 기존 timeout을 초기화시키고 다시 setTimeout을 실행시킴
document.body.addEventListener("mousemove", debounce("move", moveHandler, 500))
```

#### 3\_4. 커링 함수

커링함수는 `함수의 return값이 또다른 함수가 나와서 매개변수를 저장할 수 있는 고차함수의 일종`를 의미합니다.

말이 어렵죠?

아래 코드를 보시겠습니다.

```js
var curring = function(func){
    return function(a){
        return function(b){
            return func(a, b)
        }
    }
}
var getMaxWith10 = curring(Math.max)(10);
console.log(getMaxWith10(8));  //10
console.log(getMaxWith10(25)); //25
```

부분적용 함수랑 개념은 비슷하나, 한번에 하나씩 매개변수를 받고

최종 함수해서 이 매개변수들을 사용하게 됩니다.

이때 매개변수를 미리 입력받고, 실행은 나중에 한다하여 `Lazy Execution`이라고 합니다.

근데 보면 콜백지옥마냥 Code의 Depth가 늘어나는것을 볼 수 있습니다.

이때 `화살표함수`를 쓰면 이런문제를 말끔하게 해결할 수가 있습니다.

```js
var curring = function(func){
    return function(a){  2️⃣
        return function(b){  1️⃣
            return function(c){
                return func(a, b, c)
            }
        }
    }
}

curring = func=>a=>b=>c=>func(a, b, c);
```

이때 이게 왜 Closure와 관계가 있는가 할 수 있는데

1️⃣ 위치에서 내부에서 `식별자 a`가 사용되고 있기 때문에

2️⃣ 의 LexEnv가 GC의 대상에서 벗어나게 되기 때문에 Closure의 동작과 정확하게 일치합니다.

이때 활용 사례 한번 보겠습니다.

```js
var getInformation = baseUrl => path => id => fetch(baseUrl+path+"/"+id);
var imageUrl = "http://imageAddress.com/"
var productUrl = "http://productAddress.com/"

var getImage = getInformation(productUrl)
var getEmoticon = getImage("emoticon');
var getIcon = getImage('icon');

var emoticon1 = getEmoticon(100)
var cmoticon2 = getEmoticon(102)
...
```

보는것처럼 모든 Fetch마다 FullPath를 넣는것이 아니라

커링함수를 통해서 공통되는 부분을 Lazy Execution으로 실행하는것을 볼 수가 있습니다.