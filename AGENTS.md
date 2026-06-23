# Agent Instructions

이 저장소(`dev-ai-tools`)는 개발용 AI 도구(Claude Code, Codex) 설정을 통합 관리하는 메타 저장소다. 두 개의 git submodule로 구성된다.

repo 개요·셋업은 [README.md](./README.md)에 있다. 이 문서는 agent 작업 규칙과, 작업 위치별로 어느 문서를 볼지 가리키는 index다.

## Submodules

각 submodule은 동일 패턴이다: **sync 대상은 `home/` 하위뿐이고, 그 구조가 그대로 `~/.<tool>/`에 미러링된다.** 루트의 다른 파일(README·AGENTS·sync 스크립트 등)은 repo 관리용이라 sync 대상이 아니다.

- **[claude-config/](./claude-config)** — `home/`이 `~/.claude/`로 sync.
- **[codex-config/](./codex-config)** — `home/`이 `~/.codex/`로 sync.

**머신 종속 설정**은 sync 영역 밖, 각 submodule의 `local/`에 둔다 — 구조는 `*.example`로 커밋하고 실제 값은 git-ignored. 적용 메커니즘은 도구별로 다르다(Claude=settings deep-merge, Codex=config.toml seed-if-absent). **한쪽 메커니즘을 다른 쪽에 무단 이식하지 말 것.**

## 작업 위치에 따른 참고 문서

작업 대상이 정해지면 해당 문서를 먼저 읽고 그 지침을 따른다.

| 작업 대상 | 먼저 읽을 문서 |
|---|---|
| `claude-config/` 내부 | [claude-config/AGENTS.md](./claude-config/AGENTS.md) |
| `codex-config/` 내부 | [codex-config/AGENTS.md](./codex-config/AGENTS.md) |
| 루트 레벨(submodule 갱신·README 등) | 이 문서 + [README.md](./README.md) |

> submodule의 AGENTS.md(= `CLAUDE.md`)는 그 하위 트리에 진입할 때 자동 로드된다. 각 submodule의 상세 구조·사용법은 그 repo의 README.md에 있다.

## 작업 규칙

- **sync 게이트**: `home/` 수정 후 `*-diff-with-home`으로 차이를 확인해 사용자에게 공유하는 데까지만 한다. **`~/.<tool>/` 반영(`*-sync-to-home` 실행이든 수동 복사든)은 사용자가 명시적으로 지시했을 때만.** agent가 임의로 sync하지 말 것.
- **커밋 순서**: submodule에서 먼저 commit → 부모에서 submodule 포인터 변경을 별도 commit.
- 두 submodule은 독립적으로 버전 관리된다. 한쪽 변경을 다른 쪽에 무단으로 옮기지 말 것. rules·skills를 도구 간에 옮길 땐 도구명·경로 차이(`.claude` ↔ `.codex`, Claude 전용 도구명 등)를 반드시 변환한다(자세한 변환 규칙은 [codex-config/skill-authoring.md](./codex-config/skill-authoring.md) 참고).

## 원격 저장소 (여러 원격 미러링)

- **`.gitmodules`의 submodule URL은 상대 경로(`../claude-config.git`)로 유지한다. 절대 URL로 되돌리지 말 것.** URL을 바꿨다면 `git submodule sync`로 로컬 `.git/config`에 반영한다. (상대 경로가 필요한 이유는 README 참고.)
- **push는 submodule을 먼저, 그다음 부모** 순서로 한다. 여러 원격(관례상 `origin`/`upstream` — README 참고)을 모두 동기화하려면 각 원격에 모두 push해야 거울이 어긋나지 않는다(사용자가 일부 원격만 push하라고 지시할 수 있으니 지시를 따른다).

## 문서 작성 원칙

문서는 **목적**으로 나눈다(청중별 복제 금지).

- **README.md = repo 설명(canonical)** / **AGENTS.md(= `CLAUDE.md`) = 작업 규칙 + index.** 같은 내용을 양쪽에 복제하지 말 것 — 개요·상세는 README나 전용 문서를 가리킨다.
- 길어지는 주제(예: skill 작성 가이드)는 **별도 문서**로 빼고 AGENTS에선 한 줄로 가리킨다.
- AGENTS.md는 자동 로드되니 짧게 유지한다(README는 자동 로드 안 됨).
