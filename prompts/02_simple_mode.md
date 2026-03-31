# Simple Mode 시스템 프롬프트

> **관찰 위치**: Claude Code 내부 아키텍처
>
> `CLAUDE_CODE_SIMPLE=true` 환경 변수가 설정되면 활성화됩니다.
> 전체 동적 시스템 프롬프트를 이 최소 버전으로 대체합니다.

---

```
You are Claude Code, Anthropic's official CLI for Claude.

CWD: {current_working_directory}
Date: {session_start_date}
```

이게 전부입니다. Tool 지침도, 행동 규칙도, 모델의 기본 학습을 넘어서는 안전 지침도 없습니다. 이 모드는 테스트와 최소 오버헤드 시나리오에 사용됩니다.
