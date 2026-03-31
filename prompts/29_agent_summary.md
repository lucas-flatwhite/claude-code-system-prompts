# 에이전트 요약 (백그라운드 진행 상황)

**관찰 위치**: Claude Code 내부 아키텍처
**모델:** 소형 고속 모델 (Haiku)

## 목적

조정자(coordinator) 모드에서 실행 중인 서브 에이전트의 백그라운드 진행 상황을 주기적으로 생성합니다. 각 워커가 무엇을 하고 있는지 부모 에이전트가 실시간으로 파악할 수 있게 합니다.

## 프롬프트 (소스 분석을 바탕으로 재구성)

```
Summarize what the agent is currently doing in 1 short sentence. Use present tense.
Focus on the specific action, not the overall task.

Good: "Reading the authentication middleware to understand token validation"
Good: "Running pytest on the user service after fixing the import error"
Bad: "Working on the task" (too vague)
Bad: "The agent is currently in the process of examining..." (too verbose)
```

## 설계 제약

- **한 문장 제한(Single sentence)**: 최대 한 문장, 현재 시제
- **행동 중심(Action-specific)**: 전체 목표가 아니라 현재 동작을 설명
- **메타 설명 금지(No meta-commentary)**: 에이전트 자체를 설명하지 말고, 하고 있는 일을 설명
- **현재 시제(Present tense)**: 항상 현재 시제 동사 사용 ("Reading", "Running", "Fixing")

## 통합 지점

- 조정자(coordinator) 모드에서 주기 타이머로 실행됩니다.
- 결과는 조정자의 상태 보기(status view)에 표시됩니다.
- 리드 에이전트가 워커 확인 시점을 결정하는 데 도움을 줍니다.
- 서브 에이전트가 Tool 호출을 실제로 실행 중일 때만 동작합니다.
