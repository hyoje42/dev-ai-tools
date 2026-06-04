# dev-ai-tools

개발용 AI 도구(Claude Code, Codex) 설정을 통합 관리하는 메타 저장소.

두 개의 설정 저장소를 git submodule로 관리하며, 각각의 도구 홈 디렉터리(`~/.claude/`, `~/.codex/`)와 동기화한다.

## 구조

```
dev-ai-tools/
├── claude-config/        # Claude Code 설정 (submodule)
│   ├── CLAUDE.md         # repo 관리용 meta 문서 (sync 대상 X)
│   ├── claude-sync-to-home / claude-diff-with-home
│   └── home/             # 이 폴더가 ~/.claude/로 sync
│       ├── rules/
│       ├── skills/
│       └── settings.json
└── codex-config/         # Codex 설정 (submodule)
    ├── AGENTS.md         # agent용 meta 문서 (sync 대상 X)
    ├── README.md         # 사람용 개요 (sync 대상 X)
    ├── codex-sync-to-home / codex-diff-with-home
    └── home/             # 이 폴더가 ~/.codex/로 sync
        ├── AGENTS.md     # Codex 전역 지시문 (~/.codex/AGENTS.md로 sync)
        ├── rules/dev-tools/
        └── skills/
```

각 submodule의 공통 규칙: **`home/` 하위 구조는 `~/.<tool>/`을 그대로 미러링**한다. 루트의 다른 파일(meta 문서, sync 스크립트, `_backup/` 등)은 sync 대상이 아니다.

- **[claude-config/](./claude-config)** — `home/` 내용을 `~/.claude/`로 동기화. 자세한 내용은 [claude-config/CLAUDE.md](./claude-config/CLAUDE.md) 참고.
- **[codex-config/](./codex-config)** — `home/` 내용을 `~/.codex/`로 동기화. 자세한 내용은 [codex-config/README.md](./codex-config/README.md) 참고.

## 원격 저장소 (여러 곳에 미러링 가능)

이 저장소는 부모·submodule을 **여러 원격에 동시에 미러링**할 수 있도록 구성돼 있다. 각 원격은 같은 내용의 거울(mirror)이다.

핵심은 `.gitmodules`가 submodule 주소를 **상대 경로(`../claude-config.git`)** 로만 적어 둔다는 점이다. 그래서 git이 **"부모 repo를 어느 원격에서 받아왔는지"에 맞춰 submodule 주소를 자동으로 계산**한다. 부모를 어느 원격에서 클론하든, submodule도 같은 원격에서 받아온다 — 호스트별 주소를 따로 적거나 설정할 필요가 없다.

> 여러 원격에 미러링한 경우, 변경을 공유하려면 **부모·submodule 모두 각 원격에 push**해야 거울이 어긋나지 않는다.

## 클론 및 초기화

submodule까지 한 번에 가져오려면 `--recurse-submodules`를 쓴다. **어느 원격 주소로 클론하든** submodule은 같은 원격에서 자동으로 따라온다.

```bash
git clone --recurse-submodules <parent-repo-url> dev-ai-tools
```

`--recurse-submodules` 없이 이미 클론한 경우:

```bash
git submodule update --init --recursive
```

### 원격을 추가로 연결 (선택)

한 작업본에서 다른 원격으로도 push하려면, 부모와 각 submodule에 같은 이름의 remote를 추가한다. 관례상 처음 클론한 곳을 `origin`, 추가하는 곳을 `upstream`으로 둔다. submodule 주소는 부모 주소와 같은 상대 위치(`../<name>.git`)를 따른다.

```bash
git remote add upstream <parent-repo-url>                 # 부모
git -C claude-config remote add upstream <claude-config-url>
git -C codex-config  remote add upstream <codex-config-url>

# push는 각 원격에 (submodule을 먼저, 그다음 부모)
git -C claude-config push origin main && git -C claude-config push upstream main
git -C codex-config  push origin main && git -C codex-config  push upstream main
git push origin main && git push upstream main
```

## 작업 흐름

각 도구의 설정을 수정·동기화하는 절차는 동일한 패턴을 따른다.

### Claude Code

```bash
cd claude-config
# home/rules/, home/skills/, home/settings.json 수정
./claude-diff-with-home    # ~/.claude/와 차이 확인
./claude-sync-to-home      # ~/.claude/에 반영
git add -A && git commit   # 변경 이력 커밋
```

### Codex

```bash
cd codex-config
# home/AGENTS.md, home/rules/dev-tools/, home/skills/ 수정
./codex-diff-with-home     # ~/.codex/와 차이 확인
./codex-sync-to-home       # ~/.codex/에 반영
git add -A && git commit   # 변경 이력 커밋
```

## Submodule 업데이트

각 submodule에서 커밋한 후 부모 repo에도 submodule 포인터 변경을 반영한다.

```bash
# 각 submodule에서 작업 및 push 후
git add claude-config codex-config
git commit -m "update submodules"
```

원격 최신 변경을 가져올 때:

```bash
git submodule update --remote --merge
```

## 설계 의도

- **공통 패턴 통합** — 두 도구 모두 "repo에서 편집 → 홈 디렉터리로 sync → 커밋" 흐름이 동일하므로, 같은 메타 저장소 아래에 두어 한 번에 clone·관리한다.
- **도구별 독립성 유지** — 각 설정은 독립 repo(submodule)로 분리되어 있어 도구 단위로 버전 관리·공유가 가능하다.
- **rules/skills 일관성** — 두 도구가 비슷한 규칙·skill을 공유하므로, 한쪽에서 변경한 내용을 다른 쪽에 옮기기 쉽도록 한 작업 공간에서 비교·동기화한다.

## 관련 문서

- [docs/skill-sync-status.md](./docs/skill-sync-status.md) — skill·rule 항목별로 두 도구(Claude/Codex) 복사본이 동일한지·도구 차이로 변환됐는지(사유 포함)·검증됐는지를 기록한 동기화 검증 현황.
