# ES6
## 화살표 함수 기본
(매개변수) ⇒ { 실행할 코드 };
```javascript
const arrowPower = (x) => {
    return x * x;
};

// 매개변수가 딱 하나일 경우 소괄호 생략 가능
// 실행문에 리턴값만 존재할 경우 중괄호와 return 구문 생략 가능 -> 중괄호를 쓰면 return 구문 생략 적용 안됨 (명시 안하면 undefined 반환됨)
const square = x => x * x;

// 매개변수가 없거나 여러개인 경우 소괄호 생략 불가
const greet = () => '안녕하세요';
const add = (a, b) => a + b;

// 중괄호 본문 : return 을 꼭 명시해서 반환해야 함
const wrongSquare = x => {
    x * x;
}
console.log(wrongSquare(3)); // undefined
```

#### 화살표 함수를 사용하는 이유
기존의 함수는 자신만의 this 가 존재하는데 화살표 함수는 자기만의 this 가 존재하지 않다. 그렇기에 외부의 this 를 사용한다.

&nbsp;

### 화살표 함수 객체
```javascript
// 객체를 반환할 시, 소괄호로 감싸서 실행문이 아니고 반환할 객체를 만드는 표현식임을 표시해 주어야 한다.
// 화살표 다음 중괄호가 오면 일반 함수 중괄호 본문으로 인식하기에 반환할 객체 표현식인걸 알리기 위해 소괄호로 한번 감싸줘야한다.
const createUser = (id, name) => ({id: id, name: name});
createUser(1, 'panda');
```

&nbsp;

### 화살표 함수 콜백
```javascript
// 화살표 함수도 다른 함수에 콜백으로 전달 가능
function calculate(value, operation) {
    return operation(value);
}
console.log(calculate(3, power)); // 9 // power(3)
console.log(calculate(3, number => number * number)); // 9 // (number) => { return number * number; }
```