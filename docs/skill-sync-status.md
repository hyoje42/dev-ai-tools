# 스킬·룰 동기화 검증 현황

`claude-config/home/`과 `codex-config/home/`의 같은 항목(skill·rule)이 어떤 상태인지 항목별로 기록한다. 각 항목에 대해 (1) 두 도구 버전이 **동일/변환/재작성** 중 무엇인지, (2) 변환됐다면 **무엇이·왜** 다른지, (3) **검증 상태**를 남긴다. 개별 항목에 속하지 않는 **두 submodule 공통의 정합성 결정**(지시 문서의 규칙 귀속 등)은 아래 "검증 이력"에 남긴다.

**기계 검증**: repo 루트의 `./check-sync-status`가 모든 skill 쌍을 비교한다(rule은 2026-06-23 통합으로 비교 대상에서 제외 — 아래 'Rules' 참고). IDENTICAL 항목은 byte 단위 동일성을, DIFFERS 항목은 self/other 도구 명칭·작성자 prefix·홈 경로·호출 문법을 정규화한 뒤 non-policy 파일의 semantic 동일성을 검증한다. Claude frontmatter와 Codex `agents/openai.yaml`의 명시 호출 전용 정책은 별도로 확인하며, 단독 skill은 존재하는 쪽만 확인한다. 단독 목록(`EXPECTED_CLAUDE_ONLY`·`EXPECTED_CODEX_ONLY`)에 없는데 한쪽에만 존재하는 skill, 아래 표에 등록되지 않은 skill, 예상 상태 불일치, 정규화 후 semantic drift, 명시 호출 정책 누락이 있으면 실패한다. 이 문서는 **DIFFERS 항목의 변환 사유 기록**에 집중한다.

`EXPECTED_REWRITTEN`에 등록한 재작성 쌍은 정규화 비교 대신 스킬별로 지정한 양쪽 필수 파일을 검사하고, `EXPLICIT_ONLY`에 있으면 호출 정책도 검사한다. 공통 코드를 지정한 쌍은 해당 파일의 byte 동일성도 검사한다. 현재 `session-dashboard`의 `scripts/dashboard.py`와 `assets/index.html`·`dashboard.css`·`dashboard.js`가 대상이며, 도구별 어댑터와 문서는 서로 달라도 된다. 동작의 동등성은 이 검사만으로 보장하지 않으며, 도구별 테스트와 수동 대조 결과를 검증 이력에 기록한다.

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
| 단독 | 한쪽 도구에만 **의도적으로** 둔다. `check-sync-status`의 단독 목록에 등록하고, 짝 비교는 하지 않는다 |

## Skills

경로: `claude-config/home/skills/<name>/` ↔ `codex-config/home/skills/<name>/`

| Skill | 차이 유형 | 차이 내용 / 사유 | 검증 | 최종 점검 |
|---|---|---|---|---|
| draft-prompt | 단독 (Claude) | 아직 시작하지 않은 작업을 세 가지 중 하나로 바꾼다. 맥락이 없는 쪽에 넘길 자기완결형 프롬프트, 이 세션에서 일하기 전의 요구사항 확인, 기존 프롬프트 개선이다. OpenAI `prompt-redrafter`를 포팅하지 않고 새로 설계했으며, 원본에서는 출력 방식(초안 먼저, 질문 1~3개, 코드 블록, 사실 추가 금지)만 가져왔다. 진행 중인 작업의 인계는 handoff가 맡는다. 명시 호출 전용 | ✅ | 2026-10-29 |
| git-commit-message | 동일 | **자기완결형**: 커밋 형식·언어·승인 규칙을 SKILL.md에 직접 인라인(전역 규칙과 의도적 중복 — 독립성 우선). 워크플로·한국어 응답 템플릿 포함, claude/codex byte 동일 | ✅ | 2026-10-21 |
| handoff | 변환 | 복구 계약과 template 의미는 동일. 도구 명칭("Claude/Codex instance"), 작성자 prefix `claude-handoff-`↔`codex-handoff-`, 전역 경로 `~/.claude`↔`~/.codex`만 변환. 원 세션 없이도 사용자 의도·결정 근거·검증 상태·불확실성·승인 경계·다음 행동을 복구하는 자기완결형 handoff. 원 세션 ID를 적는 문장은 두 사본이 같다(`${CLAUDE_SESSION_ID}`를 쓰고, placeholder로 남으면 `CODEX_THREAD_ID`를 읽는다). 명시 호출 전용 정책은 Claude frontmatter ↔ Codex `agents/openai.yaml`로 변환 | ✅ | 2026-10-21 |
| inspect-session | 재작성 | 목표·결정·진행 상태를 복원하는 목적과 `locate`·`overview`·`timeline`·`show`·`grep` 명령은 같다. Claude의 projects transcript·실행 중 이름 레지스트리 대신 Codex의 sessions/archived_sessions rollout(JSONL·gzip)과 session_index 제목을 사용한다. Codex 파서는 중복 발화 제거, call_id 기반 도구 결과 연결, 압축·되돌리기 처리를 별도로 구현하며, 이벤트 번호는 원본 줄 번호다. 실행 여부는 로그만으로 단정하지 않는다. 후속 작업은 두 도구 모두 사용자가 이 세션에서 다음 행동을 지시했으면 drift를 보고한 뒤 진행하고, skill만 호출했으면 제안하고 기다린다(2026-11-11 사용자 결정). 이전 세션의 승인은 승계하지 않는다. load-handoff가 원래 세션을 조회할 때 쓸 수 있도록 두 도구 모두 자동 호출을 허용한다(2026-11-11 사용자 결정). overview는 단어 시작의 주석만 제외하고 제어 키워드·접두어·셸 인터프리터·git 전역 옵션을 처리해 git·sync 명령 후보(`pull` 포함)를 찾는다. 조건식을 보존한 전체 셸 호출과 호출 단위 결과를 보여 주며, 목록은 `--ops`(기본 15개)로 `--files`와 별도로 제한한다. 백그라운드 호출은 “백그라운드 실행”으로 표시하며, 개별 명령의 실행·성공은 단정하지 않는다. subagent 보고의 harness 고정 문구 제거는 Claude만 한다(Codex는 `<subagent_notification>`으로 따로 기록) | ✅ | 2026-11-11 |
| load-handoff | 변환 | 복원·drift 검증 계약은 동일하다. 기본은 다음 행동을 제안하고 기다리며, 사용자가 이 세션에서 계속하라고 하거나 다음 행동을 지시했으면 drift를 보고한 뒤 진행한다. 원래 세션 ID가 있고 빈 곳을 채워야 하면 inspect-session으로 조회한다. 작성자 prefix·도구 명칭 치환(교차 agent 예시는 반대 prefix)만 변환한다. description에서 호출 예시를 지워 호출 표기 차이는 없다. `@` 참조 문장은 두 사본이 같다. 명시 호출 전용 정책은 Claude frontmatter ↔ Codex `agents/openai.yaml`로 변환 | ✅ | 2026-11-11 |
| make-plan | 변환 | 플랜 파일 prefix `claude-plan-`↔`codex-plan-`(교차 agent 예시는 반대 prefix)와 세션 명칭만 변환. 본문에 호출 표기가 없어 그 항목은 차이에 해당하지 않는다. plan mode 문장은 두 사본이 같다(Claude는 전용 계획 파일, Codex는 `<proposed_plan>` 블록이 "plan mode가 요구하는 방식"이다). 명시 호출 전용 정책은 Claude frontmatter ↔ Codex `agents/openai.yaml`로 변환 | ✅ | 2026-10-21 |
| review-pr | 변환 | 본문 workflow는 tool-neutral이고 `SKILL.md`의 호출 표기만 `/review-pr`↔`$review-pr`, "Slash command"↔"Skill command"로 다름. `references/` 4종은 byte 동일. PR 번호·URL과 GitHub에만 있는 브랜치를 읽기 전용 `gh`로 받고, 안 되면 로컬 git, 그것도 안 되면 `gh` 설정을 요청한다. 생략한 base는 upstream tracking branch가 아니라 remote default branch에서 해석. description 트리거는 PR 초안 요청만 남겼고, description만 요청하면 리뷰 보고 없이 가볍게 작성 | ✅ | 2026-10-21 |
| prompt-redrafter | 단독 (Codex) | OpenAI [ChatGPT Enterprise Prompting Guide](https://github.com/openai/openai-cookbook/blob/main/examples/chatgpt/chatgpt_prompt_guide/chatgpt_prompt_guide.md)가 배포하는 [zip](https://cdn.openai.com/cookbook/chatgpt-prompt-guide/prompt-redrafter.zip)의 `SKILL.md`와 byte 단위로 같은 사본이다. 원본 갱신을 그대로 따라가려고 수정하지 않으며, `agents/openai.yaml`도 두지 않아 암묵 호출이 가능하다. draft-prompt와 목적은 비슷하지만 모드 구분, 받는 쪽에 맞춘 형식, 검증 기준을 다루는 구조가 달라 쌍으로 묶지 않는다 | ✅ | 2026-10-29 |
| review-independently | 변환 | 범용 독립 검토 workflow는 동일. review 저장을 명시적으로 요청받았을 때의 agent prefix·metadata(`claude-review-`/`claude` ↔ `codex-review-`/`codex`)와 tool home 경로만 변환. 본문에 호출 표기가 없어 그 항목은 차이에 해당하지 않는다. 명시 호출 전용 정책은 Claude frontmatter ↔ Codex `agents/openai.yaml`로 변환 | ✅ | 2026-10-21 |
| run-loop | 재작성 | 0~6 단계, in/auto 모드, 자연어 역할 설정, verifier 이름, 독립 리뷰와 재작업 한도를 맞췄다. gates/·evidence/ 변경 감지와 스펙 재승인 시 R 초기화에 이어, 승인 대상 전문을 질문 전에 표시하고 미응답 상태로 재개하면 다시 제시하는 개선도 반영했다. Claude Agent·claude-cli는 Codex의 fresh subagent·codex exec로, 호출 정책은 agents/openai.yaml로 변환한다. 질문 도구의 승인 제약, 모델 확인, CLI 샌드박스·JSONL, post.snap 재개 방식은 Codex 계약을 유지한다. 질문 도구를 쓸 수 없으면 최종 응답에 승인 대상과 질문을 함께 싣는다. 보조 스크립트 테스트와 수동 대조는 완료했으며, Codex 전체 루프의 종단 실행은 미검증이다 | ⏳ | 2026-11-12 |
| session-dashboard | 재작성 | 서버와 UI 파일 4개는 byte 동일하게 유지하고, 각 도구의 inspect-session 파서를 사용하는 source.py와 사용 지침은 별도로 변환했다. Codex는 rollout·session_index와 CODEX_THREAD_ID를, Claude는 projects 기록·이름/상태 레지스트리와 SKILL.md의 세션 ID 치환을 사용한다. 실제 실행 여부는 모두 미확인으로 표시하며, 발급된 근거에 연결한 한국어 요약과 localhost·오프라인 HTML을 제공한다. 양쪽 모두 자동·명시 호출을 허용한다. 수집 경합 수정과 도구별 변환을 마쳤으며, 자동 테스트 129개·실제 프로세스 검증 20개 묶음과 스킬 쌍 검사를 통과했다. 최종 독립 검증에서도 S1~S10을 모두 통과했다 | ✅ | 2026-11-13 |

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
- load-handoff도 같은 구조를 복원하고 기록 상태와 live state의 drift를 구분한다. 2026-10-21부터 resume·continue 요청을 받아도 다음 행동을 제안하고 기다리게 했다. 2026-11-11 사용자 결정으로 다시 바꿔, 기본은 제안하고 기다리되 사용자가 이 세션에서 계속하라고 하거나 다음 행동을 지시했으면 drift를 보고한 뒤 진행한다. skill 호출만으로는 시작하지 않는다. handoff에 기록된 승인은 다른 세션에서 받은 것이므로, 승인이 필요한 작업은 다시 묻는다.

### git-commit-guidelines(rule) + git-commit-message(skill) — 역할 분리

처음에는 skill을 통째로 rule로 흡수했으나, skill의 원래 가치가 "staged diff 보고 설명+메시지 제안"을 한 번에 부르는 **호출형 단축키**였음이 확인되어 같은 날 역할을 분리했다:

- **rule** (상시 적용): conventional format, 50자 제목, **영어 강제**, AI attribution 금지, 명시적 승인 없이 commit 금지. skill을 안 거치는 커밋에도 적용된다.
- **skill** (호출 시): staged 변경 우선 분석(없으면 전체 working tree를 보고 유연하게 판단) → 파일별 설명 → 메시지 제안 → 승인 대기 워크플로와 한국어 응답 템플릿. 형식 규칙은 rule을 참조해 중복을 없앴다.
- **2026-06-23**: ① Codex 전역 규칙을 `rules/dev-tools/`에서 `AGENTS.md`로 통합(`git-commit-guidelines` 포함). ② 이후 `git-commit-message` skill을 **자기완결형**으로 전환 — 형식·언어·승인 규칙을 SKILL.md에 직접 인라인(claude/codex 동일). 전역 규칙(codex `AGENTS.md` / claude `rules/git-commit-guidelines.md`)은 skill 안 거친 **직접 커밋**용으로 유지. skill 단독 사용·전역 설정 상이 케이스를 위해 독립성을 택한 결정이라 **의도적 중복**이며, 두 사본은 수동으로 동기 유지한다.
- **현재 상태 (2026-09-12 확인)**: 위 rule 설명과 달리 지금 전역 규칙(Claude `home/rules/git-commit-guidelines.md`, Codex `home/AGENTS.md`의 Git Commit 절)에는 승인 조항만 남아 있다. 영어 작성, conventional format, 50자 제목, AI co-author trailer 금지는 git-commit-message skill 본문에만 있고, 하네스가 붙이는 attribution은 Claude `settings.json`의 `attribution`을 빈 값으로 두어 끈다. Codex는 0.156.0에서 `commit_attribution` 키가 제거되어, 설정이 없어도 trailer를 붙이지 않는다(2026-09-23 확인).

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

### 2026-11-14 — session-dashboard Claude 어댑터의 프로젝트 판정과 턴 상태 수정

- **발견 경위**: 저장소 사본으로 실제 `~/.claude` 기록을 읽어 보니 합성 기록 테스트가 잡지 못한 결함이 두 개 있었다. 수정 대상은 Claude의 `scripts/source.py`와 `tests/session-dashboard-core.py`, `references/usage.md`이며, 공통 서버·UI 파일 4개는 변경하지 않았다.
- **프로젝트 판정**: Claude는 세션 도중 Bash에서 `cd`하면 레코드의 `cwd`가 따라 바뀐다. 마지막 `cwd`로 판정하던 탓에 부모 저장소에서 시작해 서브모듈로 이동한 세션이 중첩 저장소로 분류되어 빠졌다. 세션이 시작된 첫 `cwd`를 기준으로 바꿨다.
- **턴 상태**: 세션 기록은 대부분 `/exit` 같은 로컬 명령 출력으로 끝나서 `recorded_turn`이 항상 `불명`이었다. 메타 레코드, 컴팩션 요약, 로컬 명령 레코드를 건너뛴 마지막 대화 레코드에서 `stop_reason`을 읽는다. 응답이 기록되지 않은 사용자 요청이나 도구 결과로 끝나면 `불명`을 유지한다.
- **Codex에 옮기지 않은 이유**: Codex 어댑터는 `session_meta`의 고정된 `cwd`와 형제 파서의 `session.status`를 사용하므로 두 결함이 해당하지 않는다.
- **검증**: 회귀 테스트 2개를 추가해 Claude 핵심·서버 테스트가 45개가 되었고 UI 12개와 함께 통과했다. 실제 기록 72시간 범위에서 이동했던 세션 2개가 다시 선택되었고, 선택된 15개 중 이벤트가 없는 3개를 제외한 12개의 턴 상태가 표시되었다. `./check-sync-status`를 통과했다. 홈 동기화·커밋·push는 수행하지 않았다.

### 2026-11-13 — session-dashboard 수집 경합 수정과 Claude 변환 재개 완료

- **구현 순서**: 2026-11-12 초기 구현 이후 남은 수집 경합 수정을 2026-11-13에 재개했다. Codex의 수정과 독립 검증을 먼저 마친 뒤 공통 서버를 Claude에 동일하게 반영하고, 원시 바이트·압축 해제 결과의 해시 증거와 오류 관찰 전달을 Claude 어댑터에 변환해 적용했다.
- **변환 사유**: Codex는 `CODEX_HOME` 또는 `~/.codex`의 sessions/archived_sessions rollout(JSONL·gzip)과 `session_index.jsonl` 제목을 사용하며, 설치된 스킬 경로는 `~/.agents/skills`다. Claude는 `CLAUDE_CONFIG_DIR` 또는 `~/.claude`의 `projects` 기록과 크기를 제한해 읽은 `sessions/*.json`의 이름·상태를 사용하며, 스킬 경로는 `~/.claude/skills`다. 각 `scripts/source.py`는 같은 도구의 형제 `inspect-session` 파서에 읽기 전용 임시 스냅샷을 전달한다.
- **자기 세션과 실행 상태**: Codex는 `CODEX_THREAD_ID`를 `--self`로 넘긴다. Claude는 `SKILL.md`의 `${CLAUDE_SESSION_ID}` 치환 결과를 리터럴로 넘기며, 같은 이름의 셸 환경 변수로 대체하지 않는다. 두 도구 모두 실제 실행 여부를 미확인으로 표시한다. Claude 레지스트리 상태는 읽은 시각과 원본 수정 시각이 붙은 관측값이며, PID가 존재해도 현재 실행 중이라고 판정하지 않는다.
- **공통 동작과 호출 정책**: 호출한 에이전트가 발급 시점에 고정된 근거를 읽고 한국어 요약을 직접 작성한 뒤 가져온다. 요약의 근거는 이후 기록과 분리해 보존하며, 브라우저 새로고침은 기록만 갱신한다. localhost 서버와 단일 오프라인 HTML을 지원한다. Codex에만 UI 메타데이터용 `agents/openai.yaml`이 있고 암묵 호출을 막는 정책은 없다. Claude도 자동 호출을 허용하며, 양쪽 모두 명시 호출을 지원한다. `EXPLICIT_ONLY`에는 추가하지 않았다.
- **수집 경합과 변환 범위**: 같은 런타임에서는 체크포인트 조회부터 읽기·관찰 저장까지 `collection.lock`으로 직렬화하고, 서로 다른 런타임은 독립적으로 진행한다. 오래된 관찰은 최신 이력을 덮어쓰지 않는다. Claude의 UUID 부모 연결·분기 이력·레지스트리 이름 갱신을 유지했으며, 정상 재압축과 미완성 gzip 꼬리를 원본 교체로 오판하지 않도록 검증했다.
- **부모 검사**: `EXPECTED_REWRITTEN`에 등록하고 양쪽의 `SKILL.md`, `scripts/dashboard.py`, `scripts/source.py`, `assets/index.html`, `assets/dashboard.css`, `assets/dashboard.js`, `references/usage.md`가 존재하고 비어 있지 않은지 검사한다. 공통 서버인 `scripts/dashboard.py`와 UI 파일 3개는 byte 동일성을 추가로 강제한다. 어댑터·문서·Codex 전용 메타데이터는 의도된 차이로 유지하며, 기존 쌍·정규화·명시 호출 정책 검사는 보존했다.
- **자동 검증**: Codex 핵심·서버 38개·UI 12개·파서 18개, Claude 핵심·서버 43개·UI 12개·파서 6개로 총 129개를 통과했다. 승인된 요약과 대기 입력은 확인된 축소·삭제·교체·되돌리기 이후 원본을 복원하거나 프로세스를 재시작해도 계속 무효이며, 발급 당시 근거는 보존됨을 확인했다. 65개·200개 입력의 원자적 발급, 서로 다른 원본 512개의 스냅샷 용량과 무효화 이력 보존, 변경 없는 기록의 추가 읽기·파싱 방지도 검증했다.
- **독립 검증**: Codex의 S5·S7·S8·S9 집중 검증과 도구별 실제 프로세스 경합 10개씩 총 20개 시나리오 묶음을 통과했다. 잠금을 제거한 변형은 실패하여 경합 검사의 탐지력도 확인했다. 새 검증자의 최종 전체 검증에서도 S1~S10을 모두 통과했으며, 완료를 막는 지적 사항은 없었다. 다른 모델 계열에 의한 검증은 보장하지 않는다. 이 검증 결과는 사용자의 최종 승인이나 홈 적용 승인을 뜻하지 않는다.
- **이전 브라우저 검증**: 2026-11-12에는 온라인·오프라인 브라우저에서 320~390px 화면, 키보드 조작, 근거 조회, 오래된 요약 표시, 주입 내용 처리를 확인했다. 이번 재개에서는 UI 자산을 변경하지 않았으며, 해당 결과는 이전 검증 이력으로 유지한다.
- **부모 검사 검증**: `./check-sync-status`, 공통 파일 드리프트 게이트, JavaScript 구문 검사, 스킬 validator와 diff 검사를 통과했다. 공통 서버·UI 파일 4개의 byte 동일성과 도구별 어댑터의 의도된 차이를 확인했다.
- **최신 홈 차이와 적용 여부**: 현재 두 도구의 홈에는 이전 버전의 스킬 사본이 존재한다. 설치 경위와 시점은 확인되지 않았으며, 이번 재개 작업에서 설치하거나 동기화한 것은 아니다. 최신 `codex-diff-with-home`은 `scripts/dashboard.py` 수정 1개를, `claude-diff-with-home`은 `scripts/dashboard.py`와 `scripts/source.py` 수정 2개를 보고했다. 이번 작업에서는 홈 동기화·커밋·push를 수행하지 않았다.

### 2026-11-12 — 상태 표시줄의 잔여 사용 한도 표시를 도구별로 대조

- **Claude 변경**: `cc0d50a`는 `home/statusline-command.sh`의 5시간·7일 사용 비율 표시를 남은 비율(`100 - used_percentage`)과 `left` 표기로 바꾼다. 정수 백분율로 반올림하고 0~100 범위로 제한한다.
- **Codex 대응과 유지 사유**: `codex-config/home/config.toml`의 `[tui].status_line`에는 이미 내장 항목인 `five-hour-limit`와 `weekly-limit`가 있다. Claude가 직접 계산·출력하는 셸 스크립트와 구현 방식이 다르므로 해당 코드를 이식하지 않고 Codex 설정을 유지한다.
- **검토 범위**: Claude 커밋의 diff와 Codex 대응 설정을 수동 대조했다. 실제 상태 표시줄 화면이나 두 도구의 반올림 결과가 같은지는 이번에 검증하지 않았다.

### 2026-11-12 — run-loop 승인 대상 표시와 대기 중 재개를 Codex에 반영

- **원본과 범위**: Claude의 `bb83280`에서 바뀐 `SKILL.md`와 `references/decision-package.md`를 Codex 사본에 반영했다. 앞선 `fccb7aa`의 동작 변경은 Codex `938f5f7`에 이미 반영되어 있어 유지했다. Claude 파일과 실행 스크립트는 수정하지 않았다.
- **공통 동작**: 질문 전에 스펙의 모든 요구사항·검사 방법, 검토 결과와 컨텍스트 변경안을 항목별로 표시한다. 파일 경로나 ID 범위는 본문을 대신하지 않으며, 질문도 그 자체로 무엇을 판단하는지 알 수 있게 쓴다. `state.md`에 대기 중인 결정과 답변을 기록하고, 미응답 상태로 재개하면 단계 파일에서 판단 자료를 복원하여 다시 보여 준다.
- **Codex 변환 사유**: Claude의 `AskUserQuestion` 호출을 그대로 옮기지 않고, 실제 질문 도구가 해당 종류의 질문을 허용할 때만 사용한다. 도구를 쓸 수 없으면 최종 응답에 판단 자료와 질문을 함께 실어 이전 진행 메시지가 접혀도 내용을 확인할 수 있게 한다. 재개 전에 이미 받은 명시적 답변을 대조하여 유효한 승인을 다시 요구하지 않는다. 기존 `post.snap` 무결성 확인과 명시 호출 정책은 유지한다.
- **검증 결과**: Claude 변경분과 수동 대조를 마쳤으며, `quick_validate.py`, 기존 보조 스크립트 테스트 12개, 로컬 Markdown 참조 14개, `check-sync-status`, `git diff --check`를 통과했다. 질문 화면과 전체 루프의 실제 실행은 검증하지 않았다.
- **홈 적용과 커밋**: `codex-diff-with-home`은 이전 미적용분을 포함해 run-loop 파일 7개와 inspect-session 파일 1개의 차이를 보고했다. 이번 수정은 run-loop 문서 2개이며, 실제 홈 적용과 커밋은 하지 않았다.

### 2026-11-12 — run-loop의 Claude 개선 사항을 Codex에 반영

- **범위**: Codex의 `home/skills/run-loop/`, 보조 스크립트 테스트와 README를 수정했다. Claude 사본과 대조했으며, 이번 변경은 Claude에 이미 적용된 개선 사항의 이식이므로 Claude 파일은 수정하지 않았다.
- **공통 동작**: `--auto`, `verifier`, 자연어 역할 설정을 반영했다. 스펙 승인 단계에서 역할 설정을 바꾸면 이후 실행에 적용하고 자동 모드의 모델 조건을 다시 확인한다. 스펙 재승인 시 R은 0으로 초기화하고, 같은 ID의 슬라이스는 K를 유지한다. `## Limits`, 게이트 명령 전문 기록, 단계 표시와 부산물 제외 규칙도 보완했다.
- **검증 자료 보호**: 세션 임시 경로 대신 작업 폴더의 `gates/`에 스크립트·fixture를, `evidence/`에 검증 자료를 보존한다. 두 폴더의 파일 내용과 실행 권한을 스냅샷에 포함하여 `.loop/`가 Git에서 무시되어도 변경을 감지한다. 구현자에게는 제공받은 게이트 실행만 허용하고 스크립트·fixture 수정은 허용하지 않는다.
- **Codex 변환과 유지한 차이**: fresh subagent, 질문 도구의 승인 제약, `codex exec` 샌드박스·JSONL 처리, 빈 diff·Git 외부 작업 처리와 `logs/<run>.post.snap` 재개 방식을 유지했다. 조율자가 검증 대상 파일을 갱신한 뒤에는 재개 기준 스냅샷도 갱신하도록 명시했다. CLI 검토자는 쓰기가 필요한 검사를 미확인으로 보고하며, native agent는 권한이 허용할 때만 저장소 밖 임시 사본을 쓴다. 부산물 예외는 기존 Codex의 선언된 게이트 산출물로 제한한다.
- **역할 설정**: 사용자 지정 값이나 기본값만 제안하도록 문맥 초안과 개선 단계의 규칙을 맞췄다. 템플릿의 모델 placeholder는 임의 선택을 유도하지 않도록 제거했다. native 실행 기능이 없을 때 CLI로 바꾸는 기존 대체 경로는 유지하되, 사용자가 이미 허용했거나 별도로 승인한 경우에만 실행한다.
- **검증 결과**: 새 회귀 테스트 3개가 기존 코드에서 실패하는 것을 확인한 뒤 수정했다. 내용 변경, 생성·삭제, 실행 권한 변경을 검사하며, 중첩 경로·공백이 있는 파일명·여러 작업 폴더·Git에서 무시된 `.loop/`를 포함한다. 수정 후 기존 테스트를 포함한 12개, `quick_validate.py`, 로컬 Markdown 참조 14개, `bash -n`, `git diff --check`, `check-sync-status`를 통과했다. 실제 모델 CLI 호출과 전체 작업 루프의 종단 실행은 이번에도 검증하지 않았다.
- **홈 적용과 커밋**: `codex-diff-with-home`은 run-loop 파일 7개와 기존 inspect-session 파일 1개의 차이를 보고했다. 실제 홈 적용과 커밋은 하지 않았다.

### 2026-11-11 — run-loop 종단 테스트 뒤 Claude 사본 수정 (Codex 미반영)

- **범위**: 사용자 요청에 따라 Claude의 `home/skills/run-loop/`만 수정했다. Codex 사본은 건드리지 않았으므로 아래 항목이 두 사본 사이의 차이로 남아 있다.
- **호출 방식과 이름**: `역할.항목=값` 인자를 없애고, 역할 설정은 호출 문장이나 스펙 승인 화면에서 말로 요청하게 했다. `reviewer-5b`는 `verifier`로, `--on`과 on 모드는 `--auto`와 auto 모드로 바꿨다. Codex는 여전히 `reviewer-5b`, `--on`, 인자 문법을 쓴다.
- **5-b 변경 묶음과 재개**: 추적 파일 변경이 없거나 커밋이 없는 저장소에서는 빈 diff를 쓰게 했고, 실행 권한 변경을 스냅샷에 넣었으며, 검증을 통과한 시점의 `logs/latest.snap`과 비교해 재개하게 했다. Codex가 먼저 보강한 세 항목을 Claude 방식으로 옮긴 것이다. Codex는 `post.snap`이라는 이름을 쓴다.
- **테스트에서 나온 수정**: 게이트 부산물을 변경 목록에서 빼는 규칙, runner별 읽기 전용 문장, 스펙을 다시 승인하면 R을 다시 세는 규칙, 조율자가 roster 값을 스스로 고르지 않는 규칙, 조율자가 만든 게이트 파일과 판정 증거를 두는 `gates/`·`evidence/` 폴더와 그 해시, `5a-result.md`에 명령 전문 기록, 단계 표시 누락 방지, 템플릿의 `## Limits` 절을 넣었다.
- **검증 결과**: 사용자가 연습 저장소 세 곳에서 `/run-loop`를 직접 입력해 [0]~[6]을 돌렸다. `--auto` 조건 미달 시 in 모드 전환, 말로 한 역할 설정의 해석, claude-cli 검증자(`claude-sonnet-5-5`)와 빈 diff 처리, 계획 리뷰 2라운드 실패 뒤 재리뷰 없는 반영을 확인했다. `tree_snapshot.sh`는 임시 저장소에서 실행 권한·`gates/`·`evidence/` 변경 감지를 확인했고, `./check-sync-status`는 통과했다. 마지막 수정분(`gates/`·`evidence/`, roster 값 규칙, 단계 표시)과 K·R 한도 초과, 재개, auto 모드 정상 경로는 루프로 돌려 보지 않았다.
- **Codex에 남은 일**: 위 항목을 Codex 계약에 맞게 변환해야 한다. 이름과 호출 방식, R 규칙, `gates/`·`evidence/` 해시는 그대로 옮길 수 있고, runner별 읽기 전용 문장은 `codex exec`의 샌드박스 모드에 맞춰 다시 써야 한다.

### 2026-11-11 — inspect-session의 git·sync 명령 누락과 overview 목록 제한 수정

- **범위**: Claude의 미커밋 수정 내용을 참고해 Codex의 `state_command`·`state_shell_call`과 overview를 보완했다. Claude 스크립트·테스트 및 다른 미커밋 변경은 수정하지 않았다.
- **두 사본의 감지 규칙**: `shlex`의 기본 주석 처리를 끄고 `#`으로 시작하는 토큰부터 해당 줄 끝까지만 제외한다. URL fragment와 `a#b` 뒤의 명령을 보존하며, 제어 키워드, 명령 접두어와 옵션, `sudo -u`·`-g`의 값, `bash`·`sh`·`zsh` 뒤의 스크립트 경로를 처리한다. git 전역 옵션은 모두 건너뛰되 `-C`·`-c`·`--git-dir`·`--work-tree`·`--namespace`·`--exec-path`는 값도 함께 건너뛴다. 상태 변경 후보에 `pull`을 추가했다.
- **출력과 Codex 변환**: 두 사본 모두 백그라운드 라벨을 “백그라운드 실행”으로 바꾸고, `--ops`(기본 15개)로 명령 목록을 `--files`와 별도로 제한하며, 생략 문구는 “이전 호출 N개 생략”으로 맞췄다. Codex의 호출 단위 종료 코드 판정은 유지하고, `session_id`와 텍스트 결과를 처리하는 두 경로의 라벨을 바꿨다. `--ops`에는 기존 Codex 인자와 같은 양의 정수 검증을 적용했다.
- **Codex에서 보강한 부분**: 주석 토큰을 만나는 즉시 토큰 읽기를 멈춘다. 전체 줄을 먼저 토큰화하면 주석 안의 닫히지 않은 따옴표 때문에 명령이 누락되는 경우를 회귀 테스트로 확인했다. Claude 사본은 사용자 요청에 따라 수정하지 않았으므로, 이 경계 사례의 보강은 Codex에만 적용했다.
- **검증 결과**: `python3 -B codex-config/tests/inspect-session.py`의 18개 테스트와 `python3 -B claude-config/tests/inspect-session.py`의 6개 테스트가 모두 통과했다. 요청한 명령 감지·주석 제외 사례, 접두어와 옵션 조합, 두 백그라운드 결과 형식, `--ops` 기본값·독립 제한·최신 호출 유지·생략 개수를 확인했다. `./check-sync-status`도 통과했다.
- **홈 적용과 커밋**: 둘 다 하지 않았다. `codex-diff-with-home`은 `~/.agents/skills/inspect-session/scripts/session_digest.py` 한 파일의 변경만 보고했다.

### 2026-11-11 — run-loop의 Codex 이식

- **범위**: Claude의 미커밋 `home/skills/run-loop/`를 원본으로 Codex용 스킬을 추가했다. Claude 원본과 진행 중인 `.gitmodules`·`reference-playbook` 변경은 수정하지 않았다. Codex README와 부모의 쌍 검사·현황 문서도 함께 갱신했다.
- **유지한 동작**: 계획을 먼저 독립 리뷰하고 스펙 승인을 받은 다음, 구현·5-a 검사·5-b 독립 검증·자동 수정을 수행한다. in 모드의 최종 승인, on 모드의 전제 조건, K/R 재시도 상한, 맥락 변경과 커밋의 별도 승인도 유지한다.
- **Codex 변환**: `$ARGUMENTS`·`AskUserQuestion`·`subagent_type`·Claude 모델 별칭을 그대로 옮기지 않았다. 실제 도구 스키마에 따라 `fork_turns="none"` 또는 `fork_context=false`로 검토 대화를 분리하고, 질문 도구가 승인 요청을 지원하지 않으면 최종 응답에서 결정을 요청한다. `disable-model-invocation`은 `agents/openai.yaml`의 `allow_implicit_invocation: false`로 변환했다.
- **CLI·권한**: 로컬 `codex-cli 0.167.0`의 도움말과 OpenAI 공식 [서브에이전트 문서](https://learn.chatgpt.com/docs/agent-configuration/subagents)·[비대화형 실행 문서](https://developers.openai.com/codex/noninteractive)를 확인했다. `codex -a never exec --ephemeral`과 역할별 `read-only`·`workspace-write`를 사용하며, `--json`·`-o`로 결과를 수집한다. 셸 샌드박스가 MCP·앱의 외부 쓰기까지 제한한다고 설명하지 않는다. tmux 표시는 Codex JSONL용으로 재작성했다.
- **Codex에서 보강한 부분**: 새 파일만 생겼을 때 빈 경로 목록으로 전체 diff를 만들지 않도록 했다. 기본 완료 조건, 승인 전 모델 선택 기준, 실행 권한 변경 감지, 하위 Git 저장소별 스냅샷, 구현 전·Git 외부 작업의 재개, 마지막 검증 시점 이후 변경 대조를 명시했다. 원본에도 적용할 수 있는 개선이지만, 이번 요청은 Codex 이식이고 Claude 사본은 별도 미커밋 작업이므로 자동 역이식하지 않았다.
- **검증 결과**: 보조 스크립트 테스트 9개, `quick_validate.py`, 로컬 Markdown 참조 검사, `bash -n`, `git diff --check`, `check-sync-status`를 통과했다. 임시 사본에서 필수 참조 누락과 명시 호출 정책 해제를 검사기가 거부하는 것도 확인했다. 독립 에이전트가 네 가지 사용 시나리오를 검토했고, 빈 경로 diff와 완료 조건 누락을 수정한 뒤 재확인했다. 홈 dry-run에는 run-loop 파일 9개 추가와 기존 inspect-session·load-handoff 차이가 표시됐다. 실제 모델 CLI 호출과 전체 작업 루프의 종단 실행은 검증하지 않았으며, 홈 적용과 커밋도 수행하지 않았다.

### 2026-11-11 — inspect-session 리뷰 후 명령 오탐과 성공 단정 수정

- Codex는 셸 실행 도구만 검사하고, 양쪽 파서는 인용 문자열·주석·heredoc 본문을 명령으로 해석하지 않도록 바꿨다. 조건식을 보존해 전체 호출 결과와 개별 명령의 실행 여부를 구분한다.
- Codex는 종료 코드가 없는 결과를 완료로 간주하지 않으며, Claude는 backgroundTaskId·interrupted 기록을 별도로 표시한다. 도구별 결과 형식의 차이는 유지했다.
- 임시 기록으로 Codex 테스트 15개와 Claude 테스트 4개를 통과했다. 패치·인용 문자열 오탐, 실행되지 않은 조건부 명령, 실행 중인 호출을 포함한다. run-loop는 사용자 요청에 따라 이번 검토에서 제외했다. 홈에는 적용하지 않았다.

### 2026-11-11 — inspect-session 자동 호출 허용, load-handoff 연계와 진행 기준 변경 (양쪽 submodule과 parent)

- **배경**: `/inspect-session`을 처음 실제로 호출해 `f5d2b678` 세션을 복원한 뒤, 사용자와 결과를 검토했다. 그 자리에서 사용자가 세 가지를 결정했다.
- **결정 1, load-handoff 진행 기준**: "기본이 기다리라는거지 명시적 지시가 있으면 따르지 않을 이유는 없어." 2026-10-21에 정한 "무조건 기다림"을 바꿨다. 기본은 제안하고 기다리고, 사용자가 이 세션에서 계속하라고 하거나 다음 행동을 지시했으면 drift를 보고한 뒤 진행한다. 다른 세션의 승인은 계속 다시 묻는다. 두 사본에 같은 문장을 넣었다.
- **결정 2, inspect-session 자동 호출 허용**: load-handoff가 handoff에 기록된 원래 세션을 조회할 때 inspect-session을 쓰도록 했다. 명시 호출 전용이면 모델이 skill을 부를 수 없으므로 Claude의 `disable-model-invocation`과 Codex의 `agents/openai.yaml`을 지우고, `EXPLICIT_ONLY`에서도 뺐다. 자동 호출이 되면 description이 매 세션 skill 목록에 들어가므로 사용 조건을 description에 적었다. load-handoff 2단계에는 빈 곳을 채울 때만 inspect-session으로 조회한다는 문장을 넣었다.
- **결정 3, 첫 사용에서 찾은 약점 보완**:
  - overview가 Write/Edit만 세어서, Bash로 한 커밋·push·sync가 보이지 않았다. 셸 명령을 `&&`·`;`·`|` 단위로 나누고 heredoc 본문(커밋 메시지)은 건너뛴 뒤, 상태를 바꾸는 git·sync 명령과 성공 여부를 보여 준다. 처음 만든 정규식은 Codex 실제 기록에서 `merge-base`·`checkout-index`·`stash list`·`tag --points-at`를 잘못 잡아서, 명령 이름 뒤 경계와 읽기 전용 하위 명령을 구분하도록 고쳤다. 두 스크립트가 같은 정규식을 쓴다.
  - Claude peer 이벤트 미리보기를 subagent 보고의 harness 고정 문구가 차지했다. 이 문구를 떼고 `(subagent report)` 표시를 붙인다. Codex에는 이 문구가 없어서 적용하지 않았다.
  - "파악만" 할 때도 `HEAD`, worktree, 기록된 커밋·push·sync가 그대로인지 읽기 전용으로 확인한다. 첫 사용에서 기록에는 "sync 안 함"이었지만 실제로는 홈에 적용돼 있었다.
  - 보고 분량을 질문 크기에 맞춘다. 좁은 질문에는 짧게 답하고, 전체 파악이나 후속 작업일 때만 모든 항목을 채운다. 다른 skill에서 불렸으면 그 skill에 필요한 답만 돌려준다.
- **검증**: `python3 -B codex-config/tests/inspect-session.py`에 state ops 회귀 테스트 1개를 더했고 13개 모두 통과했다. 두 스크립트의 판정을 17개 명령 예시로 확인했다. Claude 스크립트로 `f5d2b678`의 overview를 다시 만들어 커밋 4건과 push 1건이 나오는 것을 확인했다. Claude 큰 transcript 8개와 Codex 큰 rollout 5개도 오류 없이 처리했다. `./check-sync-status`는 별도 작업인 Claude `run-loop` 미등록 2건만 실패하고, load-handoff 정규화 비교와 나머지 정책 검사는 통과했다. 자동 호출이 실제로 일어나는지는 홈 적용 전이라 확인하지 않았다.
- **홈 적용**: 하지 않았다.

### 2026-11-11 — Codex inspect-session의 실패 판정과 패치 완료 기록 수정

- **재현**: `except ValueError:`·`except sqlite3.Error:`가 실패로 분류되고, 종료 코드 0이나 `isError: false`가 있어도 본문의 오류 문자열 때문에 실패로 바뀌는 현상을 회귀 테스트로 확인했다. `patch_apply_end` 누락과 파일 목록의 생략 개수 미표시도 재현했다.
- **수정**: JSON 문자열로 감싼 결과도 해석해 종료 코드·오류 필드를 우선한다. 구조화된 결과가 없을 때만 셸 상태 헤더와 오류 표시로 시작하는 짧은 메시지를 검사한다. `patch_apply_end`의 성공·실패·불명 상태와 변경 경로를 보존하고, 타임라인에는 상태와 경로만 표시하며 상세 내용은 `show`로 조회한다. `--files`로 제한한 목록에는 생략 개수를 표시한다.
- **검증**: `python3 -B codex-config/tests/inspect-session.py`의 12개 테스트가 통과했다. 로컬 `sessions/`의 JSONL·gzip 파일을 디스크 크기순으로 고른 15개에서 도구 결과 7,295건을 비교했으며, 실패 판정은 844건에서 191건으로 줄었다. 감소한 653건 전체를 오탐으로 확정한 것은 아니다. 가장 큰 파일에서 기존에 누락된 패치 완료 기록 312건을 모두 보존했다. 사용자가 검토한 15개 파일의 목록은 받지 않았으므로 사용자 통계와 같은 표본이라는 뜻은 아니다.
- **도구 간 차이**: 이번 수정은 Codex 결과 envelope와 `patch_apply_end` 형식에 대한 보정이다. Claude 파서에는 이 형식을 이식하지 않았다. 홈 동기화는 수행하지 않는다.

### 2026-11-11 — inspect-session의 Codex 대응 추가

- **변환**: Claude의 세션 탐색과 파서를 그대로 복사하지 않고 Codex rollout 형식에 맞춰 다시 작성했다. `CODEX_HOME`과 `CODEX_THREAD_ID`를 지원하고, 제목은 로컬 `session_index.jsonl`에서 찾는다. 압축된 `.jsonl.gz` 기록과 보관된 세션도 읽는다.
- **근거**: 이 환경의 최근 Codex 기록에서 `session_meta`, `response_item`, `event_msg/item_completed` 구조와 제목 인덱스의 키를 확인했다. [공식 thread API 문서](https://learn.chatgpt.com/docs/app-server#threads)에서 조회와 재개가 별도 작업임을 확인하고, 로컬 기록이 없을 때 사용할 읽기 전용 조회 방법도 스킬에 명시했다.
- **동작 차이**: Codex는 중복 저장되는 사용자·assistant 발화를 정리하며, 옵션에 따라 인용 번호가 변하지 않도록 원본 JSONL 줄 번호를 사용한다. 마지막 턴의 완료 여부와 프로세스의 실행 여부를 구분하고, 패치 경로는 변경 시도의 일부 증거로만 표시한다. 마스킹을 적용한 뒤 출력 길이를 제한하거나 검색한다.
- **승인 범위**: 원본의 다른 세션 승인 불승계 원칙은 유지한다. 후속 작업은 live state를 확인한 뒤 현재 대화에서 이미 승인된 행동이면 진행한다. 이 기준은 2026-11-11 사용자 결정으로 두 도구에 같이 적용했다. skill 호출만으로는 transcript에 적힌 다음 작업을 시작하지 않는다는 문장을 Codex `SKILL.md`에도 넣었다.
- **검증**: `python3 -B codex-config/tests/inspect-session.py`로 임시 기록을 이용한 8개 동작 테스트를 통과했다. 제목·ID 검색, gzip·보관 경로, 중복 제거, 도구 결과·실패·질문 응답, 압축·되돌리기, 인용 번호 유지, 마스킹, 불완전 기록, 읽기 전용 동작을 확인했다. skill frontmatter 검증과 현재 Codex 세션 기록을 이용한 파싱 점검도 통과했다. 실제 `$inspect-session` 호출은 홈 적용 전이므로 확인하지 않았다.
- **부모 검사**: `check-sync-status`에서 Claude 단독 등록을 해제하고 재작성 쌍으로 등록했다. 서로 다른 파서에 문자열 정규화 일치를 요구하지 않고, 필수 파일·문서 등록·명시 호출 정책을 검사한다.
- **검사 범위**: `inspect-session` 쌍과 호출 정책 검사는 통과했다. 전체 검사는 별도 작업으로 추가된 Claude `run-loop`가 예상 목록과 문서에 아직 등록되지 않아 실패했다. 해당 스킬은 이번 작업에서 수정하지 않았다.
- **홈 적용**: 하지 않았다. `codex-diff-with-home`에서 `~/.agents/skills/inspect-session/`에 들어갈 새 파일 3개를 확인했다.

### 2026-11-11 — Claude skill inspect-session 추가 (Claude submodule과 parent)

- **배경**: 다른 세션(살아 있든 종료됐든)이 무엇을 했는지 파악하고, 필요하면 이어서 작업하려고 만들었다. handoff 파일이 없어도 원본 transcript에서 바로 복원한다.
- **조사**: 이 머신의 transcript 329개(서브에이전트 273개 별도, 약 150MB)를 집계했다.
  - 사람이 쓴 메시지는 약 0.3MB, Claude 텍스트 응답은 약 0.6MB다. 나머지는 도구 결과(18.7MB), `attachment`·`api-request*` 같은 주입 기록(약 47MB), `isMeta` 문자열, 대부분 본문이 빈 `thinking`이다.
  - 사람이 작업 도중 보낸 메시지는 `user` 레코드가 아니라 `attachment`의 `queued_command`로만 남는다. 다른 세션이 보낸 메시지는 `origin.kind == "peer"`이고 `isMeta: true`로 기록된다.
  - `parentUuid` 분기는 대부분 병렬 도구 호출이고, rewind는 한 건이었다. compact 경계는 `parentUuid: null`과 `logicalParentUuid`로 이어진다.
  - 세션 이름은 실행 중일 때만 `~/.claude/sessions/<pid>.json`에 남는다. `@` 멘션의 `[ref]`가 무엇에서 나오는지는 확인하지 못했다.
- **범위**: `claude-config/home/skills/inspect-session/`에 `SKILL.md`와 `scripts/session_digest.py`(표준 라이브러리만 사용)를 추가했다. `check-sync-status`에는 처음에 Claude 단독으로 등록했으나, 커밋 전에 Codex 대응이 추가되어 재작성 쌍(`EXPECTED_REWRITTEN`)과 `EXPLICIT_ONLY`로 등록했다(위 항목).
- **설계 판단**:
  - load-handoff를 확장하지 않고 별도 skill로 만들었다. 세션 찾기와 단계별 조회가 handoff 파일 읽기와 성격이 다르다.
  - 후속 작업은 drift를 먼저 보고한다. 사용자가 이 세션에서 다음 행동을 지시했으면 진행하고, drift 때문에 그 행동이 불분명하거나 위험해질 때만 묻는다. skill만 호출했으면 다음 행동을 제안하고 기다린다. 사용자 결정에 따른 것으로, load-handoff의 "무조건 기다림"과 다르다. load-handoff는 handoff 파일만 보고 이어서 하는 경우이고, 이 skill은 호출할 때 할 일을 함께 지시하는 경우가 많기 때문이다. 다른 세션의 승인은 다시 묻는다.
  - 사용자 결정으로 명시 호출 전용이다.
  - 레코드는 파일 순서대로 읽는다. 마지막 레코드에서 조상을 따라가면 병렬 도구 결과가 빠지므로, 조상 경로에 없는 사람 메시지 구간만 `[rewound]`로 표시한다.
  - transcript 형식은 Claude Code 내부 형식이라 모르는 레코드는 건너뛴다.
  - 흔한 토큰·키 형식을 가리고 터미널 제어 문자를 지운다. 처음에는 자른 뒤에 가려서, 경계에 걸린 토큰 일부가 `grep`·`timeline` 출력에 드러났다(임시 transcript로 재현). Codex 대응을 검토하며 찾았고, 이벤트를 만들 때 먼저 가리도록 고쳤다.
- **검증**:
  - 실제 transcript로 `locate`(ID 앞부분, 제목, 경로, `--here`), `overview`, `timeline`, `show`, `grep`을 실행했다. rewind 표시(`76f408e8`), compact 이후부터 보기(`fe2b127e`), peer 메시지 58건(`70113b2c`), 사용자가 거절한 `AskUserQuestion` 표시(`bf22e1e2`)를 확인했다. 33MB transcript는 0.2초 안에 처리했다.
  - `./check-sync-status`에서 `inspect-session` 쌍과 명시 호출 정책 검사는 통과했다. 전체 검사는 별도 작업인 Claude `run-loop`가 등록되지 않아 실패한다.
  - skill을 `/inspect-session`으로 실제 호출한 결과는 확인하지 않았다.
- **홈 적용**: 하지 않았다.

### 2026-10-29 — Claude 단독 skill draft-prompt 추가와 단독 skill 검증 지원 (Claude submodule과 parent)

- **조사**: 거친 요청으로 프롬프트를 만들거나 개선하는 skill을 찾았다. Codex에는 OpenAI [ChatGPT Enterprise Prompting Guide](https://github.com/openai/openai-cookbook/blob/main/examples/chatgpt/chatgpt_prompt_guide/chatgpt_prompt_guide.md)가 배포하는 `prompt-redrafter`가 있다. 로컬 `SKILL.md`가 [zip](https://cdn.openai.com/cookbook/chatgpt-prompt-guide/prompt-redrafter.zip) 안의 파일과 byte 단위로 같은 것을 확인했다. Anthropic 쪽 사정은 다음과 같다.
  - 공식 [`anthropics/skills`](https://github.com/anthropics/skills)와 공식 플러그인 마켓플레이스에는 대응하는 skill이 없다.
  - Console의 Prompt Generator와 Prompt Improver, `/v1/experimental/generate_prompt`·`improve_prompt`·`templatize_prompt` API는 legacy Workbench와 함께 2026-08-17에 종료되었다([릴리스 노트](https://platform.claude.com/docs/en/release-notes/overview) 2026-07-17 항목). 옛 문서 URL은 [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)로 리디렉션된다.
  - 남은 공식 도구는 두 가지다. 하나는 Cookbook의 [Metaprompt 노트북](https://github.com/anthropics/claude-cookbooks/blob/main/misc/metaprompt.ipynb)으로, 단일 턴 템플릿을 만들지만 본문에 `scratchpad`·`inner monologue` 지시가 남아 있다. 다른 하나는 Claude Code 번들 claude-api skill의 `prompt-audit`로, 낡은 패턴을 감사할 뿐 새 프롬프트는 만들지 않는다.
- **범위**:
  - `claude-config/home/skills/draft-prompt/SKILL.md`를 추가했다. 세 모드는 받는 쪽에 맞춘 자기완결형 프롬프트 작성, 같은 세션에서 일하기 전의 요구사항 확인, 기존 프롬프트 개선이다. 넘기는 프롬프트에는 완료 기준과 확인 방법을 넣고, 확인 수단이 없으면 없다고 적게 했다. 낡은 패턴 정리는 직접 하지 않고 `/claude-api prompt-audit`를 안내한다.
  - `check-sync-status`에 `EXPECTED_CLAUDE_ONLY`·`EXPECTED_CODEX_ONLY`를 추가했다. 이전에는 한쪽에만 두는 skill을 허용할 방법이 없어서, 미추적 `prompt-redrafter`가 `ONLY-CODEX`로 검증에 실패하고 있었다. 명시 호출 정책 검사는 단독 skill이면 존재하는 쪽만 본다.
- **설계 판단**:
  - redrafter를 포팅하지 않고 새로 만들었다. redrafter는 ChatGPT 사용자가 프롬프트를 한 번 다듬는 용도다. 규칙 상당수("Preserve intent", "Prefer clarity over cleverness" 등)는 `prompt-audit`의 삭제 기준("모델이 이미 아는 내용인가")에 걸리는 일반론이다. 가져온 것은 출력 방식(초안 먼저, 질문 1~3개, 코드 블록, 사실 추가 금지, 단순한 요청은 짧게)뿐이다.
  - 같은 세션 모드에서는 프롬프트를 쓰지 않는다. 같은 모델이 같은 맥락에서 자기 이해를 다시 써서 성능이 오른다는 근거를 찾지 못했다. 효과는 사용자만 아는 정보를 끌어내는 요구사항 확인에 있다고 봤다. [Claude Code best practices](https://code.claude.com/docs/en/best-practices)의 "구체적인 프롬프트는 Claude가 모르는 정보를 준다", "Give Claude a check to run"과 OpenAI 가이드의 "When not to meta-prompt"가 근거다.
  - 받는 쪽 형식은 Claude면 XML 태그(Anthropic 모범 사례), Codex·GPT 계열이면 마크다운 섹션(OpenAI 가이드)으로 정했다. 받는 쪽이 정해지지 않으면 다른 Claude 세션을 기본값으로 하고, 그 가정을 밝힌다.
  - 진행 중인 작업의 인계는 handoff가 맡는다. 이 경계를 draft-prompt `SKILL.md`의 Purpose에 적었다.
  - Codex `prompt-redrafter`는 원본 갱신을 그대로 따라가려고 수정하지 않았고, 명시 호출 정책도 넣지 않았다.
- **검증**:
  - `bash -n check-sync-status`와 `./check-sync-status`가 통과했다(exit 0).
  - 임시 사본에서 실패 사례 다섯 가지가 모두 exit 1로 잡혔다. 등록하지 않은 한쪽 skill, draft-prompt 문서 행 누락, draft-prompt의 `disable-model-invocation` 누락, Claude 단독 skill이 양쪽에 생긴 경우, Codex 쪽으로 옮겨진 경우다.
  - draft-prompt를 실제로 호출한 결과는 확인하지 않았다.
- **홈 적용**: 하지 않았다. `claude-diff-with-home`은 `skills/draft-prompt/SKILL.md` 추가와 `settings.json` 차이를 보고한다. `settings.json` 차이는 키 순서와 `~/.claude/settings.json`에만 있는 `modelSettings`이며, 이번 변경과 관계없다.

### 2026-10-21 — 새 모델 기준 skill 평가 반영과 review-pr의 PR 조회 지원 (양쪽 submodule)

- **평가 방법**: Claude Code 2.1.304와 Opus 5.5 기준으로 skill 6개를 하나씩 평가했다. 기준은 Claude Code 번들 claude-api skill의 `prompt-audit.md`와, 내장 기능(`code-review`, plan mode, 세션 재개) 및 하네스 지시와 겹치는지 여부다. skill마다 subagent 하나가 평가했고, 영향이 큰 주장(skills 문서의 description 로딩 표, handoff template 펜스, review-pr의 기본 브랜치 동작, 내장 handoff의 정체)은 다시 확인했다. 퇴역할 skill은 없었다. 바이너리의 내장 handoff(`background_handoff`, `desktop_handoff`, `cloud_handoff`)는 세션을 다른 실행 환경으로 옮기는 기능이라 이 skill과 겹치지 않는다.
- **범위 (양쪽 사본 같은 의미)**:
  - 명시 호출 전용 skill 3개(handoff, load-handoff, review-independently)의 description을 목적 한 줄로 줄였다. [skills 문서](https://code.claude.com/docs/en/skills)의 표에 따르면 `disable-model-invocation: true` skill은 "Description not in context"라서, 트리거 나열과 "Do NOT use for" 문구가 모델 판단에 쓰이지 않았다.
  - handoff: template을 감싸는 바깥 펜스를 backtick 4개로 바꿨다. 안쪽에 들여 쓴 닫는 펜스가 바깥 펜스를 143행에서 닫아, Next Action 2부터 끝까지가 template 밖에 있었다(markdown-it로 확인). 4단계에 원 세션 ID를 적는 문장을 넣었고, Resume Checklist 5번은 "Next Action 1을 제안하고 승인을 기다린다"로 바꿨다.
  - load-handoff: resume·continue 요청에도 다음 행동을 제안하고 기다리며, handoff에 기록된 승인은 다시 묻는다. Report에 Open Questions 항목을 추가했다. template은 이 절을 필수로 정하는데 보고 목록에서 빠져 있었다. 후보 나열 명령을 `ls -lt .handoffs/*/*-handoff-*.md`로 바꿨고, `@` 참조가 "절대 경로로 바뀐다"는 근거 없는 설명을 지웠다.
  - make-plan: 이 작업의 폴더가 있으면 그 폴더에 새 버전을 쓴다. 이전 규칙은 다른 날 개정하면 새 날짜 폴더를 만들어 이력이 갈라졌다. 폴더는 의미로 찾고, glob은 `ls -d .plans/[0-9][0-9][0-9][0-9][0-9][0-9]-{task-name}/`로 바꿨다. 이전 glob은 `oauth-api-auth`처럼 이름 끝만 같은 폴더도 잡았다. plan mode에서는 계획을 먼저 제시하고, plan mode가 끝난 뒤 파일을 만든다.
  - review-independently: 2026-08-07 통합 때 없어진 절차를 부정하던 문장 두 개("Do not force a peer-review protocol…", "Do not auto-discover an existing review file…")를 지우고, `.reviews/` 읽기 금지에 통합 때 빠진 이유를 되살렸다.
  - git-commit-message: 응답 template의 `[file](absolute/path)` 링크를 저장소 기준 상대 경로로 바꿨다. 2026-09-27에 response-format의 링크 규칙을 지운 근거와 같다. 펜스 안에 있던 안내문을 밖으로 옮기고 중복 문장 하나를 지웠으며, Proposed Commit에 `Branch:` 줄을 넣었다.
  - review-pr: 사용자가 밝힌 용도(다른 사람 PR 리뷰, PR을 올리기 전 셀프 체크, PR description 작성)에 맞춰 다시 썼다.
    - 버그와 회귀를 먼저 보는 기준은 유지하고, PR 설명과 구현의 대조, 관계없는 변경, 셀프 체크용 잔여물 점검을 더했다.
    - PR 번호·URL과 GitHub에만 있는 브랜치를 받는다. `gh`는 읽기 전용 하위 명령 6개만 쓰고, 안 되면 로컬 git, 그것도 안 되면 `gh auth login`을 요청한다. checkout하지 않고 `git show <sha>:<path>`로 읽는다.
    - 기본 브랜치에서 인자 없이 부르면 `main...main` 빈 diff가 되던 문제를 `@{upstream}..HEAD`와 작업 트리 변경을 보도록 고쳤다.
    - remote가 여럿이면 `--repo`를 넘긴다. `claude-config`에서 `--repo` 없이 실행한 `gh`가 `origin`(github.com) 대신 `upstream`(다른 호스트)을 골랐다.
    - description만 요청하면 리뷰 보고 없이 가볍게 쓴다. 중복이던 Quick Start 절은 지웠다.
- **사용자 결정**:
  - load-handoff는 "무조건 기다리는거고 바로 이어서 작업하는거는 없어". 2026-07-24부터 유지하던 resume 권한 해석을 뒤집었고, 항목별 메모도 고쳤다.
  - review-pr에서 버그 찾기는 계속 핵심이다("다른 사람 PR을 확인해야되니"). description만 요청하면 가볍게 처리하는 것은, 2026-09-11 보완 3의 "리뷰하면서 초안을 쓴다"를 description 단독 요청에 한해 바꾼 것이다. `draft`를 붙인 요청은 전처럼 리뷰를 먼저 한다.
  - review-pr description에 리뷰 전용 트리거 문구는 추가하지 않았다. 슬래시 없이 "PR 리뷰해줘"라고 하면 내장 `code-review`가 실행될 수 있지만, `--comment`나 `--fix` 없이는 아무것도 쓰지 않으므로 그대로 두기로 했다.
  - 다시 쓴 문장을 원문으로 되돌려 review-pr diff를 줄이는 안은 기각했다. 지금 머리말이 세 가지 용도를 명시하기 때문이다.
  - 브랜치 규칙은 이번에도 넣지 않았다. 전역 규칙에 넣으면 `main`이 보호되는 PR 기반 저장소에서 하네스의 "branch first" 기본값까지 꺼 버린다. 2026-09-27 결정을 유지하고, git-commit-message의 `Branch:` 줄로 커밋 제안에서 브랜치가 드러나게 했다.
- **변환 판단**:
  - `check-sync-status`에는 줄 단위 예외가 없어서, 도구마다 동작이 다른 세 문장은 두 사본에서 같은 문장으로 썼다.
    - handoff 세션 ID 문장은 `${CLAUDE_SESSION_ID}`를 쓰고, placeholder로 남으면 `CODEX_THREAD_ID` 환경 변수를 읽게 한다. 정규화는 대문자 `CLAUDE`/`CODEX`를 바꾸지 않는다.
    - make-plan의 plan mode 문장은 "plan mode가 요구하는 방식으로 먼저 제시한다"로 썼다. Claude plan mode는 전용 계획 파일을 쓰고, Codex Plan Mode는 파일 쓰기를 금지하고 `<proposed_plan>` 블록을 요구한다(openai/codex `codex-rs/collaboration-mode-templates/templates/plan.md`).
    - load-handoff의 `@` 문장은 "이미 첨부됐으면 그 내용을 쓴다"는 조건문이다.
  - Codex가 `codex-config` 변경을 리뷰하면서 `target-resolution.md`의 세 곳을 고쳤다. 첫째, base 브랜치도 새로 fetch하고 fetch가 실패하면 중단한다. 둘째, GitHub에만 있는 브랜치는 `<remote>/<branch>` 대신 `FETCH_HEAD` SHA를 쓴다. single-branch clone에서는 fetch해도 remote-tracking ref가 생기지 않음을 확인했다. 셋째, fetch 조건을 "없거나, 낡았거나, 최신인지 확인이 필요할 때"로 넓혔다. 세 가지 모두 Claude에도 해당하므로 양쪽에 반영했다.
  - 다만 Codex 버전은 SHA를 셸 변수에 담았다. 두 도구 모두 명령마다 새 셸을 띄우므로 다음 명령에서는 변수가 비고, 그러면 `git show "$sha":<path>`가 `git show :<path>`가 되어 로컬 index를 오류 없이 읽는다(scratch에서 확인). 그래서 `git rev-parse FETCH_HEAD`가 출력한 SHA를 이후 명령에 그대로 적게 바꿨고, `SKILL.md`의 fetch 조건 문장도 같은 뜻으로 맞췄다. Codex는 최종본을 확인했다.
  - 참고로 `~/.codex/models_cache.json`의 GPT-6 계열 기본 프롬프트 4개에는 "branch"라는 말이 나오지 않는다.
- **검증**:
  - `./check-sync-status`가 exit 0으로 끝났다. review-pr `references/` 4종과 git-commit-message `SKILL.md`는 byte 동일하다.
  - scratch에서 make-plan glob, load-handoff 후보 나열, review-pr의 base·head fetch 순서(로컬로 만든 `refs/pull/1/head`)를 실행해 확인했다.
  - `claude -p`로 실행한 프로젝트 skill에서 `${CLAUDE_SESSION_ID}`가 그 세션 ID로 치환되는 것을 확인했다. `codex exec`로는 `CODEX_THREAD_ID`가 셸 명령에 전달되고 resume ID와 같은 것을 확인했다.
  - 실제 GitHub의 `pull/<N>/head`, 사용자 skill 경로(`~/.claude/skills/`)에서의 치환, 바뀐 skill의 실제 실행 결과는 확인하지 않았다.
- **홈 적용**: 하지 않았다. `claude-diff-with-home`은 이번 skill 변경 외에 `settings.json` 차이도 보고한다. `~/.claude/settings.json`에만 있는 `modelSettings`, `advisorModel`, `skipWorkflowUsageWarning`은 sync하면 사라진다. `modelSettings`는 2026-09-11 유지 결정대로 두고, 나머지 두 값도 사용자 결정으로 같은 방식으로 둔다. 사용자는 두 값이 사라져도 문제가 된 적이 없었다고 했고, 필요하면 `/config`에서 다시 켠다.

### 2026-09-27 — 새 모델 프롬프트 지침에 맞춰 지시문·skill 정리 (양쪽 submodule)

- **범위**:
  - Claude output style `fluent-korean-concise`의 상태 공유 조항에서 "사소한 진행 상황을 중계하지 않습니다"를 지우고, 언제 상태를 알릴지 적었다.
  - Claude `response-format.md`의 §2 Code References를 지웠고, Codex `home/AGENTS.md`의 링크 형식 줄도 지웠다.
  - 양쪽 skill 사본에 같은 수정을 넣었다. review-pr는 push·PR 생성 금지와 "초안은 요청할 때만"을 한 번씩만 남기고, 인자 해석 질문을 리뷰 대상이 달라질 때로 좁혔다. `target-resolution.md`의 "변경 파일을 끝까지 읽지 말라"도 지웠다. load-handoff는 새 handoff 제안을 사용자가 멈추거나 도구를 바꾼다고 말할 때로 좁혔다. handoff의 `**NEVER**`와 git-commit-message의 `Do NOT`은 평서문으로 바꿨다.
  - 문서 정정: `claude-config/README.md`의 모델별 하네스 설명을 2.1.280과 Opus 5.5 기준으로 고쳤다. 이 문서의 review-pr 판단 근거(Codex 내장 리뷰)와 thinking 환경 변수 설명도 바로잡았다.
- **판단 근거**: Claude Code 2.1.280에 번들된 claude-api skill의 `prompt-audit.md`는 현재 모델이 지시를 더 문자 그대로 따르므로, 예전 모델에 맞춘 강조·반복·진행 보고 억제 문구가 과하게 적용된다고 설명한다. 특히 "don't narrate"류 문구가 있으면 Opus 5.5가 진행 상황을 너무 적게 말한다고 하는데, 이 세션에서도 하네스가 상태 공유를 재촉했다. OpenAI GPT-5.6 가이드도 승인 요구를 반복하면 불필요한 확인 질문이 늘어난다고 한다. 링크 형식 조항은 Claude 하네스의 "Reference code as `file_path:line_number` — it's clickable."과 충돌했다. 사용자가 VS Code 확장을 거의 쓰지 않아 하네스 형식을 따르기로 했다. Codex는 GPT-6 기본 프롬프트가 같은 링크 형식을 이미 지정한다. 하네스의 "If on the default branch, branch first."에 대응하는 규칙은 사용자 결정으로 넣지 않았다. 커밋은 git-commit-message 제안을 거치므로, 브랜치를 만들려 하면 승인 전에 드러난다.
- **변환 판단**: skill 수정은 두 사본에 같은 문자열 치환을 적용했다. output style의 응답 구성 절반은 Claude 전용이라 Codex에 옮기지 않는다.
- **검증**: `./check-sync-status`와 `git diff --check`를 통과했다. 행동 변화는 아직 Opus 5.5와 GPT-6에서 다시 시험하지 않았다.

### 2026-09-27 — Codex 지시문을 GPT-6 기본 프롬프트에 맞춤 (승인 조항은 양쪽 반영)

- **범위**: Codex `home/AGENTS.md`에서 "Before editing a file, briefly explain…"과 "Prefer `rg`…" 두 줄을 지웠다. 커밋 승인 조항 네 곳에는 이유 구절 "because later edits change what the commit would contain"을 똑같이 붙였다. 네 곳은 Codex `home/AGENTS.md`, Claude `home/rules/git-commit-guidelines.md`, git-commit-message `SKILL.md`의 두 사본이다.
- **판단 근거**: 사용자는 Codex에서 GPT-6 계열을 주로 쓰기로 했다. `models_cache.json`의 `gpt-6-sol` 기본 프롬프트는 도구를 쓰기 전에 commentary로 먼저 알리라고 하고(61행), 검색에는 `rg`를 먼저 쓰라고 한다(98행). 그래서 지운 두 줄은 중복이다. 반면 파괴적 명령 금지와 사용자 변경 보존은 GPT-6 기본 프롬프트에 없어서(`gpt-5.6-luna`에는 있었다) AGENTS.md에 남겼다. GPT-6 23행 "Do not request permission again when the user has already authorized an action in an earlier turn"은 "승인은 그 제안에만 적용된다"는 조항과 부딪히므로, 조항에 이유를 붙여 의도한 예외임을 드러냈다. `developer_instructions`의 어조 조항("사용자 메시지의 어조를 모방하지 않고")은 `gpt-5.6-luna`의 "You match the tone…"과 충돌했지만, GPT-6에는 그 문장이 없어 그대로 두었다.
- **변환 판단**: 이유 구절은 모델과 무관하게 유효하고 git-commit-message는 byte 동일 쌍이라, Claude 규칙과 skill에도 같은 문장을 넣었다. 지운 두 줄은 Codex 기본 프롬프트와의 중복만 정리한 것이다. Claude 쪽 대응 규칙(`response-format.md` §1, `tool-usage.md` §1)은 Claude 하네스에 같은 내용이 없어 유지한다.
- **검증**: `./check-sync-status`를 통과했다.

### 2026-09-23 — Claude sync에서 claude.ai 동기화 skill 경로 제외 (Codex 미적용)

- **범위**: `claude-config/claude-sync-to-home`에 Claude Code 소유 경로 목록(`skills/synced`, `skills/.trash`)을 두고, 고아 검사·sync 전 백업·빈 디렉터리 정리에서 제외했다. `tests/sync-home.sh`에 이 경로를 보존하는지 확인하는 단언을 추가하고, README의 스크립트 설명에 제외 규칙을 적었다.
- **판단 근거**: Claude Code 2.1.280은 claude.ai 계정에서 켠 skill을 터미널 세션에서도 `~/.claude/skills/synced/`로 내려받는다(changelog: "Added syncing of the skills and plugins enabled on your claude.ai account to terminal sessions"). 이 머신은 2026-09-23 10:26에 자동 업데이트되면서 파일 207개가 생겼다. sync 스크립트는 `skills/`를 repo가 관리하는 디렉터리로 보므로 이 파일들을 고아로 분류해 삭제를 제안했다. skills 문서가 `synced`라는 이름을 예약해 두었고, 동기화를 끄면 `~/.claude/skills/.trash/`로 옮긴다고 설명하므로 두 경로 모두 하네스 소유로 본다.
- **변환 판단**: Codex에는 적용하지 않는다. `codex-sync-to-home`은 `~/.agents/skills/`에서 repo에 있는 skill 이름만 다루고, `~/.codex/skills/.system/`은 이미 제외하므로 같은 문제가 없다.
- **검증**: 스크립트 문법 검사와 `tests/settings-merge.sh`, `tests/sync-home.sh`를 통과했다. 추가한 단언은 이전 스크립트에서 실패하는 것을 확인했다. 실제 홈 대상 dry-run에서도 `skills/synced` 언급이 사라졌다.

### 2026-09-23 — 도구·모델 버전 변경으로 무효가 된 설정 정리 (양쪽 submodule)

- **범위**: Codex `home/config.toml`에서 `commit_attribution`과 `[features] codex_git_commit`을 지웠다. `local/config.override.toml.example`에서는 모델 예시를 `gpt-5-codex`에서 `gpt-6-sol`로, project trust 예시를 `trust_level = "trusted"`로 바꾸고, TOML scope 설명 예시를 `[tui] status_line_use_colors`로 바꿨다. Codex README와 `skill-authoring.md`의 loader 링크는 `codex-rs/ext/skills/src/host_roots.rs`로 고쳤다. Claude `home/statusline-command.sh`에서는 transcript 기반 대체 비용 계산과 가격표를 없애고, `local/settings.override.json.example`의 `Glob`·`Grep` allow 항목을 지웠다.
- **판단 근거**: Codex 0.156.0에서 `codex doctor`는 `commit_attribution`을 인식하지 못하는 설정으로 보고하고(상류 PR #22412에서 제거), `codex features list`는 `codex_git_commit`을 `removed`로 표시한다. `gpt-5-codex`는 모델 카탈로그에 없고, `trusted = true`는 알 수 없는 필드다. 기존 loader 링크는 404였다. Claude statusline의 대체 가격표는 Opus 5.5(입력 $4, 캐시 읽기 0.05x), Sonnet 5(입력 $2), Fable 5.1(캐시 읽기 0.025x)과 맞지 않았다. 비용은 status JSON의 `cost.total_cost_usd`가 정확하므로, 가격표를 따라 고치는 대신 대체 경로를 없앴다. 2026-08-21 항목에서 transcript 합계를 대체 경로로 남긴 결정은 이번에 대체했다. Linux·macOS·WSL에서는 `Glob`·`Grep`이 기본 도구 목록에 없고, settings의 allow 규칙으로도 되살아나지 않는다(Claude Code tools-reference).
- **변환 판단**: 두 submodule의 변경은 서로 대응하지 않는다. Claude의 `attribution`은 유효한 설정이라 유지한다. Codex status line은 네이티브 `estimated-thread-cost`를 쓰므로 statusline 변경은 이식 대상이 아니다.
- **검증**: 두 submodule의 스크립트 문법 검사와 `tests/*.sh`를 모두 통과했다. statusline은 같은 transcript에서 이전 버전과 토큰 합계가 같았고, 이전 형식(v2) 캐시가 남아 있으면 무시하고 처음부터 다시 계산했다. 새 baseline과 예시 값을 합친 Codex 설정은 `codex doctor`에서 인식하지 못하는 설정 경고가 없었다.
- **홈 적용**: 하지 않았다. `codex-merge-config`는 기존 키를 지우지 않으므로, sync 후에도 `~/.codex/config.toml`에 남은 `commit_attribution`과 `[features] codex_git_commit`은 따로 지워야 한다.

### 2026-09-11 — Claude 중심 skill·rule·settings 경량화 (skill은 Codex 동시 반영)

- **범위**: 모델과 Claude Code 하네스가 좋아지면서 근거가 사라졌거나 줄여도 되는 조항을 Claude `home/`에서 찾아 정리했다. 짝 사본이 있는 skill은 Codex `home/skills/`도 같이 고쳤고, rule, settings, output style은 Claude에만 반영했다. 커밋 승인 범위 문장만 예외로, 두 도구의 전역 커밋 규칙과 git-commit-message skill 두 사본에 함께 넣었다(아래 "커밋 규칙 보강"). 단위별 감사, 반박 검증, 완결성 검토를 거친 항목 가운데 반박 검증을 통과했거나 사용자가 고른 항목만 반영했다. 1차 반영은 `claude-config@709d333`·`codex-config@6e538b7`에 들어갔고, 2차 반영은 그 뒤 커밋에 들어간다.
- **skill 1차**: handoff의 Writing Checklist를 지웠다. 체크 항목이 모두 워크플로 단계와 template에 이미 있었다. make-plan은 Rules 절을 지우고, 본문에 없던 세 조항(Context 절 필수, 검증하지 않은 주장을 옮기지 않음, 같은 세션에서도 새 버전 파일 생성)만 워크플로와 Version Management 절로 옮겼다.
- **skill 2차**:
  - handoff·load-handoff를 명시 호출 전용으로 바꿨다. Claude는 `disable-model-invocation: true`, Codex는 `agents/openai.yaml`의 `policy.allow_implicit_invocation: false`를 쓴다. 500자가 넘는 description이 매 세션 skill 목록에 들어갔는데, 사용자가 description을 줄이는 대신 이 방식을 골랐다. 이제 `/handoff`·`/load-handoff`(Codex는 `$handoff`·`$load-handoff`)로 직접 불러야 한다.
  - load-handoff: 존재하지 않는 명령 `/load_handoff`를 `/load-handoff`로 고쳤다. 45줄짜리 Response Template은 `## Report` 절로 바꾸되 2026-07-24에 넣은 복원 항목은 모두 남겼다. 처음에는 한 문단이었고, 비교 검증 뒤 필수 항목 불릿으로 바꿨다(아래 "전후 비교 검증과 보완"). Purpose 절과, 워크플로 1단계의 경로 해석과 겹치던 오류 처리 두 줄을 지웠다. "재개 후 작업이 남으면 새 handoff를 만든다"는 "세션을 마칠 때 제안하고, 사용자가 요청할 때만 작성한다"로 바꿨다.
  - handoff: 지식 구분 단계를 의도 복구 단계의 한 항목으로 합치고 단계 번호를 다시 매겼다. Best Practices에서 다른 항목과 겹치는 "Be operational"과 "Be explicit about uncertainty"를 지웠다. 2026-07-24 복구 계약(template 필드, 줄 수보다 복구 가능성 우선, Resume Checklist)은 그대로 둔다.
  - make-plan: 8단계 워크플로를 3단계(위치 결정, 근거 확보, 작성과 보고)로 줄였다. 탐색이 얕아지지 않도록 코드 탐색과 참조 자료 검증 문장은 남겼다.
  - review-pr: 전면 재작성하지 않고 네 곳만 고쳤다. description에서 리뷰 전용 트리거 4개('review this PR' 등)를 빼고 PR 초안 트리거만 남겼다. 초안 전 리뷰는 유지하되 발견 사항을 병합을 막는 문제부터 간결하게 보고하고, 초안을 쓸 때 빌드·테스트는 스스로 돌리지 않는다(아래 "전후 비교 검증과 보완"). 저장소 PR 템플릿이나 `CONTRIBUTING.md`가 있으면 그것을 따르고, 기본 `references/pr-body-template.md`는 Summary·Changed files·Testing·Risk 4개 절로 줄였다. When Not To Use에서 "항상 리뷰부터 한다" 항목은 지웠다.
- **review-pr 판단 근거**: Codex에서 review-pr를 실제로 부른 5건 중 4건이 PR 초안 요청이었고, 사용자는 더 짧게 써 달라는 요청(7번 중 6번), 변경 파일을 넣어 달라는 요청(3번), 저장소 PR 템플릿 사용을 반복했다. Claude 쪽 리뷰 요청은 내장 `/code-review`와 겹친다. Codex에도 내장 `/review`와 `codex review`가 있어 리뷰 요청은 양쪽 모두 내장 기능과 겹친다. 그래도 description 트리거는 암묵 호출에만 영향을 주고 `$review-pr`로 리뷰를 직접 부를 수 있어, 두 사본을 같은 의미로 유지했다(2026-09-27 정정: 처음에는 Codex에 내장 리뷰 명령이 없다고 적었다). 리뷰 단계를 빼고 초안 전용 skill로 재작성하는 안은 사용자가 리뷰 후 초안 흐름의 가치를 들어 기각했다.
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
- **판단 근거**: Claude Code 2.1.266 문서 기준으로, 작업 디렉터리 안의 읽기와 내장 read-only 집합의 git 명령은 어떤 권한 모드에서도 프롬프트가 없어 allow가 동작을 바꾸지 않는다. auto 모드는 작업 디렉터리 읽기를 분류기 없이 승인하므로 비밀 파일을 막는 장치는 deny뿐이라 deny는 중복과 공개 자료(인증서)만 줄였다. 사용자 설정에서 `/etc/**`처럼 슬래시 하나로 시작하는 패턴은 `~/.claude/` 기준으로 해석되어 한 번도 동작한 적이 없었고(`//` 접두사가 파일시스템 루트), 사용자 결정으로 재앵커 대신 삭제했다. `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING`은 Opus 4.6·Sonnet 4.6에만 적용되고 Fable·Sonnet 5·Opus 4.7 이후에는 효력이 없다. `MAX_THINKING_TOKENS`의 0이 아닌 값은 adaptive reasoning 모델에서 무시되지만, Haiku 4.5 같은 비-adaptive 모델에는 적용된다. 다만 제거한 값 128000은 Claude Code가 최대 출력 토큰보다 1 작게 제한하므로 기본 상한과 같아, 제거해도 동작 차이가 없다(2026-09-27 정정: 처음에는 두 변수 모두 Opus 4.6·Sonnet 4.6에만 적용된다고 적었다).
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
