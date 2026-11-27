# VectorStore

Chroma : 디스크에 문서/벡터를 보관할 수 있는 벡터 데이터베이스. 프로그램을 재시작하면 같은 저장소를 다시 열어 재사용한다.

- 생성할 때 속성
    - `collection_name` : 폴더 경로와 컬렉션 이름을 함께 구분하기에 폴더 경로와 일치해야 함
    - `embedding_function` : 문서/질문을 벡터로 바꿀 임베딩 객체
    - `persist_directory` : 저장소 경로 문자열
- `add_documents(list[Document], ids=list[str])` : 최초 저장. 문서 목록과 식별자를 저장소에 전달하여 연결한 임베딩 도구가 본문을 벡터로 바꾸고, 본문/metadata/벡터가 각각의 ID 와 함께 보관된다.
- `get()` : 저장된 문서를 가져온다.
    - `limit=N` : 최대 N건 조회
    - `include=[”key”]` : 해당 key 내용을 포함하여 가져온다.

```python
from pathlib import Path
from dotenv import load_dotenv
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
from langchain_chroma import Chroma

load_dotenv()

LESSION_DIR = Path(__file__).resolve()
DATABASE_DIR = LESSION_DIR / ".chroma-lab" / "center-policy-v1" # 디스크의 자료 위치

QUESTION = "강의를 취소하면 결제한 돈을 돌려받을 수 있나요?"
EMBEDDING_MODEL = "text-embedding-3-small"

# 고정 ID: 저장할 문서 항목을 구분하는 값. 같은 파일명의 여러 본문도 별도 ID로 식별.
DOCUMENTS = [
    Document(id="refund", page_content="수업 시작 24시간 전까지 취소하면 전액 환불됩니다.", metadata={"source": "center_policy.txt", "topic": "policy"}),
    Document(id="hours", page_content="토요일 실습실은 오전 10시부터 오후 5시까지 운영합니다.", metadata={"source": "center_policy.txt", "topic": "policy"}),
    Document(id="rag", page_content="RAG는 외부 문서를 검색해 모델 요청에 넣는 방식입니다.", metadata={"source": "course_guide.txt", "topic": "lesson"}),
]

embeddings = OpenAIEmbeddings(model=EMBEDDING_MODEL, request_timeout=20, max_retries=0)

store = Chroma(
    collection_name="center-policy-v1", # 폴더 경로와 일치해야 함
    embedding_function=embeddings, # 문서/질문을 벡터로 바꿀 임베딩 객체 연결
    persist_directory=str(DATABASE_DIR) # Path 객체를 문자열로 바꿔서 전달
)

# 자료 존재 여부를 최대 한 건 조회
stored = store.get(limit=1)
print(type(stored).__name__, stored["ids"]) # dict [] # (아무것도 없는 최초엔 빈 리스트)

if not stored["ids"]: # 디스크에 없을 경우 최초 저장
    ids = [str(document.id) for document in DOCUMENTS]
    # 문서 목록과 식별자를 저장소에 전달
    # -> 연결한 임베딩 도구가 본문을 벡터로 바꾸고, 본문/metadata/벡터가 각각의 ID 와 함께 보관된다.
    store.add_documents(DOCUMENTS, ids=ids)

# 조회(재사용). 재사용 문서의 최선성/완전성을 보장하지는 않는다.
saved = store.get(include=["documents", "metadatas"]) # documents, metatdatas 를 포함하여 가져온다.
print("문서 수:", len(saved["ids"]))
for item in zip(saved["ids"], saved["documents"], saved["metadatas"]): # 같은 위치의 ID/본문/metadata 를 튜플로 묶어 순회
    print(item) # ex. ('refund', '수업 시작 24시간 전까지 취소하면 전액 환불됩니다.', {'source': 'center_policy.txt', 'topic': 'policy'})

all_retriever = store.as_retriever(search_kwargs={"k": 2}) # Chroma 에서 검색기 생성
all_result = all_retriever.invoke(QUESTION)

# filter : metadata 를 읽어서 필터링
# metadata 의 topic 이 policy 인 문서에서만 검색
policy_retriever = store.as_retriever(search_kwargs={"k": 2, "filter": {"topic": "policy"}})
policy_result = policy_retriever.invoke(QUESTION)
```

- [VectorStore](#vectorstore)
