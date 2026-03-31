# 자리 비움 요약 (세션 요약)

**관찰 위치**: Claude Code 내부 아키텍처
**함수:** `buildAwaySummaryPrompt()`
**모델:** 소형 고속 모델 (Haiku)

## 목적

사용자가 자리를 비웠다가 세션으로 돌아올 때 1~3문장 요약을 생성합니다. UI의 "while you were away" 카드에 표시됩니다.

## 시스템 프롬프트

시스템 프롬프트는 선택적 세션 메모리 접두부(prefix)와 함께 동적으로 조립됩니다.

```
[Session memory (broader context):
{memory content}]

The user stepped away and is coming back. Write exactly 1-3 short sentences. Start by stating the high-level task — what they are building or debugging, not implementation details. Next: the concrete next step. Skip status reports and commit recaps.
```

## 설계 제약

- **간결성(Brevity)**: 정확히 1~3개의 짧은 문장
- **작업 우선(Task-first)**: 사용자가 무엇을 만들거나 디버깅하는지부터 시작
- **실행 가능성(Actionable)**: 구체적인 다음 단계를 포함
- **불필요한 정보 배제(No noise)**: 상태 보고와 커밋 요약은 생략
- **구현 세부사항 회피(Implementation detail avoidance)**: 코드 세부사항이 아니라 상위 수준 작업에 집중

## 입력 처리

- 큰 세션에서 `"prompt too long"` 오류를 피하기 위해 마지막 30개 메시지(`RECENT_MESSAGE_WINDOW = 30`)만 사용합니다.
- 더 넓은 컨텍스트를 위해 세션 메모리 내용을 포함합니다.
- 응답 속도를 위해 추론 표시(thinking)를 비활성화합니다.
- 캐시 오염을 피하기 위해 `skipCacheWrite: true`를 사용합니다.

## 오류 처리

- 중단, 빈 대화 기록(transcript), 어떤 오류든 발생 시 `null`을 반환합니다.
- `APIUserAbortError` 예외를 조용히 흡수합니다.
