# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트: 오늘의 만화 TODO 앱

미루는 습관을 고치기 위해 하루 일과를 웹툰 만화 칸으로 시각화하는 TODO 앱. 할 일을 완료하면 만화 칸이 채워지고, 미완료 항목은 빈칸으로 남아 시각적 압박감을 준다. 하루가 끝나면 Claude AI가 달성률을 분석해 피드백을 제공한다.

- **대상 사용자**: 미루는 습관을 고치고 싶고 시각적 동기 부여가 필요한 사람
- **핵심 가치**: 복잡한 백엔드 없이 프론트엔드 시각 효과에 집중
- **배포 환경**: GitHub Pages (단일 HTML 파일)

## 기술 스택

- **언어**: HTML / CSS / JavaScript (단일 파일, 프레임워크 없음)
- **AI**: OpenRouter API — 무료(`:free`) 모델 사용
- **저장소**: `localStorage` (날짜별 자동 초기화)
- **배포**: GitHub Pages
- **외부 의존성 없음** — CDN, npm 패키지, 백엔드 서버 모두 사용하지 않음

## 핵심 기능

1. 할 일 추가 / 체크 / 삭제 — 최대 6개 권장 (만화 칸 2×3 그리드)
2. 만화 칸 렌더링 — 완료 시 이모지 + AI 캡션으로 칸 채우기, 미완료 시 해치 패턴 빈칸
3. AI 캡션 생성 — 완료된 항목마다 OpenRouter 무료 모델로 짧은 만화 대사 생성
4. 하루 피드백 — 달성률 기반 AI 코멘트, 미완료 항목 명시적으로 지적
5. 자정 자동 초기화 — localStorage에 저장된 날짜 비교, 새 날이면 데이터 리셋
6. 달성률 진행 바 — 완료 개수 / 전체 개수 시각화
7. 지난 기록 보기 — 자정 초기화 직전에 그날의 날짜/요일/달성률/피드백을 `localStorage`(`webtoon_todo_history`, 최근 30일)에 보관, 날짜별로 한눈에 비교 가능

## 5W1H 기획

| 기획 요소 | 세부 정의 | 구현 포인트 |
|---|---|---|
| Who (누가) | 미루는 습관을 고치고 싶고 시각적 동기 부여가 필요한 사람 | 사용자 친화적인 웹툰 스타일 UI/UX |
| What (무엇을) | 우선순위 달성도에 따라 칸이 완성되는 투두 코믹 대시보드 | 체크 여부에 따른 이미지 렌더링(완성/빈칸) 핵심 로직 |
| Where (어디서) | PC 및 모바일 웹 브라우저 | 반응형 HTML/CSS, GitHub Pages 배포 |
| When (언제) | 아침(계획 입력)부터 하루 끝(달성 확인)까지 | 자정 기준 데이터 리셋, 달성률 기반 피드백 |
| Why (왜) | 지루한 일정 관리에 페널티를 줘서 성취감과 재미를 높이기 위해 | 프론트엔드 시각 연출(애니메이션)에 개발 우선순위 집중 |
| How (어떻게) | 일정 입력 후 완료할 때마다 해당 빈칸에 만화 그림이 채워지는 방식 | AI에게 텍스트 입력폼, 데이터 정렬, 조건부 이미지 출력 로직을 단계별로 지시 |

## AI 역할 및 프롬프트 가이드

AI가 하는 일:

1. 완료된 할 일 → 만화 칸 캡션 생성 (20자 이내, 생동감 있는 한국어)
2. 하루 마무리 → 달성률 기반 전체 피드백 (2~3문장, 미완료 항목 명확히 지적)

### Claude Code에게 지시할 프롬프트 (그대로 붙여넣기)

```
index.html 단일 파일로 웹툰 스타일 TODO 앱 만들어줘.

- 할 일 체크하면 만화 칸이 채워지고 미완료는 빈칸(해치 패턴)으로 남음
- OpenRouter API (무료 모델) 연동해서 완료 항목에 만화 캡션 생성
- 하루 마무리 버튼 누르면 달성률 기반 AI 피드백 생성 (미완료 항목 따끔하게 지적)
- localStorage로 날짜 바뀌면 자동 초기화
- GitHub Pages 배포용 단일 파일로 만들어줘
- OpenRouter 무료 모델 API 키를 코드 상수로 내장 (사용자 입력 UI 없음)
```

## API 설정

- **제공자**: [OpenRouter](https://openrouter.ai)
- **모델**: 무료 티어 모델(`:free` 접미사가 붙은 슬러그) 중 하나 고정 사용 — 현재 `index.html`에는 `deepseek/deepseek-v4-flash-0731:free` 사용 중. ⚠️ 무료 모델 목록은 몇 주 단위로 교체/제거되므로, 존재하지 않는 슬러그를 썼다가는 400 에러가 난다 (실제로 겪었음). 400/404가 나면 [openrouter.ai/models](https://openrouter.ai/models)에서 `:free` 모델을 다시 확인해 교체할 것
- **엔드포인트**: `https://openrouter.ai/api/v1/chat/completions` (OpenAI 호환 Chat Completions 스펙)
- **인증**: `Authorization: Bearer <OPENROUTER_API_KEY>` 헤더. 권장 헤더 `HTTP-Referer`, `X-Title`도 함께 전송
- **키 관리**: 사용자 키 입력 UI 없음 — API 키는 `index.html` 상단에 상수(`const OPENROUTER_API_KEY = "..."`)로 하드코딩
- `max_tokens`: 캡션 100 / 피드백 300

⚠️ **보안 주의**: GitHub Pages는 정적 공개 배포이므로 코드에 내장한 키는 페이지 소스로 누구나 볼 수 있다. 반드시 무료 모델 전용/저권한 키를 사용하고, 남용이 감지되면 즉시 OpenRouter 대시보드에서 키를 폐기·재발급할 것.

## 파일 구조

```
webtoon-todo/
├── index.html       ← 전체 앱 (HTML + CSS + JS 한 파일)
└── CLAUDE.md        ← 이 기획 문서 (Claude Code 자동 로드)
```

## 코딩 규칙

- **단일 파일 원칙** — `index.html` 하나에 모든 코드 작성, 외부 파일 분리 금지
- **프레임워크 금지** — React, Vue 등 사용 안 함. 순수 JS만
- **상태 관리** — `tasks` 배열 하나로 모든 상태 관리, 변경 시 항상 localStorage 저장
- **날짜 초기화** — 앱 시작 시 localStorage의 날짜와 오늘을 비교, 다르면 전체 리셋
- **API 키 관리** — OpenRouter 무료 모델 키를 `index.html` 상단 상수로 하드코딩, 사용자 입력 UI는 두지 않음 (자세한 내용은 [API 설정](#api-설정) 참고)
- **한국어 UI** — 모든 텍스트, 버튼, 피드백은 한국어로

## 배포

1. GitHub 레포 생성 (Public, 이름: `webtoon-todo`)
2. `index.html` + `CLAUDE.md` 파일 push
3. Settings → Pages → Branch: `main` / `(root)` → Save
4. 배포 주소: `https://[유저명].github.io/webtoon-todo`

API 키는 사용자 입력 없이 코드에 내장된 OpenRouter 무료 모델 키를 그대로 사용한다. 공개 저장소에 올라가는 만큼 무료/저권한 키만 사용할 것 (자세한 내용은 [API 설정](#api-설정) 참고).
