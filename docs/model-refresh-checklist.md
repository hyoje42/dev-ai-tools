# 새 모델·하네스 점검 지침

새 모델을 기본값으로 쓰기 시작하거나, Claude Code·Codex 버전이 올라가 하네스가 주입하는 지침과 도구 구성이 달라졌을 때 이 repo가 관리하는 전역 지침과 skill을 다시 점검하는 절차다. 에이전트에게 "새 모델 점검"을 맡길 때는 이 문서를 읽고 따르게 한다. 문서 끝의 [점검 기록](#점검-기록)에는 점검마다 확인한 사실과 수정 후보를 남긴다.

## 원칙

- **모델별로 지침을 나누지 않는다.** Claude Code의 rules는 `paths:` 조건만 지원하고 output style은 전역 하나라서, 모델별 분기는 훅 같은 우회 장치를 새로 유지해야 한다. 사본도 모델 수와 도구 수를 곱한 만큼 늘어난다.
- **새 모델을 계기로 빼는 작업을 먼저 한다.** 새 모델에 맞춰 조항을 더하는 것이 아니라, 하네스가 이미 맡은 내용과 더는 필요 없는 보정을 지운다. 지난 두 번의 모델 대응 커밋(`claude-config`의 `8695533`, `0f73919`)도 대부분 빼는 작업이었다.
- **새 조항은 모델 이름 없이, 이유를 붙인 평서문으로 쓴다.** 강조어 없이 이유를 적은 문장은 이전 모델로 돌아가도 그대로 쓸 수 있다.
- **사용자 정책은 점검 대상이 아니다.** 커밋 승인, sync gate, 문서 언어, 한국어 문체처럼 사용자가 정한 규칙은 모델이 바뀌어도 그대로 둔다.
- **모델마다 달라야 하는 값은 프롬프트가 아니라 설정에 둔다.** Claude Code는 `settings.json`의 `modelSettings`에 모델별 effort를 저장한다. `run-loop`의 roster도 역할별 모델을 설정으로 받는다.
- **모델 변화와 하네스 변화를 구분한다.** 도구 구성과 주입 지침은 모델이 아니라 Claude Code 버전에 따라 바뀌는 경우가 많다. 2026-11-13 점검에서 나온 차이도 대부분 버전에서 왔다.

## 점검 시점

- 기본 모델을 새 모델로 바꿨을 때 점검한다.
- 평소에 쓰다가 지침을 과하게 따르거나 무시하는 동작이 반복해서 보일 때 점검한다.
- 변경 이력에 도구 구성, 권한, 시스템 프롬프트와 관련된 항목이 보일 때 점검한다.

## 범위와 순서

전역 층을 먼저 보고 skill을 마지막에 본다. 전역 층은 모든 답변에 적용되므로 영향이 가장 크다.

1. `claude-config/home/output-styles/`: 시스템 프롬프트 층에 들어가므로 하네스 지침과 직접 부딪히는 곳이다.
2. `claude-config/home/rules/`와 `claude-config/home/settings.json`
3. `claude-config/home/skills/`

Codex는 같은 절차를 `codex-config/home/AGENTS.md`, `config.toml`의 `developer_instructions`, `home/skills/`에 적용한다. Claude에서 고친 내용을 옮길 때는 [codex-config/skill-authoring.md](../codex-config/skill-authoring.md)의 변환 규칙을 따른다.

## 절차

### 1. 기준 사실을 모은다

추측으로 조항을 고치지 않도록, 편집 전에 아래 사실을 먼저 확인한다.

- 이 문서의 점검 기록과 `.handoffs/`에 있는 이전 점검 작업 기록을 먼저 읽는다. 이미 검토해서 남기기로 한 조항을 다시 후보로 올리지 않기 위해서다.
- 버전과 모델을 확인한다: `claude --version`
- 모델별 기본 도구 구성을 확인한다. 아래 명령은 모델마다 짧은 요청을 한 번 보낸다.

  ```bash
  for m in fable opus sonnet; do
    claude -p --model "$m" --no-session-persistence --permission-mode dontAsk \
      --output-format stream-json --verbose "Reply with the single word ok." \
      | jq -c 'select(.type=="system" and .subtype=="init") | {model, tools}'
  done
  ```

- `~/.claude/cache/changelog.md`에서 직전 점검 버전 이후의 항목을 읽는다. 도구, 권한, 설정 키, skill frontmatter와 관련된 변경을 찾는다.
- `claude-config/claude-diff-with-home`으로 추적본과 `~/.claude`가 같은지 확인한다. 다르면 지금 적용 중인 지침이 점검 대상과 다르다는 뜻이다.
- 대상 모델로 연 세션에서, 하네스가 준 지침 가운데 rules와 output style에 겹치거나 어긋나는 내용이 있는지 확인한다. 세션이 자기 지침을 읽고 보고한 내용은 관찰일 뿐이다. `claude-config/README.md`에 사실로 적을 내용은 이전처럼 바이너리 분석으로 확인한다.
- 바이너리 분석은 `readlink -f "$(command -v claude)"`로 찾은 실행 파일에서 세 가지를 본다. 첫째는 내장 모델 목록에 적힌 모델별 `capabilities`(`lean_prompt`, `*_prompt_bundle`, `turn_updates` 등)다. 둘째는 `# Communicating with the user` 문자열 근처에서 답변 형식의 갈래를 고르는 코드다. 셋째는 서버가 내려 준 값이 그 선택을 바꾸는지이며, `~/.claude.json`의 `clientDataCacheSlots`와 `cachedGrowthBookFeatures`에서 확인한다. 함수 이름은 버전마다 바뀌므로 문자열로 찾는다.
- 사용자가 `/doctor prompt-audit`을 실행한다. 이전 모델에 맞춰 쓴 표현을 CLAUDE.md, skill, agent, command에서 찾아 주는 Claude Code 명령이며, 에이전트가 직접 실행할 수 없다.

### 2. 조항을 분류한다

조항마다 무엇에 기대고 있는지를 보고 아래 셋 가운데 하나로 나눈다.

| 종류 | 예 | 할 일 |
|---|---|---|
| 사용자 정책 | 커밋 승인, sync gate, 문서 언어, 한국어 문체 | 그대로 둔다. |
| 하네스 보정 | `rg` 사용, 상태 공유 시점, output style의 양보 조항 | 1단계에서 모은 사실과 대조한다. |
| 모델 결점 보정 | 특정 모델이 덜 따르거나 과하게 따라서 넣은 문구 | 새 모델에서 문제가 재현되는지 본다. |

분류가 애매한 조항은 `git log -S'<문구>'`로 그 조항을 넣은 커밋을 찾아 이유를 확인한다.

### 3. 판정한다

- **제거**: 하네스가 같은 내용을 이미 지시하거나, 보정하려던 결점이 새 모델에서 재현되지 않을 때 지운다.
- **수정**: 문구가 모호해서 문자 그대로 따르면 의도와 달라질 때 고친다. 하네스의 기본 동작이 사용자 정책과 어긋날 때는 정책을 이유와 함께 명시한다.
- **유지**: 하네스에 대응하는 지침이 없거나, 그 조항이 하네스 지침의 부작용을 막고 있을 때 그대로 둔다.
- **보류**: 근거가 추측뿐이면 고치지 않고 점검 기록에 후보로 남긴다. 실제 사용 중에 문제가 재현되면 그때 고친다.

### 4. skill을 점검한다

- skill이 지정하는 도구 이름이 1단계에서 확인한 도구 구성에 있는지 확인한다. 헤드리스 실행 명령은 기본 도구 구성과 비교하지 말고, skill에 적힌 플래그 그대로 실행해서 init 이벤트의 `tools`를 본다. `--tools`와 `--allowedTools`에 적은 이름에 따라 도구 구성이 달라지기 때문이다.
- 강조어를 찾는다: `rg -n '\b(NEVER|MUST|ALWAYS|IMPORTANT|CRITICAL)\b' claude-config/home/skills -g '*.md'`
- 멈추고 사용자를 기다리는 단계에 이유가 붙어 있는지 확인한다. 최근 하네스는 작업을 중간에 멈추지 말라고 지시하므로, 이유 없는 멈춤 조건은 무시될 수 있다.
- subagent 실행과 effort 지정을 요구하는 문구가 Agent 도구 설명이 허용하는 조건과 맞는지 확인한다.
- 본문에 모델 이름이나 모델 ID가 고정돼 있는지 확인한다.
- skill이 가리키는 명령, 다른 skill, 스크립트 경로가 실제로 있는지 확인한다.

### 5. 적용하고 기록한다

- 편집하기 전에 수정 목록을 사용자에게 보여 주고, 사용자가 고른 항목만 고친다.
- `home/`을 고친 뒤에는 `*-diff-with-home`으로 차이를 확인해 공유하는 데까지만 한다. sync와 commit은 [AGENTS.md](../AGENTS.md)의 게이트를 따른다. 한쪽 submodule만 고쳤다면 `./check-sync-status`와 [skill-sync-status.md](./skill-sync-status.md)로 다른 쪽에 대응하는 항목이 있는지 확인한다.
- 조항을 하나씩 끄고 시나리오를 돌려 검증하지 않는다. 모델 출력은 실행마다 달라서 한두 번의 결과로는 판단하기 어렵다. 지운 뒤 평소처럼 쓰다가 문제가 재현되면 `git revert`로 되돌린다.
- output style과 하네스의 관계에 대해 확인한 사실은 `claude-config/README.md`의 "Output style" 절에 적는다. 이 문서의 점검 기록에는 날짜, 모델, 버전, 확인한 사실, 수정 후보와 처리 상태를 적는다.

## 점검 기록

### 2026-11-13: Fable 5.2, Claude Code 2.1.323

범위는 Claude의 전역 지침(output style, rules, settings)과 skill 9개다. Codex는 점검하지 않았다. 아래 수정 후보는 아직 하나도 적용하지 않았다.

**확인한 사실**

- 기본 도구 구성은 Fable 5.2, Opus 5.5, Sonnet 5.5가 모두 같고, `Grep`과 `Glob`이 없다. `--tools`나 `--allowedTools`에 두 이름을 적으면 제공된다. `run-loop`의 reviewer 명령과 implementer 명령을 각각 헤드리스로 실행해 두 도구가 동작하는 것을 확인했다.
- Fable 5.2 세션이 받은 지침에는 `# Communicating with the user` 같은 답변 형식 절이 보이지 않고, Fable 5.1이 받던 한 줄도 보이지 않는다. 대신 작업을 중간에 멈추지 말고 끝까지 하라는 절이 있다. 세션의 자기 관찰과 바이너리 분석 결과가 일치한다. 내장 모델 목록에서 Fable 5.2는 `lean_prompt`와 `fable_5_2_prompt_bundle`을 갖고, 답변 형식 선택 코드는 이 조합에서 아무 절도 넣지 않는다. Fable 5.1은 `fable_5_1_prompt_bundle`로 한 줄짜리 지침을 받는다.
- rules는 사용자 메시지 쪽 system-reminder로, output style은 시스템 프롬프트의 `# Output Style` 절로 들어온다. `claude-config/README.md`의 설명과 일치한다.
- 추적본과 `~/.claude`의 차이는 `settings.json` 하나다.
- skill 본문에는 대문자 강조어가 없다.

**수정 권장**

1. `claude-config/README.md:100`에 Fable 5.2 설명을 추가한다. 바이너리로 확인한 뒤 적용했다(커밋 전). 같은 행의 Opus 5, Opus 5.5, Fable 5 설명은 2.1.280 기준 그대로이며 2.1.323에서 다시 확인하지 않았다. 2.1.323의 내장 모델 목록만 보면 Fable 5도 답변 형식 절을 받지 않는 조합이므로, 그 문장은 다음 점검에서 다시 확인한다.

**결정 필요**

2. Bash 도구 설명에 기본 브랜치에 있으면 커밋 전에 브랜치를 먼저 만들라는 지침이 있다. 이 repo와 두 submodule은 `main`에 바로 커밋해 왔다. 그 방식을 유지하려면 현재 브랜치에 커밋한다는 문장을 `home/rules/git-commit-guidelines.md`(모든 프로젝트)나 이 repo의 `AGENTS.md`(이 repo만) 가운데 한 곳에 넣어야 한다. 실제로 브랜치를 만드는 동작은 아직 관찰하지 못했다.

**보류한 후보**

3. `claude-sync-to-home`을 실행하면 `~/.claude/settings.json`에서 `model`과 `modelSettings`가 지워진다. `/model`과 `/effort`로 고른 값이 이 두 키에 저장되는데, sync는 baseline과 override를 합친 결과만 쓰기 때문이다. sync 뒤에 두 명령으로 다시 고르면 되므로 중요도는 낮다. 번거로워지면 sync가 두 키를 보존하게 하거나 값을 `local/settings.override.json`에 적는다. 2026-09-11 작업 기록(`.handoffs/260911-claude-instruction-slimming/`)에도 같은 문제가 미결 질문으로 남아 있다.
4. `home/rules/response-format.md`의 "Explain Before Editing"은 output style의 상태 공유 조항(`fluent-korean-concise.md:49`)과 겹친다. 다만 2026-09-11에 "변경 묶음마다 한 번 설명"으로 다듬어서 남긴 조항이므로 사용자 정책으로 보고 그대로 둔다.
5. `fluent-korean-concise.md:95`의 "답변 전 점검" 절은 앞 절의 내용을 다시 묻는 항목이 대부분이다. 2026-09-11에 이 스타일의 내부 반복 정리를 "1~2주 써 본 뒤 결정"으로 미뤘고 그 기간이 지났으므로, 지울지는 사용자가 결정한다.
6. `fluent-korean-concise.md:35`의 "목록은 최대 6개의 불릿"은 목록 하나의 기준인지 답변 전체의 기준인지 적혀 있지 않다. 같은 파일 28행의 "대안을 반드시 남긴다"는 추천 하나를 제시하라는 하네스 지침과 방향이 다르다.
7. `home/skills/git-commit-message/SKILL.md:10`은 `home/rules/git-commit-guidelines.md`와 같은 문장이다. 2026-09-11 점검에서 이 skill의 본문 축소는 기각됐다. 관찰된 실패가 지침을 덜 따라서 생긴 것이었기 때문이며, 그 판단을 따라 그대로 둔다.

**유지**

- `home/rules/tool-usage.md`의 `rg` 조항은 기본 도구 구성에 `Grep`과 `Glob`이 없어서 검색 방법을 정하는 유일한 지침이 됐다.
- `home/rules/response-format.md`의 문서 언어 조항은 `language: korean` 설정이 영어 지침 파일까지 한국어로 쓰게 만드는 것을 막는다.
- output style의 양보 조항과 상태 공유 조항은 그대로 둔다. Fable 5.2에는 대응하는 하네스 지침이 없어 상태 공유 조항이 필요하고, 양보 조항은 답변 형식 절을 받는 Fable 5와 5.1에서 쓰인다.
- `home/skills/run-loop/references/runners.md`의 헤드리스 명령은 그대로 둔다. implementer 명령에는 `--tools`가 없지만 `--allowedTools`에 적힌 `Grep`과 `Glob`이 제공된다. 처음에는 기본 도구 구성만 보고 두 도구가 빠진다고 판단했으나, 명령을 실행해 보니 틀린 판단이었다.
- skill의 멈춤 조건(`inspect-session`, `load-handoff`, `draft-prompt`, `git-commit-message`)에는 이유가 붙어 있다. `run-loop`의 effort 지정 문구는 Agent 도구 설명의 조건과 맞는다.
- `draft-prompt/SKILL.md:35`의 모델 계열 언급은 받는 쪽에 맞춘 형식 안내이며 모델 결점 보정이 아니다. 47행이 가리키는 `/claude-api prompt-audit`은 변경 이력에 있다.
