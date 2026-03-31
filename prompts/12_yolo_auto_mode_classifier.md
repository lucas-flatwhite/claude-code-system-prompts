# YOLO/Auto-Mode 분류기 시스템

> **관찰 위치**: Claude Code 내부 아키텍처
>
> Tool 호출을 자동 승인할지 차단할지 결정하는 2단계 보안 분류 시스템입니다. Claude Code의 "auto-approve" 모드 핵심입니다.

---

## 아키텍처 개요

YOLO 분류기(classifier)는 Claude Code에서 **가장 보안에 민감한 구성 요소**입니다. 사용자가 "auto mode"(이전 명칭은 "YOLO mode")를 활성화하면, 이 분류기가 모든 Tool 호출을 실행 전에 평가합니다.

### 동작 방식

1. **기본 프롬프트(Base Prompt)**: `yolo-classifier-prompts/auto_mode_system_prompt.txt`에서 로드됩니다 (`bun:bundle` feature flag `TRANSCRIPT_CLASSIFIER`로 빌드 시 번들됨).
2. **권한 템플릿(Permissions Template)**: 사용자 유형에 따라 런타임에 교체됩니다.
   - **외부 사용자**: `permissions_external.txt`
   - **Anthropic 직원** (`USER_TYPE=ant`): `permissions_anthropic.txt`
3. **사용자 규칙(User Rules)**: 사용자 지정 가능한 세 개의 섹션이 `settings.autoMode`에서 주입됩니다.
   - `allow` — 사용자가 명시적으로 허용한 액션
   - `soft_deny` — 차단되어야 하지만 사용자가 재정의할 수 있는 액션
   - `environment` — 사용자 환경에 대한 컨텍스트

### 분류기 Tool 스키마

분류기는 구조화 출력을 위해 강제 Tool 호출(`classify_result`)을 사용합니다.

```json
{
  "name": "classify_result",
  "description": "Report the security classification result for the agent action",
  "input_schema": {
    "type": "object",
    "properties": {
      "thinking": {
        "type": "string",
        "description": "Brief step-by-step reasoning."
      },
      "shouldBlock": {
        "type": "boolean",
        "description": "Whether the action should be blocked (true) or allowed (false)"
      },
      "reason": {
        "type": "string",
        "description": "Brief explanation of the classification decision"
      }
    },
    "required": ["thinking", "shouldBlock", "reason"]
  }
}
```

### 시스템 프롬프트 조립

```
buildYoloSystemPrompt(context):
  1. Load BASE_PROMPT from auto_mode_system_prompt.txt
  2. Replace <permissions_template> with external OR anthropic template
  3. Replace <user_allow_rules_to_replace> with user's allow rules
  4. Replace <user_deny_rules_to_replace> with user's deny rules  
  5. Replace <user_environment_to_replace> with user's environment info
```

### CLAUDE.md 통합

분류기는 사용자의 CLAUDE.md 내용도 앞에 덧붙은 사용자 메시지 형태로 받습니다.

```
The following is the user's CLAUDE.md configuration. These are
instructions the user provided to the agent and should be treated
as part of the user's intent when evaluating actions.

<user_claude_md>
{content}
</user_claude_md>
```

### 안전 허용 목록 Tool

다음 Tool들은 사전 승인되어 classifier를 완전히 건너뜁니다.

- 읽기 전용 Tool (파일 읽기, glob, grep)
- 분류기 Tool 자체 (`classify_result`)
- 여러 안전한 유틸리티

### 대화 기록(Transcript)

분류기는 대화의 **축약 기록(transcript)**을 받습니다.
- 사용자 메시지: 텍스트만
- assistant 메시지: **tool_use 블록만** (모델이 분류기에 영향을 주는 텍스트를 작성하지 못하도록 텍스트는 제외됨)

### PowerShell 거부 지침

Windows에서는 추가 차단 규칙이 주입됩니다.
- 시스템 상태를 변경하는 PowerShell 명령
- 레지스트리 수정
- 서비스 관리 명령

## 핵심 설계 결정

1. **텍스트 제외(Text exclusion)**: 모델이 작성한 텍스트를 통한 프롬프트 주입을 막기 위해 assistant 텍스트 블록을 의도적으로 transcript에서 제외합니다.
2. **캐시 친화성(Cache-friendly)**: CLAUDE.md 접두부는 분류기 호출 간 프롬프트 캐싱을 위해 `cache_control`을 사용합니다.
3. **기능 게이트(Feature-gated)**: 전체 시스템은 `TRANSCRIPT_CLASSIFIER` 빌드 플래그 뒤에 있습니다.
4. **이중 템플릿(Dual templates)**: Anthropic 직원은 외부 사용자와 다른 기본 permission 값을 받습니다.
5. **사용자 맞춤화(User customization)**: 세 구역의 `allow`/`deny`/`environment` 시스템으로 사용자가 분류기 동작을 조정할 수 있습니다.
