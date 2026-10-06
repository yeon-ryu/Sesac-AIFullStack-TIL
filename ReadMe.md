# TIL (Today I Learned)

## Node.js 설치
1. Node.js 설치 (기본 세팅으로)
2. node -v , npm -v 로 잘 설치됐는지 버전 확인

## VSCode 환경 설정
1. 설정 오픈
2. Editor > Inline Suggest: Enabled 체크 해제 (자동 코딩 방지)
3. Extensions > Emmet > Trigger Expension On Tab 체크 (Tab 키로 태그 자동 완성)
4. 워크스페이스 왼쪽 아래에서 Restricted Mode 를 누르고 trust 눌러주면 모든 확장 프로그램이 Enable 상태가 됨

## VSCode 확장 프로그램
### 마크다운 자동 목차 생성 확장 프로그램
1. Markdown All in One 확장 프로그램 설치
2. Ctrl + Shift + P 로 커맨드 팔레트 열기
3. Markdown All in One: Create Table of Contents 선택

### Code Runner
코드 실행 버튼 하나로 쉽게 하게 해줌  
설정 > Code-runner: Run In Terminal 체크 해주면 터미널에서 결과 확인 가능 (디폴트 아웃풋 창)  

### Live Server
Enable(Workspace) 를 누르고 오른쪽 아래 Go Live 를 누르면 저장한 변경사항이 바로 반영되는 웹 페이지 오픈

## React 개발 환경 설정
### Next.js 프로젝트 생성
```bash
npx create-next-app@latest 프로젝트명
```

### React 실행
```bash
npm install
npm run dev
```

## Java 세팅
### JDK 21 설치
테무린 jdk 21 : https://adoptium.net/temurin/releases/?version=21&os=any&arch=any

1. zip 파일 다운 및 압축 풀기
2. 환경 변수 설정
    1. 윈도우 검색 창 > 시스템 환경 변수 편집 > 환경 변수
    2. 시스템 변수 > 새로 만들기 > JAVA_HOME [jdk bin 폴더가 있는 경로]
    3. 시스템 변수 > Path 편집 > 새로 만들기 > `%JAVA_HOME%\bin` 추가 > JAVA_HOME 이 제일 위에 위치하도록 이동 (순서대로 찾기에 jdk 를 빨리 쉽게 찾게 하기 위해)
3. jdk 세팅 확인
    
    ```bash
    # cmd 창
    java -version
    ```
    

### IntelliJ 설치
https://www.jetbrains.com/ko-kr/idea/download/?section=windows

1. exe 파일 다운 및 설치 진행
    1. Open Folder as Project 체크해두면 편함
    2. .java 파일과 연결
    3. Add “bin” folder to PATH 는 이미 하긴 했지만 체크
2. ~~인코딩 설정~~
    1. IntelliJ 가 설치된 위치의 bin 폴더 > .vmoptions 파일 중 자신의 OS bit 와 맞는 파일(idea64.exe.vmoptions)을 열어 `-Dfile.encoding=UTF-8` 을 추가한 후 저장
