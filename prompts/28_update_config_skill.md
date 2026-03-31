# Update Config 스킬 (/update-config)

**관찰 위치**: Claude Code 내부 아키텍처
**등록:** `registerBundledSkill('update-config', ...)`
**사용 가능 범위:** 모든 사용자

## 목적

`settings.json` 설정 파일을 관리합니다. hook, permission 배열, 일반 설정을 포함합니다. Claude Code 설정 계층을 편집할 수 있도록 안내형 인터페이스를 제공합니다.

## 프롬프트 (바이너리 분석을 바탕으로 재구성)

이 프롬프트는 매우 길며(~476줄) 세 가지 주요 영역을 다룹니다.

### 설정 파일 계층

```
Settings are loaded from three levels (highest to lowest priority):
1. Project settings: .claude/settings.json (checked into repo)
2. User settings: ~/.claude/settings.json (private, all projects)
3. Enterprise settings: /etc/claude-code/settings.json (managed by admins)

Lower-priority settings are overridden by higher-priority ones.
```

### Hook 시스템

이 스킬은 전체 hook 생명주기를 문서화합니다.

```
Hooks are commands that run at specific points in Claude Code's lifecycle:
- PreToolUse: Runs before a tool is executed. Can block the tool.
- PostToolUse: Runs after a tool completes. Can modify the result.
- PreCompact: Runs before conversation compaction.
- PostCompact: Runs after conversation compaction.
- Notification: Runs when Claude wants to notify the user.
- Stop: Runs when Claude's turn ends.

Each hook receives a JSON payload on stdin with:
- session_id: Current session ID
- tool_name: Name of the tool (PreToolUse/PostToolUse)
- tool_input: Tool input parameters (PreToolUse/PostToolUse)
- tool_output: Tool output (PostToolUse only)

Hook exit codes:
- 0: Success, continue normally
- 2: Block the tool (PreToolUse only)
- Other: Log error but continue
```

### Permission 배열

```
Permission settings control which tools can run without user approval:
- allowedTools: Tool names or patterns that are auto-approved
- deniedTools: Tool names or patterns that are always blocked
- trust: Trust levels for different tool categories
```

## 아키텍처 노트

- 이 스킬은 "simple" 설정 작업(키-값 설정)과 "complex" 작업(hook, permission)을 구분합니다.
- hook 편집 시 `settings.json` 파일에 대해 Edit Tool을 직접 사용합니다.
- 각 편집 후 JSON 문법을 검증합니다.
- 하나의 인터페이스에서 세 가지 설정 레벨을 모두 지원합니다.
