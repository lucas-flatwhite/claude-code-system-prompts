# 메모리 지침 (CLAUDE.md 시스템)

**관찰 위치**: Claude Code 내부 아키텍처
**변수:** `MEMORY_INSTRUCTION_PROMPT`

## 목적

시스템 프롬프트 안에서 로드된 모든 CLAUDE.md 메모리 파일을 감싸는 메타 지침입니다. 이 한 줄이 사용자 제공 지침이 기본 동작보다 절대적으로 우선함을 규정합니다.

## 프롬프트

```
Codebase and user instructions are shown below. Be sure to adhere to these instructions.
IMPORTANT: These instructions OVERRIDE any default behavior and you MUST follow them
exactly as written.
```

## 메모리 파일 로딩 순서

파일은 우선순위의 역순으로 로드됩니다 (나중일수록 우선순위 높음).

1. **관리 메모리(Managed memory)** (`/etc/claude-code/CLAUDE.md`) — 모든 사용자를 위한 전역 지침
2. **사용자 메모리(User memory)** (`~/.claude/CLAUDE.md`) — 모든 프로젝트에 적용되는 개인 전역 지침
3. **프로젝트 메모리(Project memory)** (`CLAUDE.md`, `.claude/CLAUDE.md`, `.claude/rules/*.md` in project roots) — 코드베이스에 커밋되는 지침
4. **로컬 메모리(Local memory)** (`CLAUDE.local.md` in project roots) — 개인 프로젝트 전용 지침

## 파일 탐색

- 사용자 메모리는 `~/.claude/`에서 로드됩니다.
- 프로젝트 및 로컬 파일은 현재 디렉터리에서 루트까지 거슬러 올라가며 탐색됩니다.
- 현재 디렉터리에 가까운 파일일수록 우선순위가 높습니다 (나중에 로드됨).
- 각 디렉터리에서 `CLAUDE.md`, `.claude/CLAUDE.md`, `.claude/rules/` 아래 모든 `.md` 파일을 확인합니다.

## @include 지시어

메모리 파일은 전이적 파일 포함을 지원합니다.

- 문법: `@path`, `@./relative/path`, `@~/home/path`, 또는 `@/absolute/path`
- 말단 텍스트 노드에서만 동작합니다 (코드 블록 내부 제외).
- 처리한 파일을 추적하여 순환 참조를 방지합니다.
- 존재하지 않는 파일은 조용히 무시됩니다.
- 최대 include 깊이: 5
- 텍스트 파일 확장자만 허용합니다 (이미지, PDF 등 로드 방지).

## 프론트매터(Frontmatter) 지원

메모리 파일은 조건부 주입을 위한 `paths` 필드를 가진 YAML 프론트매터(frontmatter)를 지원합니다.

```yaml
---
paths:
  - src/components/**
  - "*.tsx"
---
```

`paths` 프론트매터(frontmatter)가 있는 파일은 활성 파일이 glob 패턴과 일치할 때만 주입됩니다.

## 구성

- `MAX_MEMORY_CHARACTER_COUNT`: 40000자 (파일당 권장 최대치)
- `claudeMdExcludes` 설정: 특정 CLAUDE.md 파일을 제외하는 glob 패턴
- `MAX_INCLUDE_DEPTH`: 전이적 포함 5단계
- 메모리 파일의 HTML 주석은 주입 전에 제거됩니다.
