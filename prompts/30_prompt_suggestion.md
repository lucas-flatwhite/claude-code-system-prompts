# 프롬프트 제안 서비스

**관찰 위치**: Claude Code 내부 아키텍처
**변수:** `SUGGESTION_PROMPT`
**모델:** 소형 고속 모델 (Haiku)

## 목적

Claude가 한 턴을 마친 뒤 사용자가 다음에 할 가능성이 높은 명령이나 질문을 예측합니다. 제안은 UI에서 클릭 가능한 옵션으로 나타나며, 타이핑 없이 대화 흐름을 이어갈 수 있게 합니다.

## 시스템 프롬프트 (소스 분석을 바탕으로 재구성)

```
You are predicting what the user will say next to an AI coding assistant.
Given the conversation so far, suggest 1-3 short follow-up prompts the user
might naturally say next.

Rules:
- Each suggestion should be 2-8 words
- Match the user's communication style (formal/informal, language)
- Suggestions should be natural next steps, not generic
- Prioritize actionable requests over questions
- Do NOT suggest things the assistant just completed
- If the task seems done, suggest verification or next logical steps

Return JSON array of strings.

Examples:
["Run the tests", "Show me the diff", "Deploy to staging"]
["Fix the type error", "Add error handling"]
["Looks good, commit it"]
```

## 필터링 로직

이 서비스는 제안에 여러 필터를 적용합니다.
- 최근 실행된 명령과 중복을 제거합니다.
- 서로 너무 비슷한 제안을 걸러냅니다.
- 방금 완료한 작업을 가리키는 제안을 제거합니다.
- 제안 수는 최대 3개로 제한합니다.

## 통합 지점

- 각 assistant 턴이 끝날 때마다 실행됩니다.
- 응답을 막지 않도록 비동기로 실행됩니다.
- 결과는 터미널 UI에서 클릭 가능한 칩(chip) 형태로 표시됩니다.
- 제안 수용률 측정을 위한 분석 로그(analytics)가 남습니다.
