# Codex 설정: claude-config에서 무엇을 가져오고 무엇을 제외했나

`codex-config`는 `claude-config`의 구조·skill을 기반으로 만들어졌다. 두 submodule은 독립이라 각 README는 자기 repo만 설명하고 서로를 참조하지 않는다. 도구 간 제외·변환 정책은 메타 repo 차원의 관심사라 여기(부모)에 모아 둔다.

## 제외한 항목 (claude-config → codex-config)

- `claude-config/reference-skills/` — submodule이라 복사하지 않았다.
- `claude-config/_backup/` — Claude 홈 백업이라, Codex에는 별도의 `_backup/`만 둔다.
- `claude-config/home/settings.json` — Codex와 설정 형식이 달라 그대로 동기화하지 않는다(아래 변환 정책 참고).

## 설정 변환 정책 (Claude `settings.json` → Codex)

Claude `home/settings.json`의 항목 중 Codex에 대응이 없거나 형식이 다른 것은 옮기지 않았다.

- `env.CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` — Claude 전용 실험 플래그. Codex엔 같은 기능이 없어, 관련 skill(`setup-team-agents`)은 2026-06-12에 퇴역해 `outdated/`로 옮겼다.
- `env.CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING`, `env.MAX_THINKING_TOKENS` — Claude 전용 환경 변수. Codex의 reasoning effort는 `~/.codex/config.toml`의 모델·추론 설정으로 관리한다.
- `permissions.allow`/`permissions.deny`, `defaultMode` — Claude 권한 DSL. Codex는 `~/.codex/rules/default.rules`·sandbox·approval 정책을 쓰며, 기존 승인 규칙을 덮어쓰면 위험하므로 동기화 대상에서 제외했다.
- `skipDangerousModePermissionPrompt`, `skipAutoPermissionPrompt` — Claude 전용 프롬프트 설정.

Codex에 반영한 방식:

- 언어/응답/도구 사용·git 커밋 규칙 → `codex-config/home/AGENTS.md` (Codex가 로드하는 전역 규칙은 이 단일 파일).
- 커스텀 skill → `codex-config/home/skills/`.
- model·reasoning effort·project trust·plugin 활성화 같은 머신 종속 값 → `~/.codex/config.toml`을 직접 편집(이 repo의 스크립트는 config.toml을 덮어쓰지 않는다).

## Skill 변환 규칙

skill을 Claude ↔ Codex로 옮길 때의 형식·도구명·경로 변환 규칙은 [codex-config/skill-authoring.md](../codex-config/skill-authoring.md) 참고.
