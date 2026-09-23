# React
## Async / Await

이 방식을 사용하면 async 함수 안에서는 비동기가 동기처럼 처리되기에 함수 통째로 비동기로 던질 때는 이 방식이 나을 것 같고 함수 내에서 비동기, 동기 식으로 동작해야 하는 작업이 각각 있다면 Promise 의 .then 방식이 나을 것 같다.

**async** : 함수 앞에 'async' 를 붙여야 내부에서 'await' 를 사용할 수 있다.

**await** : Promise 가 끝날 때까지 기다렸다가 **resolve 된 결과 값**을 받는 변수에 할당한다.

**※ async 가 붙어있는 함수는 항상 Promise 를 반환한다. useEffect 는 clean-up 을 위해 return 을 사용하기에 return 은 클린업 코드나 비어있어야 하기에 useEffect 함수 자체에 async 는 사용하면 안된다.**

```jsx
function 기능() {
	// promise 를 반환하는 함수
}

async function run() {
	let result = await 기능();
}
```

**에러 처리** : try-catch-finally 블록으로 감싼다.

```jsx
try{ } catch(error) { } finally { }
```

```jsx
// 함수 앞에 'async' 를 붙여야 내부에서 'await' 를 사용할 수 있다.
async function run() {
    console.log('작업 시작...');

    // 에러가 발생할 수 있는 코드는 try 블록으로 감싼다
    try{
        // await : increase(0) Promise 가 끝날 때까지 기다렸다가 resolve 된 값을 result 변수에 할당한다.
        let result = await increase(0); // Promise 를 반환하는 함수
        console.log(result);

        result = await increase(result);
        console.log(result);

        result = await increase(result);
        console.log(result);
    } catch (e) {
        // try 블록 안에서 promise 가 reject 되면 catch 블록이 실행된다.
        console.log("에러!", e);
    } finally {
        console.log("작업 종료");
    }

    console.log("이러면 이 함수 내부에서는 비동기가 동기식으로 처리되는거 아닌가?");
    console.log("함수 내부에서 동기/비동기가 나뉘어야하면 .then 방식, 함수 통째로 비동기로 던질거면 async/await 방식");
}

run();
```

## Fetch API

**Fetch API** : 페이지 일부만 구현하는데 필요한 도구. **JavaScript 에 내장**된 기능

**Ajax** : 페이지 새로고침 없이 필요한 데이터만 서버와 비동기적으로 주고받는 기술

Fetch API 는 비동기 작업이기에 Promise 를 반환한다.

`let promise = fetch(url, [options]);`

- url : 우리가 데이터를 요청하고 싶은 서버 주소
- [options] : 선택 매개변수. http 메서드, 요청 헤더, 요청 본문 등을 **객체**로 지정할 수 있다. 아무것도 안 넣으면 **기본 값인 GET 방식**으로 요청을 보낸다.

fetch 함수 참고 : https://developer.mozilla.org/ko/docs/Web/API/Window/fetch

fetch API 는 404 500 에러 시, reject 처리를 하지 않기에 ok 나 status 코드로 처리하는게 확실하다.

```jsx
const promise = fetch(url, {method : 'GET'});
```

json : java 와 js 끼리 데이터를 주고 받기 위해서 공통으로 사용하는 규약

```jsx
async function 함수() {
		const response = await fetch(url, {method : 'GET'}); // Response { status : 200, body : {}, ... }
		
		if(!response.ok) throw new Error(); // fetch 를 사용하려면 거의 필수 처리
		
		// .json : 본문 내용을 JSON 으로 파싱해서 자바스크립트 객체로 변환해준다.
		const responseJson = await response.json(); // .json() 작업도 비동기 작업이다.
}
```

1. Response > fetch(url, [options]);
2. 내용물 > 결과.json();

## await 와 then

then 사용하니 하나 이상의 비동기 있으니 번거로워서 그냥 비동기 로직만 따로 함수 분리해서 async await 쓰는게 낫겠다….

```jsx
const API_URL = 'https://jsonplaceholder.typicode.com/users';

// .then() 체이닝 사용
function fetchDataWithThen() {
    console.log('then 방식으로 fetch 시작');

    fetch(API_URL)
    // 1단계 : 서버의 응답(Response 객체)이 도착하면 실행
    .then(response => {
        // 성공이 아닐 시 에러 던지기
        // fetch API 는 404 500 에러 시, reject 처리를 하지 않기에 ok 나 status 코드로 처리하는게 확실하다
        if(!response.ok) {
            throw new Error('HTTP error!!');
        }

        // 실제 데이터본문(body) 추출을 위해 .json() 호출
        // .json() 역시 Promise 를 반환하므로, return 해줘야 다음 .then 에서 받을 수 있다.
        return response.json();
    })
    // 2단계 : .json() Promise 가 성공적으로 완료되면, 파싱된 데이터(data) 를 받아서 실행
    .then(data => {
        console.log('then 실제 데이터', data);
    })
    .catch(error => {
        console.error('then 데이터를 가져오는데 실패했습니다.', error);
    })
    ;
}

async function fetchDataWithAsyncAwait() {
    console.log('async/await 방식으로 fetch 시작');
    
    try {
        // 1단계 : fetch 가 완료되어 response 객체가 올때까지 기다린다.
        const response = await fetch(API_URL);

        if(!response.ok) {
            throw new Error('HTTP error!!');
        }

        // 2단계 : .json() 이 완료되어 실제 데이터가 올 때까지 기다린다.
        const data = await response.json();
        console.log('async/await 실제 데이터', data);
    } catch (error) {
        console.error('async/await 데이터를 가져오는데 실패했습니다.', error);
    }
}

fetchDataWithThen();
fetchDataWithAsyncAwait();
```

## Axios

CDN 으로 추가해서 확인

https://axios.rest/pages/getting-started/first-steps#using-unpkg

```jsx
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```

Axios : fetch 를 대체하는 기술

**외부 라이브러리**를 사용하고 **자동으로 처리**해주는게 많아 간편하다

마찬가지로 비동기이고 `axios.http메서드(url);` 형태

- 4xx, 5xx HTTP 에러를 자동으로 reject 처리
- json 파싱을 자동으로 해서 .data 에 담아준다.

```jsx
async function callApiWithAxios() {
    try {
        const response = await axios.get(API_URL); // {status:200, data:{}, headers, ...}
        console.log('axios 결과', response.data);
    } catch(error) {
        console.error('axios 에러', error);
    }
}
```

## data component

### Item list (then)

```jsx
const { useState, useEffect } = React;

const API_URL = 'https://jsonplaceholder.typicode.com/users';

function Title() {
    return <h1>회원 목록</h1>
}

// 회원 목록 전체
function ItemList() {
    const [users, setUsers] = useState([]);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);

    useEffect(() => {
        // setTimeout(() => { // 로딩 화면 테스트용 타임아웃

        setError(null);
        setLoading(true);

        fetch(API_URL)
            .then(response => {
                if(!response.ok) throw new Error('네트워크 응답에 문제가 있습니다.');
                return response.json();
            }).then(responseUsers => {
                setUsers(responseUsers);
            }).catch(error => {
                console.error('데이터를 불러오는데 실패!');
                setError(error);
            }).finally(() => {
                setLoading(false);
            });

        // }, 1000);
    }, []);

    if(loading) {
        return <p>로딩중...</p>
    }

    if(error) {
        return <p>오류가 발생했습니다 : {error.message}</p>
    }

    return <div>
        {users.map(user => <Item key={user.id} user={user} />)}
    </div>
}

// 회원 한 명의 카드 모양
function Item({ user }) {

    return <div>
        <h4>{ user.name }</h4>
        userId : { user.id }<br/>
        email : { user.email }<br/>
        company name : { user.company?.name }<br/>
    </div>
}

function App() {
    return (<>
        <Title />
        <ItemList />
    </>);
}

ReactDOM.createRoot(document.getElementById("root")).render(<App />);
```

### 날씨 API (async / await)

1. 사용자의 위치를 얻는다. (비동기)
2. 그 위치정보를 이용하여 날씨 api 호출 (비동기)

```jsx
const { useState, useEffect } = React;

const API_KEY = '776ac9802aa53ec7389f6e2aa4b2bc5b';

function Weather() {
    const [weatherData, setWeatherData] = useState(null);
    const [loading, setLoading] = useState(null);
    const [error, setError] = useState(null);

    useEffect(() => {
        const getPosition = () => {
            return new Promise((resolve, reject) => {
                // 1. 사용자의 위치정보 얻기
                navigator.geolocation.getCurrentPosition(resolve, reject); // 권한 허용되면 resolve, 거절하면 reject 로 전해진다.
            });
        };

        const fetchWeather = async () => {
            try {
                setLoading(true);
                setError(null);

                // 1-1. 사용자의 위치정보 얻기 실행
                const position = await getPosition();
                console.log(position); // {coords : {latitude : 위도, longitude : 경도}}

                const { latitude : lat, longitude : lon } = position.coords;

                // 2. 날씨 api 호출
                const response = await fetch(`https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${API_KEY}&units=metric&lang=kr`);
                
                if(!response.ok) throw new Error('날씨 정보 가져오기 실패!');

                const result = await response.json();
                console.log(result);
                setWeatherData(result);
            } catch (e) {
                console.error(e);
                setError(e);
            } finally {
                setLoading(false);
            }
        };

        fetchWeather();
    }, []);

    if(loading) {
        return <div>날씨 정보를 불러오는 중...</div>
    }

    if(error) {
        return <div>오류 발생 : {error.message}</div>
    }

    if(!weatherData) {
        return <div>날씨 정보 없음</div>
    }

    return <>
        <h1>{weatherData.name} 날씨</h1>
        <h4>{`현재 온도 : ${weatherData.main.temp}`}</h4>
        <h4>{`체감 온도 : ${weatherData.main.feels_like}`}</h4>
        <h4>{`풍속 : ${weatherData.wind.speed}`}</h4>
    </>
}

function App() {
    return (<>
        <h1>오늘의 날씨</h1>
        <Weather />
    </>);
}
```

- [React](#react)
  - [Async / Await](#async--await)
  - [Fetch API](#fetch-api)
  - [await 와 then](#await-와-then)
  - [Axios](#axios)
  - [data component](#data-component)
    - [Item list (then)](#item-list-then)
    - [날씨 API (async / await)](#날씨-api-async--await)
