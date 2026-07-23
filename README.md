# dev-ai-tools

개발용 AI 도구(Claude Code, Codex) 설정을 통합 관리하는 메타 저장소.

두 도구의 설정을 git submodule로 묶어 한 번에 clone·버전 관리하고, 각 도구가 실제로 읽는 홈 경로와 동기화한다.

## 구성

- **[claude-config/](./claude-config)** — Claude Code 설정(rules·skills·settings)을 관리하고 `~/.claude/`와 동기화한다.
- **[codex-config/](./codex-config)** — Codex 설정(skills·전역 지시문·공통 `config.toml` baseline)을 관리한다. 전역 지시문·config는 `~/.codex/`, user skill은 공식 경로인 `~/.agents/skills/`와 동기화한다.

각 submodule의 디렉터리 구조·스크립트·사용법 등 상세는 **해당 repo의 README**에 있다([claude-config/README.md](./claude-config/README.md), [codex-config/README.md](./codex-config/README.md)). 이 문서는 메타 repo 전체에 공통인 내용만 다룬다.

## 원격 저장소 (여러 곳에 미러링 가능)

이 저장소는 부모·submodule을 **여러 원격에 동시에 미러링**할 수 있도록 구성돼 있다. 각 원격은 같은 내용의 거울(mirror)이다.

핵심은 `.gitmodules`가 submodule 주소를 **상대 경로(`../claude-config.git`)** 로만 적어 둔다는 점이다. 그래서 git이 "부모 repo를 어느 원격에서 받아왔는지"에 맞춰 submodule 주소를 자동 계산한다 — 호스트별 주소를 따로 적을 필요가 없다.

> 여러 원격에 미러링한 경우, 변경을 공유하려면 **부모·submodule 모두 각 원격에 push**해야 거울이 어긋나지 않는다.

## 클론 및 초기화

```bash
git clone --recurse-submodules <parent-repo-url> dev-ai-tools
```

`--recurse-submodules` 없이 이미 클론한 경우:

```bash
git submodule update --init --recursive
```

### 원격을 추가로 연결 (선택)

부모와 각 submodule에 같은 이름의 remote를 추가한다(관례상 처음 클론한 곳을 `origin`, 추가하는 곳을 `upstream`). submodule 주소는 부모와 같은 상대 위치(`../<name>.git`)를 따른다.

```bash
git remote add upstream <parent-repo-url>
git -C claude-config remote add upstream <claude-config-url>
git -C codex-config  remote add upstream <codex-config-url>

# push는 submodule을 먼저, 그다음 부모
git -C claude-config push origin main && git -C claude-config push upstream main
git -C codex-config  push origin main && git -C codex-config  push upstream main
git push origin main && git push upstream main
```

## 작업 흐름

두 도구 모두 "repo에서 편집 → diff 확인 → 필요 시 홈으로 sync" 흐름을 따른다(도구별 구체 명령은 각 repo의 README 참고). 단, 설정 파일 적용 방식은 다르다: Claude `settings.json`은 JSON deep-merge, Codex `config.toml`은 현재 홈 파일 + repo baseline + local override를 TOML merge한다.

1. submodule의 `home/` 하위(rules·skills·전역 설정·공통 config baseline)를 수정한다.
2. `*-diff-with-home`으로 실제 홈 대상과의 차이를 확인해 공유한다(Codex skill 대상은 `~/.agents/skills/`).
3. **사용자 명시 지시가 있을 때만** `*-sync-to-home`으로 홈에 반영한다.
4. submodule에서 먼저 commit → 부모에서 submodule 포인터를 commit한다.

## Submodule 업데이트

```bash
# 각 submodule에서 작업·push 후, 부모에서 포인터 갱신
git add claude-config codex-config
git commit -m "update submodules"

# 원격 최신 변경을 가져올 때
git submodule update --remote --merge
```

## 설계 의도

- **공통 패턴 통합** — 두 도구 모두 "repo에서 편집 → diff 확인 → 필요 시 홈으로 sync → 커밋" 흐름이 같아, 한 메타 저장소에서 함께 관리한다. 홈 설정 파일은 도구별 포맷에 맞게 merge한다.
- **도구별 독립성 유지** — 각 설정은 독립 submodule이라 도구 단위로 버전 관리·공유가 가능하다.
- **skill 일관성·rule 정합성** — 두 도구가 skill을 공유하므로 한 작업 공간에서 비교·동기화한다(`check-sync-status`). 규칙은 도구별 위치가 달라(claude `home/rules/` ↔ codex `home/AGENTS.md`) 기계 비교 대신 같은 공간에서 함께 관리해 어긋나지 않게 한다.

## 관련 문서

- [docs/skill-sync-status.md](./docs/skill-sync-status.md) — skill이 두 도구(Claude/Codex) 간 동일한지·도구 차이로 변환됐는지·검증됐는지, 그리고 rule의 도구 간 배치(claude `home/rules/` ↔ codex `home/AGENTS.md`)를 기록한 동기화 현황.
- [docs/codex-config-from-claude.md](./docs/codex-config-from-claude.md) — codex-config가 claude-config에서 무엇을 가져오고/제외했는지와 설정·skill 변환 정책(두 submodule 관계는 부모에서 설명).
- `./check-sync-status` — 두 submodule의 skill 쌍을 기계 비교한다. IDENTICAL 항목은 byte 동일성, DIFFERS 항목은 도구명·prefix·홈 경로·호출 문법을 정규화한 semantic 동일성을 검증하며 한쪽 누락과 명시 호출 정책 누락도 실패 처리한다. 규칙은 도구별 위치가 달라(claude `home/rules/` ↔ codex `home/AGENTS.md`) 비교 대상이 아니다.
