# 0\. Callback Function

콜백함수는 사실 JS에만 있는 내용은 아닙니다.

Callback Func는 Call(부르다)Back(나중에) 입니다.

즉, `나중에 불려서 실행될 함수`를 Callback Func라고 합니다.

책에 재미있는 예시가 있는데

스폰지밥이 아침 6시에 약속이 있다고 했을때

1. **알람시계(CallBack Func)가 없을 경우**
    ![image.png](https://devocean.sk.com/editorImg/2023/11/5/6099fc12ba68a41c8cd596cfff67a168832f3f1e66d106f106aa07a1fb829532)
    6시가 되는것을 알기 위해서 뜬눈으로 6시까지 기다려야 합니다
2. **알람시계(CallBack Func)가 있는 경우**
    ![image.png](https://devocean.sk.com/editorImg/2023/11/5/b568e9a71aae09ea355a631fab552686113c1c8d4d55ac82885ef54370269978)
    나중에 알람시계가 동작할것을 알기 때문에 기다릴 필요 없이 편하게 다른일(여기서는 잠)을 할 수 있습니다.

간단하게 정리하면

`콜백함수 : 다른 함수의 매개변로 들어가는 함수`

를 의미하게 됩니다.

# 1\. Callback Function의 제어권

Callback Func는 다른 함수의 매개변수로 들어가게 됩니다.

때문에 해당 과정에서 자신의 실행 권한을 다른 함수에게 전달하게 됩니다.

이부분을 나눠서 살펴보겠습니다.

#### 1\_1. 호출 시점

```js
var count = 0;
var iter_func = function(){
    console.log(count);
    if (++count>4){
        //설정된 setInterval을 Clear해서 종료시킴
        clearInterval(timer);
    }
}
var timer = setInterval(iter_func, 300)
```

위 코드의 경우 300ms마다 count를 1씩 증가시키고 그 값을 출력하면서

이 값이 4를 넘어가면 반복을 종료합니다.

이때 setInterval을 사용하는 문법은 아래와 같습니다.
`var intervalId = scope.setInterval(func, delay[, param1, param2])`
(param1, 2는 func에 매개변수로 들어가게 됩니다)

코드에서 timer라는 변수가 무엇을 의미하는지 모호한데

이 친구는 `interval이 갖게되는 setInterval Object의 ID값`이 담기게 됩니다.
(그래서 이 변수를 CallBack Func 내부에서 사용해서 중지시키는것을 볼 수 있죠)

이제 CallBack 함수의 실행에 따라 실행 주체&제어권을 살펴보면 아래와 같습니다

| code | 호출 주체 & 제어권 |
| ---- | ----------- |
| iter\_func() | 사용자 |
| setInterval(iter\_func, 300) | setInterval |

`iter_func()` : 사용자가 원하는 시점에 실행하고 결과값을 받아옴
`setInterval(iter_func, 300)` : setInterval 함수가 알아서 300ms마다 실행시킴(사용자에게 제어권이 없음)

#### 1\_2. 매개변수를 전달하는 시점

Array.prototype.map함수를 잠시 살펴보겠습니다

```js
var myarr2 = [10, 20, 30].map(function(idx, val) {
    console.log(idx, val);
    return val+5
)}
console.log(myarr2)
/*
10, 0
20, 1
30, 2

[5, 6, 7]
*/
```

Array.prototype.map의 경우`Array.prototype.map(callback[, this])`의 문법으로 동작합니다.

이때 위 함수가 무언가 잘못 동작하는게 보이시죠?

value와 index가 반대로 출력되고 있습니다.

실제로 Array.prototype.map의 경우 Callback함수의 매개변수를 value,index형태로 주게 되어있습니다.

`Array.prototype.map(callback(currentValue[, index[, array]])[, thisArg])`

이처럼 Callback함수의 경우 호출되는 함수에 의해서 어떤 순서로 매개변수가 들어갈지 제어권이 넘어가 있는것을 볼 수 있습니다.
(=호출되는 함수가 어떻게 매개변수를 넣을지 이미 정해져 있다)

#### 1\_3. Callback함수 내부에서의 this의 결정

Callback함수의 경우 기본적으로 함수이기 때문에 this = window가 됩니다.

하지만 일반적으로 Callback함수를 받는 경우

Callback함수 내부에서 this로 사용할 Object를 매개변수로 받습니다.

이때 받은 매개변수가 어떤식으로 사용할지는 아래 Array.prototype.map을 구현하는 함수에서 볼 수 있습니다.

```js
Array.prototype.map = function (callback, thisArg){
    var mappedArr = [];
    for (var i = 0; i < this.length/*여기서 this는 나중에 받는 Array를 의미함*/; i++){
        //callback 함수를 실핼할 때 thisArg를 binding함과 동시에 실행시킴
        var tempval = callback.call(thisArg||window, this[i], i, this)
        mappedArr.push(tempval)
    }
    return mappedArr;
}
```

위와 같은 받식으로 thisArg를 받고

이를 func.prototype.call/apply/bind를 사용해서 호출함수 내부에서 this를 binding합니다.

정리하면 호출하는 함수가 Callback Func에서 this를 어떤것으로 사용할지를 결정하게 됩니다.

```js
[1, 2, 3].map(function(x, idx) {console.log(this)})
//>> 이 경우 별도로 binding이 안되었으므로 Window{...}
document.body.innderHTML += '<button id = "a">클릭</button>'
document.body.querySelector("#a").addEventListener("click", function(e){console.log(this)})
//>> 이 경우는 addEventListener가 this를 앞의 Object를 binding하게 정해져 있으므로
//<button id = "a">클릭</button>
```

# 2\. this 그때는 맞고 지금은 틀리다 in Callback Function

CallBack Func에서도 역시 this의 본질은 유지됩니다.

1. function으로 실행될 경우 window를 카리킨다
2. object의 method로 실행될 경우 object를 가리킨다
3. 위 두가지는 모두 실행되는 시점에 결정된다.

아래 예시를 보실까요?

```js
var obj = {
    vals : [1, 2, 3],
    logValues : function(v, i) {
        console.log(this, v, i)
    }    
}
obj.logValues(1, 2)                    // {vals : [1, 2, 3], logValues : f}
[4, 5, 6].map(obj.logValues)           // Window{...}
```

`obj.logValues()` : Object의 Func로서 실행되므로 this = obj
`Array.map(obj.logValues)` : Array의 Callback Func로 실행되므로 this = window

결국 그 함수가 어떻게 정의되어있던 실행 시점이 this를 결정하게 됩니다.

# 3\. Wrong Way\. this를 쓰지않고 object에 바로 접근하기

일반저긍로 this를 통해서 자기자신에 접근하지만

JS같은 경우 자신의 Object이름을 사용해서 바로 접근이 가능합니다.

```js
var obj1 = {
    name : "obj1",
    func : function() {
        console.log(obj1.name)
    }
}
obj1
//obj1
```

위처럼 사용하면 this가 window를 지목하는 문제에서 벗어날 수가 있습니다.

하지만 위같이 사용할 경우 해당 func의 재사용성이 떨어지게 됩니다.

```js
var obj1 = {
    name : "obj1",
    func : function() {
        console.log(obj1.name)
    }
}
var obj2 = {
    name : 'obj2',
    func : obj1.func
}
obj2.func()
//>>obj1
```

재사용성이 없는 코드에서만 쓸 수 있는 치트정도로 넘어가겠습니다.

# 4\. 콜백 지옥과 비동기 제어

callback 지옥은

**콜백** 안에 **콜백** 안에 **콜백** 안에 **콜백**....

처럼 콜백 내부에서 콜백이 실행되는 구조입니다.

이 경우 콜백만큼 indent가 들어가서 코드의 가독성이 떨어지고 유지보수를 어렵게 만드는 요인이 됩니다.

```js
setTimeout(function(name) {
    var coffeeList = name;
    console.log(coffeeList);    
    
    setTimeout(function(name) {
        coffeeList = coffeeList + "," +name;
        console.log(coffeeList);    
        
        setTimeout(function(name) {
            coffeeList = coffeeList + "," +name;
            console.log(coffeeList);    
            
            setTimeout(function(name) {
                coffeeList = coffeeList + "," +name;
                console.log(coffeeList);    
                
            }, 500, "카페모카")
        }, 500, "아메리카노")
    }, 500, "에스프레소")
}, 500, "말차라떼")
```

이때 위 구조를 기명함수로 바꿀 수는 있지만

한번 사용하는 익명함수를 기명함수로 만들어 쓰는것을 다소 비효율적이라고 볼 수 있죠

```js
setTimeout(function(name) {
    var coffeeList = name;
    console.log(coffeeList);        
    innser_func(coffeeList)
}, 500, "말차라떼")

function innser_func(prev) {
    setTimeout(function(name) {
    var coffeeList = prev + "," + name;
    console.log(coffeeList);            
}, 500, "에스프레소")}
```

이러한 CallBack 지옥을 막기 위해서

ES6에서 `Promise, Generator`가

ES2017에서 `async/await`이 도입되었습니다.

## 4\_1. Promise

Promise는 대표적으로 `fetch api`가 있습니다.

```js
fetch("http://www.google.com")
.then(resp=>resp.text())
.then(fin_resp=>console.log(fin_resp))
```

위 방법은 사실 풀어쓰면 아래와 같을겁니다.

```js
function my_fetch(domain) = {    

    setTimeout(function(domain) {
        //xtthprequests함수를 임의로 fetch_sync로씀
        var resp = fetch_sync(domain).text()
        
        setTimeout(function(resp) {
            console.log(resp)
        }, 10, resp)
    }, 10, domain)
}
setTimeout(my_fetch, 10, "http://www.google.com")
```

promise를 사용하면서 CallBack을 사용한 코드가 보다 직관적이고 간결한것을 확인할 수 있습니다.

위 콜백지옥 예제를 promise를 사용해서 구현하면 보다 직관직인 코드를 만들 수 있습니다.

```js
new Promise(function(resolve){
    setTimeout(function(){
        var name = "말차라떼";
        console.log(name);
        //resolver를 사용하면
        //다음 then의 input으로 넘어감
        resolve(name);
    }, 500)
}).then(function(prevName){
    return new Promise(function (resolve){
        setTimeout(function(){
            var name = prevName + ", 에스프레소"
            console.log(name);
            resolve(name);
        }, 500)
    })
}).then(function(prevName){
    return new Promise(function (resolve){
        setTimeout(function(){
            var name = prevName + ", 아메리카노"
            console.log(name);
            resolve(name);
        }, 500)
    })
}).then(function(prevName){
    return new Promise(function (resolve){
        setTimeout(function(){
            var name = prevName + ", 카페모카"
            console.log(name);
            resolve(name);
        }, 500)
    })
})
```

Promise를 좀더 간결하게 쓰는 방법이 있으나, 이 부분은 5장 클로저에서 다룹니다.

## 4\_2. Generator

Generator는 저는 실무에서 써본적이 많이 없습니다(Python에서는 좀 쓰지만요)

Generator는 일반 함수와 유사하나

**함수는 return을 만나고 종료된다면, Generator를 종료된는 시점에서 다시 시작합니다.**

Generator를 사용해서 위 코드를 다시 구현해 보면 아래와 같습니다.

```js
var addCoffee = function(prevName, name){
    setTimeout(function() {
        //callback내부에서 coffeeGenerator 내부의 진행을 만듦
        coffeeMaker.next(prevName? prevName + "," + name : name);        
    }, 500)
}

//*를 통해서 generator인것을 명시해줌
var coffeeGenerator = function*() {
    //main에서 coffeeMaker.next()를 실행했을 때 아래 코드에서 멈춤
    //그리고 coffeeMaker.next로 받은 매개변수가
    //return값으로 돌아옴
    var malcha = yield addCoffee("", "말차라떼");
    //yield addCoffee("", "말차라떼")에서
    //coffeeMaker.next()가 실행되면서 다음 코드가 실행됨
    console.log(malcha);
    var espresso = yield addCoffee(malcha, "에스프레소")
    console.log(espresso);
    var americano = yield addCoffee(espresso, "아메리카노")
    console.log(americano);
    var moca = yield addCoffee(americano, "카페모카")
    console.log(moca);
}

var coffeeMaker = coffeeGenerator();
coffeeMaker.next()
```

> Python같은 경우도 동일한 generator 문법이 있으니, 이처럼 비동기를 목적으로 사용하는 거는 굉장히 신박한 방법인거 같습니다

## 4\_3. async/await

ES2017에 추가된 비동기처리 문법 입니다.

`async` 키워드 같은경우 아예 함수 자체를 비동기 함수로 만들어 버립니다.

`await` 키워드는 async 함수 내부에서 비동기함수의 결과를 대기하는 역할을 합니다.

이 두가지를 합치면 기존 Promise/then 구문을 아래처럼 고칠수가 있게 됩니다.

```js
var addCoffee = function(name){
    return new Promise(function (resolve){
        setTimeout(function(){
            //resolve에 넣으면
            //해당 값이 await 이후 return됨
            resolve(name);            
        }, 500)
    })
}
var coffeeMaker = async function(){
    var coffeeList = "";
    var _addCoffee = async function(name){
        coffeeList += (coffeeList ? "," : "") + await addCoffee(name);
        //Promise를 통해서 await하고 return을 받을 수 있게 구성
    }
    await _addCoffee('말차라떼')
    console.log(coffeeList)
    await _addCoffee('에스프레소')
    console.log(coffeeList)
    await _addCoffee('아메리카노')
    console.log(coffeeList)
    await _addCoffee('카페모카')
    console.log(coffeeList)
}

coffeeMaker()
```

> Python같은 경우로 async def를 통해서 비동기함수를 만들 수 있습니다. 하지만 Python같은 경우 asyncio package를 통해서만 비동기함수를 실행할 수 있고
> JS같은 경우는 main에서 바로 비동기함수를 실행할 수 있다는 문법적인 차이점이 있습니다.