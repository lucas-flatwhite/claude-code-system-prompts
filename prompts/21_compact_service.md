# Compact 서비스 (대화 요약)

**관찰 위치**: Claude Code 내부 아키텍처
**변수:** `NO_TOOLS_PREAMBLE`, `BASE_COMPACT_PROMPT`, `PARTIAL_COMPACT_PROMPT`, `PARTIAL_COMPACT_UP_TO_PROMPT`
**모델:** Sonnet (via cache-sharing fork or streaming fallback)

## 목적

컨텍스트 윈도가 한계에 가까워질 때 상세한 대화 요약을 생성합니다. Compact 서비스에는 세 가지 동작 모드가 있습니다. 전체 압축, 최근 메시지 부분 압축, 이전 메시지 부분 압축입니다.

## 도구 금지 서문

요약 중 모델이 Tool을 호출하지 못하도록 첫 번째 지침으로 주입됩니다. cache-sharing fork 경로는 부모의 전체 Tool 세트를 상속하므로 이것이 중요합니다.

```
CRITICAL: Respond with TEXT ONLY. Do NOT call any tools.

- Do NOT use Read, Bash, Grep, Glob, Edit, Write, or ANY other tool.
- You already have all the context you need in the conversation above.
- Tool calls will be REJECTED and will waste your only turn — you will fail the task.
- Your entire response must be plain text: an <analysis> block followed by a <summary> block.
```

## 기본 Compact 프롬프트 (전체 요약)

전체 대화를 요약할 때 사용됩니다.

```
Your task is to create a detailed summary of the conversation so far, paying close
attention to the user's explicit requests and your previous actions.
This summary should be thorough in capturing technical details, code patterns, and
architectural decisions that would be essential for continuing development work
without losing context.
```

### 필수 요약 섹션

1. **주요 요청과 의도(Primary Request and Intent)** — 모든 명시적 사용자 요청과 의도
2. **핵심 기술 개념(Key Technical Concepts)** — 논의된 기술, 프레임워크, 패턴
3. **파일과 코드 구간(Files and Code Sections)** — 전체 코드 스니펫과 함께 검토/수정/생성한 파일
4. **오류와 수정(Errors and Fixes)** — 발생한 오류와 해결 단계
5. **문제 해결(Problem Solving)** — 해결된 문제와 진행 중인 트러블슈팅
6. **모든 사용자 메시지(All User Messages)** — Tool 결과가 아닌 모든 사용자 메시지 (의도 변화 추적에 중요)
7. **보류 작업(Pending Tasks)** — 남아 있는 작업 항목
8. **현재 작업(Current Work)** — 요약 직전 진행 중이던 작업의 정확한 설명
9. **선택적 다음 단계(Optional Next Step)** — 최근 사용자 요청과 직접 맞닿는 경우에만 다음 단계

## 부분 Compact 프롬프트 (최근 메시지)

이전 컨텍스트는 유지하고 최근 메시지만 요약할 때 사용됩니다.

```
Your task is to create a detailed summary of the RECENT portion of the conversation —
the messages that follow earlier retained context. The earlier messages are being kept
intact and do NOT need to be summarized. Focus your summary on what was discussed,
learned, and accomplished in the recent messages only.
```

## 부분 Compact Up-To 프롬프트 (이전 컨텍스트)

새 메시지는 그대로 두고 공간 확보를 위해 이전 메시지를 요약할 때 사용됩니다.

```
Your task is to create a detailed summary of this conversation. This summary will be
placed at the start of a continuing session; newer messages that build on this context
will follow after your summary (you do not see them here). Summarize thoroughly so
that someone reading only your summary and then the newer messages can fully understand
what happened and continue the work.
```

8번 섹션("Current Work")은 "Work Completed"로, 9번 섹션은 "Context for Continuing Work"로 대체됩니다.

## 분석 블록

세 변형 모두 `DETAILED_ANALYSIS_INSTRUCTION`을 포함하며, 모델이 `<summary>` 전에 추론을 `<analysis>` 태그로 감싸도록 요구합니다. analysis 블록은 초안 scratchpad 역할을 하며, 요약이 컨텍스트에 들어가기 전에 `formatCompactSummary()`가 제거합니다.

```
Before providing your final summary, wrap your analysis in <analysis> tags to organize
your thoughts and ensure you've covered all necessary points. In your analysis process:

1. Chronologically analyze each message and section of the conversation. For each
   section thoroughly identify:
   - The user's explicit requests and intents
   - Your approach to addressing the user's requests
   - Key decisions, technical concepts and code patterns
   - Specific details like: file names, full code snippets, function signatures, file edits
   - Errors that you ran into and how you fixed them
   - Pay special attention to specific user feedback
2. Double-check for technical accuracy and completeness.
```

## 사용자 지정 지침 지원

Compact 프롬프트는 CLAUDE.md 또는 hook을 통한 사용자 지정 요약 지침을 지원합니다.

```
There may be additional summarization instructions provided in the included context.
If so, remember to follow these instructions when creating the above summary.
```

## 아키텍처 노트

- **Cache-sharing fork**: 부모의 프롬프트 캐시를 상속하는 분기 프로세스입니다. 전체 Tool 세트도 상속하므로 `NO_TOOLS_PREAMBLE`가 필요합니다.
- **Streaming fallback**: fork 경로가 실패하면 (예: 모델이 `maxTurns: 1` 상태에서 Tool 호출 시도) 표준 스트리밍 질의로 넘어갑니다.
- **Proactive mode integration**: Proactive mode 또는 Kairos가 활성화되면 proactive 작업에 대한 추가 컨텍스트가 덧붙습니다.
