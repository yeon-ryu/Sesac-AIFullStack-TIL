# 기초 파이썬
LLM(Large Language Model) : 요청 지시에 따라 답을 만들어 주는 모델

## 환경 설정

파이썬 패키지를 설치할 때 pip 를 사용함 `pip install`

파이썬 패키지는 기본적으로 프로젝트 별이 아니라 파이썬을 이용하는 환경에 설치됨 (한 PC 당 파이썬 패키지 환경 다 공유) (프로젝트 별 환경X) → 프로젝트 별 가상 환경을 설치한다.

Java 에서는 JDK 를 설치했듯이 파이썬을 위해 conda 를 설치

### conda

conda : 가상 환경을 만들고 파이썬 패키지를 설치

저장소 : 채널

1. Anaconda
    - 파이썬과 데이터 분석용 라이브러리가 함께 들어있다. → 설치 용량이 큼
    - default 채널 사용 - 부분 유료
2. Miniconda
    - 파이썬 + conda (버전 패키지 관리) → 기본 패키지만 설치되어 있다.
    - default 채널 사용 - 부분 유료
3. **Miniforge**
    - 기본 패키지 위주(파이썬 + conda)로 설치하나 위의 2개와 저장소가 다르다.
    - conda-forge 라는 오픈소스 저장소에서 패키지를 다운받는다. - 무료

Miniforge 사용

```bash
conda install // conda-forge 저장소에서 가져옴
pip install // PyPI 저장소에서 가져옴
```

### conda-forge 설치

https://conda-forge.org/ → Download Installer


### 가상 환경 생성 및 접속

Miniforge Prompt 사용 (Cmd 창도 사용 가능)

&nbsp;

버전 확인

```bash
conda -V # conda 26.9.4
python -V # Python 3.14.10
```

&nbsp;

채널 확인

```bash
conda config --show channels
```

&nbsp;

가상 환경 생성

```bash
# conda create -n 가상환경이름 파이썬버전
conda create -n pystudy_env python=3.12
```

[y] : y 를 누르거나 엔터를 누른다.

&nbsp;

가상 환경 활성화/비활성화

```bash
conda activate 가상환경이름 # 가상 환경 활성화
conda deactivate # 가상 환경 비활성화
```

가상 환경이 활성화 되면 (가상환경이름) 이 된다.

&nbsp;

가상 환경 초기화 (안되어 있으면 가끔 가상 환경으로 잘 세팅이 안된다)

```bash
# 가상 환경 활성화된 상태에서
conda init cmd.exe
```

### VSCode 사용

프로젝트 시, 폴더명은 기본적으로 숫자로 시작하면 안된다. 수업 중에만 숫자 사용함.

파이썬 파일은 `.py` 확장자를 사용한다.

1. VSCode 확장 프로그램 Python 설치
2. Ctrl + Shift + P : python select interpreter
3. 사용할 가상 환경 선택 → 오른쪽 아래에 실행하는 가상 환경 표시되는 것 확인

실행 할 때 Run Python File 을 선택해야 제대로 인터프리터를 사용해서 실행된다.

## 변수와 Collections

변수명 규칙 : 스네이크 케이스 사용

- str (문자열)
- int (정수) : _(밑줄) 로 긴 숫자의 가독성을 높일 수 있음
- print(문자열)
- f-string : 문자열 앞에 f 를 붙이고 문자열 안 중괄호 안에 변수명을 넣으면 해당 값이 삽입된다.

```python
team_name = '새싹' # 문자열 # str (문자열)
price = 123_456 # 정수 (_ : 긴 숫자를 읽기 편하게 표시한 것. 숫자에 아무런 영향을 주지 않는다.) # int (정수)

print(team_name) # 콘솔에 출력

# f-string : 문자열 앞에 f 를 붙이고 중괄호 안에 변수명을 넣으면 해당 값이 삽입 된다.
print(f"팀 이름: {team_name}")
```

- 리스트 : 순서가 있고 내용을 변경할 수 있는 자료형

```python
# 리스트 : 순서가 있고, 내용을 변경할 수 있는 자료형
scores = [88, 90, 67]
print(scores[0]) # 88

# 값 추가, 제거
scores.append(100) # 끝에 값 추가
scores.insert(2, 75) # 2번 인덱스에 75 삽입

print(scores) # [88, 90, 75, 67, 100]

last_score = scores.pop() # 인덱스를 주지 않으면 마지막 값을 꺼내서 반환하고 제거
print(f"꺼낸 점수 : {last_score}, 수정된 점수: {scores}") # 꺼낸 점수 : 100, 수정된 점수: [88, 90, 75, 67]
```

- 딕셔너리 : 키와 값을 한 쌍으로 저장하는 자료형

```python
# 딕셔너리 : 키와 값을 한 쌍으로 저장하는 자료형
user = {'name' : '홍길동', 'age' : 30}
print(f"사용자 정보: {user}, 이름 : {user['name']}") # 사용자 정보: {'name': '홍길동', 'age': 30}, 이름 : 홍길동

# 키가 없으면 새로 추가, 있으면 기존 값을 덮어씀
user['job'] = '의적'
user['age'] = 33
print(user) # {'name': '홍길동', 'age': 33, 'job': '의적'}

# 존재하지 않는 키를 대괄호로 조회하려 할 경우 KeyError 발생
# print(user['email'])

# get(키, 기본값) : 키가 없어도 오류 대신 기본 값 반환
print(user.get('email', '정보 없음')) # 정보 없음

# 리스트와 딕셔너리 조합
users = [
    {'name':'홍길동'},
    {'name':'신사임당', 'job':'화가'}
]

print(users[1]['name']) # 신사임당
print(users[0].get('job', '정보 없음')) # 정보 없음
```

## 문자열과 제어문

- 슬라이싱 : 문자열의 일부를 읽는다. [시작 인덱스 : 끝 인덱스]
    - 끝 인덱스는 포함되지 않는다.
    - 시작이나 끝을 생략하고 처음부터나 끝까지 가능하다.
- split() : 구분자를 안 넣어주면 띄어쓰기를 기준으로 분해한다.

```python
# 문자열 핵심 기능
# 슬라이싱 : 문자열의 일부를 읽는다. [시작 인덱스 : 끝 인덱스]
# 끝 인덱스는 포함되지 않는다.

txt = 'helloworld'
print(f"슬라이싱[0:5]: {txt[0 : 5]}") # txt[:5] 로 시작을 생략하면 처음부터
print(txt[5:]) # 끝을 생략하면 끝까지
```

Python 은 중괄호 대신 들여쓰기로 코드 블록을 구분한다.

- if 문 : 조건문 끝에 콜론을 쓰고, 다음 줄에 들여쓰기 한 부분을 실행 범위로 인식. `else if` 대신 `elif` 를 사용
- for 문 : for 문 끝에 콜론을 쓰고, 다음 줄에 들여쓰기 한 부분을 실행 범위로 인식.

```python
# 제어문 : if, for
score = 85

# Python 은 중괄호 대신 들여쓰기로 코드 블록을 구분한다.
# if문이나 for문 끝에 콜론을 쓰고, 다음 줄에서 들여쓰기 한 부분을 실행 범위로 인식

if score >= 90:
    # 미리 변수 선언을 안해도 밖에서 사용 가능
    grade = 'A'
elif score >= 80:
    grade = 'B'
else:
    grade = 'F'
print(f'점수 : {score}, 등급 : {grade}') # 점수 : 85, 등급 : B

# for in 반복문 : 리스트의 값을 하나씩 꺼내 반복
foods = ['햄버거', '피자', '보쌈']

for food in foods:
    if food != '피자':
        print(f'오늘의 메뉴 : {food}')
    else:
        print('오늘의 메뉴는 피자!')
```

## Comprehension

리스트 컴프리헨션 : 반복과 조건을 이용하여 새 리스트를 만든다.

- len(문자열) : 문자열 길이 반환
- 문자열.upper() : 문자열을 대문자로 변환해서 반환

```python
fruits = ['orange', 'apple', 'banana', 'kiwi']

# 5글자 이상을 대문자로 변경
long_fruits = []

for fruit in fruits:
    if len(fruit) >= 5:
        long_fruits.append(fruit.upper())

print(long_fruits) # ['ORANGE', 'APPLE', 'BANANA']
print(fruits) # ['orange', 'apple', 'banana', 'kiwi']

# comprehension 방식으로 표기
compact_fruits = [fruit.upper() for fruit in fruits if len(fruit) >= 5]
print(compact_fruits) # ['ORANGE', 'APPLE', 'BANANA']
```

## 함수

함수명 규칙 : 스네이크 케이스 사용

`def 함수명(매개변수: 타입힌트 = 기본값) -> 반환값타입힌트 :`

기본 값이 있는 매개변수를 없는 매개변수보다 뒤에 둔다.

```python
# 함수와 타입 힌트

# 함수 정의
# 매개변수명: 타입힌트 = 기본값
# 타입 힌트에 안 맞는 타입을 넣어도 컴파일 에러가 발생하지는 않는다.
def create_greeting(name: str, age: int = 20) -> str:
    return f"안녕하세요, {age}세 {name}님."

print(create_greeting('홍길동', 30))
print(create_greeting('유관순'))

# 키워드 인자는 순서가 바뀌어도 된다.
print(create_greeting(age=45, name='신사임당'))

greeting = create_greeting('영희')
print(greeting)
```

## JSON

기능을 사용할 때는 import

- 직렬화 : `json.dumps(객체)` : json 문자열화
    - `json.dump(파일)` : 파일에 직접 저장
    - ensure_ascii : 디폴트는 true, 한글을 읽기 쉽게 False 설정하기도 함
- 역직렬화 : `json.loads(JSON 문자열)`
    - `json.load(파일)` : 파일에서 직접 읽어옴

※ JSON 에서 문자열은 큰 따옴표(”) 로 감싸인다.

```python
import json

feedback = {'id' : 1, 'text' : '실습이 도움이 됐어요.'}

# 요청에 붙일 JSON 문자열을 만든다.
# json.dumps() : JSON 문자열 반환 (Python -> JSON)
# json.dump() : 파일에 직접 저장 (Python -> JSON)
request_json = json.dumps(feedback, ensure_ascii=False) # json 으로 변화 (직렬화)
print(request_json) # {"id": 1, "text": "실습이 도움이 됐어요."}

# 응답 예시에서 필요한 값 얻기
reply = '{"id":1, "score":75}' # 내부 문자열을 작은 따옴표로 만들어 놓으면 loads 로 읽어올 때 오류가 발생함
print(reply)

# 역직렬화
# json.loads() : JSON 문자열 읽기 (JSON -> Python)
# json.load() : 파일에서 읽기 (JSON -> Python)
result = json.loads(reply)
print(result['score']) # 75
```

## 예외 처리

try-except

```python
import json

# 예외 처리
broken_reply = '{"id":1, "score":75' # 중괄호가 덜 닫힌 JSON 문

try:
    broken_result = json.loads(broken_reply)
except json.JSONDecodeError: # catch 구문
    print("응답을 JSON으로 읽을 수 없습니다.")
```

raise : 에러를 던짐

```python
# payload 타입이 딕셔너리가 아닐 경우 or payload 에 "items" 가 포함되어 있지 않다면
if not isinstance(payload, dict) or "items" not in payload:
    raise ValueError("응답은 items 키가 있는 JSON 객체여야 합니다.")
```

## LLM 을 위한 파이썬

""" : 줄바꿈을 포함한 문자열 작성

파일을 직접 실행할 경우 `__main__` 이라는 이름으로 들어간다. 다른 곳에서 모듈로 불러올 경우 모듈명이 실행된다.

### 프롬프트 생성

```python
import json

FEEDBACKS = [
    {'id':1, 'text':'진도가 조금 빠르지만 실습으로 이해하고 있습니다.'},
    {'id':2, 'text':'API키, 설정에서 막혀서 수업을 거의 못 따라갔습니다.'},
    {'id':3, 'text':'예제를 더 연습하고 싶습니다.'},
]

def build_prompt(feedbacks):
    feedbacks_json = json.dumps(feedbacks, ensure_ascii=False) # ensure_ascii : 한글 읽기 좋게 부가 설정

    # feedbacks 을 분석하여 객체를 만들라는 프롬프트
    # """ : 줄바꿈을 포함한 문자열 작성
    return f"""
            수강평을 분석해 JSON 객체만 출력하세요. 최상위 키는 "items" 입니다.
            각 항목은 id, sentiment, score, key_reason을 포함하세요.
            id 는 원본과 동일하게, setiment 는 positive/negative/mixed 중 하나,
            score 는 0~100 정수, key_reason 은 한 문장으로 작성하세요.
            수강평: {feedbacks_json}
            """

def main():
    prompt = build_prompt(FEEDBACKS)
    print("[모델에 보낼 프롬프트]")
    print(prompt)

# 파일을 직접 실행할 경우 __main__ 으로 자동으로 들어간다.
if __name__ == '__main__':
    main()
```

출력

```
[모델에 보낼 프롬프트]

            수강평을 분석해 JSON 객체만 출력하세요. 최상위 키는 "items" 입니다.
            각 항목은 id, sentiment, score, key_reason을 포함하세요.
            id 는 원본과 동일하게, setiment 는 positive/negative/mixed 중 하나,
            score 는 0~100 정수, key_reason 은 한 문장으로 작성하세요.
            수강평: [{"id": 1, "text": "진도가 조금 빠르지만 실습으로 이해하고 있습니다."}, {"id": 2, "text": "API 키 설정에서 막혀서 수업을 거의 못 따라갔습니다."}, {"id": 3, "text": "예제를 더 연습하고 싶습니다."}]
```

### 프롬프트 결과 반환 받기

```python
import json

SAMPLE_PEPLY = '{"items":[{"id":1,"sentiment":"mixed","score":75,"key_reason":"진도는 빠르지만 실습에 도움이 됩니다."}' \
', {"id":2,"sentiment":"negative","score":35,"key_reason":"환경 설정이 학습을 막았습니다."}' \
', {"id":3,"sentiment":"mixed","score":65,"key_reason":"추가 연습이 필요합니다."}]}'

BROKEN_JSON = '{"items":[]'
MISSING_FIELD_REPLY = '{"items":[{"id":1,"sentiment":"mixed","key_reason":"진도는 빠르지만 실습에 도움이 됩니다."}]}'
RESPONSE_EXAMPLES = {
    "정상 응답" : SAMPLE_PEPLY,
    "깨진 JSON" : BROKEN_JSON,
    "필수 필드 누락" : MISSING_FIELD_REPLY
}

def parse_response(reply):
    payload = json.loads(reply)
    # payload 타입이 딕셔너리가 아닐 경우 or payload 에 "items" 가 포함되어 있지 않다면
    if not isinstance(payload, dict) or "items" not in payload:
        # 에러를 던짐
        raise ValueError("응답은 items 키가 있는 JSON 객체여야 합니다.")

    items = payload["items"]
    # items 타입이 list 가 아닐 경우
    if not isinstance(items, list):
        raise ValueError("items 는 리스트여야 합니다.")

    required_keys = ["id", "sentiment", "score", "key_reason"]
    for item in items:
        if not isinstance(item, dict):
            raise ValueError("각 항목은 딕셔너리여야 합니다.")
        for key in required_keys:
            if key not in item:
                raise ValueError(f"필수 필드가 없습니다: {key}")

    return items

def show_result(reply):
    try:
        items = parse_response(reply)
    except json.JSONDecodeError:
        print("응답을 JSON 으로 읽을 수 없습니다.")
        return
    except ValueError as error:
        print("응답 형식 오류 /", error)
        return

    for item in items:
        print(f"{item['id']}: {item['sentiment']} / {item['score']}점")
        print(f"근거: {item['key_reason']}")

def main():
    # for key값, value값 in 딕셔너리.items() : 반복할 때 키와 값을 한 쌍 씩 꺼낼 수 있다.
    for name, reply in RESPONSE_EXAMPLES.items():
        print(f'\n[{name}]')
        show_result(reply)
```

출력

```
[정상 응답]
1: mixed / 75점
근거: 진도는 빠르지만 실습에 도움이 됩니다.
2: negative / 35점
근거: 환경 설정이 학습을 막았습니다.
3: mixed / 65점
근거: 추가 연습이 필요합니다.

[깨진 JSON]
응답을 JSON 으로 읽을 수 없습니다.

[필수 필드 누락]
응답 형식 오류 / 필수 필드가 없습니다: score
```

## NumPy

Numpy : 숫자 배열을 만들고 배열 전체를 계산하는 기능을 제공하는 Python 라이브러리

벡터 : 같은 기준과 순서에 맞춰 나열한 숫자 목록

숫자 리스트로 된 벡터로 유사도 점수를 계산할 수 있다. → 벡터의 내적을 구해서 유사도를 구한다.

내적 : 같은 위치의 숫자끼리 곱한 뒤 모두 더한 값

```
액션, 스릴러, 코미디, 감동

모아나 : [0, 0, 0, 1]
토이스토리 : [0, 0, 1, 1]
```

NumPy 설치

```bash
# 가상 환경에 설치
# 1. Command Prompt -> 경로 앞에 (가상환경명) 표시되어 있는 것 확인
pip install numpy
```

- numpy.array(파이썬 배열) : Numpy 배열을 반환해준다.

```python
# Numpy : 숫자 배열을 만들고 배열 전체를 계산하는 기능을 제공하는 Python 라이브러리
import numpy as np

left = [1, 2]
right = [3, 4]

print("리스트의 + :", left + right) # 리스트의 + : [1, 2, 3, 4]

# np.array(파이썬 리스트) : numpy 의 리스트로 반환
# Numpy 의 배열에는 쉼표(,)가 없다.
# Numpy 배열의 +는 같은 위치의 숫자끼리 계산한다.
print("Numpy 배열의 + :", np.array(left) + np.array(right)) # Numpy 배열의 + : [4 6]
```

### 코사인 유사도

코사인 유사도(정규화)(단위벡터) : 내적 / (리스트1 벡터의 길이 * 리스트2 벡터의 길이)

- numpy.dot(리스트1, 리스트2) : 내적
- numpy.linalg.norm(리스트1) : 벡터의 길이

```python
import numpy as np

# 벡터 : 같은 기준과 순서에 맞춰 나열한 숫자 목록
# 액션, 스릴러, 코미디, 감동
moana = np.array([0, 0, 0, 1])
toy_story = np.array([0, 0, 1, 1])
action_movie = np.array([1, 0, 0, 0])

print("\n모아나 :", moana)
print("토이스토리 :", toy_story)
print("액션 영화 :", action_movie)

# 코사인 유사도 : 두 벡터의 방향이 얼마나 비슷한지 점수로 나타내는 방법 중 하나
def cosine_similarity(a, b):
    # 내적 : 같은 위치의 숫자끼리 곱한 뒤 모두 더한 값
    dot = np.dot(a, b)

    # 벡터의 길이 : 원점에서 해당 좌표까지의 길이
    length_a = np.linalg.norm(a)
    length_b = np.linalg.norm(b)

    # 내적을 두 길이의 곱으로 나눠 방향을 비교한다.
    return dot / (length_a * length_b)

print(f"\n 모아나와 자기 자신의 유사도 : {cosine_similarity(moana, moana)}") # 1.0
print(f"\n 모아나와 토이스토리의 유사도 : {cosine_similarity(moana, toy_story)}") # 0.7071...
print(f"\n 모아나와 액션 영화의 유사도 : {cosine_similarity(moana, action_movie)}") # 0.0
```

- [기초 파이썬](#기초-파이썬)
  - [환경 설정](#환경-설정)
    - [conda](#conda)
    - [conda-forge 설치](#conda-forge-설치)
    - [가상 환경 생성 및 접속](#가상-환경-생성-및-접속)
    - [VSCode 사용](#vscode-사용)
  - [변수와 Collections](#변수와-collections)
  - [문자열과 제어문](#문자열과-제어문)
  - [Comprehension](#comprehension)
  - [함수](#함수)
  - [JSON](#json)
  - [예외 처리](#예외-처리)
  - [LLM 을 위한 파이썬](#llm-을-위한-파이썬)
    - [프롬프트 생성](#프롬프트-생성)
    - [프롬프트 결과 반환 받기](#프롬프트-결과-반환-받기)
  - [NumPy](#numpy)
    - [코사인 유사도](#코사인-유사도)
