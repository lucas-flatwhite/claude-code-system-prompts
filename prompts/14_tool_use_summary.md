# Tool 사용 요약 생성기

> **관찰 위치**: Claude Code 내부 아키텍처
>
> Haiku를 사용해 Tool 호출이 무엇을 수행했는지 요약하는 짧은 git commit 스타일 라벨을 생성합니다. 모바일 앱에서는 한 줄짜리 항목(row)으로 표시됩니다.

---

## 시스템 프롬프트

```
Write a short summary label describing what these tool calls accomplished. It appears as a single-line row in a mobile app and truncates around 30 characters, so think git-commit-subject, not sentence.

Keep the verb in past tense and the most distinctive noun. Drop articles, connectors, and long location context first.

Examples:
- Searched in auth/
- Fixed NPE in UserService
- Created signup endpoint
- Read config.json
- Ran failing tests
```

## 사용자 프롬프트 템플릿

```
User's intent (from assistant's last message): {lastAssistantText (first 200 chars)}

Tools completed:

Tool: {name}
Input: {truncated input, max 300 chars}
Output: {truncated output, max 300 chars}

Label:
```

## 기술 세부사항

| 속성 | 값 |
|----------|-------|
| **Model** | Haiku |
| **Query Source** | `tool_use_summary_generation` |
| **Prompt Caching** | 활성화 |
| **Criticality** | 비중요 (실패는 기록되지만 무시됨) |
| **Input Truncation** | Tool 입력/출력당 300자 |
