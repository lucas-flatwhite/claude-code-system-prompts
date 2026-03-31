# 세션 제목 생성기

**관찰 위치**: Claude Code 내부 아키텍처
**변수:** `SESSION_TITLE_PROMPT`
**모델:** Haiku (via `queryHaiku`)
**출력 형식:** JSON (`{ "title": "..." }`)

## 목적

대화 내용으로부터 간결한 sentence-case 세션 제목(3~7단어)을 생성합니다. SDK, CCR 원격 세션, REPL 브리지 전반에서 AI 생성 세션 제목의 단일 기준 역할을 합니다.

## 시스템 프롬프트

```
Generate a concise, sentence-case title (3-7 words) that captures the main topic or goal of this coding session. The title should be clear enough that the user recognizes the session in a list. Use sentence case: capitalize only the first word and proper nouns.

Return JSON with a single "title" field.

Good examples:
{"title": "Fix login button on mobile"}
{"title": "Add OAuth authentication"}
{"title": "Debug failing CI tests"}
{"title": "Refactor API client error handling"}

Bad (too vague): {"title": "Code changes"}
Bad (too long): {"title": "Investigate and fix the issue where the login button does not respond on mobile devices"}
Bad (wrong case): {"title": "Fix Login Button On Mobile"}
```

## 입력 처리

`extractConversationText()` 함수는 메시지 배열을 하나의 텍스트 문자열로 평탄화합니다.
- 메타 메시지와 사람이 아닌 메시지는 건너뜁니다.
- 최근 컨텍스트가 우선되도록 마지막 1000자만 잘라 사용합니다.
- `msg.origin.kind === 'human'` 기준으로 필터링합니다.

## 통합 지점

- **SDK 출력 경로(print path)**: 비대화형 세션 제목 생성
- **CCR 원격 세션(remote sessions)**: `useRemoteSession`을 통한 대화형 세션 제목 생성
- **REPL 브리지(REPL bridge)**: 사용자 메시지 3개 이후 fire-and-forget 방식으로 제목을 갱신
