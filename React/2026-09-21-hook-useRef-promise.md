# React Hooks
- useState : 화면에 출력되고 변경 즉시 UI 가 달라져야 하는 값
- useEffect : 렌더링 후 동기화
- useMemo : 값이나 객체 참조를 저장하여 재사용 (최적화, useEffect 의 의도치 않은 동작 제어)
- useCallback : 함수를 저장하여 재사용 (최적화, useEffect 의 의도치 않은 동작 제어)
- useRef : 렌더링 사이에 기억해야 하지만, 변경만으로 리렌더링할 필요가 없는 값을 저장

## useMemo
리렌더링 중 **의존성 배열에 있는 상태값이 변경되지 않았다면 이전에 기억한 값을 사용**한다.  
`const 기억저장변수 = useMemo(() => { return 계산하여 기억할 값; }, [계산에 사용한 의존성 상태값]);`  
useEffect 는 렌더링이 완료된 후에 실행되고, useMemo 는 렌더링 중에 실행된다. (평범하게 위에서 아래 코드로 실행되고 실행될 때는 그만큼 아래 코드가 기다림)  
비용이 높은데 불필요한 재 계산이 많이 발생할 경우 **최적화를 위해** 사용  
```jsx
const { useState, useMemo } = React;

const hardCalculator = (num) => {
    console.log("어려운 계산 시작");
    // 실은 안에서 별다른 동작을 안해도 시간을 꽤 잡아먹나? > 이정도여야 겨우 지연 발생하면 신경 안 써도 될듯...
    for(let i = 0; i < 999999999; i++) { }
    return num + 10000;
}

const easyCalculator = (num) => {
    console.log('쉬운 계산 시작');
    return num + 1;
}

function App() {
    const [hardNumber, setHardNumber] = useState(1);
    const [easyNumber, setEasyNumber] = useState(1);

    // state 인 number 들이 변경될 때마다 App 컴포넌트가 리렌더링이 되어 코드가 다시 실행된다.
    // 쉬운 계산만 건드려도 App 컴포넌트가 리렌더링 되기에 hardCalculator 도 실행되어 딜레이 된다.
    // const hardSum = hardCalculator(hardNumber);

    // useEffect 는 렌더링이 완료된 후에 실행되고, useMemo 는 렌더링 중에 실행된다.
    // 리렌더링 중 의존성 배열에 있는 상태값이 변경되지 않았다면 이전에 기억한 값을 사용
    // 값을 계산하여 반환하는 함수, 계산에 사용한 의존성 배열
    const hardSum = useMemo(() => {
        return hardCalculator(hardNumber);
    }, [hardNumber]);
   
    const easySum = easyCalculator(easyNumber);

    return (<>
        <h3>어려운 계산기</h3>
        <input type="number"
            value={hardNumber}
            onChange={(e) => setHardNumber(Number(e.target.value))} />
        <span> + 10000 = {hardSum}</span>
        <hr />
        <h3>쉬운 계산기</h3>
        <input type="number"
            value={easyNumber}
            onChange={(e) => setEasyNumber(Number(e.target.value))} />
        <span> + 1 = {easySum}</span>
    </>);
}
```

### useEffect 와 함께 사용
- useMemo : 렌더링에서 사용하는 값(참초)을 재사용을 하기 위하여 사용 (계산 생략을 위해 사용)
- useEffect : 렌더링 후 동기화를 위해 사용 (사이드 이펙트를 위해 사용)
```jsx
const { useState, useEffect, useMemo } = React;

function App() {
    const [isKorea, setIsKorea] = useState(true);
    const [number, setNumber] = useState(0);

    console.log('렌더링...');

    // 그냥 변수로 만들고 useEffect 로 의존성을 걸면 다른 값 변경으로 리렌더링할 때 마다 location 이 새로 생성되고 useEffect 가 실행된다.
    // const location = {
    //     country : isKorea ? '한국' : '외국'
    // };

    // 객체를 return 받는다. isKorea 가 변경되지 않으면 저장된 location 값을 활용(update X)
    const location = useMemo(() => ({
        country : isKorea ? '한국' : '외국'
    }), [isKorea]);

    useEffect(() => {
        console.log('useEffect 호출 됨... location 이 변경되었습니다.');
    }, [location]);

    return (<>
        <h2>지금 당신이 있는 위치는?</h2>
        <p>국가 : {location.country}</p>
        <button onClick={() => setIsKorea(!isKorea)}>국가 토글하기</button>
        <hr/>
        <input type="number"
            value={number}
            onChange={(e) => setNumber(Number(e.target.value))} />
    </>);
}
```

## useCallback
함수를 기억하여 함수 참조를 재사용  
`const 함수명 = useCallback(기억할 함수 내용, [의존성 배열]);`  
```jsx
const { useState, useEffect, useCallback } = React;

function App() {
    const [number, setNumber] = useState(0);
    const [toggle, setToggle] = useState(false);

    // number 값이 바뀌면 printNumber 함수를 새로 만들고 아니면 재사용
    const printNumber = useCallback(() => {
        console.log('current number', number);
    }, [number]);

    useEffect(() => {
        console.log('printNumber 함수가 변경되었습니다.');
    }, [printNumber]);

    return (<>
        <input type="number"
            value={number}
            onChange={(e) => setNumber(Number(e.target.value))} />
        <button onClick={printNumber}>숫자 출력</button>
        <hr/>
        <button onClick={() => setToggle(!toggle)}>토글 버튼</button>
        <p>토글 상태 : {String(toggle)}</p>
    </>);
}
```

### 스타일 객체를 만드는 함수
```jsx
const { useState, useEffect, useCallback } = React;

function Square({genSquareStyle}) {
    console.log('자식 컴포넌트(Square) 렌더링');

    const [style, setStyle] = useState({});

    useEffect(() => {
        console.log('자식의 useEffect 실행(스타일이 변경됨)');
        setStyle(genSquareStyle());
    }, [genSquareStyle]);

    return <div style={style}></div>
}

function App() {
    const [size, setSize] = useState(200);
    const [isDark, setIsDark] = useState(false);

    // 스타일 객체를 만드는 함수
    const genSquareStyle = useCallback(() => {
        return {
            backgroundColor : 'orange',
            width : `${size}px`,
            height : `${size}px`
        }
    }, [size]);

    return (<>
        <div style={{ backgroundColor : isDark ? 'black' : 'white' }}>
            <input type="range" min={100} max={300}
                value={size}
                onChange={(e) => setSize(Number(e.target.value))} />
            <button onClick={() => setIsDark(!isDark)}>테마 변경</button>
        </div>

        <Square genSquareStyle={genSquareStyle} />
    </>);
}

ReactDOM.createRoot(document.getElementById("root")).render(<App />);
```
자식 컴포넌트는 App 에 변화가 있을 때마다 계속 리렌더링 되는 중

## useRef
**값이 바뀌어도 리렌더링을 일으키지 않고, 리렌더링 되어도 값이 초기화 되지 않는다.**  
다른 이유로 리렌더링 됐을 때 최신값이 보인다.  
**일반 지역 변수를 사용하면 useState 가 바뀌면 리렌더링되어 선언문도 다시 실행되어 값이 초기화 된다.**  
타이머 id, 이전 값, DOM 요소 참조 등 화면에는 즉시 반영하지 않아도 되지만 컴포넌트가 기억해야 하는 값에 적합하다.  
`const 객체명 = useRef(초기값);`  
useRef 는 초기값을 .current 프로퍼티에 값을 저장한다. `객체명.current` 로 값에 접근  
```jsx
const { useState, useRef } = React;

function Counter() {
    
    console.log('렌더링 됨..');
    
    // useState 값이 바뀌면 React 가 컴포넌트를 다시 렌더링한다. > 화면에 바로 보여줘야 하는 값은 state 로 관리
    const [count, setCount] = useState(0);
    
    // useState 가 바뀌면 리렌더링되어 선언문도 다시 실행되어 값이 초기화 된다.
    let variable = 0;

    // useRef 는 초기값을 .current 프로퍼티에 값을 저장한다.
    // 값이 바뀌어도 리렌더링을 일으키지 않고, 리렌더링 되어도 값이 초기화 되지 않는다.
    // 즉 타이머 id, 이전 값, DOM 요소 참조 등 화면에는 즉시 반영하지 않아도 되지만 컴포넌트가 기억해야 하는 값에 적합하다.
    // 다른 이유로 리렌더링 되면 최신값이 화면에 보인다.
    const countRef = useRef(0);

    const increaseCount = () => {
        setCount(count + 1);
    };

    const increaseVariable = () => {
        variable += 1;
        console.log('지역 변수(variable) 값 :', variable);
    };

    const increaseCountRef = () => {
        countRef.current += 1;
        console.log('Ref(countRef) 값 :', countRef.current);
    };

    return <>
        <h1>State : {count}</h1>
        <h2>Variable : {variable}</h2>
        <h3>Ref : {countRef.current}</h3>
        <button onClick={increaseCount}>state 증가</button>
        <button onClick={increaseVariable}>variable 증가</button>
        <button onClick={increaseCountRef}>ref 증가</button>
    </>
}
```

&nbsp;

input 태그의 ref 속성에 useRef() 객체를 연결하면 React 가 렌더링 후 .current 에 해당 input 의 실제 DOM 요소를 넣어준다.  
input 포커스, 스크롤 이동, 영상 재생 등 **실제 DOM 요소 기능을 호출할 때** 사용  
```jsx
const { useState, useEffect, useRef } = React;

function LoginComponent() {
    const [form, setForm] = useState({
        username : '',
        password : ''
    });
		
    const usernameRef = useRef(); // 초기값을 넣지 않았기에 current 프로퍼티 값은 undefined
    
    // 렌더링 후 input 의 ref 속성에 넣어준 usernameRef.current 에 input DOM 이 왔으므로 처리
    useEffect(() => {
        console.log('Login 컴포넌트가 마운트 되었습니다.');
        console.log(usernameRef.current); // <input type="text" name="username" value="">
        
        // username input 에 마우스 포커스가 가 있도록
        usernameRef.current.focus();
    }, []);

    const onChangeHandler = (e) => {
        setForm({...form, [e.target.name] : e.target.value});
    };

    const onClickHandler = () => {
        alert(`username : ${form.username}\npassword : ${form.password}`);
        setForm({username : '', password : ''});
        usernameRef.current.focus();
    };

    return <>
        <input type="text"
            name = "username"
            value = {form.username}
            onChange = {onChangeHandler}
            ref = {usernameRef} /> {/* input 태그의 ref 속성에 usernameRef 를 연결하면 React 가 렌더링 후 usernameRef.current 에 해당 input DOM 요소를 넣어준다. */}
        <br />
        <input type="password"
            name = "password"
            value = {form.password}
            onChange = {onChangeHandler} />
        <br />
        <button onClick={onClickHandler}>로그인</button>
    </>
}

function App() {
    return (<>
        <LoginComponent />
    </>);
}

ReactDOM.createRoot(document.getElementById("root")).render(<App />);
```
```jsx
const usernameRef = useRef();

<input type="text"
    name = "username"
    ref = {usernameRef} />
    
// 렌더링이 끝나기 전에는 실제 DOM 요소가 아직 준비되지 않았을 수 있으므로, DOM 을 직접 다루는 코드는 useEffect 안에서 처리
useEffect(() => {
    // usernameRef input DOM 에 마우스 포커스가 가 있도록
    usernameRef.current.focus();
}, []);s
```

## Custom Hook
반복되는 훅 로직을 별도의 함수로 분리하여 재사용 가능하다.  
- 이름을 use 로 시작한다.
- 컴포넌트나 다른 커스텀 훅의 최상위에서 호출한다. (조건문, 반복문, 이벤트 핸들러 안에서 호출 X)
- 커스텀 훅은 로직을 공유한다. (로직은 재사용하지만 각 컴포넌트에서 호출할 때마다 state 는 독립적으로 만들어진다.)
```jsx
const { useState, useEffect } = React;

// 커스텀 훅
// 이름을 use 로 시작한다.
// 컴포넌트나 다른 커스텀 훅의 최상위에서 호출한다. (조건문, 반복문, 이벤트 핸들러 안에서 호출 X)
// 커스텀 훅은 로직을 공유한다. (로직은 재사용하지만 각 컴포넌트에서 호출할 때마다 state 는 독립적으로 만들어진다.)
function useDelayTimer(delay) {
    // 시간이 delay 만큼 지났는지 boolean 값 반환
    const [isReady, setIsReady] = useState(false);

    useEffect(() => {
        console.log(`${delay/1000}초 타이머 (재)시작`);
        setIsReady(false);

        // 타이머는 컴포넌트가 제거되더라도 기본적으로 남아서 백그라운드에서 동작한다.
        const timer = setTimeout(() => { 
            setIsReady(true);
            console.log('타이머 종료! 준비 완료')
        }, delay);

        // 만약 컴포넌트가 제거(update) 되면 unmount 되기 전 타이머를 제거한다.
        return () => clearTimeout(timer);
    }, [delay]);

    return isReady;
}


function App() {
    const isReady = useDelayTimer(5000);

    return (<>
        <h1>커스텀 훅 실습</h1>
        {isReady ? <h2>준비 완료! 안녕하세요 ~</h2> : <h2>잠시만 기다려 주세요~</h2>}
    </>);
}

ReactDOM.createRoot(document.getElementById("root")).render(<App />);
```

# Async (비동기)
## 콜백 지옥
비동기 결과값을 사용하여 다음 작업을 하기 위하여 계속해서 콜백을 호출함.
```javascript
function sayHello() {
    console.log(`(After 3sec later...) 음식이 나왔습니다.`);
}

console.log('햄버거를 주문합니다.');
// setTimeout(sayHello, 3000); // 비동기 // 최소한 3000ms 가 지난 뒤에야 실행할 기회를 얻는다.
console.log('감자튀김을 주문합니다.');

console.log(`\n콜백 지옥 체험`);

// 완료 후 실행할 콜백 추가
function increase(number, callback) {
    setTimeout(() => {
        const result = number + 10;
        console.log(result);

        if(callback) {
            callback(result);
        }
    }, 1000);
}

// 여러 비동기 작업의 순서를 중첩 콜백만으로 관리할 때의 콜백 지옥 현상
console.log('작업 시작...');
increase(0, (result) => {
    console.log('첫번째 작업 결과 : ', result);
    // increase(result, (result) => { });
});
```

## Promise
Promise : 아직 끝나지 않은 비동기 작업의 미래 결과를 나타내는 객체 (비동기 작업의 결과를 담는 객체)  
대기(Pending), 성공(fufilled), 실패(rejected) 상태 중 하나를 가진다.
```javascript
// resolve : 성공 시 호출하여 성공 결과를 담음, reject : 실패 시 호출하여 실패 이유를 담음
// 한번 성공이나 실패 시 상태가 변경되지 않는다.
const promise = new Promise((resolve, reject) => {});

promise
	.then((resolve값) => {})
	.catch((reject값) => {})
	.finally(() => {});
```
```javascript
function increase(number) {
    const promise = new Promise((resolve, reject) => {
        setTimeout(() => {
            const result = number + 10;
            if(result > 10) {
                const e = new Error('숫자가 너무 큽니다.');
                reject(e); // 실패 처리 > .catch 매개변수로 전달된다.
            } else {
                resolve(result); // 계산된 결과 성공처리 > .then 매개변수로 전달된다.
            }
        }, 1000);
    });

    return promise;
}

console.log('작업 시작');
increase(0).then((number) => { // resolve 값을 받아서 실행
    console.log(number);
    return increase(number);
}).catch((error) => { // .then 중간에 reject 하면 여기서 잡힌다.
    console.log(error);
}).finally(() => { // 성공하던 실패하던 무조건 마지막에 실행
    console.log("모든 작업이 끝났습니다.");
});
```