# Codex 설정: claude-config에서 무엇을 가져오고 무엇을 제외했나

`codex-config`는 `claude-config`의 구조·skill을 기반으로 만들어졌다. 두 submodule은 독립이라 각 README는 자기 repo만 설명하고 서로를 참조하지 않는다. 도구 간 제외·변환 정책은 메타 repo 차원의 관심사라 여기(부모)에 모아 둔다.

## 제외한 항목 (claude-config → codex-config)

- `claude-config/reference-skills/` — submodule이라 복사하지 않았다.
- `claude-config/_backup/` — Claude 홈 백업이라, Codex에는 별도의 `_backup/`만 둔다.
- `claude-config/home/settings.json` — Codex와 설정 형식이 달라 그대로 가져오지 않는다. Codex 쪽은 `home/config.toml` baseline과 `local/config.override.toml`을 TOML merge로 적용한다(아래 변환 정책 참고).

## 설정 변환 정책 (Claude `settings.json` → Codex)

Claude `home/settings.json`의 항목 중 Codex에 대응이 없거나 형식이 다른 것은 옮기지 않았다.

- `env.CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` — Claude 전용 실험 플래그. Codex엔 같은 기능이 없어, 관련 skill(`setup-team-agents`)은 2026-06-12에 퇴역해 `outdated/`로 옮겼다.
- `env.CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING`, `env.MAX_THINKING_TOKENS` — Claude 전용 환경 변수. Codex의 reasoning effort는 `~/.codex/config.toml`의 모델·추론 설정으로 관리한다.
- `permissions.allow`/`permissions.deny`, `defaultMode` — Claude 권한 DSL. Codex는 `~/.codex/rules/default.rules`·sandbox·approval 정책을 쓰며, 기존 승인 규칙을 덮어쓰면 위험하므로 동기화 대상에서 제외했다.
- `attribution.commit`/`attribution.pr`/`attribution.sessionUrl` — Codex에 직접 같은 JSON 구조는 없으므로, 대응 가능한 공통값은 `home/config.toml`의 `commit_attribution = ""` 같은 Codex TOML 설정으로 둔다.
- `skipDangerousModePermissionPrompt`, `skipAutoPermissionPrompt` — Claude 전용 프롬프트 설정.

Codex에 반영한 방식:

- 언어/응답/도구 사용·git 커밋 규칙 → `codex-config/home/AGENTS.md` (Codex가 로드하는 전역 규칙은 이 단일 파일).
- 커스텀 skill 원본 → `codex-config/home/skills/`, sync 대상 → `~/.agents/skills/`.
- 공통 Codex config 값(예: attribution·feature 기본값) → `codex-config/home/config.toml` baseline.
- model·reasoning effort·project trust 같은 머신 종속 override → `codex-config/local/config.override.toml` 또는 기존 `~/.codex/config.toml`. sync는 현재 `~/.codex/config.toml` + baseline + override를 merge해서 같은 키는 교체하고 없는 키는 추가하며, repo가 모르는 기존 키는 보존한다.

## Skill 변환 규칙

skill을 Claude ↔ Codex로 옮길 때의 형식·도구명·경로 변환 규칙은 [codex-config/skill-authoring.md](../codex-config/skill-authoring.md) 참고. 특히 명시 호출 전용 설정은 Claude `SKILL.md`의 `disable-model-invocation: true`를 Codex `agents/openai.yaml`의 `policy.allow_implicit_invocation: false`로 변환한다.
