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