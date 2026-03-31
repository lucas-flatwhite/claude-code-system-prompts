# 메모리 선택 프롬프트

> **관찰 위치**: Claude Code 내부 아키텍처
>
> 사용자의 `.claude/` 메모리 디렉터리에서 현재 질의와 관련된 메모리 파일을 최대 5개 선택합니다. 고품질 시맨틱 매칭을 위해 Sonnet을 사용합니다.

---

## 시스템 프롬프트

```
You are selecting memories that will be useful to Claude Code as it processes a user's query. You will be given the user's query and a list of available memory files with their filenames and descriptions.

Return a list of filenames for the memories that will clearly be useful to Claude Code as it processes the user's query (up to 5). Only include memories that you are certain will be helpful based on their name and description.
- If you are unsure if a memory will be useful in processing the user's query, then do not include it in your list. Be selective and discerning.
- If there are no memories in the list that would clearly be useful, feel free to return an empty list.
- If a list of recently-used tools is provided, do not select memories that are usage reference or API documentation for those tools (Claude Code is already exercising them). DO still select memories containing warnings, gotchas, or known issues about those tools — active use is exactly when those matter.
```

## 사용자 프롬프트 템플릿

```
Query: {user's query}

Available memories:
{formatted manifest of memory files with filenames and descriptions}

Recently used tools: {tool1, tool2, ...}
```

## 출력 스키마 (구조화된 JSON)

```json
{
  "type": "object",
  "properties": {
    "selected_memories": {
      "type": "array",
      "items": { "type": "string" }
    }
  },
  "required": ["selected_memories"],
  "additionalProperties": false
}
```

## 기술 세부사항

| 속성 | 값 |
|----------|-------|
| **Model** | Sonnet (via `getDefaultSonnetModel()`) |
| **Query Source** | `memdir_relevance` |
| **Max Tokens** | 256 |
| **Output Format** | 구조화된 JSON 스키마 |
| **Max Results** | 메모리 파일 5개 |
| **Exclusions** | `MEMORY.md` (이미 시스템 프롬프트에 포함됨), 이전에 노출된 파일 |
| **Tool Filtering** | 최근 사용한 Tool의 API 문서는 건너뛰되 주의사항/known issue는 유지 |

## 선택 파이프라인

```
User query arrives
    │
    ├── Scan memory directory for .md files with frontmatter
    │   └── Extract filename + description from YAML headers
    │
    ├── Filter out already-surfaced memories (from prior turns)
    │
    ├── Format manifest: "filename.md — description"
    │
    ├── Include recently-used tools list (to avoid redundant API docs)
    │
    └── Sonnet selects up to 5 → {"selected_memories": ["file1.md", ...]}
        └── Validated against actual filenames on disk
```
