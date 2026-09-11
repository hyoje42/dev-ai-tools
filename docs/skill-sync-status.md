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
| git-commit-message | 동일 | **자기완결형**: 커밋 형식·언어·승인 규칙을 SKILL.md에 직접 인라인(전역 규칙과 의도적 중복 — 독립성 우선). 워크플로·한국어 응답 템플릿 포함, claude/codex byte 동일 | ✅ | 2026-09-11 |
| handoff | 변환 | 복구 계약과 template 의미는 동일. 도구 명칭("Claude/Codex instance"), 작성자 prefix `claude-handoff-`↔`codex-handoff-`, 전역 경로 `~/.claude`↔`~/.codex`만 변환. 원 세션 없이도 사용자 의도·결정 근거·검증 상태·불확실성·승인 경계·다음 행동을 복구하는 자기완결형 handoff. 명시 호출 전용 정책은 Claude frontmatter ↔ Codex `agents/openai.yaml`로 변환 | ✅ | 2026-09-11 |
| load-handoff | 변환 | 복원·drift 검증·resume 권한 계약은 동일. 작성자 prefix·도구 명칭 치환(교차 agent 예시는 반대 prefix), 호출 예시 `/load-handoff`↔`$load-handoff`만 변환. 명시 호출 전용 정책은 Claude frontmatter ↔ Codex `agents/openai.yaml`로 변환 | ✅ | 2026-09-11 |
| make-plan | 변환 | 플랜 파일 prefix `claude-plan-`↔`codex-plan-`(교차 agent 예시는 반대 prefix)와 세션 명칭만 변환. 본문에 호출 표기가 없어 그 항목은 차이에 해당하지 않는다. 명시 호출 전용 정책은 Claude frontmatter ↔ Codex `agents/openai.yaml`로 변환 | ✅ | 2026-09-11 |
| review-pr | 변환 | 본문 workflow는 tool-neutral이고 `SKILL.md`의 호출 표기만 `/review-pr`↔`$review-pr`로 다름. `references/` 4종은 byte 동일하며, 생략한 base는 upstream tracking branch가 아니라 remote default branch에서 해석. description 트리거는 PR 초안 요청만 남겼고, 초안은 같은 변경을 리뷰해 병합을 막는 문제부터 보고한 뒤 저장소 PR 템플릿을 우선해 작성 | ✅ | 2026-09-11 |
| review-independently | 변환 | 범용 독립 검토 workflow는 동일. review 저장을 명시적으로 요청받았을 때의 agent prefix·metadata(`claude-review-`/`claude` ↔ `codex-review-`/`codex`)와 tool home 경로만 변환. 본문에 호출 표기가 없어 그 항목은 차이에 해당하지 않는다. 명시 호출 전용 정책은 Claude frontmatter ↔ Codex `agents/openai.yaml`로 변환 | ✅ | 2026-08-21 |

## Rules

2026-06-23부로 Codex의 작업 규칙을 `home/AGENTS.md`로 통합하고 `home/rules/dev-tools/`를 제거했다. 2026-08-20에는 언어 선택과 한국어 문체의 역할을 분리했다(아래 검증 이력 참고). 따라서 rule/config는 더 이상 도구 간 파일 단위로 짝지어지지 않는다:

- **Claude**: `home/rules/*.md` 개별 작업 규칙 + `home/output-styles/fluent-korean-concise.md` 응답 구성·한국어 문체 (`~/.claude/rules/` auto-load + settings의 output style 선택). `fluent-korean.md`는 상류 대조본으로 남아 있고 활성 스타일이 아니다
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
- **현재 상태 (2026-09-12 확인)**: 위 rule 설명과 달리 지금 전역 규칙(Claude `home/rules/git-commit-guidelines.md`, Codex `home/AGENTS.md`의 Git Commit 절)에는 승인 조항만 남아 있다. 영어 작성, conventional format, 50자 제목, AI co-author trailer 금지는 git-commit-message skill 본문에만 있고, 하네스가 붙이는 attribution은 Claude `settings.json`의 `attribution`과 Codex `config.toml`의 `commit_attribution`을 빈 값으로 두어 끈다.

### review-independently — 2026-08-07 통합 / 2026-08-21 전달 가능성 보강

- 기존 `write-review`와 `read-review`를 하나의 `review-independently`로 통합했다. 붙여넣은 agent 응답, 파일·문서, code, diff, current changes, 기술 질문, system state, design decision 등 입력 형식과 무관하게 실제 artifact와 source를 기준으로 독립 조사·검증하고 의견을 제시한다.
- 기본 결과는 chat 응답이다. 사용자가 save, record, document, write 등으로 보존을 명시한 경우에만 `.reviews/YYMMDD-{topic-slug}/{agent}-review-YYYY-MM-DD-HHMMSS.md`를 작성한다. filename prefix와 문서 metadata에 reviewing agent를 모두 기록한다.
- 저장 review는 원 conversation 없이도 이해할 수 있도록 background, scope, evidence, analysis, conclusion, limitations, open questions를 포함한다. code defect나 operational risk에서 유용할 때만 severity와 `file:line` finding을 사용한다.
- 기존 review-response 왕복, accept/dispute/discuss 강제, author standpoint, 자동 review file 탐색을 제거했다. 검토 대상 자체를 수정하지 않고 기존 `.reviews/` 내용도 사용자가 지목한 경우에만 읽는다.
- Codex의 공유 `~/.agents/skills/`에는 repo에서 사라진 이름을 임의 삭제할 수 없으므로, sync script에 이 repo가 관리했던 정확한 퇴역 이름(`read-review`, `write-review`)만 별도 등록했다. diff에서 잔존을 표시하고 sync 시 양쪽 skill 경로를 백업한 뒤 삭제 여부를 묻는다. 다른 공유 skill은 계속 보존한다.
- 2026-08-21에 전달 가능성을 보강했다. 이 skill의 chat 출력을 그대로 복사해 다른 agent에게 넘기고 "이 agent는 이렇게 판단했다, 계획이나 구현을 조정할지 검토하라"고 요청하는 것이 실제 주 용도임을 확인했다. 자기완결성 요구가 저장 review에만 있고 chat 응답에는 없던 공백을 메웠다.
- chat 응답은 조사 범위(확인했으나 문제가 없던 자료와 참고한 외부 source 포함)를 서술하고, 세션 밖으로 복사해도 성립하도록 "the file above" 같은 세션 의존 지시 대신 대상과 artifact를 명시한다.
- finding의 지위를 구분한다. 수정이 필요한 사항과 수신자가 판단해 거절할 수 있는 사항을 구별하고, review가 복종을 요구하는 지시가 아니라 판단을 청하는 의견임을 명시한다.
- 조사·인용 지침의 code 편중을 해소했다. 기존 code 문구는 유지한 채 document·붙여넣은 응답·design decision 갈래를 추가하고, `path:line` 외에 절·인용구·발화 단위로 비파일 대상을 지목할 수 있게 했다. 외부 source 조회는 외부 도구의 현재 동작, version 의존 세부사항, 인용 source의 실제 내용, 변경 가능한 사실에 판단이 걸릴 때의 조건부 의무로 올렸고, `Boundaries`에서 자신의 기억을 evidence 목록에서 배제했다.
- 현재 Claude/Codex 차이는 저장 filename prefix·Reviewing agent metadata(`claude` / `codex`)와 tool home 경로다.

### make-plan — 2026-06-04 드리프트 2건 수정

포팅 과정에서 들어간 실수 2건을 발견·수정했다(둘 다 `codex-config/home/skills/make-plan/SKILL.md`):

1. **중복 문구**: "... referenced by future Codex sessions or other coding agents **or other coding agents**." → 중복 절 제거(Claude 버전은 한 번만 등장).
2. **버전 예시 의미 붕괴**: "다른 에이전트의 플랜을 이어받는" 예시가 `claude-`→`codex-` 일괄치환으로 **소스 파일명까지** 바뀌어 위 줄과 중복·자기모순(`continues from codex`)이 됨. → 소스를 `claude-plan.md` / `(continues from claude)`로 복원해 교차 에이전트 예시의 의미를 되살림.

2026-07-03 재점검에서 Codex 쪽 "other agents" 설명문이 `codex-plan.md`를 예시로 들어 자기 prefix를 다른 agent 예시처럼 보이게 하던 문구를 발견했다. → Claude 문장 구조를 유지하되 Codex 기준의 다른 agent 예시(`claude-plan.md`, `cursor-plan.md`)만 남겨 대칭을 맞췄다.

2026-07-23에는 `make-plan`·`read-review`·`write-review`의 명시 호출 전용 여부를 description 문구에 의존하지 않고 제품별 정책으로 강제했다. Claude는 `SKILL.md` frontmatter의 `disable-model-invocation: true`, Codex는 `agents/openai.yaml`의 `policy.allow_implicit_invocation: false`를 사용한다.

## 검증 이력

### 2026-09-11 — Claude 중심 skill·rule·settings 경량화 (skill은 Codex 동시 반영)

- **범위**: 모델과 Claude Code 하네스가 좋아지면서 근거가 사라졌거나 줄여도 되는 조항을 Claude `home/`에서 찾아 정리했다. 짝 사본이 있는 skill은 Codex `home/skills/`도 같이 고쳤고, rule, settings, output style은 Claude에만 반영했다. 커밋 승인 범위 문장만 예외로, 두 도구의 전역 커밋 규칙과 git-commit-message skill 두 사본에 함께 넣었다(아래 "커밋 규칙 보강"). 단위별 감사, 반박 검증, 완결성 검토를 거친 항목 가운데 반박 검증을 통과했거나 사용자가 고른 항목만 반영했다. 1차 반영은 `claude-config@709d333`·`codex-config@6e538b7`에 들어갔고, 2차 반영은 그 뒤 커밋에 들어간다.
- **skill 1차**: handoff의 Writing Checklist를 지웠다. 체크 항목이 모두 워크플로 단계와 template에 이미 있었다. make-plan은 Rules 절을 지우고, 본문에 없던 세 조항(Context 절 필수, 검증하지 않은 주장을 옮기지 않음, 같은 세션에서도 새 버전 파일 생성)만 워크플로와 Version Management 절로 옮겼다.
- **skill 2차**:
  - handoff·load-handoff를 명시 호출 전용으로 바꿨다. Claude는 `disable-model-invocation: true`, Codex는 `agents/openai.yaml`의 `policy.allow_implicit_invocation: false`를 쓴다. 500자가 넘는 description이 매 세션 skill 목록에 들어갔는데, 사용자가 description을 줄이는 대신 이 방식을 골랐다. 이제 `/handoff`·`/load-handoff`(Codex는 `$handoff`·`$load-handoff`)로 직접 불러야 한다.
  - load-handoff: 존재하지 않는 명령 `/load_handoff`를 `/load-handoff`로 고쳤다. 45줄짜리 Response Template은 `## Report` 절로 바꾸되 2026-07-24에 넣은 복원 항목은 모두 남겼다. 처음에는 한 문단이었고, 비교 검증 뒤 필수 항목 불릿으로 바꿨다(아래 "전후 비교 검증과 보완"). Purpose 절과, 워크플로 1단계의 경로 해석과 겹치던 오류 처리 두 줄을 지웠다. "재개 후 작업이 남으면 새 handoff를 만든다"는 "세션을 마칠 때 제안하고, 사용자가 요청할 때만 작성한다"로 바꿨다.
  - handoff: 지식 구분 단계를 의도 복구 단계의 한 항목으로 합치고 단계 번호를 다시 매겼다. Best Practices에서 다른 항목과 겹치는 "Be operational"과 "Be explicit about uncertainty"를 지웠다. 2026-07-24 복구 계약(template 필드, 줄 수보다 복구 가능성 우선, Resume Checklist)은 그대로 둔다.
  - make-plan: 8단계 워크플로를 3단계(위치 결정, 근거 확보, 작성과 보고)로 줄였다. 탐색이 얕아지지 않도록 코드 탐색과 참조 자료 검증 문장은 남겼다.
  - review-pr: 전면 재작성하지 않고 네 곳만 고쳤다. description에서 리뷰 전용 트리거 4개('review this PR' 등)를 빼고 PR 초안 트리거만 남겼다. 초안 전 리뷰는 유지하되 발견 사항을 병합을 막는 문제부터 간결하게 보고하고, 초안을 쓸 때 빌드·테스트는 스스로 돌리지 않는다(아래 "전후 비교 검증과 보완"). 저장소 PR 템플릿이나 `CONTRIBUTING.md`가 있으면 그것을 따르고, 기본 `references/pr-body-template.md`는 Summary·Changed files·Testing·Risk 4개 절로 줄였다. When Not To Use에서 "항상 리뷰부터 한다" 항목은 지웠다.
- **review-pr 판단 근거**: Codex에서 review-pr를 실제로 부른 5건 중 4건이 PR 초안 요청이었고, 사용자는 더 짧게 써 달라는 요청(7번 중 6번), 변경 파일을 넣어 달라는 요청(3번), 저장소 PR 템플릿 사용을 반복했다. Claude 쪽 리뷰 요청은 내장 `/code-review`와 겹친다. Codex에는 내장 리뷰 명령이 없지만, description 트리거는 암묵 호출에만 영향을 주므로 `$review-pr`로 리뷰를 직접 부를 수 있어 두 사본을 같은 의미로 유지했다. 리뷰 단계를 빼고 초안 전용 skill로 재작성하는 안은 사용자가 리뷰 후 초안 흐름의 가치를 들어 기각했다.
- **전후 비교 검증과 보완**: 축소한 load-handoff와 review-pr가 결과물 품질을 떨어뜨리지 않는지 옛 버전과 새 버전을 같은 입력으로 비교했다. 결론은 두 skill 모두 새 버전을 유지하는 것이고, 되돌린 부분은 없다.
  - 조건: `claude-opus-5`로 읽기 전용 도구만 허용하고 세션 기록을 남기지 않았다. load-handoff는 evofit 저장소에서 버전마다 3회 실행했다. review-pr는 병합 전으로 되돌린 복제본에서 실행했다. voice-exec-app `feat/integrate-skill-shortcuts`와 tinicore-bootstrap `fix/triggered-jobs-next-run-at`(PR 템플릿 있음)은 버전마다 2회, self-evolving-plugin `fix/codex-hook-output-schema`는 버전마다 1회다.
  - load-handoff 결과: 드리프트 탐지와 "다음 행동을 실행하지 않음"은 두 버전이 같았다. 새 버전 응답은 33~47줄로, 이모지와 영어 헤더를 쓰던 옛 버전(58~82줄)보다 읽기 쉬웠다. 다만 새 버전은 3회 모두 성공 기준과 완료·진행·대기 구분을 빠뜨렸고, 1회는 handoff 파일 경로도 빠뜨렸다.
  - review-pr 결과: voice-exec-app의 실제 High 버그를 새 버전은 2회 모두 High로 찾았다. 저장소 PR 템플릿은 옛 버전이 2회 모두 무시했고, 새 버전은 2회 모두 따랐다. 초안 길이는 옛 82~111줄(7개 절)에서 새 62~75줄(4개 절)로 줄었다. 요청 없는 빌드·테스트 시도는 옛 버전 5회 중 3회, 새 버전 5회 중 1회였다. 초안을 쓸 때 리뷰가 얕아지는 문제는 없었고, 새 버전도 High부터 Low까지 모두 보고했다. 회당 비용은 옛 $1.7~2.5, 새 $1.7~1.8이다.
  - 한계: 실행 횟수가 적고, Codex에서는 실행하지 않았다.
  - 보완 1 (load-handoff `## Report`, 두 사본): 한 문단으로 나열하던 항목을 필수 항목 불릿 8개로 바꿨다. 항목은 작업과 handoff 파일 경로, 기록 상태와 현재 상태·드리프트, 성공 기준, 금지 사항과 승인 조건, 채택·기각 결정, 완료·진행·대기로 나눈 진행 상황, 검증하지 않은 주장, 첫 다음 행동과 기대 결과·중단 조건이다. 배치는 자유지만 항목을 빼면 안 되고, 해당하지 않으면 "none"으로 적는다. 문단 형태에서 같은 항목이 반복해서 빠졌기 때문이다.
  - 보완 2 (review-pr Preconditions, 두 사본): "사용자나 저장소 PR 템플릿·체크리스트가 요구하는 검사는 실행한다"는 문구를 바꿨다. 이제 초안을 쓸 때는 빌드나 테스트를 스스로 돌리지 않는다. 사용자나 템플릿·체크리스트가 요구했는데 실행하지 않은 검사가 있으면, 초안을 보여 준 뒤 목록으로 알리고 돌릴지 사용자에게 묻는다. 비교 검증에서 새 버전이 tinicore-bootstrap의 체크리스트 문구를 근거로 요청 없이 `cargo fmt/test`를 시도했고, 사용자는 "사용자가 요청 안하면 안하는거는 맞는데 사용자에게 물어보라해 다 정리하고 나서"라고 결정했다.
  - 보완 3 (review-pr Preconditions 첫 불릿, 두 사본): "report only merge-blocking issues in a few lines"를 "report the findings concisely with merge-blocking issues first (or say none were found)"로 바꾸고, Risk 절에 넘기는 대상도 "remaining risk"에서 "risks that matter for merging or rollout"로 바꿨다. 비교 검증에서 새 버전은 "병합을 막는 문제만" 지시를 따르지 않고 High부터 Low까지 모두 보고했는데, 오히려 실제 High 버그를 2회 모두 찾는 좋은 결과가 나왔다. 사용자도 리뷰하면서 초안을 쓰는 방식을 원한다. 지시와 실제 동작이 어긋난 채로 두면 나중에 모델이 지시를 글자 그대로 따를 때 리뷰가 좁아질 수 있으므로, 지시를 원하는 동작에 맞췄다.
- **skill 변환 판단**: 모든 skill 변경을 두 사본에 같은 의미로 반영했다. 남은 차이는 기존 변환 항목(도구 명칭, 작성자 prefix, 홈 경로, 호출 표기)과 명시 호출 정책 파일뿐이다. 표의 make-plan 행에 적혀 있던 호출 표기 변환은 현재 양쪽 본문에 호출 표기가 없어 뺐다.
- **Claude rule (Codex 미적용)**:
  - `tool-usage.md`의 "Prefer Relative Paths"를 지웠다. Claude Code 2.1.268의 Read·Edit·Write 스키마가 절대 경로를 요구하고, 작업 중에도 `cd` 뒤 상대 경로 Read가 실패했다.
  - `response-format.md`: 상태 공유 빈도는 output style이 정하므로 §1은 "변경 묶음마다 한 번 설명"으로 좁히고 §4(관련 Edit 묶기)는 지웠다. §3의 "기술 용어와 코드 식별자는 원문 유지" 문장도 지웠다. Claude의 `language` 설정과 output style이 같은 내용을 이미 지시한다.
  - `agent-instruction-files.md`(1985B → 1211B)와 `python-guidelines.md`(764B → 369B)는 반복과 예시를 덜어 압축했다. §3 Language는 원문 그대로 두었고, python 규칙에 uv 같은 새 정책은 넣지 않았다.
- **Claude settings (Codex 미적용)**: `home/settings.json`에서 KST `date` 허용 규칙 3개, `skipDangerousModePermissionPrompt`, `env.CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`를 지웠다. 마지막 값을 지우면 남는 환경 변수가 없어 `env` 블록도 함께 없앴다.
  - date 규칙은 정확히 일치하는 명령에만 적용되는데, 세션 기록에 남은 KST date 실행 6건은 모두 `$(...)` 치환이나 `;`·`&&`로 이어진 복합 명령이어서 한 번도 규칙과 맞지 않았다.
  - `skipDangerousModePermissionPrompt`는 사용자 선호가 아니라 bypass 모드 경고를 한 번 수락했을 때 하네스가 사용자 설정에 기록하는 값이 baseline에 커밋된 것이다. 감사 시점의 세션 기록에 bypass 모드 사용은 없었다.
  - `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`는 사용자 결정으로 지웠다. [agent teams 문서](https://code.claude.com/docs/en/agent-teams)에 따르면 이 값이 켜져 있을 때 Claude가 이름을 붙인 subagent는 확인 없이 팀원으로 실행되므로, 요청하지 않아도 팀이 생길 수 있다. 최근 세션 기록의 Agent 호출 8건에는 모두 이름이 없어 팀 기능을 쓴 흔적이 없고, 팀 구성을 돕던 `setup-team-agents`는 2026-06-12에 퇴역했다. 세션 간 메시징(`SendMessage`, `@세션` 언급)은 v2.1.224부터 설정 없이 켜지는 별도 기능이라 이 값과 관계없다([cross-session messaging 문서](https://code.claude.com/docs/en/cross-session-messaging)). 팀이 필요하면 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 claude`로 실행하거나 머신 전용 `local/settings.override.json`에 넣는다.
- **Codex에 옮기지 않은 이유**:
  - Codex `home/AGENTS.md`에는 편집 전 설명, 관련 변경 묶기, 상대 경로 우선 조항이 남아 있다. Claude 쪽 삭제 근거는 Claude Code의 도구 스키마와 output style이었는데, Codex는 승인 프롬프트와 `apply_patch`로 편집하므로 같은 근거가 성립하지 않는다. Codex 조항은 Codex 동작을 기준으로 따로 판단한다.
  - `agent-instruction-files`·`python-guidelines` 압축은 의미를 바꾸지 않았으므로 Codex `AGENTS.md`의 같은 절을 맞춰 고칠 필요가 없다.
  - settings 변경은 Claude 권한 DSL, 하네스 상태값, Claude 전용 agent team 스위치라 Codex에 대응 설정이 없다. Codex의 승인 규칙은 sync가 보존하는 `~/.codex/rules/default.rules`에 있다.
  - response-format §3의 문장을 지운 결과, Claude §3과 Codex `AGENTS.md`의 문서 언어 조항은 같은 문장이 되었다.
- **`check-sync-status`**: `EXPLICIT_ONLY`에 handoff·load-handoff를 추가해 명시 호출 정책 검사 대상이 4개(handoff, load-handoff, make-plan, review-independently)가 되었다. 옛 표기 `/load_handoff`만 따로 치환하던 sed를 지우고, `/load-handoff`를 다른 slash 호출 표기와 같은 규칙으로 정규화한다.
- **커밋 규칙 보강 (Claude·Codex 공통)**: 경량화와 별도로 커밋 관련 변경 2건을 넣었다.
  - "An approval covers only the proposal it answers; after further edits, propose again even if the user earlier said to commit and push." 문장을 네 곳의 기존 승인 규칙 끝에 똑같이 붙였다. 전역 규칙 두 곳은 Claude `home/rules/git-commit-guidelines.md`와 Codex `home/AGENTS.md`의 Git Commit 조항이고, skill 두 곳은 git-commit-message `SKILL.md` Commit Rules의 첫 항목(Claude·Codex 사본)이다.
  - 도입 근거: evofit 세션 `69c04bc5`에서 모델이 커밋 제안에 대한 "커밋하고 push 해" 승인으로 git-commit-message skill을 써서 커밋했다. 그 직후 다른 작업(표 수정안)에 대한 "좋아 반영해"를 근거로, 새 커밋 요청 없이 커밋과 push를 한 번 더 실행했다. 당시에도 규칙, skill, 하네스에 "한 맥락의 승인은 다음 맥락으로 이어지지 않는다"는 문장이 있었으므로, 승인이 어느 제안에 묶이는지를 커밋 조항에 직접 적었다.
  - 배치 판단: 문장이 특정 도구 기능에 의존하지 않아 변환 없이 두 도구에 넣었다. skill에도 넣은 이유는 2026-06-23 결정에 있다. 그 결정에 따라 skill은 단독으로 써도 동작하도록 승인 규칙을 전역 규칙과 의도적으로 중복해 담고, 두 곳을 수동으로 맞춘다. 승인 범위 문장도 그 승인 규칙의 일부다. evofit 사례가 이 skill로 커밋한 직후에 일어났으므로, skill 본문에 있어야 실제 효과도 난다. 매 세션 로드되는 Claude 규칙은 305B에서 439B로 늘었다.
  - git-commit-message의 Workflow 2단계(목록 작성)와 9단계(커밋 직전 재확인)에서 `git status --short`를 `git status --short --branch`로 바꿨다. Claude Code 2.1.268의 Bash 도구 설명에 "If on the default branch, branch first."가 들어왔는데, `--short`만으로는 현재 branch와 detached HEAD가 보이지 않는다. 이 작업을 시작할 때 두 submodule이 detached 상태였던 것이 실제 사례다. 요청 없는 branch 생성은 관찰되지 않았으므로 branch 생성 정책은 skill에 넣지 않았다. 두 사본을 똑같이 바꿔 IDENTICAL 상태를 유지한다.
- **Claude output style (Codex 미적용)**: `home/output-styles/fluent-korean-concise.md`에서 스타일 자신의 기준과 어긋나는 두 곳을 고쳤다.
  - "적용 범위" 절: "코드, 식별자, 명령어, 로그, 커밋 메시지에는 ..."을 "코드와 코드 주석, 식별자, 명령어, 로그, 커밋 메시지에는 프로젝트의 언어와 기존 관례를 적용합니다."로 바꿨다. Claude의 `language` 설정이 넣는 "Use korean for all explanations, comments" 문장은 코드 주석까지 한국어로 쓰라는 뜻으로 읽힐 수 있다. 상류 `fluent-korean`과 Codex `developer_instructions`는 이미 코드 주석을 지침 적용 대상에서 뺀다고 명시하므로, 이번 수정으로 세 문서의 적용 범위가 맞춰졌다.
  - "표현 예시" 절: 모범 예문 "사본에 기재된 문구는 작업이 진행되는 상황을 보여 줍니다."를 "사본에 적힌 문구는 작업이 어디까지 됐는지 보여 줍니다."로 바꿨다. 같은 스타일의 어휘 선택 절이 뜻이 흐려지기 쉬운 표현으로 지정한 '진행하다'를 모범 예문이 쓰고 있었다.
  - Codex 후속 작업: 2026-09-09 항목에 적은 대로, `developer_instructions`를 이 스타일의 "한국어 문장" 절로 바꾸는 작업은 아직 하지 않았다. 옮길 때는 이번 두 곳 수정이 들어간 문구를 기준으로 삼는다.
- **보류**: review-independently 본문 축소, git-commit-message 본문 축소, fluent-korean-concise 내부 반복 정리는 반영하지 않았다. review-independently는 주 용도인 chat 응답 모드의 사용 기록이 없고, git-commit-message의 관찰된 실패는 지침을 덜 따라서 생겼다. fluent-korean-concise 안의 반복 정리(약 10KB를 8.5KB로 줄이는 작업)는 활성화한 지 얼마 되지 않았으므로 1~2주 써 본 뒤 판단한다. 같은 스타일에서 기준과 어긋난 두 곳은 반영을 마쳤다(위 "Claude output style" 불릿).
- **유지 결정**: 사용자가 다음 두 가지는 바꾸지 않기로 정했다. 같은 결과를 보고 다시 고치지 않도록 기록한다.
  - effort 기본값(`modelSettings`): `home/settings.json`과 `local/settings.override.json` 어디에도 넣지 않는다. Claude Code는 `/effort`로 고른 값을 `~/.claude/settings.json`의 `modelSettings`에 저장하지만, `claude-sync-to-home`은 이 파일을 baseline(override가 있으면 baseline과 override를 merge한 결과)으로 다시 쓰므로 sync할 때마다 그 값이 사라진다. 사용자는 이 동작을 알고 그대로 두기로 했다.
  - template 제목 언어: handoff, make-plan, review-independently, review-pr template의 번역 조항은 사용자가 읽을 수 있도록 본문을 한국어로 쓰게 하려는 것이다. 사용자 원문은 "handoff, make-plan, review-independently, review-pr 이 스킬들은 내가 이해를 해야되니깐 한국어로 적으라는 거엿어. 근데 뭐 제목정도 살짝 영어로 써잇는거는 큰 상관없잖아?"이다. 따라서 섹션 제목이 영어로 남은 산출물은 문제로 보지 않고, template도 고치지 않는다.
- **이전 기록 정정**: 2026-09-09 항목은 남긴 `date` 명령 3개를 "스킬이 실행하는" 명령으로 적었다. 실제로 스킬이 쓰는 형식은 `%y%m%d`와 `%Y-%m-%d-%H%M%S` 두 가지이고, `%Y-%m-%d`는 어느 스킬도 쓰지 않았다. 이번에 세 규칙을 모두 지웠다.
- **검증**: `./check-sync-status`가 exit 0으로 끝났다(DIFFERS 5개 정규화 비교와 명시 호출 정책 4개 통과). `bash -n check-sync-status`와 두 submodule의 `git diff --check`에 오류가 없고, `codex-config/tests/sync-home.sh`는 마지막 본문 축소 뒤 다시 실행해 통과했다. review-pr `references/` 4종과 git-commit-message `SKILL.md`는 `cmp`로 byte 동일함을 확인했다. 커밋 규칙 보강, output style 수정, agent team 스위치 삭제, 비교 검증 뒤 보완보다 앞선 변경은 Claude 홈에 이미 반영되어 있어, `claude-diff-with-home`은 `settings.json`, `skills/load-handoff/SKILL.md`, `skills/review-pr/SKILL.md`, `skills/git-commit-message/SKILL.md`, `rules/git-commit-guidelines.md`, `output-styles/fluent-korean-concise.md` 6건을 보고한다. Codex 홈에는 아무것도 반영하지 않아 `codex-diff-with-home`이 수정 7개(`~/.codex/AGENTS.md`와 skill 파일 6개)와 `agents/openai.yaml` 추가 2개를 보고한다.

### 2026-09-09 — Claude output style을 fluent-korean-concise로 교체 (Codex 반영 예정)

- **범위**: Claude의 `home/output-styles/fluent-korean-concise.md` 추가와 `settings.json`의 `outputStyle` 전환. Codex의 `home/config.toml` `developer_instructions`는 건드리지 않았다.
- **내용**: 새 스타일은 상류 `fluent-korean`의 버전 갱신이 아니라 직접 작성한 개정본이다. 상류의 절 구조(상황과 목표, 동작 범위, 문장 단위, 구 단위, 추가 사항, 세부 동작)를 버리고, 응답 구성 절반(결과 우선, 기본 분량, 정보 선택, 형식 선택, 어조, 작업 중 상태 공유, 예외와 우선순위)과 한국어 문장 절반으로 재구성했다. 54줄에서 100줄로 늘었다.
- **변환 판단**: 이번 커밋에서는 Codex를 함께 고치지 않고, **나중에 별도 작업으로 반영한다**. 2026-08-20에 정한 계층 분리는 그대로 유지되며, 그때까지 Codex의 한국어 문체는 상류 `fluent-korean` 기준의 `developer_instructions`가 담당한다. 새 스타일의 응답 구성 절반은 Claude Code의 답변 형식·상태 공유를 전제로 하므로 옮길 대상이 아니고, 반영 범위는 한국어 문장 절반으로 한정한다.
- **후속 작업**: 지금 두 도구의 한국어 문체가 갈라진 것은 실수가 아니라 반영을 미룬 상태다. 후속 작업에서 `developer_instructions`를 `fluent-korean-concise`의 "한국어 문장" 절로 교체하고, 그 결과를 이 문서에 다시 기록한다. 그때까지는 이 항목이 미완 표시 역할을 한다.
- **문서 정합성**: `claude-config/README.md`의 output style 절을 두 스타일 구성으로 고치고, 상류 diff를 수동 반영하는 대상이 `fluent-korean`뿐임을 명시했다. 이 문서의 Rules 절도 활성 스타일 파일명을 바꿨다.
- **검증**: `home/` 하위 skill은 변동이 없어 `./check-sync-status` 결과에 영향이 없다. output style은 여전히 기계 검증 대상이 아니며 수동 대조로 남는다.

### 2026-09-09 — Claude `settings.json` 권한·env 정리 (Codex 미적용)

- **범위**: Claude의 `home/settings.json`만 바꿨다. `permissions.allow`에서 `Read(./**/*)`와 git 읽기 명령 24개를 제거하고, 스킬(`handoff`·`make-plan`·`review-independently`)이 실행하는 `date` 명령 3개만 남겼다. `permissions.deny`는 33개에서 28개로 정리했다: `.envrc`(`.env*`에 포함)·`*.asc`·`*.cer`·`*.crt`·`*.csr`·`/etc/**`를 제거하고, `.git` 규칙 3개를 `./**/.git/**` 하나로 합치고, `~/.git-credentials`·`~/.npmrc`·`~/.netrc`를 추가했다. `env`에서 `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING`과 `MAX_THINKING_TOKENS`를 제거했다.
- **판단 근거**: Claude Code 2.1.266 문서 기준으로, 작업 디렉터리 안의 읽기와 내장 read-only 집합의 git 명령은 어떤 권한 모드에서도 프롬프트가 없어 allow가 동작을 바꾸지 않는다. auto 모드는 작업 디렉터리 읽기를 분류기 없이 승인하므로 비밀 파일을 막는 장치는 deny뿐이라 deny는 중복과 공개 자료(인증서)만 줄였다. 사용자 설정에서 `/etc/**`처럼 슬래시 하나로 시작하는 패턴은 `~/.claude/` 기준으로 해석되어 한 번도 동작한 적이 없었고(`//` 접두사가 파일시스템 루트), 사용자 결정으로 재앵커 대신 삭제했다. 두 env 변수는 Opus 4.6·Sonnet 4.6에만 적용되고 Fable·Sonnet 5·Opus 4.7 이후에는 효력이 없다.
- **변환 판단**: Codex에는 적용하지 않는다. 권한 DSL과 thinking env 변수는 [codex-config-from-claude.md](./codex-config-from-claude.md)에 Claude 전용으로 기록된 항목이고, Codex는 자체 rules·sandbox·approval 정책과 모델 reasoning 설정을 쓴다. 대응 항목이 없으므로 드리프트가 아니다.
- **검증**: `jq empty`와 `claude-config/tests/settings-merge.sh`를 통과했고, `claude-diff-with-home` dry-run에서 예정된 변경만 확인했다. 홈에는 적용하지 않았다. `${VAR}` 확장이 섞인 Bash 명령은 정적 deny 검사를 거치지 않고 auto 모드 분류기로 넘어가 `.git/config` 읽기가 허용되는 것을 확인했는데, 설정으로 막을 수 없는 하네스 동작이라 기록만 남긴다.

### 2026-08-21 — status line 세션 비용 표시 정렬

- **범위**: Claude의 `home/settings.json`·`home/statusline-command.sh`와 Codex의 `home/config.toml`을 대조했다.
- **판단**: Claude는 사용자 명령이 transcript를 읽어 세션 비용을 계산하지만, Codex의 status line은 미리 정의된 항목만 지원한다. 따라서 Claude 구현을 이식하지 않고 Codex 네이티브 `estimated-thread-cost` 항목을 사용한다. 비용 정보가 없는 계정에서는 해당 항목이 자동으로 생략된다.
- **배치**: 두 도구 모두 세션 토큰 합계 뒤, 사용량 제한 앞에 비용을 표시하도록 맞췄다. Claude는 이미 이 순서였으므로 수정하지 않았고, Codex에만 `estimated-thread-cost`를 추가했다.
- **비용 출처**: 이어서 Claude의 `home/statusline-command.sh`가 비용을 읽는 위치를 transcript 재계산에서 status JSON의 `cost.total_cost_usd`로 바꿨다. 재계산 방식은 해당 값이 `--resume` 시 0으로 초기화되던 문제(anthropics/claude-code#13088) 때문에 도입했는데, 지금은 그 값이 세션 재개 후에도 유지되고 transcript에 기록되지 않는 호출(ai-title 재생성 등)까지 포함하므로, 재계산 결과가 약 10% 낮게 측정된다. transcript 합계는 JSON이 값을 보고하지 않을 때만 쓰이는 대체 경로로 남기고 `(!)` 표시를 덧붙였다.
- **이식 여부**: 이 변경은 Codex에 이식하지 않는다. Codex는 네이티브 `estimated-thread-cost` 항목이 비용을 자체적으로 산출하므로, 어떤 출처를 신뢰할지 선택하는 문제 자체가 발생하지 않는다.
- **검증**: `./check-sync-status`, `git diff --check`, `codex-diff-with-home`을 통과했으며, Codex 홈 대상에는 `config.toml` 변경 한 건만 예정돼 있다.

### 2026-08-21 — review-independently 전달 가능성·외부 근거 보강

- **범위**: `review-independently`의 `SKILL.md` 6개 항목(Boundaries 1건, Workflow 3건, Review Format Rules 2건). `references/review-template.md`는 이미 `Inputs`·`Evidence Reviewed`·`Limitations`로 같은 정보를 요구하고 있어 수정하지 않았다.
- **동기**: skill 점검에서 "참고한 자료를 서술하라"는 요구가 저장 review에만 있고 chat 응답에는 없다는 점, 조사·인용 지침이 code에 편중되어 있다는 점을 확인했다. 이후 실제 용도가 이 agent의 검토 의견을 다른 agent에게 전달하는 것임이 드러나면서, chat 응답의 자기완결성이 가장 중요한 공백으로 확정됐다.
- **판단**: 새 skill을 만들지 않았다. 의견 전달은 `handoff`의 작업 상태 이관과 다르고, 대상을 독립적으로 판단하는 `review-independently`가 이미 맞는 skill이다. `Boundaries`의 "this session's prior involvement are not evidence"는 전달 용도에서 오히려 의견의 가치를 지탱하는 핵심 조항이므로 유지했다.
- **결과**: 줄 수 증가 없이 기존 6개 줄만 확장했다(6 insertions, 6 deletions). code 검토 문구를 그대로 보존해 기존 동작에 회귀가 없다.
- **부수 정정**: 표와 항목별 메모에 남아 있던 "호출 표기(`/review-independently` ↔ `$review-independently`) 변환" 기술을 제거했다. 현재 양쪽 `SKILL.md` 본문에 호출 표기가 없어 실제 차이가 아니다.
- **변환**: claude → codex는 frontmatter의 `disable-model-invocation`, 저장 prefix(`claude-review-`/`codex-review-`), tool home 경로만 다르다. 본문 6개 항목은 byte 동일하게 포팅했고 `diff`로 잔여 차이가 이 4종뿐임을 확인했다.

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
