# 초기 설정 실행 지침

저장소 루트에서 아래 명령을 실행하여 tmux와 Codex 설정을 홈에 동기화한다. 사용자가 이 문서의 실행을 지시하면 홈 동기화를 승인한 것으로 간주한다.

`rsync`, `xclip`, `tmux`, `python3`는 이미 설치되어 있고 서브모듈도 초기화되어 있다고 가정한다. 실행 전에 확인하고, 가정과 다르면 임의로 설치하거나 초기화하지 말고 사용자에게 어떻게 진행할지 묻는다.

동기화 전에 tmux는 `cmp -s .tmux.conf "$HOME/.tmux.conf"`, Codex는 `./codex-config/codex-diff-with-home`으로 차이를 확인한다. 이미 동기화된 항목은 사용자에게 알리고 해당 sync 명령을 생략한다. 차이가 있는 항목만 아래 명령으로 동기화한다.

```bash
printf 'y\n' | ./tmux-sync-to-home
./codex-config/codex-sync-to-home < /dev/null
```

tmux 파일이 이미 동기화되어 있어도 실행 중인 서버에는 아래 명령으로 설정을 다시 반영한다. 서버가 없다면 먼저 `tmux new-session -d -s dev-ai-tools`로 세션을 만든다.

```bash
tmux source-file "$HOME/.tmux.conf"
tmux show-options -gv mouse
tmux show-options -gv mode-keys
```

마우스 설정이 `on`, 복사 모드가 `vi`인지 확인하고 결과를 보고한다. Codex 설정은 새 세션에서 적용되도록 안내한다. 커밋이나 푸시는 하지 않는다.
