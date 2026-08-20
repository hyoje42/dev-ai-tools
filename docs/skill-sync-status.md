# 스킬·룰 동기화 검증 현황

`claude-config/home/`과 `codex-config/home/`의 같은 항목(skill·rule)이 어떤 상태인지 항목별로 기록한다. 각 항목에 대해 (1) 두 도구 버전이 **동일/변환/재작성** 중 무엇인지, (2) 변환됐다면 **무엇이·왜** 다른지, (3) **검증 상태**를 남긴다. 개별 항목에 속하지 않는 **두 submodule 공통의 정합성 결정**(지시 문서의 규칙 귀속 등)은 아래 "검증 이력"에 남긴다.

**기계 검증**: repo 루트의 `./check-sync-status`가 모든 skill 쌍을 비교한다(rule은 2026-06-23 통합으로 비교 대상에서 제외 — 아래 'Rules' 참고). IDENTICAL 항목은 byte 단위 동일성을, DIFFERS 항목은 self/other 도구 명칭·작성자 prefix·홈 경로·호출 문법을 정규화한 뒤 non-policy 파일의 semantic 동일성을 검증한다. Claude frontmatter와 Codex `agents/openai.yaml`의 명시 호출 전용 정책은 별도로 확인한다. 한쪽에만 존재하거나 아래 표에 등록되지 않은 skill, 예상 상태 불일치, 정규화 후 semantic drift, 명시 호출 정책 누락이 있으면 실패한다. 이 문서는 **DIFFERS 항목의 변환 사유 기록**에 집중한다.

새로 점검하거나 항목이 바뀌면 이 표와 "검증 이력"을 갱신한다.

## 범례

**검증 상태**

| 표기 | 의미 |
|---|---|
| ✅ 검증완료 | 두 버전을 비교·점검했고, 의도된 차이만 존재(또는 동일) |
| ⏳ 진행중 | 아직 점검/정리 중 |
| ⚠️ 드리프트 | 실수로 인한 불일치 발견 — 수정 필요/예정 |

**차이 유형**

| 표기 | 의미 |
|---|---|
| 동일 | 변환 불필요(tool-neutral). 두 파일이 byte 단위로 같음 |
| 변환 | 도구명·경로·기능 차이로 **의도적으로** 다름 |
| 재작성 | 도구 역량 차이로 **구조까지** 다름 |

## Skills

경로: `claude-config/home/skills/<name>/` ↔ `codex-config/home/skills/<name>/`

| Skill | 차이 유형 | 차이 내용 / 사유 | 검증 | 최종 점검 |
|---|---|---|---|---|
| git-commit-message | 동일 | **자기완결형**: 커밋 형식·언어·승인 규칙을 SKILL.md에 직접 인라인(전역 규칙과 의도적 중복 — 독립성 우선). 워크플로·한국어 응답 템플릿 포함, claude/codex byte 동일 | ✅ | 2026-07-23 |
| handoff | 변환 | 복구 계약과 template 의미는 동일. 도구 명칭("Claude/Codex instance"), 작성자 prefix `claude-handoff-`↔`codex-handoff-`, 전역 경로 `~/.claude`↔`~/.codex`만 변환. 원 세션 없이도 사용자 의도·결정 근거·검증 상태·불확실성·승인 경계·다음 행동을 복구하는 자기완결형 handoff | ✅ | 2026-07-24 |
| load-handoff | 변환 | 복원·drift 검증·resume 권한 계약은 동일. 작성자 prefix·도구 명칭 치환(교차 agent 예시는 반대 prefix), 호출 예시 `/load_handoff`↔`$load-handoff`만 변환 | ✅ | 2026-07-24 |
| make-plan | 변환 | 플랜 파일 prefix `claude-plan-`↔`codex-plan-`, 세션 명칭, 호출 표기 `/make-plan`↔`$make-plan`. 명시 호출 전용 정책은 Claude frontmatter ↔ Codex `agents/openai.yaml`로 변환 | ✅ | 2026-07-23 |
| review-pr | 변환 | 본문 workflow는 tool-neutral이고 `SKILL.md`의 호출 표기만 `/review-pr`↔`$review-pr`로 다름. `references/` 4종은 byte 동일하며, 생략한 base는 upstream tracking branch가 아니라 remote default branch에서 해석 | ✅ | 2026-07-23 |
| review-independently | 변환 | 범용 독립 검토 workflow는 동일. review 저장을 명시적으로 요청받았을 때의 agent prefix·metadata(`claude-review-`/`claude` ↔ `codex-review-`/`codex`), tool home 경로, 호출 표기(`/review-independently`↔`$review-independently`)만 변환. 명시 호출 전용 정책은 Claude frontmatter ↔ Codex `agents/openai.yaml`로 변환 | ✅ | 2026-08-07 |

## Rules

2026-06-23부로 Codex의 작업 규칙을 `home/AGENTS.md`로 통합하고 `home/rules/dev-tools/`를 제거했다. 2026-08-20에는 언어 선택과 한국어 문체의 역할을 분리했다(아래 검증 이력 참고). 따라서 rule/config는 더 이상 도구 간 파일 단위로 짝지어지지 않는다:

- **Claude**: `home/rules/*.md` 개별 작업 규칙 + `home/output-styles/fluent-korean.md` 한국어 문체 (`~/.claude/rules/` auto-load + settings의 output style 선택)
- **Codex**: `home/AGENTS.md`의 언어 선택·작업 규칙 + `home/config.toml`의 `developer_instructions` 한국어 문체 (`~/.codex/rules/`는 지시문으로 로드되지 않음: [agents-md 가이드](https://developers.openai.com/codex/guides/agents-md), [#23788](https://github.com/openai/codex/issues/23788))

`check-sync-status`는 계속 **skills만** 비교한다. rule/config 내용의 도구 간 정합성은 Codex의 `home/AGENTS.md`·`developer_instructions`와 Claude의 `home/rules/`·output style을 사람이 직접 대조한다.

## 퇴역·역할 변경 항목

| 항목 | 처리 | 일자 | 비고 |
|---|---|---|---|
| setup-team-agents (skill) | 양쪽 `outdated/skills/`로 이동, sync 제외 | 2026-06-12 | 팀 agent 기능이 도구에 네이티브로 들어오면서 사장. 설계 기록용으로 보관 — 각 submodule `outdated/README.md` 참고 |
| git-commit-message (skill) | 역할 분리 후 자기완결형으로 재확장 | 2026-06-12 / 2026-06-23 | 2026-06-12에는 형식 규칙을 rule로 분리하고 skill을 호출형 워크플로로 얇게 유지했다. 2026-06-23에는 skill 독립성을 위해 형식·언어·승인 규칙을 SKILL.md에 다시 인라인했다(전역 rule과 의도적 중복). 현재 claude/codex skill은 byte 동일 |
| write-review / read-review (skills) | `review-independently` 하나로 통합 | 2026-08-07 | review 작성과 response 처리가 실제로는 같은 독립 조사·검토였으므로 분리된 왕복 protocol을 제거. 기본 chat 응답 + 명시적 요청 시에만 저장하는 단일 skill로 대체 |

## 항목별 메모

### handoff / load-handoff — 2026-06-12 재설계 / 2026-07-24 복구 계약 보강

- 저장 경로를 도구별 `.claude/handoffs/` ↔ `.codex/handoffs/`에서 **공유 `.handoffs/` + 작성자 prefix**(`claude-handoff-*` / `codex-handoff-*`)로 통일. make-plan의 cross-agent 방식과 일관되며, 한 도구가 남긴 handoff를 다른 도구가 자연스럽게 발견할 수 있다.
- handoff 템플릿을 `references/handoff-template.md`로 분리하고, 템플릿을 반복하던 Real Example(~100줄)과 중복 bash 절차를 제거 (289줄 → ~50줄).
- load-handoff에 "handoff 내용을 현재 코드와 대조 검증" 단계 추가 (make-plan의 검증 규칙과 같은 정신).
- 2026-07-24에는 handoff를 단순 대화 요약이 아니라 **원 세션 없이도 복구 가능한 자기완결형 컨텍스트**로 보강했다. 사용자 목표·성공 기준·명시 제약, 결정과 기각안의 근거, 작성 시점의 branch/`HEAD`/worktree와 검증 결과, session-only context, 사실·추론·미결 질문, 승인 경계, 다음 행동의 성공·중단 조건을 구분해 기록한다. 임의의 줄 수보다 복구 가능성을 우선한다.
- load-handoff도 같은 구조를 복원하고 기록 상태와 live state의 drift를 구분한다. 단순 load/inspect 요청은 실행 전에 대기하지만, 명시적인 resume/continue 요청은 handoff 범위 안의 안전한 다음 행동을 승인한 것으로 해석하되 기록된 approval gate와 stop condition은 계속 지킨다.

### git-commit-guidelines(rule) + git-commit-message(skill) — 역할 분리

처음에는 skill을 통째로 rule로 흡수했으나, skill의 원래 가치가 "staged diff 보고 설명+메시지 제안"을 한 번에 부르는 **호출형 단축키**였음이 확인되어 같은 날 역할을 분리했다:

- **rule** (상시 적용): conventional format, 50자 제목, **영어 강제**, AI attribution 금지, 명시적 승인 없이 commit 금지. skill을 안 거치는 커밋에도 적용된다.
- **skill** (호출 시): staged 변경 우선 분석(없으면 전체 working tree를 보고 유연하게 판단) → 파일별 설명 → 메시지 제안 → 승인 대기 워크플로와 한국어 응답 템플릿. 형식 규칙은 rule을 참조해 중복을 없앴다.
- **2026-06-23**: ① Codex 전역 규칙을 `rules/dev-tools/`에서 `AGENTS.md`로 통합(`git-commit-guidelines` 포함). ② 이후 `git-commit-message` skill을 **자기완결형**으로 전환 — 형식·언어·승인 규칙을 SKILL.md에 직접 인라인(claude/codex 동일). 전역 규칙(codex `AGENTS.md` / claude `rules/git-commit-guidelines.md`)은 skill 안 거친 **직접 커밋**용으로 유지. skill 단독 사용·전역 설정 상이 케이스를 위해 독립성을 택한 결정이라 **의도적 중복**이며, 두 사본은 수동으로 동기 유지한다.

### review-independently — 2026-08-07 통합

- 기존 `write-review`와 `read-review`를 하나의 `review-independently`로 통합했다. 붙여넣은 agent 응답, 파일·문서, code, diff, current changes, 기술 질문, system state, design decision 등 입력 형식과 무관하게 실제 artifact와 source를 기준으로 독립 조사·검증하고 의견을 제시한다.
- 기본 결과는 chat 응답이다. 사용자가 save, record, document, write 등으로 보존을 명시한 경우에만 `.reviews/YYMMDD-{topic-slug}/{agent}-review-YYYY-MM-DD-HHMMSS.md`를 작성한다. filename prefix와 문서 metadata에 reviewing agent를 모두 기록한다.
- 저장 review는 원 conversation 없이도 이해할 수 있도록 background, scope, evidence, analysis, conclusion, limitations, open questions를 포함한다. code defect나 operational risk에서 유용할 때만 severity와 `file:line` finding을 사용한다.
- 기존 review-response 왕복, accept/dispute/discuss 강제, author standpoint, 자동 review file 탐색을 제거했다. 검토 대상 자체를 수정하지 않고 기존 `.reviews/` 내용도 사용자가 지목한 경우에만 읽는다.
- Codex의 공유 `~/.agents/skills/`에는 repo에서 사라진 이름을 임의 삭제할 수 없으므로, sync script에 이 repo가 관리했던 정확한 퇴역 이름(`read-review`, `write-review`)만 별도 등록했다. diff에서 잔존을 표시하고 sync 시 양쪽 skill 경로를 백업한 뒤 삭제 여부를 묻는다. 다른 공유 skill은 계속 보존한다.
- 현재 Claude/Codex 차이는 저장 filename prefix·Reviewing agent metadata(`claude` / `codex`), tool home 경로, 호출 표기(`/review-independently` ↔ `$review-independently`)다.

### make-plan — 2026-06-04 드리프트 2건 수정

포팅 과정에서 들어간 실수 2건을 발견·수정했다(둘 다 `codex-config/home/skills/make-plan/SKILL.md`):

1. **중복 문구**: "... referenced by future Codex sessions or other coding agents **or other coding agents**." → 중복 절 제거(Claude 버전은 한 번만 등장).
2. **버전 예시 의미 붕괴**: "다른 에이전트의 플랜을 이어받는" 예시가 `claude-`→`codex-` 일괄치환으로 **소스 파일명까지** 바뀌어 위 줄과 중복·자기모순(`continues from codex`)이 됨. → 소스를 `claude-plan.md` / `(continues from claude)`로 복원해 교차 에이전트 예시의 의미를 되살림.

2026-07-03 재점검에서 Codex 쪽 "other agents" 설명문이 `codex-plan.md`를 예시로 들어 자기 prefix를 다른 agent 예시처럼 보이게 하던 문구를 발견했다. → Claude 문장 구조를 유지하되 Codex 기준의 다른 agent 예시(`claude-plan.md`, `cursor-plan.md`)만 남겨 대칭을 맞췄다.

2026-07-23에는 `make-plan`·`read-review`·`write-review`의 명시 호출 전용 여부를 description 문구에 의존하지 않고 제품별 정책으로 강제했다. Claude는 `SKILL.md` frontmatter의 `disable-model-invocation: true`, Codex는 `agents/openai.yaml`의 `policy.allow_implicit_invocation: false`를 사용한다.

## 검증 이력

### 2026-08-20 — 한국어 언어 선택과 문체 계층 분리

- Claude의 `settings.json`에서 `language = korean`과 `outputStyle = fluent-korean`이 별도 역할을 담당하는 구조를 Codex의 실제 instruction 계층에 맞게 변환했다. Codex의 기본 응답 언어는 `home/AGENTS.md`, 선택된 한국어의 형태론·문체 품질은 `home/config.toml`의 `developer_instructions`가 담당한다.
- Claude output style의 YAML metadata는 옮기지 않고, 한국어 본문과 대조 예시는 보존했다. Codex 쪽에는 출력 언어를 강제하지 않고 사용자 언어 요구·프로젝트별 산출물 관례·코드와 기존 언어를 우선하는 범위 설명을 추가했다.
- 기술 용어 규칙은 더 구체적인 `developer_instructions`에만 남겨 `home/AGENTS.md`와의 중복을 제거했다. 한국어 형태론과 예시의 정밀도를 위해 부모의 영어 기본 원칙에는 language-specific output style 본문만 허용하는 좁은 예외를 기록했다.
- Codex `developer_instructions`는 실제 `developer` role에 추가되는 scalar string이라 project/local config가 같은 키를 정의하면 전체 교체된다. Claude의 named output style과 동일한 기능으로 문서화하지 않았으며, tool-specific 계층 차이는 `docs/codex-config-from-claude.md`에 기록했다.
- `codex-merge-config`의 multiline string round-trip, baseline·override 우선순위, sync 결과 전달을 회귀 테스트로 검증한다. `check-sync-status`는 여전히 skill만 기계 검증하며, 이번 rule/config 변환은 수동 대조 대상으로 유지한다.

### 2026-08-16 — submodule 지시 문서에서 부모 전용 규칙 분리

- 두 submodule `AGENTS.md`의 서두를 "A submodule of `dev-ai-tools`"에서 "standalone repo이며 메타 repo의 submodule로도 임베드될 수 있다"로 바꿨다. 각 repo는 단독 clone으로도 사용되므로, 지시 문서가 특정 부모 repo의 존재를 전제하지 않게 했다.
- 양쪽 `AGENTS.md`에서 **Commit order**(submodule 먼저 → 부모 포인터 bump) 규칙을 제거했다. 이 규칙은 부모가 있을 때만 성립하는 오케스트레이션이며, canonical 위치는 부모 `AGENTS.md`의 work rules와 `README.md`의 작업 흐름이다. 두 submodule에 대칭 적용했으므로 도구 간 드리프트가 아니다.
- codex `skill-authoring.md`의 "동기화 검증" 섹션(부모 루트 `check-sync-status` 사용법 설명)을 제거했다. submodule 문서가 부모 repo에만 존재하는 스크립트를 가리키면 단독 clone에서 깨진 참조가 된다. 해당 설명의 canonical 위치는 부모 `README.md`와 이 문서의 "기계 검증" 문단이다. Claude 쪽에는 대응 섹션이 없어 제거 대상이 없었다(비대칭이 아니라 원래 없던 항목).
- 부모 `AGENTS.md`의 cross-submodule commit gate에 **변환 사유를 이 문서에 기록**하라는 요구를 추가했다. 기존에는 codex `skill-authoring.md`에만 적혀 있어, 위 제거로 요구가 사라지는 것을 막기 위해 부모로 올렸다.
- `home/` 하위는 건드리지 않았으므로 skill 표의 상태·최종 점검일은 변동 없다. `./check-sync-status`도 그대로 통과한다.

### 2026-08-07 — write-review + read-review → review-independently 통합

- 별도 review 작성·응답 단계가 같은 독립 조사·검토 workflow를 중복 표현한다고 판단해 두 skill을 `review-independently` 하나로 대체했다.
- 모든 입력을 같은 증거 중심 workflow로 검토하고 chat에 응답하되, 사용자가 기록을 명시한 경우에만 날짜·주제·agent가 드러나는 `.reviews/` 경로에 self-contained review를 저장하도록 했다.
- 양쪽 submodule의 explicit-invocation-only 정책과 tool-specific agent prefix·home path 변환을 유지하고, `check-sync-status`의 예상 skill 목록과 호출 문법 정규화를 최종 이름에 맞췄다.
- Codex diff/sync가 공유 skill 경로의 다른 항목은 보존하면서 퇴역한 두 이전 이름만 감지·백업·확인 후 정리하도록 migration test를 추가했다.

### 2026-07-24 — handoff 복구 계약 정렬 · semantic drift gate 추가

- Codex에 먼저 보강한 `handoff`·`load-handoff`의 복구 계약을 Claude 형식으로 이식했다. 본문과 template 의미는 같고, 작성자 prefix·도구명·홈 경로·호출 문법만 제품별로 다르다.
- `check-sync-status`가 DIFFERS 항목을 단순히 "서로 다름"으로만 판정하던 사각지대를 제거했다. 이제 `agents/` 정책 파일을 제외한 파일 목록을 맞춘 뒤 self/other 도구 명칭, 작성자 prefix, 홈 경로, slash/`$` 호출 문법을 정규화해 본문·reference의 semantic 동일성을 검사한다.
- `make-plan`의 작성자명 길이에 따른 주석 정렬 공백은 의미 없는 차이로 정규화한다. Claude `disable-model-invocation: true`와 Codex `agents/openai.yaml`은 기존 전용 검사에서 계속 확인한다.
- 전체 7개 skill을 재검증했다. `git-commit-message`는 byte-identical이고, 나머지 6개는 문서화된 변환 후 semantic match이며 한쪽에만 존재하거나 깨진 reference는 없다.

### 2026-07-23 — 명시 호출 정책 강제 · review base 수정 · 검증 gate 강화

- `make-plan`·`read-review`·`write-review`에 제품별 명시 호출 전용 정책을 추가하고, 같은 원칙을 각 submodule의 `AGENTS.md`와 `codex-config/skill-authoring.md`에 기록했다. 자동 호출을 막기 위해 description 문구를 반복하던 부분은 제거했다.
- `review-pr`에서 base를 생략할 때 feature branch의 upstream tracking ref를 base로 잘못 사용할 수 있던 규칙을 수정했다. 이제 대상 branch의 remote가 광고하는 default branch를 우선하며, 없으면 다른 명확한 remote default와 local `main`/`master`/`develop` 순서로 해석한다.
- [Codex 공식 loader](https://github.com/openai/codex/blob/main/codex-rs/core-skills/src/loader.rs)가 `$CODEX_HOME/skills`를 하위 호환용 deprecated 경로, `$HOME/.agents/skills`를 user skill 경로로 명시하는 것을 확인해 sync 대상을 `~/.agents/skills/`로 전환했다. repo 원본은 비교 구조를 유지하기 위해 `codex-config/home/skills/`에 둔다. 공유 경로의 다른 skill과 `~/.codex/skills/.system/`은 관리 대상에서 제외한다.
- `check-sync-status`를 단순 현황 출력에서 검증 gate로 강화했다. 예상 IDENTICAL/DIFFERS 목록, 양쪽 존재 여부, 문서 등록 여부, 명시 호출 정책을 검사하고 드리프트가 있으면 non-zero로 종료한다.
- 전체 skill 7종을 재검증했다. `git-commit-message`는 IDENTICAL, 나머지 6종은 문서화된 변환에 따른 DIFFERS이며 한쪽에만 존재하는 skill은 없다.

### 2026-07-03 — skill 호출 표기 변환 반영 및 전 항목 재검증

- `./check-sync-status` 실행 결과 한쪽에만 존재하는 skill은 없었다. `git-commit-message`는 IDENTICAL, `handoff`·`load-handoff`·`make-plan`·`read-review`·`review-pr`·`write-review`는 DIFFERS로 확인했다.
- 각 DIFFERS 항목을 diff로 대조한 결과 모두 의도된 변환이었다. 공통 패턴은 도구명(Claude/Codex), 저장 prefix(`claude-*`/`codex-*`), 전역 경로(`~/.claude`/`~/.codex`), 호출 표기(Claude slash command `/...` ↔ Codex skill command `$...`) 차이다.
- `review-pr`는 이전 문서에서 byte 동일로 기록돼 있었으나, 현재 Codex skill의 호출 표기가 `$review-pr`로 바뀌어 `SKILL.md`만 의도적으로 다르다. `references/` 4종은 여전히 byte 동일이다.
- rule 쪽도 수동 대조했다. Codex `home/AGENTS.md`는 Claude `home/rules/`의 response-format·tool-usage·python·git-commit·agent-instruction 내용을 단일 파일에 통합한 상태다. response-format §3은 양쪽 모두 "Document Output Language" 규칙으로 갱신되어, 문서·저장 산출물 작성 시 사용자 요청 언어/기존 문서 언어를 따르도록 맞춰졌다.

### 2026-06-23 — agent 지시 rule 추가 · Codex rules 로딩 검증 · dev-tools 통합 제거

- **새 rule**: "agent 지시 파일(AGENTS.md / CLAUDE.md) 작성" 전역 지시 추가. AGENTS.md는 개발 내용 위주로 간결하게(상세는 다른 문서 가리키기), CLAUDE.md는 `@AGENTS.md` import로 단일 소스화. 기본값이며 사용자의 명시적 지시가 우선. 배치: claude `home/rules/agent-instruction-files.md`(영어) + codex `home/AGENTS.md` 섹션(영어, 2026-06-23 영문화).
- **Codex rules 로딩 검증**: 공식 문서 확인 결과 Codex 전역 지시문은 `~/.codex/AGENTS.override.md`/`AGENTS.md` **단일 파일**만 로드하고 `~/.codex/rules/*.md`는 지시문으로 자동 로드하지 않는다(rules-디렉터리 auto-load는 미구현 — [agents-md 가이드](https://developers.openai.com/codex/guides/agents-md), [#23788](https://github.com/openai/codex/issues/23788)).
- **dev-tools 통합 제거**: 위 결론에 따라 `codex-config/home/rules/dev-tools/`(4개)를 삭제하고 Codex 전역 규칙을 `home/AGENTS.md` 단일 소스로 통합. response-format·tool-usage·python은 이미 AGENTS.md 섹션에 있던 죽은 사본이었고, `git-commit-guidelines`는 AGENTS.md 'Git 커밋' 섹션으로 이전해 기존 미적용 갭을 해소. `git-commit-message` skill의 참조 경로도 AGENTS.md로 수정. 당시 response-format의 §4(Plan Mode Output Language)는 Codex에 plan mode가 없어 통합 시 의도적으로 누락(Claude 전용)이었으나, 2026-07-03 현재는 양쪽 모두 문서·저장 산출물 언어 규칙으로 갱신됐다.
- **연관 정리**: codex `README.md`(구조·동기화 위치), `codex-diff-with-home`·`codex-sync-to-home`의 `MANAGED_DIRS`(`rules/dev-tools` 제거), repo 루트 `check-sync-status`(Rules 비교 제거 — 이제 skills만), 이 문서의 'Rules' 섹션을 갱신.
- **git-commit-message 자기완결화**: 사용자 요청으로 commit 규칙(형식·언어·승인)을 양쪽 skill의 SKILL.md에 인라인 → 전역 설정과 무관하게 동작(독립성 우선, 전역 규칙과 의도적 중복). 전역 규칙은 직접 커밋 커버용으로 유지. 이로써 claude/codex `git-commit-message` SKILL.md는 byte 동일이 됨.
- **load-handoff 자기완결화**: skill 독립성 점검 중 발견 — step 2가 `handoff` skill의 `references/handoff-template.md`(외부 skill 파일)를 가리키던 것을 제거하고 표준 handoff 구조(섹션 목록)를 SKILL.md에 직접 서술. claude·codex 동일 적용(짝 결합 의존 해소). 나머지 skill(handoff·make-plan·read-review·review-pr·write-review)은 번들 references가 자기 디렉터리 안에서 해소됨을 확인.
- **남은 작업**: 이미 sync한 머신은 `~/.codex/rules/dev-tools/`가 잔류할 수 있으니 수동 삭제 권장(Codex가 읽지 않아 무해하지만 정리 차원).

### 2026-06-14 — load-review → read-review 이름 변경

- `load-review` skill을 `read-review`로 rename(`write-review`와 대비되는 read/write 짝). 양쪽 submodule(`claude-config`·`codex-config`)의 `home/skills/load-review/` 디렉터리를 `read-review/`로 옮기고(`git mv`), `SKILL.md` frontmatter·heading·invocation(`/read-review`)·description, `references/response-template.md`, `write-review`의 교차 참조, 이 문서의 표·메모를 일괄 갱신.
- 동작·변환 차이는 종전과 동일. 같은 날 `./check-sync-status` 실행 결과 `skills/read-review`는 정상 매칭 DIFFERS 쌍(한쪽에만 존재 없음)이고, claude↔codex diff는 의도된 변환(응답자 prefix `claude-response-`↔`codex-response-`, 예시 review 파일명, meta `Author` 값)뿐임을 확인. 표의 "최종 점검"도 2026-06-14로 갱신.

### 2026-06-12 — 구조 개편 및 기계 검증 도입

- **범위**: skill 7종(setup-team-agents 퇴역, git-commit-message 역할 분리 후 유지, write-review/load-review 추가), rule 4종(git-commit-guidelines 신설 포함).
- **방법**: `./check-sync-status`로 전 쌍 비교. IDENTICAL 3건(review-pr, python-guidelines, git-commit-guidelines), DIFFERS 항목은 diff로 의도된 변환만 존재함을 확인.
- **결과**: 드리프트 0건. DIFFERS 항목(git-commit-message, handoff, load-handoff, load-review, make-plan, write-review, response-format, tool-usage)은 이 문서의 표에 사유를 기록했다.

### 2026-06-04 — 전체 항목 정합성 감사

- **범위**: skill 6종 + rule 3종. 각 항목의 claude/codex 버전을 line-by-line 비교.
- **방법**: 항목별 병렬 감사로 "의도된 변환 vs 실수(drift)"를 구분하고, 발견 항목은 별도로 **적대적 재검증**(intentional 변환이면 기각). `review-pr`의 references 4종도 포함.
- **결과**: 확정 드리프트 **2건**(둘 다 make-plan) 수정, 기각 0건, 나머지 **8건은 동일 또는 의도된 변환**으로 확인. 수정분은 `codex-diff-with-home` 확인 후 `codex-sync-to-home`으로 `~/.codex/`에 반영(사후 diff "No differences found").
- **재현**: `diff claude-config/home/<path> codex-config/home/<path>`. 단, 도구명·경로·기능 치환은 **의도된 변환**이라 버그가 아니다. 버그 신호 = 중복 단어, 절반만 끝난 치환, 역방향 치환(Codex 파일에 남은 "Claude" 지시), 깨진 문법, 의도치 않은 로직 분기.
