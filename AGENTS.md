# Agent Instructions

이 저장소(`dev-ai-tools`)는 개발용 AI 도구(Claude Code, Codex)의 설정을 통합 관리하는 메타 저장소다. 두 개의 git submodule로 구성된다.

사람용 개요는 [README.md](./README.md)에 있다. 이 문서는 agent를 위한 안내다.

## Submodules

- **[claude-config/](./claude-config)** — Claude Code의 rules, skills, settings.json을 관리하고 `~/.claude/`로 동기화한다.
- **[codex-config/](./codex-config)** — Codex의 AGENTS.md, rules, skills를 관리하고 `~/.codex/`로 동기화한다.

## 작업 위치에 따른 참고 문서

작업 대상이 결정되면 해당 submodule의 agent용 문서를 먼저 읽고 그 지침을 따른다.

| 작업 대상 | 먼저 읽을 문서 |
|---|---|
| `claude-config/` 내부 작업 | [claude-config/CLAUDE.md](./claude-config/CLAUDE.md) |
| `codex-config/` 내부 작업 | [codex-config/AGENTS.md](./codex-config/AGENTS.md) |
| 루트 레벨 작업(submodule 갱신, README 등) | 이 문서 + [README.md](./README.md) |

각 submodule은 독립된 git 저장소이므로, 변경 후에는 submodule 내부에서 먼저 commit한 뒤 부모 repo에서 submodule 포인터 변경을 별도 commit한다.

## 주의사항

- 두 submodule은 독립적으로 버전 관리된다. 한쪽 변경을 다른 쪽에 무단으로 옮기지 말 것.
- rules·skills 내용을 두 도구 사이에서 옮길 때는 도구별 도구명·경로 차이(예: `.claude` ↔ `.codex`, Claude 전용 도구명 등)를 반드시 변환한다. 자세한 변환 규칙은 [codex-config/README.md](./codex-config/README.md)의 "Skill 작성 가이드"와 "제외한 항목" 절 참고.
