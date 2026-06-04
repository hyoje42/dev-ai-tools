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

## 원격 저장소 (여러 원격 미러링)

부모 repo와 두 submodule은 **여러 원격에 미러링**될 수 있다. 관례상 처음 클론한 곳을 `origin`, 추가 원격을 `upstream`으로 둔다.

- **`.gitmodules`의 submodule URL은 상대 경로(`../claude-config.git`)로 유지한다. 절대 URL로 되돌리지 말 것.** 상대 경로여야 "부모를 클론한 원격"에 맞춰 submodule 주소가 자동 해석된다. URL을 바꿨다면 `git submodule sync`로 로컬 `.git/config`에 반영한다.
- **push는 submodule을 먼저, 그다음 부모** 순서로 한다. 여러 원격을 모두 동기화하려면 각 원격(`origin`, `upstream` 등)에 모두 push해야 거울이 어긋나지 않는다. (단, 사용자가 일부 원격만 push하라고 지시할 수 있으니 지시를 따른다.)

## 주의사항

- 두 submodule은 독립적으로 버전 관리된다. 한쪽 변경을 다른 쪽에 무단으로 옮기지 말 것.
- rules·skills 내용을 두 도구 사이에서 옮길 때는 도구별 도구명·경로 차이(예: `.claude` ↔ `.codex`, Claude 전용 도구명 등)를 반드시 변환한다. 자세한 변환 규칙은 [codex-config/README.md](./codex-config/README.md)의 "Skill 작성 가이드"와 "제외한 항목" 절 참고.
