# Claude Code 시스템 프롬프트

**Claude Code**의 내부 프롬프트 아키텍처와 에이전트 동작 방식을 문서화한 독립 연구 프로젝트입니다. Claude Code는 Anthropic의 AI 기반 소프트웨어 엔지니어링 어시스턴트입니다.

이 문서는 동작 분석, 출력 관찰, 공개 토론 자료 조사를 바탕으로 작성되었습니다. 현대의 에이전트형 AI 코딩 어시스턴트가 어떻게 설계되는지 이해하기 위한 교육 자료로 활용할 수 있습니다.

## 개요

Claude Code는 정교한 다층 프롬프트 아키텍처를 사용합니다. 메인 시스템 프롬프트는 고정 문자열이 아니라, 모듈식 섹션 빌더(section-builder) 함수들을 통해 런타임에 동적으로 조립됩니다. 경계 마커가 전역 캐시 가능한 접두부(prefix)와 세션별 접미부(suffix)를 구분해 주며, 이를 통해 API 호출 전반에서 프롬프트 캐싱이 가능합니다.

핵심 정체성 프롬프트 외에도, 이 시스템에는 특수 에이전트 프롬프트, 멀티 워커 조정자(coordinator), Tool 호출 자동 승인을 위한 2단계 보안 분류기, 그리고 메모리 선택, 세션 검색, Tool 사용 요약을 위한 각종 유틸리티 프롬프트가 포함됩니다.

## 프롬프트 카탈로그

### 핵심 정체성

| # | 프롬프트 | 설명 |
|---|--------|-------------|
| 01 | [메인 시스템 프롬프트](prompts/01_main_system_prompt.md) | 정체성, 동작, Tool 사용 지침, 톤, 효율성 전반을 다루는 동적 조립형 마스터 프롬프트 |
| 02 | [Simple Mode](prompts/02_simple_mode.md) | `CLAUDE_CODE_SIMPLE`로 활성화되는 최소 4줄 프롬프트 |
| 03 | [기본 에이전트 프롬프트](prompts/03_default_agent_prompt.md) | 모든 서브 에이전트가 상속하는 기본 프롬프트 |
| 04 | [사이버 위험 지침](prompts/04_cyber_risk_instruction.md) | 허용된 행위와 금지된 행위 사이의 보안 경계 |

### 오케스트레이션

| # | 프롬프트 | 설명 |
|---|--------|-------------|
| 05 | [조정자(Coordinator) 시스템 프롬프트](prompts/05_coordinator_system_prompt.md) | 4단계 워크플로와 동시성 규칙을 갖춘 멀티 워커 오케스트레이터 |
| 06 | [팀메이트 프롬프트 부록](prompts/06_teammate_prompt_addendum.md) | swarm 및 team 모드용 통신 프로토콜 |

### 특수 에이전트

| # | 프롬프트 | 설명 |
|---|--------|-------------|
| 07 | [검증 에이전트](prompts/07_verification_agent.md) | 구현을 깨뜨리는 데 집중하는 적대적 테스트 전문가 |
| 08 | [Explore 에이전트](prompts/08_explore_agent.md) | 엄격한 수정 금지 제약 아래에서 수행하는 읽기 전용 코드베이스 탐색 |
| 09 | [에이전트 생성 아키텍트](prompts/09_agent_creation_architect.md) | 사용자 요구사항으로부터 새 에이전트 구성을 설계 |
| 10 | [상태 줄 설정 에이전트](prompts/10_statusline_setup_agent.md) | 다양한 셸 환경에서 터미널 상태 줄을 구성 |

### 보안 및 권한

| # | 프롬프트 | 설명 |
|---|--------|-------------|
| 11 | [권한 설명기](prompts/11_permission_explainer.md) | 사용자 승인 전에 Tool 위험 수준을 설명 |
| 12 | [Auto Mode 분류기](prompts/12_yolo_auto_mode_classifier.md) | Tool 호출 자동 승인을 위한 2단계 보안 분류기 |

### Tool 관련 설명

| # | 프롬프트 | 설명 |
|---|--------|-------------|
| 13 | [Tool별 프롬프트](prompts/13_tool_prompts.md) | Bash, Edit, Agent, fork 동작 의미를 포함한 30개 이상의 Tool 설명 |

### 유틸리티 및 헬퍼

| # | 프롬프트 | 설명 |
|---|--------|-------------|
| 14 | [Tool 사용 요약](prompts/14_tool_use_summary.md) | 완료된 Tool 배치에 대해 git commit 스타일 라벨을 생성 |
| 15 | [세션 검색](prompts/15_session_search.md) | 지난 대화 세션 전반에 대한 시맨틱 검색 |
| 16 | [메모리 선택](prompts/16_memory_selection.md) | 현재 질의 컨텍스트에 맞는 관련 메모리 파일을 선택 |
| 17 | [Auto Mode 비평](prompts/17_auto_mode_critique.md) | 사용자가 작성한 auto-mode 분류기 규칙을 검토 |
| 20 | [세션 제목](prompts/20_session_title.md) | Haiku 기반 3~7단어 세션 제목 생성기 |
| 29 | [에이전트 요약](prompts/29_agent_summary.md) | 조정자(coordinator) 모드에서 서브 에이전트의 진행 상황을 주기적으로 갱신 |
| 30 | [프롬프트 제안](prompts/30_prompt_suggestion.md) | 클릭 가능한 제안을 위해 사용자의 후속 명령을 예측 |

### 컨텍스트 윈도 관리

| # | 프롬프트 | 설명 |
|---|--------|-------------|
| 21 | [Compact 서비스](prompts/21_compact_service.md) | 분석/요약 블록을 사용하는 다중 변형 대화 요약 |
| 22 | [자리 비움 요약](prompts/22_away_summary.md) | 복귀한 사용자를 위한 1~3문장 세션 요약 |

### 동적 섹션

| # | 프롬프트 | 설명 |
|---|--------|-------------|
| 18 | [Proactive Mode](prompts/18_proactive_mode.md) | tick 기반 간격 조절과 터미널 포커스 인식을 갖춘 자율 에이전트 |
| 23 | [Chrome 브라우저 자동화](prompts/23_chrome_browser_automation.md) | 브라우저 확장 통합: GIF 녹화, 탭 관리, 대화상자 처리 |
| 24 | [메모리 지침](prompts/24_memory_instruction.md) | CLAUDE.md 로딩, @include 지시어, 프론트매터(frontmatter) glob 처리 |

### 번들 스킬

| # | 프롬프트 | 설명 |
|---|--------|-------------|
| 19 | [Simplify 스킬](prompts/19_simplify_skill.md) | 코드 재사용성, 품질, 효율성을 위한 3에이전트 병렬 리뷰 |
| 25 | [Skillify 스킬](prompts/25_skillify.md) | 인터뷰 기반 스킬 생성기로, `SKILL.md` 파일을 생성 |
| 26 | [Stuck 스킬](prompts/26_stuck_skill.md) | 멈추거나 느려진 세션을 위한 내부 진단 도구 |
| 27 | [Remember 스킬](prompts/27_remember_skill.md) | 자동 메모리 항목을 `CLAUDE.md` 파일로 승격 |
| 28 | [Update Config 스킬](prompts/28_update_config_skill.md) | `settings.json`, hook, permission 배열을 관리 |

## 아키텍처

### 프롬프트 조립

메인 시스템 프롬프트는 섹션 빌더(section-builder) 파이프라인을 통해 구성됩니다.

```
System Prompt Assembly
    |
    |   Static Prefix (globally cached)
    |-- Identity and Cyber Risk
    |-- Permission modes, hooks, reminders
    |-- Code style, security, error handling
    |-- Reversibility, blast radius
    |-- Tool preferences, parallel calls
    |-- Tone and style rules
    |-- Output efficiency patterns
    |
    |   CACHE BOUNDARY
    |
    |   Dynamic Suffix (session-specific)
    |-- Agent tools, skills, verification
    |-- Memory file content
    |-- Model overrides
    |-- Environment info (CWD, OS, git state)
    |-- Language preferences
    |-- Custom output styles
    |-- MCP server instructions
    |-- Context window management
```

### Auto Mode 분류기

자동 승인 시스템은 아래 요소로 조립되는 별도의 분류기(classifier) 프롬프트를 사용합니다.

1. **기본 프롬프트(Base prompt)**: 분류기 지침 포함
2. **기본 규칙(Default rules)**: `allow`, `deny`, `environment` 섹션 포함
3. **사용자 재정의(User overrides)**: 각 섹션 전체를 대체
4. **2단계 분류(2-stage classification)**: Stage 1은 빠르게 실행되고, 불확실하면 Stage 2가 확장 추론을 사용

### 컨텍스트 윈도 파이프라인

```
User Message
    |
    v
[Micro-Compaction]  -- Cache-aware tool result deletion
    |
    v
[Compact Service]   -- Full/partial summarization
    |
    v
[Prompt Suggestion] -- Predict next user command
    |
    v
[Away Summary]      -- Session recap if user was idle
```

### 메모리 시스템

```
Memory Loading Order (first loaded = lowest priority):
    |
    |-- Enterprise managed config
    |-- User global config
    |-- Project config (shared, checked in)
    |-- Project rules directory
    |-- Local config (private, git-ignored)
    |
    |   @include directives resolve transitively (max depth: 5)
    |   Frontmatter paths field enables conditional injection
```

### 환경 변수

| 변수 | 효과 |
|----------|--------|
| `CLAUDE_CODE_SIMPLE` | 최소 4줄 시스템 프롬프트를 활성화 |
| `USER_TYPE=ant` | 내부 전용 섹션과 모델 재정의를 활성화 |
| Feature flags | Proactive mode, verification agent, fork subagent 등을 제어 |

## 저장소 구조

```
claude-code-system-prompts/
    README.md
    prompts/
        01_main_system_prompt.md        19_simplify_skill.md
        02_simple_mode.md               20_session_title.md
        03_default_agent_prompt.md      21_compact_service.md
        04_cyber_risk_instruction.md    22_away_summary.md
        05_coordinator_system_prompt.md 23_chrome_browser_automation.md
        06_teammate_prompt_addendum.md  24_memory_instruction.md
        07_verification_agent.md        25_skillify.md
        08_explore_agent.md             26_stuck_skill.md
        09_agent_creation_architect.md  27_remember_skill.md
        10_statusline_setup_agent.md    28_update_config_skill.md
        11_permission_explainer.md      29_agent_summary.md
        12_yolo_auto_mode_classifier.md 30_prompt_suggestion.md
        13_tool_prompts.md
        14_tool_use_summary.md
        15_session_search.md
        16_memory_selection.md
        17_auto_mode_critique.md
        18_proactive_mode.md
```

## 목적

이 프로젝트는 AI 연구자, 에이전트형 시스템을 구축하는 개발자, 그리고 프로덕션급 AI 코딩 어시스턴트의 설계 패턴을 이해하고자 하는 모든 사람을 위한 교육 자료입니다.

여기서 문서화한 패턴은 더 넓은 AI 엔지니어링 커뮤니티와 관련된 다음 주제를 다룹니다.
- 멀티 에이전트 오케스트레이션과 조정
- 자율 Tool 사용을 위한 보안 분류 체계
- 컨텍스트 윈도 관리와 대화 압축
- 계층적 재정의 의미 체계를 갖춘 메모리 시스템
- 지연 시간 최적화를 위한 프롬프트 캐싱 전략

## 면책 조항

이 프로젝트는 독립 연구 프로젝트입니다. 내용은 Claude Code의 동작과 아키텍처에 대한 분석 및 관찰을 바탕으로 합니다. 이 프로젝트는 Anthropic과 어떠한 방식으로도 제휴, 승인, 연관되어 있지 않습니다. 모든 상표는 각 소유자에게 귀속됩니다.
