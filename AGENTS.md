# Agent Instructions

이 저장소(`dev-ai-tools`)는 개발용 AI 도구(Claude Code, Codex)의 설정을 통합 관리하는 메타 저장소다. 두 개의 git submodule로 구성된다.

사람용 개요는 [README.md](./README.md)에 있다. 이 문서는 agent를 위한 안내다.

## Submodules

각 submodule은 동일한 패턴을 따른다. **sync 대상 파일은 `home/` 서브디렉터리 안에 있으며, 그 구조가 그대로 `~/.<tool>/`에 미러링된다.** 루트의 다른 파일(`CLAUDE.md`, `README.md`, sync 스크립트 등)은 repo 관리용이지 sync 대상이 아니다.

- **[claude-config/](./claude-config)** — `claude-config/home/`이 `~/.claude/`로 sync.
- **[codex-config/](./codex-config)** — `codex-config/home/`이 `~/.codex/`로 sync.

## 작업 위치에 따른 참고 문서

작업 대상이 결정되면 해당 submodule의 meta 문서를 먼저 읽고 그 지침을 따른다.

| 작업 대상 | 먼저 읽을 문서 |
|---|---|
| `claude-config/` 내부 작업 | [claude-config/CLAUDE.md](./claude-config/CLAUDE.md) |
| `codex-config/` 내부 작업 | [codex-config/AGENTS.md](./codex-config/AGENTS.md) |
| 루트 레벨 작업(submodule 갱신, README 등) | 이 문서 + [README.md](./README.md) |

`*-config/home/` 안의 파일을 수정한 경우, 해당 submodule의 `*-sync-to-home` 스크립트를 실행해 `~/.<tool>/`에 반영한다. 파일을 수정하기 전후로 `*-diff-with-home`을 돌려 어떤 변화가 적용되는지 확인한다.

각 submodule은 독립된 git 저장소이므로, 변경 후에는 submodule 내부에서 먼저 commit한 뒤 부모 repo에서 submodule 포인터 변경을 별도 commit한다.

## 주의사항

- 두 submodule은 독립적으로 버전 관리된다. 한쪽 변경을 다른 쪽에 무단으로 옮기지 말 것.
- rules·skills 내용을 두 도구 사이에서 옮길 때는 도구별 도구명·경로 차이(예: `.claude` ↔ `.codex`, Claude 전용 도구명 등)를 반드시 변환한다. 자세한 변환 규칙은 [codex-config/README.md](./codex-config/README.md)의 "Skill 작성 가이드"와 "제외한 항목" 절 참고.
