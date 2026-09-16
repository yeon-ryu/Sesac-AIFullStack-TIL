# React
#### **React** : JavaScript 를 위한 **UI 라이브러리**  
JavaScript 처럼 Dom 을 하나하나 조작하는 방식 : 명령형 프로그래밍  
UI : 사용자 인터페이스

&nbsp;

## React 특징
- **엘리먼트** : 요소 노드, 어트리뷰트 노드, 텍스트 노드로 만들어지는 가상 DOM 개념
- **컴포넌트** : **엘리먼트를 반환**하는 개념을 가진 **화면의 구성 단위**
    - 클래스형 컴포넌트
    - 함수형 컴포넌트
- **Virtual DOM** : Render Tree 를 참고하여 Virtual DOM 을 생성하여 변경 사항에 대해 **실제 DOM 을 바로 건드려 전체 DOM 을 업데이트 하는 대신 변경이 발생한 부분만** 업데이트한다.
- **JSX** : React 에서 UI 를 구성하는데 사용하는 자바스크립트의 확장 문법. 자바스크립트 안에서 HTML 과 유사한 문법을 사용하여 작성할 수 있다. Babel 같은 도구로 JavaScript 코드로 변환된다.

&nbsp;

### 동작 원리
#### 브라우저 렌더링 엔진 동작 과정
1. HTML Parsing : HTML 을 파싱하여 DOM Tree 를 생성
2. CSS Parsing : CSS 파일을 파싱해서 CSSOM Tree 를 생성
3. Render : 두개의 Tree 를 결합하여 Render Tree 생성
4. Layout :   (가장 오래 걸리고 비용이 비싼 작업) Render Tree를 화면에 배치하는 것과 관련하여 각 노드의 위치와 크기를 계산하여 화면 상에 실제 픽셀로 변환하고 레이어를 생산
5. Paint : (가장 오래 걸리고 비용이 비싼 작업) 레이어를 생성하여 실제 화면에 표시

&nbsp;

#### React 의 Re-Randering
1. Diffing : Virtual DOM 과 변경된 Virtual DOM 을 비교한다.
2. Bath Update : 트리에서 바뀐 부분만 확정으로 실제 변경한다.

&nbsp;

### React 시작
1. React CDN 방식으로 추가 [React CDN 링크](https://ko.legacy.reactjs.org/docs/cdn-links.html)
2. Babel CDN 방식으로 추가 [Babel CDN 링크](https://babeljs.io/docs/babel-standalone#installation)
#### JSX 는 JavaScript 의 확장문법이기에 JavaScript 로 변환하기 위해 Babel 이 필요하다.
```jsx
<head>
    <script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
</head>

<body>
    <div id="root"></div>
    
    <!-- 바벨을 사용중임을 명시 (JSX 문법을 사용하기 위해) -->
    <script type="text/babel">
        function App() {
            return (<></>);
        }

        // react가 관리할 시작 지점 설정
        const root = ReactDOM.createRoot(document.getElementById("root"));
        root.render(<App/>);
    </script>
</body>
```

&nbsp;

## 엘리먼트 생성과 렌더링
리액트는 root 라는 id 를 가진 div 안을 렌더링 하는 방식으로 사용한다.  
React 엘리먼트는 일종의 설계 정보 이다.
```jsx
<head>
    <script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
</head>

<body>
    <div id="root"></div>

    <script>
        console.log(React); // 여러 기능이 들어있는 자바스크립트 객체
        
        // react가 관리할 시작 지점 설정
        const root = ReactDOM.createRoot(document.getElementById('root')); // ReactDom : react-dom 에서 사용

        // class 는 이미 javascript 에서 예약되었기 때문에 react 에서는 className 을 사용한다.
        // 실제 DOM 노드가 아니라 React Element 이다. React.createElement('태그명', { 속성 객체 }, '자식 내용');
        const greeting = React.createElement('h1', { className : 'greeting' }, '안녕, 리액트!');
        console.log(greeting); // {type : "h1", props : {className: 'greeting', children: '안녕, 리액트!'}}

        root.render(greeting);
    </script>
</body>
```

&nbsp;

## JSX 문법
주석 : {/* */}
1. { } 안에 자바스크립트 표현식(하나의 값으로 평가되는 코드)을 넣을 수 있다.
2. HTML 속성과 약간 다른 규칙을 따른다. (카멜 케이스)
    - class → className
    - onclick → onClick
3. 인라인 스타일은 '객체'형태로 전달해야 한다.
    - `<p style={ {color : ‘white’} }></p>`
    - `<p style={customStyle}></p>`
4. 모든 태그는 반드시 닫혀야 한다.
    - `<hr> → <hr/>`
    - `<br> → <br/>`
    - `<input> → <input/>`
5. 같은 레벨의 부모 태그가 있으면 안된다. 무조건 하나의 태그로 한번 감싸야 함.
    - Fragment : 여러 요소를 하나로 묶어주기 위한 특별한 빈 태그 <></>

```jsx
<script type="text/babel">
    const user = {
        name : '홍길동',
        age : 20
    }

    function introduce(user) {
        return `안녕하세요, 저는 ${user.name}이고 ${user.age}세 입니다.`;
    }

    function handleClick() {
        alert('버튼이 클릭되었습니다!');
    }

    const customStyle = {
        backgroundColor : 'cornflowerblue',
        color : 'white',
        padding : '10px',
        borderRadius : '5px'
    };

    function JsxRules() {
        return (<>
            {/* JSX 내부 주석 */}
            <h2>1. { } 안에 자바스크립트 표현식(하나의 값으로 평가되는 코드)을 넣을 수 있다.</h2>
            <h3>이름 : {user.name}</h3>
            <p>소개 : {introduce(user)}</p>

            {/* class -> className, ondlick -> onClick */}
            <h2>2. HTML 속성과 약간 다른 규칙을 따른다.</h2>
            <button className="my-button" onClick={handleClick}>클릭</button>

            <h2>3. 인라인 스타일은 '객체'형태로 전달해야 한다.</h2>
            <p style={customStyle}>이 문단에는 커스텀 스타일이 적용되었습니다.</p>

            <h2>4. 모든 태그는 반드시 닫혀야 한다.</h2>
            한 줄 짜리 태그도 에외는 없다. <br/> <input/> <hr/>
        </>);
    }

    ReactDOM.createRoot(document.getElementById("root")).render(<JsxRules />);
</script>
```

&nbsp;

## Component
컴포넌트 : UI 를 재사용 가능한 '부품'으로 나눈 것(ex. 헤더, 버튼, 카드)  
React 엘리먼트를 반환하는 **자바스크립트 함수** 이고 반드시 **대문자로 시작** 해야 한다.

&nbsp;

### 클래스형 컴포넌트
```jsx
<script type="text/babel">
    /**
     * 클래스형 컴포넌트, 컴포넌트는 앞 글자를 반드시 대문자로 시작
     * 자바스크립트의 class 문법을 사용하고, React.Component 를 상속 받아 만든다.
     */
    class TitleClass extends React.Component {
        // 클래스형 컴포넌트 안에서는 반드시 render() 함수를 구현해야 함
        render() {
            return React.createElement('h1', {className : 'classComponent'}, '클래스형 컴포넌트 입니다.');
        }
    }

    ReactDOM.createRoot(document.getElementById('root')).render(<TitleClass />);
</script>
```

&nbsp;

### 함수형 컴포넌트
```jsx
 <script type="text/babel">
    /**
     * 함수형 컴포넌트
     */
    function OldWay() {
        return React.createElement('div', null,
            React.createElement('h1', {className: 'greeting'}, "createElement 방식"),
            React.createElement('p', null, "코드가 길고 복잡해짐")
        );
    }

    ReactDOM.createRoot(document.getElementById('root')).render(<OldWay />);
</script>
```

&nbsp;

### 함수형 컴포넌트 - JSX 방식
Babel 이 이러한 JSX 문법을 위와 같은 JS 방식 코드로 번역해준다.
```jsx
 <script type="text/babel">
    // JSX 방식
    function NewWay() {
		    // 스크립트 안에 HTML 처럼 쓰고 있는 이건 JSX 문법
        return (
            <div>
                <h1 className='greeting'>JSX 방식</h1>
                <p>HTML처럼 보여서 훨씬 직관적이다!!</p>
            </div>
        );
    }

    ReactDOM.createRoot(document.getElementById('root')).render(<NewWay />);
</script>
```

&nbsp;

### 컴포넌트 조합(합성)
```jsx
<script type="text/babel">
    function App() {
        return (
            // JSX 에서 여러 태그를 반환할 때는 반드시 하나의 부모 요소로 감싸야 한다.
            // 여러 요소를 하나로 묶어주기 위한 특별한 빈 태그(Fragment)
            <>
                <OldWay />
                <hr />
                <NewWay />
            </>
        );
    }

    ReactDOM.createRoot(document.getElementById('root')).render(<App />);
</script>
```

&nbsp;

## Re-Rendering
코드상으론 전체를 렌더링하더라도 Virtual DOM 이 차이점만 비교해서 실제 DOM 이 그 부분만 렌더링하기에 React 가 빠르다.
```jsx
<script type="text/babel">
    function Clock() {
        return (<>
            <h1>실시간 시계</h1>
            <h2>현재 시간 : {new Date().toLocaleTimeString()}</h2>
        </>);
    }

    const root = ReactDOM.createRoot(document.getElementById("root"));

    // Clock 컴포넌트를 그려줘
    // 코드상으론 전체를 렌더링하지만 Virtual DOM 이 차이점만 비교해서 실제 DOM 이 그 부분만 렌더링하기에 빠르다.
    function tick() {
        root.render(<Clock />);
    }

    // 1초마다 tick 실행
    setInterval(tick, 1000);
</script>
```

&nbsp;

## Props
부모 컴포넌트가 자식 컴포넌트에게 데이터를 전달하는 '통로’. properties 의 줄임말. **읽기 전용 데이터**  
함수의 매개변수처럼 부모가 값을 넘기면 자식이 받은 값을 사용한다.
- props 는 {name : '홍길동', age : 20} 같은 객체 값으로 전달된다.
- 변수나 숫자 같은 값을 넘길 때는 {}를 사용한다.
- 태그 사이에 넣은 값은 props.children 으로 전달된다.
- props 를 넘기지 않았을 경우, 매개변수에서 기본값을 설정했다면 기본값이 적용되고 아니라면 undefined 가 나온다.
```jsx
<script type="text/babel">
    /**
     * Props : 부모 컴포넌트가 자식 컴포넌트에게 데이터를 전달하는 '통로'
     * 읽기 전용 데이터 (자식은 Props 를 수정할 수 없음)
     * 데이터는 항상 위에서 아래로 흐릅니다. (부모한테서 자식한테로 흐른다.)
     * 함수의 매개변수처럼, 부모가 값을 넘기고 자식은 받은 값을 사용한다.
     */

    /*
    // props = {name : '유관순', age = '20'}
    function Greeting(props) {
        return (
            <h1>안녕하세요, {props.name}님! {props.age}세</h1>
        );
    }
    */

    function Greeting({name = '게스트', favoriteNumber, children}) {
        return (<>
            <h1>안녕하세요, {name}님!</h1>
            <h2>제가 좋아하는 숫자는 {favoriteNumber} 입니다.</h2>
            <strong>전달된 자식 요소(children) : {children}</strong>
        </>);
    }

    function App() {
        const name = ['홍길동', '유관순', '이순신'];
        let idx = 0;

        return (<>
            {
                /* 변수나 숫자 값은 중괄호 안에 넣어서 전달 */
                /* 태그 사이에 넣은 값은 props.children 으로 전달 된다. */
                name.map(n => <Greeting key={idx++} name={n} favoriteNumber={5}>props 공부 중!</Greeting>)
            }
            <Greeting>
                <div>여러 줄도 가능</div>
                <div>props 를 넘기지 않으면 기본값이 적용되거나 undefined 가 들어갑니다.</div>    
            </Greeting>
        </>);
    }

    const root = ReactDOM.createRoot(document.getElementById("root"));
    root.render(<App />);
</script>
```