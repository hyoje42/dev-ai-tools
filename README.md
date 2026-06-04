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

## 클론 및 초기화

submodule을 함께 가져오려면 `--recurse-submodules` 옵션을 사용한다.

```bash
git clone --recurse-submodules <repo-url> dev-ai-tools
```

이미 클론한 경우:

```bash
git submodule update --init --recursive
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
