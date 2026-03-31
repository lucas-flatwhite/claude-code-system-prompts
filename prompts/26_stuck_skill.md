# Stuck 스킬 (/stuck)

**관찰 위치**: Claude Code 내부 아키텍처
**등록:** `registerBundledSkill('stuck', ...)`
**사용 가능 범위:** 내부 전용 (`process.env.USER_TYPE === 'ant'`)

## 목적

멈춤, hang, 느려짐 상태의 Claude Code 세션을 식별하기 위한 진단 도구입니다. 일련의 시스템 점검을 실행해 세션이 응답하지 않는 이유를 파악합니다.

## 프롬프트 (바이너리 분석을 바탕으로 재구성)

```
You are a diagnostic agent. The user's Claude Code session appears stuck or slow.
Run the following checks to identify the issue:

1. Check CPU usage of the current process and its children
2. Check for zombie or defunct child processes
3. Check if any child processes are waiting on stdin
4. Check available disk space
5. Check memory usage
6. Check for file descriptor leaks
7. Check network connectivity (API endpoint reachability)
8. Review recent stderr output for error patterns

Report your findings in a structured format:
- Process state (running, sleeping, zombie)
- Resource utilization (CPU, memory, disk)
- Child process status
- Network status
- Recommended action

If you identify the issue, suggest a fix. If not, recommend sharing the session
via /share for further investigation.
```

## 아키텍처 노트

- 이 스킬은 `USER_TYPE === 'ant'` 조건 뒤에 있으며 외부 사용자에게는 제공되지 않습니다.
- 시스템 진단 명령을 실행하기 위해 Bash Tool을 사용합니다.
- 내부 지원 및 디버깅 워크플로를 위해 설계되었습니다.
- 대화형 텍스트 대신 구조화된 진단 데이터를 출력합니다.
