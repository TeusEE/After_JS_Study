# 데이터 타입

### 데이터 타입의 종류

#### 1. 기본형(Primitive Type)

값을 그대로 할당이나 복제

- Number
- String
- Boolean
- null
- undefined
- Symbol

#### 2. 참조형(Reference Type)

값이 저장된 주소값을 할당(참조)

- Array
- Function
- Date
- RegExp
- Map, WeakMap
- Set, WeakSet

### 식별자와 변수

변수와 식별자를 혼용하는 경우가 많습니다. 식별자라고 해야할 곳에 변수를, 변수라고 해야할 곳에 식별자를 쓰는 식으로 말입니다. 둘의 차이가 있지만 모를 수 있기에 혼란스러울 수 있습니다.
차이점을 정리하자면,

> 변수는 `'변할 수 있는 수'`이다.
>
> 변할 수 있는 무언가를 명사로 확장시켰습니다. 여기서 '무언가'란 데이터를 말합니다. (숫자, 문자열, 객체, 배열)
>
> 식별자는 `'어떤 데이터를 식별하는 데 사용하는 이름'` 즉 변수명입니다.

### 변수 선언과 데이터 할당

#### 변수 선언

> `변수 선언(Variable Declaration)`은 값을 저장하기 위한 `메모리 공간을 확보`하고 `변수 이름과 확보된 메모리 공간의 주소를 연결`해서 값을 저장할 수 있게 준비하는 과정입니다.

```javascript
// 변수 선언
var a; // 변수 선언
```

- 변수 선언은 `변수 이름을 등록`하고 `값을 저장할 메모리 공간을 확보`한다.
  - 변수 선언한 이후, 변수에 값을 아직 할당하지 않았다.
  - 따라서, 확보된 메모리 공간은 비어있을 것으로 생각할 수 있으나 확보된 메모리 공간에는 자바스크립트 엔진에 의해 `undefined` 값이 `암묵적으로 할당`되어 초기화된다.

#### 데이터 할당

```javascript
var a; // 변수 선언
a = "abc"; // 데이터 할당

var a = "abc"; // 변수 선언 + 데이터 할당
```

> `데이터 할당(Data Allocation)`은 데이터를 저장하기 위한 `별도의 메모리 공간 다시 확보(데이터 영역)`해서 데이터를 저장하고 `그 주소를 변수 영역에 저장`한다.

>

**_[ 💡 NOTE - 데이터 영역이라는 다른 메모리 영역을 사용하는 이유 ]_**

데이터 변환을 자유롭게 할 수 있게 함과 동시에 메모리를 더욱 효율적으로 관리하기 위한 고민의 결과입니다.

숫자형 데이터는 64비트 공간을 확보하지만, 문자열은 정해진 규격이 없습니다.
한 글자마다 각각 필요한 메모리 용량이 가변적이며 전체 글자수 역시 가변적이기 때문입니다.

데이터 변환 시 "확보된 공간을 변환된 데이터 크기에 맞게 늘리는 작업"이 선행되어야 하는데
마지막 데이터는 상관없지만, 중간에 있는 데이터는 전체 영역을 옮기고 늘리는 작업이 있어서 효율적이지 못합니다.
'데이터 영역' 이라는 새로운 영역을 나누면서 데이터가 변했을 때 '데이터 영역상에 새롭게 영역을 확보'하고 할당한 후 주소를 넣어주는 방식으로 진행하면 효율적입니다.

>

### 기본형 데이터와 참조형 데이터

#### 불변값

- `변수` VS `상수`를 구분하는 성질은 `변경 가능성`

  - **변수 영역 메모리**를 바꿀 수 있으면 변수, 없으면 상수. 데이터 할당이 한번 이뤄진 변수 공간에 다른 데이터를 재할당할 수 있는지 여부과 관건

##### 불변성

- `불변값` VS `상수`
  : 불변성 여부 구분할 때 변경 가능성의 대상은 `데이터 영역 메모리`
- 기본형 데이터는 모두 불변값

```javascript
var a = "abc";
a = a + "def"; // 새로운 문자열 'abcdef'를 만들어 그 주소를 변수 a에 저장.(abc / abcdef는 완전히 별도의 데이터)

var b = 5; // 데이터 영역에 5가 없으므로 새로 데이터 공간을 만들어서 5 저장.
var c = 5; // 데이터 영역에 저장된 5의 주소를 재활용해 c에 저장
b = 7; // 데이터 영역에 7을 만들고, b에 다시 저장. 5, 7은 다른 값을 변경할 수 없고 변수에서 새로운 값이 필요하면 새로 만들어야함.
```

- 위 코드와 주석을 보면,
- 불변값: 다른 값으로 변경할 수 없고, 새로 만드는 동작을 통해서만 변경이 가능하다. `한번 만들어진 값은 가비지 컬렉팅이 되지 않는 한 영원히 변하지 않음`

#### 가변값

기본형 데이터는 모두 불변값이라고 했습니다. 그렇다면 참조형 데이터는 모두 가변값일 것 같지만,
`기본적인 성질은 가변값인 경우가 많지만` 설정에 따라 변경 불가능한 경우도 있고, 아예 불변값으로 활용하는 방안도 있습니다.

```javascript
// 참조형 데이터의 할당
var obj = {
  a: 10,
  b: "bbb",
};
```

**변수 영역**
| 주소 | 1002 | 1003 | 1004 |
| ------ | -------- | -------- | -------- |
| 데이터 | | 이름:obj1<br/>값: @5001 | |

**데이터 영역**

| 주소   | 5001    | 5002 | 5003 | 5004  |
| ------ | ------- | ---- | ---- | ----- |
| 데이터 | @7103~? |      | 1    | "bbb" |

**객체@5001의 변수 영역**

| 주소   | 7103                | 7104             | 7105 |
| ------ | ------------------- | ---------------- | ---- |
| 데이터 | 이름:a<br/>값:@5003 | 이름:b<br/>@5004 |      |

1. 빈 공간(@1002)을 확보하고, 그 주소의 이름을 obj1으로 지정한다.
2. 임의의 데이터 저장 공간(@5001)에 데이터를 저장하려고 보니 여러 개의 프로퍼티로 이뤄진 데이터 그룹이다. 이 그룹 내부의 프로퍼티들을 저장하기 위해 별도의 변수 영역을 마련하고, 그 영역의 주소(@7103 ~ ?)를 @5001에 저장한다.

3. @7103, @7104에 각각 a와 b라는 프로퍼티 이름을 지정한다.

4. 데이터 영역에서 숫자 1을 검색한다. 검색 결과가 없으므로 임의로 @5003에 저장하고, 이 주소를 @7103에 저장한다. 문자열 ‘bbb’ 역시 임의로 @5004에 저장하고, 이 주소를 @7104에 저장한다.

기본형 데이터와의 차이는 `객체의 변수(프로퍼티) 영역`이 별도로 존재하는 점입니다.
객체가 별도로 할애한 영역은 변수 영역일 뿐 '데이터 영역'은 기존의 메모리 공간을 그대로 활용하고 있습니다.

데이터 영역에 저장된 값은 모두 불변값입니다. 하지만 변수에는 다른 값을 얼마든지 대입할 수 있기때문에 `흔히 참조형 데이터는 가볍값이다` 라고 하는 것 입니다.

##### 프로퍼티의 값의 변경

```javascript
obj1.a = 20;
```

이렇게 값을 변경하면 데이터 영역에 20이 생기고 그 영역의 주소값이 a 프로퍼티 변수 영역의 값으로 들어갈 뿐 `obj1의 값이 변하지 않습니다.`

기본형과 참조형을 비교 정리하자면,

기본형일 때 `변경은 새로 만드는 동작`에 의해서만 가능했지만

참조형은 `새로운 객체`가 만들어지는 것이 아니라 기존 객체 내부의 값만 바뀌게 됩니다.

>

**_[ 💡 NOTE - 가비지 컬렉터 ]_**

자바스크립트는 가비지 컬렉터(GC)라는 **자동 메모리 관리 형식을** 활용합니다.
가비지 컬렉터의 목적은 메모리 할당을 모니터링하고 할당된 메모리의 블록이 더이상 필요하지 않은 시점을 확인하여 회수하는 것입니다.

어떤 데이터에 대해 자신의 주소를 참조하는 변수의 개수를 `참조 카운트`라고 하는데,
참조 카운트가 `0`인 메모리 주소는 가비지 컬렉터의 수거 대상이 되고, 가비지 컬렉터는 런타임 환경에 따라 특정 시점이나 메모리 사용량이 포화 상태에 임박할 때마다 자동으로 수거 대상들을 수거한다.

>

#### 변수 복사

결론 부터 말하자면 어떤 데이터(기본형, 참조형)이든 변수에 할당하기 위해서는 주소값을 복사해야 합니다.

깊게 따지자면 `자바스크립트에서 모든 데이터 타입은 참조형 데이터일 수 밖에 없다` 라는 것입니다.

다만 기본형은 `주소값을 복사하는 과정이 한번`만 이뤄지고, `참조형은 한단계 더` 거치게 됩니다.

```javascript
var a = 10; // 10 이라는 값을 데이터 영역에 확보
var b = a; // 식별자 a를 검색하고 그 값을 데이터 영역에서 찾아옵니다. 저장된 데이터 영역의 주소값을 변수 영역에 대입합니다.

var obj1 = { c: 10, d: "ddd" }; // 변수 영역에 빈공간을 확보해 식별자는 obj1로 지정합니다. 데이터 영역에 빈공간을 확보한 뒤 별도의 변수 영역(객체 변수 영역)을 확보해 그 주소를 저장합니다. 객체 변수 영역에 c, d의 식별자를 입력한 다음 데이터 영역에 값을 검색(재활용)해서 넣거나, 새로운 영역을 만들어 대입합니다.

var obj2 = obj1; // 식별자 obj1을 검색해 그 값(데이터 영역에 해당하는 값 - 여기선 객체 변수 영역 주소값)을 가져와 식별자 obj2에 대입합니다.
```

### 불변 객체

불변 객체는 최근의 React, Vue 등의 라이브러리나 프레임워크 뿐만 아니라 함수형 프로그래밍, 디자인 패턴 등에서 매우 중요한 기초가 되는 개념입니다.

어떤 상황에서 불변 객체가 필요한가 보면
`값으로 전달받은 객체에 변경을 가하더라도 원본 객체는 변하지 않아야 하는 경우`가 있습니다.

이런 상황에서 불변성을 유지하려면 여러 방법이 있습니다.

- 새로운 배열을 만들어서 재할당(사용자 정의 함수)
- 각종 라이브러리 사용(immutable.js, baobab.js)
- Spread Operator 사용하기
- Object.assign 사용하기
- JSON 변환 후 다시 parsing

### 얕은 복사와 깊은 복사

> **얕은 복사(Shallow Copy)**: 1단계의 값만 복사
>
> **깊은 복사(Deep Copy)**: 내부의 모든 값들을 하나하나 전부 복사

여기서 말하는 복사는 주소값을 가져와서 저장하는 것이 아닌 새로운 객체를 생성해서 재할당하는 것입니다.

보통 중첩 객체, 중첩 배열이 아니면 **얕은 복사**를 진행하고, 그렇다면 **깊은 복사**를 진행합니다.

#### 얕은 복사 방법

- Array.prototype.slice()

- Object.assign()

- Spread 연산자 (전개 연산자)

#### 깊은 복사 방법

- JSON.parse && JSON.stringify
- 재귀 함수를 구현한 복사(사용자 정의 함수)
- lodash clonedeep 함수 사용
- structuredClone 함수 사용

### undefined와 null

#### undefined

undefined는 사용자가 명시적으로 지정할 수도 있지만 자바스크립트 엔진이 자동으로 부여하는 경우도 있습니다. 다음 세 경우가 이에 해당합니다.

- 값을 대입하지 않은 변수에 접근할 때
- 객체 내부의 존재하지 않는 프로퍼티에 접근하려고 할 때
- return문이 없거나 호출되지 않는 함수의 실행 결과

그런데 값을 대입하지 않은 변수, 이 경우에 대해 배열의 경우 undefined가 아니라 `empty`라고 해서 비어있는 요소가 되어버린다.

위 상황은 배열의 length 값을 직접적으로 변경하거나 new 연산자로 Array 생성자 함수를 호출하는 상황 때 발생된다.

`비어있는 요소`와 `undefined를 할당한 요소`는 출력 결과부터 달라집니다.

이렇게 여러 상황이 발생되는 것을 막기 위해 undefined가 되는 상황은 자바스크립트 엔진이 반환되는 경우로만 제한하는 것이 좋습니다.

#### null

`비어있음`을 명시적으로 나타내고 싶을 때에는 undefined 대신 null 을 사용하면 됩니다.

한가지 주의할 점은 `type of null` 이 **object**라는 것입니다.

이건 자바스크립트 자체 버그이기 때문에..

null을 걸러야 하는 상황이 있으면 일치 연산자(===)를 사용해서 정확히 판별해야 합니다.