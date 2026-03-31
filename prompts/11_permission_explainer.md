# 권한 설명기 시스템 프롬프트

> **관찰 위치**: Claude Code 내부 아키텍처
>
> 사용자가 승인하기 전에 Tool/명령이 무엇을 하는지, 왜 실행되는지, 위험 수준이 무엇인지 설명하는 경량 보조 질의(side-query)입니다. 구조화된 Tool 출력을 사용해 JSON 형식 응답을 보장합니다.

---

## 시스템 프롬프트

```
Analyze shell commands and explain what they do, why you're running them, and potential risks.
```

## 구조화 출력 스키마 (Tool: `explain_command`)

```json
{
  "explanation": "What this command does (1-2 sentences)",
  "reasoning": "Why YOU are running this command. Start with 'I' - e.g. 'I need to check the file contents'",
  "risk": "What could go wrong, under 15 words",
  "riskLevel": "LOW | MEDIUM | HIGH"
}
```

### 위험 수준 정의

| 수준 | 설명 | 예시 |
|-------|-------------|---------|
| `LOW` | 안전한 개발 워크플로 | `ls`, `cat`, `git status` |
| `MEDIUM` | 복구 가능한 변경 | 파일 편집, `npm install` |
| `HIGH` | 위험하거나 되돌리기 어려움 | `rm -rf`, `DROP TABLE`, force push |

## 런타임 동작

- **메인 루프 모델(main loop model)**을 사용합니다 (별도 Haiku 모델 아님).
- 권한 프롬프트(permission prompt)와 동시에 **보조 질의(side query)**로 실행됩니다.
- `"why"` 컨텍스트를 위해 최근 assistant 메시지 3개(최대 1000자)를 추출합니다.
- 사용자 설정 `permissionExplainerEnabled: false`로 비활성화할 수 있습니다.
