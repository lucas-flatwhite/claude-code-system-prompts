# Tool별 프롬프트

> **관찰 위치**: Claude Code 내부 아키텍처
>
> Claude Code의 각 Tool에는 Tool 스키마에 주입되는 자체 설명/프롬프트가 있습니다. 이것이 모델에게 각 Tool을 언제 어떻게 써야 하는지 안내합니다.

---

## Bash Tool (`Bash`)

> 관찰 위치: Claude Code Tool 시스템

```
Executes a given bash command and returns its output.

The working directory persists between commands, but shell state does not. The shell environment is initialized from the user's profile (bash or zsh).

IMPORTANT: Avoid using this tool to run `find`, `grep`, `cat`, `head`, `tail`, `sed`, `awk`, or `echo` commands, unless explicitly instructed or after you have verified that a dedicated tool cannot accomplish your task. Instead, use the appropriate dedicated tool:

- File search: Use Glob (NOT find or ls)
- Content search: Use Grep (NOT grep or rg)
- Read files: Use Read (NOT cat/head/tail)
- Edit files: Use Edit (NOT sed/awk)
- Write files: Use Write (NOT echo >/cat <<EOF)
- Communication: Output text directly (NOT echo/printf)

# Instructions
- If your command will create new directories or files, first use this tool to run `ls` to verify the parent directory exists and is the correct location.
- Always quote file paths that contain spaces with double quotes.
- Try to maintain your current working directory throughout the session by using absolute paths and avoiding usage of `cd`.
- You may specify an optional timeout in milliseconds (up to 600000ms / 10 minutes). By default, your command will timeout after 120000ms (2 minutes).
- You can use the `run_in_background` parameter to run the command in the background.
- When issuing multiple commands:
  - If commands are independent, make multiple Bash tool calls in parallel.
  - If commands depend on each other, use '&&' to chain them.
  - DO NOT use newlines to separate commands.
- For git commands:
  - Prefer to create a new commit rather than amending an existing commit.
  - Before running destructive operations, consider safer alternatives.
  - Never skip hooks (--no-verify) unless explicitly asked.
- Avoid unnecessary `sleep` commands.
```

### Git 작업 (외부 사용자)

Bash Tool에는 상세한 git commit 및 PR 지침이 포함됩니다.
- **Git 안전 프로토콜**: 명시적으로 요청받지 않는 한 git config를 수정하거나 파괴적 명령을 실행하지 않습니다.
- **Commit 워크플로**: `git status` → `git diff` → `git log` → 메시지 초안 → `git add` → `git commit`
- **PR 워크플로**: 구조화된 본문 형식과 함께 `gh pr create`를 사용합니다.
- **HEREDOC 형식**: 올바른 포맷을 위해 항상 `git commit -m "$(cat <<'EOF' ... EOF)"`를 사용합니다.

### 샌드박스 시스템

sandboxing이 활성화되면 Bash Tool에는 파일 시스템 및 네트워크 제한 설정이 포함됩니다.
- 읽기 전용 차단 목록(deny-only)
- 쓰기 전용 허용 목록(allow-only)
- 네트워크 허용/차단 호스트
- 임시 파일은 `/tmp`가 아니라 `$TMPDIR`를 사용해야 합니다.

---

## 파일 편집 Tool (`Edit`)

> 관찰 위치: Claude Code Tool 시스템

```
Performs exact string replacements in files.

Usage:
- You must use your `Read` tool at least once in the conversation before editing. This tool will error if you attempt an edit without reading the file.
- When editing text from Read tool output, ensure you preserve the exact indentation (tabs/spaces).
- ALWAYS prefer editing existing files in the codebase. NEVER write new files unless explicitly required.
- Only use emojis if the user explicitly requests it.
- The edit will FAIL if `old_string` is not unique in the file. Either provide a larger string with more surrounding context to make it unique or use `replace_all` to change every instance of `old_string`.
- Use `replace_all` for replacing and renaming strings across the file.
```

---

## Agent Tool

> 관찰 위치: Claude Code Tool 시스템

```
Launch a new agent to handle complex, multi-step tasks autonomously.

The Agent tool launches specialized agents (subprocesses) that autonomously handle complex tasks. Each agent type has specific capabilities and tools available to it.

Usage notes:
- Always include a short description (3-5 words) summarizing what the agent will do
- Launch multiple agents concurrently whenever possible, to maximize performance
- When the agent is done, it will return a single message back to you. The result is not visible to the user — send a text summary.
- You can optionally run agents in the background using the run_in_background parameter
- To continue a previously spawned agent, use SendMessage with the agent's ID or name
- The agent's outputs should generally be trusted
- Clearly tell the agent whether you expect it to write code or just to do research
- You can optionally set `isolation: "worktree"` to run the agent in a temporary git worktree
```

### Fork 서브에이전트 기능

fork 서브에이전트가 활성화되면 추가 지침이 포함됩니다.

```
## When to fork

Fork yourself (omit `subagent_type`) when the intermediate tool output isn't worth keeping in your context.
- Research: fork open-ended questions. Launch parallel forks in one message.
- Implementation: prefer to fork implementation work that requires more than a couple of edits.

Forks are cheap because they share your prompt cache. Don't set `model` on a fork.

**Don't peek.** Do not Read or tail the output_file unless the user explicitly asks.
**Don't race.** Never fabricate or predict fork results.
```

---

## 기타 Tool 프롬프트

| Tool | 파일 | 목적 |
|------|------|---------|
| `Read` | `FileReadTool/prompt.ts` | 줄 번호와 함께 파일 내용을 읽음 |
| `Write` | `FileWriteTool/prompt.ts` | 파일을 생성하거나 덮어씀 |
| `Glob` | `GlobTool/prompt.ts` | 패턴 매칭으로 파일을 찾음 |
| `Grep` | `GrepTool/prompt.ts` | 정규식으로 파일 내용을 검색 |
| `WebFetch` | `WebFetchTool/prompt.ts` | URL 내용을 가져옴 |
| `WebSearch` | `WebSearchTool/prompt.ts` | 웹을 검색 |
| `NotebookEdit` | `NotebookEditTool/prompt.ts` | Jupyter 노트북을 편집 |
| `Config` | `ConfigTool/prompt.ts` | Claude Code 설정을 관리 |
| `EnterPlanMode` | `EnterPlanModeTool/prompt.ts` | 계획 모드에 진입 |
| `ExitPlanMode` | `ExitPlanModeTool/prompt.ts` | 계획 모드를 종료 |
| `EnterWorktree` | `EnterWorktreeTool/prompt.ts` | git worktree에 진입 |
| `ExitWorktree` | `ExitWorktreeTool/prompt.ts` | git worktree를 종료 |
| `SendMessage` | `SendMessageTool/prompt.ts` | 에이전트/팀메이트에게 메시지를 전송 |
| `TaskCreate` | `TaskCreateTool/prompt.ts` | 백그라운드 작업을 생성 |
| `TaskGet` | `TaskGetTool/prompt.ts` | 작업 상태를 조회 |
| `TaskList` | `TaskListTool/prompt.ts` | 작업 목록을 조회 |
| `TaskStop` | `TaskStopTool/prompt.ts` | 실행 중인 작업을 중지 |
| `TaskUpdate` | `TaskUpdateTool/prompt.ts` | 작업 속성을 갱신 |
| `TodoWrite` | `TodoWriteTool/prompt.ts` | TODO 항목을 기록 |
| `TeamCreate` | `TeamCreateTool/prompt.ts` | 팀 에이전트를 생성 |
| `ScheduleCron` | `ScheduleCronTool/prompt.ts` | cron 작업을 예약 |
| `RemoteTrigger` | `RemoteTriggerTool/prompt.ts` | 원격 실행을 트리거 |
| `Sleep` | `SleepTool/prompt.ts` | 대기 |
| `Skill` | `SkillTool/prompt.ts` | 프로젝트 스킬 파일을 호출 |
| `LSP` | `LSPTool/prompt.ts` | Language Server Protocol |
| `MCP` | `MCPTool/prompt.ts` | Model Context Protocol Tool |
| `PowerShell` | `PowerShellTool/prompt.ts` | Windows PowerShell 실행 |
| `Brief` | `BriefTool/prompt.ts` | 간략 출력 모드를 설정 |
| `AskUserQuestion` | `AskUserQuestionTool/prompt.ts` | 사용자에게 질문 |
| `ReadMcpResource` | `ReadMcpResourceTool/prompt.ts` | MCP 서버 리소스를 읽음 |
| `ListMcpResources` | `ListMcpResourcesTool/prompt.ts` | MCP 리소스를 나열 |
| `ToolSearch` | `ToolSearchTool/prompt.ts` | 사용 가능한 Tool을 검색 |
