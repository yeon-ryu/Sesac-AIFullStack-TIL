# Lang Chain
프롬프트를 프로그램 코드 안에서 재사용할 수 있는 템플릿으로 만들고, 이를 실제 인공지능 모델과 연결 → LangChain 의 `PromptTemplate` 과 `LCEL 체인(Chain)` 을 통해 조립 가능한 레고 블록의 형태로 제작

**LangChain** : 모델X, LLM 을 활용하는 프로그램에서 프롬프트 템플릿, 모델 호출, 응답처리, Parser, 검색기 등을 연결하는 **라이브러리**. 필요한 단계를 연결해 하나의 실행 흐름(체인)으로 구성할 수 있다.

## 환경 설정

```bash
# langchain-core 와 langchain-openai 설치
pip install langchain-core langchain-openai
```

```bash
# 검증을 위한 pydantic 설치
pip install pydantic
```

```bash
# langchain rag 수업을 위해 자료를 로딩하기 위해 설치 (deprecated 된 라이브러리)
pip install langchain-community

# pdf 로딩 라이브러리
pip install pypdf
```

## PromptTemplate

- `PromptTemplate.from_template(”템플릿 만들 문자열”)` : 문자열 템플릿 생성
    - 문자열에서 변수는 `{}` 를 사용하되, f-string 은 아니다.
- `ChatPromptTemplate.from_messages([(”역할”, “메세지)])` : 역할이 붙은 메세지 템플릿 생성
    - 파이썬에서 소괄호 안에 문자열들을 넣었을 때,  `,` 가 존재하면 튜플로, 존재하지 않으면 하나의 문자열로 반환한다.
- `template.partial(변수명=”변수값”)` : 변수를 invoke 로 실행하기 전에 미리 채워 놓을 수 있다.
- `template.invoke({"변수명" : "변수값"})` : 값을 채워 템플릿 실행

```python
from langchain_core.prompts import ChatPromptTemplate, PromptTemplate

def show_prompt_template():
    # 랭체인이 변수값을 채워줄 것이기에 f-string 이 아니다.
    template = PromptTemplate.from_template("{audience}에게 {topic}에 대해 쉽게 설명하세요.")

    # template 의 값을 채워 실행
    prompt_value = template.invoke({"audience": "Java 를 배운 Python 입문자", "topic": "LLM"})

    print("문자열 템플릿:", prompt_value.to_string())

def show_chat_prompt_template():
    template = ChatPromptTemplate.from_messages([
        ("system", "초보자에게 쉬운 말로 설명하세요"),
        ("user", "질문: {question}\n답변 형식:{answer_format}") # to_messages() 로 보이면 type 은 "human" 이 된다.
    ])

    prompt_value = template.invoke({
        "question": "LangChain 에서 프롬프트 템플릿을 만들려면 어떻게 하나요?",
        "answer_format": "한 줄 정의와 짧은 예시"
    })

    print("메세지 템플릿:")
    for message in prompt_value.to_messages():
        print(f"[{message.type}] {message.content}")

if __name__ == "__main__":
    show_prompt_template()
    show_chat_prompt_template()
```

## 체인 연결

Chain 을 연결할 때는 `|` 로 연결한다.

- `prompt | model | StrOutputParser()` : `prompt` 결과를 `model` 에 입력하고, `model` 의 결과를 `StrOutputParser()` 에 넣는다.
    - 실행은 `invoke()` 됐을 때 실행된다.
    - prompt 실행 값이 model 에 들어가 요청하고, 모델의 응답 결과 값이 parser 에 들어감
- LCEL (엘셀) : 파이프로 연결한 체인
- Runnable : 파이프로 연결한 실행 단계
- `StrOutputParser()` : 모델의 응답 메세지(AI Message)에서 텍스트를 꺼내서 **문자열로 반환**

```python
from dotenv import load_dotenv
from openai import APIError
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import ChatOpenAI

def build_chain(model):
    prompt = ChatPromptTemplate.from_messages([
        ("system", "LLM 활용을 가르치는 초보자도 이해하기 쉬운 강의를 하는 강사입니다. 세 문장 이내로 설명하세요."),
        ("user", "{audience}에게 {topic}을 쉽게 설명하세요.")
    ])

    # LCEL (엘셀) 체인을 넘긴다.
    # | (파이프 연산자) : 왼쪽 작업 결과를 오른쪽 작업의 입력으로 넘기도록 연결해준다.
    return prompt | model | StrOutputParser()

def main():
    load_dotenv()

    # 모델 API 응답 요청 방식 1 : chat completions
    # 모델 API 응답 요청 방식 2 : Responses : OpenAI 내장 웹 검색, 파일 검색 등이 필요할 경우
    model = ChatOpenAI(
        model="gpt-4.1-mini",
        temperature=0.7,
        max_completion_tokens=300,
        timeout=20, # 20초
        max_retries=0,
        use_responses_api=False # API 응답 요청 방식을 chat completions 방식으로 사용 (수업에서 원래 사용하던 방식)
    )

    # 체인을 완성해서 반환 받음
    chain = build_chain(model)

    # 체인을 실행하여 모델에 직접 요청
    try:
        result = chain.invoke({"audience": "Python 입문자", "topic": "LangChain"})
    except APIError as error:
        print("[요청 실패]", type(error))
        return
    print("[결과]", result)

if __name__ == "__main__":
    main()
```

## 데이터 검증

`class 클래스명(부모클래스)` : 부모 클래스를 상속 받은 클래스

- 클래스의 필드에 접근할 때 `.` 으로 접근 가능하다.

Pydantic : 클래스에 적은 타입과 규칙으로 입력 데이터를 검증하는 Python 라이브러리

- `BaseModel` 을 상속해 필요한 항목을 선언하고, 검증을 통과한 값을 객체로 다룸
    - `BaseModel` 은 타입 힌트를 실제 검증 규칙으로 활용한다.
    - `ConfigDict(extra="forbid")` : 클래스의 검사 설정. 선언하지 않은 추가 필드는 허용하지 않음
    - `Literal["사과", "바나나"]` : Literal 에 들어간 값만 허용
    - `Field()` : 해당 필드의 조건
    - `@model_validator` : 아래 메서드를 pydantic 검증에 등록한다. 속성의 `(mode=”after”)` 를 붙이면 필드 검증 통과 후 체크한다. → pydantic 검증 때 자동 체크
- `JsonOutputParser` : **JSON 을 Python 값으로 읽는다.** 체인에 사용할 수 있다. 괄호가 안 닫혀있는 등의 문제가 발생해도 오류를 발생 시키지 않고 잘 읽어와준다.
- `PydanticOutputParser` : JSON 을 Python 값으로 읽어오며 검증을 통한다. 마찬가지로 괄호가 안 닫혀있는 등의 문제가 발생해도 오류를 발생 시키지 않고 잘 읽어와준다. `.` 으로 필드를 읽어올 수 있다.
    - `pydantic_object` : 검증 기준이 될 클래스. 해당 타입 객체로 반환해준다.
    - 검증을 통과하지 못하면 `ValidationError` / `OutputParserException` 를 발생시킨다.
    - `parser.get_format_instructions()` : 클래스의 항목과 규칙을 모델에게 전달할 JSON 응답 형식 안내로 만든다.

```python
from typing import Literal
from pydantic import BaseModel, ConfigDict, Field
from langchain_core.output_parsers import JsonOutputParser, PydanticOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_core.exceptions import OutputParserException
from langchain_openai import ChatOpenAI
from openai import APIError
from dotenv import load_dotenv

# Pydantic : 클래스에 적은 타입과 규칙으로 입력 데이터를 검증하는 Python 라이브러리
# BaseModel 을 상속해 필요한 항목을 선언하고, 검증을 통과한 값을 객체로 다룬다.
class SentimentAnalysis(BaseModel):
    # 이 클래스의 검사 설정, 선언하지 않은 추가 필드는 허용하지 않음
    model_config = ConfigDict(extra="forbid")

    # Literal : 안에 들어간 값만 허용
    label: Literal["긍정, 부정", "중립"] = Field(description="허용된 감정 분류")
    # str : 타입 힌트 <- BaseModel 은 타입 힌트를 실제 검증 규칙으로 사용한다.
    # min_length=1 : "" 는 허용하지 않나 " " 는 허용된다.
    reason: str = Field(min_length=1, description="판단 근거")

# label 이 "보류" 가 되거나 할 경우 PydanticOutputParser 로 읽어왔을 때 ValidationError(OutputParserException) 발생
SAMPLE_REPLY = '{"label": "중립", "reason": "어려움과 긍정적인 평가가 함께 있습니다."}'
SENTENCE = "수업이 어렵지만 예제가 있어서 따라갈 만하다."

def build_chain(model):
    parser = PydanticOutputParser(pydantic_object=SentimentAnalysis)
    # 클래스의 항목과 규칙을 모델에게 전달할 JSON 응답 형식 안내로 만든다.
    format_instructions = parser.get_format_instructions()

    prompt = PromptTemplate.from_template(
        "아래 수강평의 감정을 분류하고 이유를 적으세요.\n"
        "{format_instructions}\n"
        "수강평: {sentence}"
    )

    # format_instructions 는 변수지만 변하지 않을 값이기에 미리 채워둔다.
    prompt = prompt.partial(format_instructions=format_instructions)

    return prompt | model | parser

def main():
    # # JsonOutputParser : JSON 을 Python 값으로 읽는다.
    # format_only = JsonOutputParser()
    # data = format_only.invoke(SAMPLE_REPLY)
    # print("JSON 읽기만 한 값:", data["label"])

    # # PydanticOutputParser : JSON 을 클래스 규칙으로 검증하고, 통과한 값을 그 클래스의 객체로 반환
    # parser = PydanticOutputParser(pydantic_object=SentimentAnalysis)
    # result = parser.invoke(SAMPLE_REPLY)
    # print("JSON 검증하며 읽은 값:")
    # print("결과 타입:", type(result).__name__) # SentimentAnalysis
    # print("감정:", result.label)
    # print("이유:", result.reason)

    load_dotenv() # 환경변수 로드
    model = ChatOpenAI( # 모델 연결 정보
        model="gpt-4.1-mini",
        temperature=0,
        max_completion_tokens=300,
        timeout=20,
        max_retries=0,
        use_responses_api=False
    )

    # 체인 만들고 요청 실행
    try:
        result = build_chain(model).invoke({"sentence": SENTENCE})
    except OutputParserException:
        print("응답 검증 실패 : 감정 분류와 이유 형식을 확인하세요.")
        return
    except APIError as error:
        print("API 요청 실패:", type(error).__name__)
        return

    # 응답 결과 사용
    print("결과 타입:", type(result).__name__)
    print("감정:", result.label)
    print("이유:", result.reason)

if __name__ == "__main__":
    main()
```

### 커스텀 검증

개별 정책 사항은 `BaseModel` 상속 클래스에 커스텀 메소드(`@model_validator`)를 만들어 검증한다.

```python
from pydantic import BaseModel, ConfigDict, Field, model_validator

class Quiz(BaseModel):
    model_config = ConfigDict(extra="forbid")

    question: str = Field(min_length=1, description="퀴즈 문제")
    choices: list[str] = Field(min_length=4, max_length=4, description="보기 네 개")
    answer: str = Field(min_length=1, description="보기 중 정답 문자열")
    explanation: str = Field(min_length=1, description="정답 설명")

    # 로직적 정책 사항은 개발자가 체크
    # @model_validator : 아래 메서드를 pydantic 검증에 등록한다. -> pydantic 자동 검증
    @model_validator(mode="after")
    def check_answer(self):
        if any(not choice.strip() for choice in self.choices):
            raise ValueError("보기는 비어 있을 수 없습니다.")
        # set() : 중복을 제외한 값들의 집합을 만듦
        if len(set(self.choices)) != len(self.choices):
            raise ValueError("보기는 서로 달라야 합니다.")
        if self.answer not in self.choices:
            raise ValueError("정답은 보기 중 하나여야 합니다.")
        return self
```

- `enumerate` : 반복할 때 번호를 붙여 번호와 값을 한 쌍씩 꺼낼 수 있게 해주는 함수

```python
for number, choice in enumerate(quiz.choices, start=1):
```

## RAG

RAG (Retrieval-Augmented-Generation) : 검색 증감 생성 (참고할 자료를 요청에 넣는 방식)

### 문서 로딩

- `Path(file).resolve()` : 현재 파일이 존재하는 절대 경로
- (Deprecated 됨) `~~TextLoader(”파일 경로”)` : 전달된 파일을 읽을 객체를 만든다.~~
    - `~~load()` : 파일을 읽고 문서가 하나라도 항상 list[Document] 를 반환~~
- TextLoader 대신 `Document()` 로 직접 객체를 생성한다.
    - page_content : path.read_text() 로 파일 본문을 직접 읽는다.
    - metadata : source 등 키 값을 채운다.
- `리스트.extend(리스트2)` : 리스트 안에 있는 원소들을 다른 리스트에 덧붙인다.
- Document 객체
    - `metadata["source"]` : 파일 출처 (직접 채우면 경로가 아닐 때도 있다)
    - `metadata["source"].name` : 파일명
    - `page_content` : 파일 본문

```python
from pathlib import Path
# langchain_community 는 deprecated 됨
# from langchain_community.document_loaders import TextLoader
from langchain_core.documents import Document

# 현재 파일의 절대 경로 : Path(__file__).resolve()
# 현재 파일이 존재하는 폴더(parent) 안의 data 폴더에 파일들 존재
DATA_DIR = Path(__file__).resolve().parent / "data"
FILENAMES = ["center_policy.txt", "course_guide.txt"]

def main():
    documents = []

    for filename in FILENAMES:
        path = DATA_DIR / filename
        # TextLoader : 전달된 파일을 읽을 객체를 만든다.
        # load() : 하나의 문서를 읽더라도 늘 list[Document] 를 반환한다.
        # loaded = TextLoader(str(path), encoding="utf-8").load()

        # extend() : 리스트를 펼쳐서 다른 리스트 안에 넣어준다.
        # documents.extend(loaded)

        # Document : 본문과 부가 정보를 함께 보관하는 객체 생성
        document = Document(
            page_content=path.read_text(encoding="utf-8"),
            metadata={"source": path.name} # source 에 바로 파일명
        )

        documents.append(document)

    for document in documents:
        # document.metadata["source"] 는 원래 파일 경로를 보여주지만 파일명만 보여주게 덮어쓰기
        # document.metadata["source"] = Path(document.metadata["source"]).name
        print("타입:", type(document).__name__)
        print("출처:", document.metadata["source"]) # 파일명
        print("본문 앞부분:", document.page_content[:100]) # 본문
        print()

if __name__ == "__main__":
    main()
```

### PDF 문서 로딩

텍스트를 추출하여 문자열로 사용

- `PdfReader(경로)` : PDF 를 읽고 구조를 반환
    - `pages` : 페이지들이 순서대로 들어있다.
    - `페이지.extract_text()` : 해당 페이지 안의 텍스트를 문자열로 추출.
    이미지로 된 텍스트는 이 방식으로 텍스트를 추출할 수 없다. 이미지로 된 텍스트를 추출하려면 OCR 이나 Docling 같은 도구가 필요하다.
    표 같은 것도 정렬이 어긋날 수 있기에 필요하면 전용 도구를 사용한다.

```python
from pathlib import Path
from langchain_core.documents import Document
from pypdf import PdfReader

DATA_FILE = Path(__file__).resolve().parent / "data" / "course_handbook.pdf"

def main():
    # PdfReader(경로) : PDF 를 읽고 구조를 반환
    reader = PdfReader(DATA_FILE)

    documents: list[Document] = []

    # pages : pdf 페이지들이 순서대로 들어있다.
    for page_number, page in enumerate(reader.pages, start=1):
        text = page.extract_text() or "" # 현재 페이지 안의 텍스트를 문자열로 추출
        
        if not text.strip(): # 내용 없는 페이지는 패스
            continue
        
        document = Document(
            page_content=text.strip(),
            metadata={"source": DATA_FILE.name,
                      "page": page_number}
        )
        documents.append(document)

    print("PDF 전체 페이지 수:", len(reader.pages))
    print("만들어진 Document 수:", len(documents))

    for document in documents:
        print(f"\n{document.metadata["source"]} / {document.metadata["page"]} 페이지")
        print(document.page_content[:300])


if __name__ == "__main__":
    main()
```

- [Lang Chain](#lang-chain)
  - [환경 설정](#환경-설정)
  - [PromptTemplate](#prompttemplate)
  - [체인 연결](#체인-연결)
  - [데이터 검증](#데이터-검증)
    - [커스텀 검증](#커스텀-검증)
  - [RAG](#rag)
    - [문서 로딩](#문서-로딩)
    - [PDF 문서 로딩](#pdf-문서-로딩)
