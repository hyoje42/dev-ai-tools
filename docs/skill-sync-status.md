# 스킬·룰 동기화 검증 현황

`claude-config/home/`과 `codex-config/home/`의 같은 항목(skill·rule)이 어떤 상태인지 항목별로 기록한다. 각 항목에 대해 (1) 두 도구 버전이 **동일/변환/재작성** 중 무엇인지, (2) 변환됐다면 **무엇이·왜** 다른지, (3) **검증 상태**를 남긴다.

**기계 검증**: repo 루트의 `./check-sync-status`가 모든 skill 쌍을 비교해 IDENTICAL / DIFFERS / 한쪽에만 존재를 보고한다(rule은 2026-06-23 통합으로 비교 대상에서 제외 — 아래 'Rules' 참고). "동일" 주장의 byte 동일성은 이 스크립트로 검증하고, 이 문서는 **DIFFERS 항목의 사유 기록**에 집중한다. DIFFERS인데 이 문서에 사유가 없으면 드리프트로 간주한다.

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
| git-commit-message | 동일 | **자기완결형**: 커밋 형식·언어·승인 규칙을 SKILL.md에 직접 인라인(전역 규칙과 의도적 중복 — 독립성 우선). 워크플로·한국어 응답 템플릿 포함, claude/codex byte 동일 | ✅ | 2026-06-23 |
| handoff | 변환 | 도구 명칭("Claude/Codex instance"), 작성자 prefix `claude-handoff-`↔`codex-handoff-`, 전역 경로 `~/.claude`↔`~/.codex`. `SKILL.md` + `references/handoff-template.md` 구성 | ✅ | 2026-06-12 |
| load-handoff | 변환 | handoff와 같은 계열: 작성자 prefix·도구 명칭 치환 (교차 agent 예시는 반대 prefix) | ✅ | 2026-06-12 |
| make-plan | 변환 | 플랜 파일 prefix `claude-plan-`→`codex-plan-`, 세션 명칭. 2026-06-04 드리프트 2건 수정(아래 메모) | ✅ | 2026-06-12 |
| read-review | 변환 | peer review 응답자 prefix `claude-response-`↔`codex-response-`, Author 메타데이터, 예시 review 파일 prefix만 도구별로 치환. "검토만 하고 수정 금지", side-effect-free verification, response file 기록 의도는 동일 | ✅ | 2026-06-14 |
| review-pr | 동일 | 변환 불필요(tool-neutral). `SKILL.md` + `references/` 4종 전부 byte 동일 | ✅ | 2026-06-12 |
| write-review | 변환 | peer review 작성자 prefix `claude-review-`↔`codex-review-`, Reviewer 메타데이터, 전역 경로 `~/.claude`↔`~/.codex`만 치환. 독립 리뷰, 기존 `.reviews/` 격리, review file만 생성하는 경계는 동일 | ✅ | 2026-06-12 |

## Rules

2026-06-23부로 **Codex 전역 규칙을 `home/AGENTS.md`로 통합**하고 `home/rules/dev-tools/`를 제거했다(아래 검증 이력 참고). 따라서 rule은 더 이상 도구 간 파일 단위로 짝지어지지 않는다:

- **Claude**: `home/rules/*.md` 개별 파일 (`~/.claude/rules/`로 auto-load)
- **Codex**: `home/AGENTS.md` 섹션 (단일 파일만 operative — `~/.codex/rules/`는 지시문으로 로드되지 않음: [agents-md 가이드](https://developers.openai.com/codex/guides/agents-md), [#23788](https://github.com/openai/codex/issues/23788))

`check-sync-status`는 이제 **skills만** 비교한다. rule 내용의 도구 간 정합성은 codex `home/AGENTS.md` ↔ claude `home/rules/`를 사람이 직접 대조한다.

## 퇴역·역할 변경 항목

| 항목 | 처리 | 일자 | 비고 |
|---|---|---|---|
| setup-team-agents (skill) | 양쪽 `outdated/skills/`로 이동, sync 제외 | 2026-06-12 | 팀 agent 기능이 도구에 네이티브로 들어오면서 사장. 설계 기록용으로 보관 — 각 submodule `outdated/README.md` 참고 |
| git-commit-message (skill) | 역할 분리: 형식 규칙을 rule `git-commit-guidelines.md`로 이전하고, 워크플로 부분만 얇은 skill로 유지 | 2026-06-12 | 형식·승인·영어 강제는 rule(상시 적용), "staged만 보고 설명+제안"이라는 호출형 워크플로와 한국어 응답 템플릿은 skill(단축키)로 분리. 둘은 중복 없이 상호 참조 |

## 항목별 메모

### handoff / load-handoff — 2026-06-12 재설계

- 저장 경로를 도구별 `.claude/handoffs/` ↔ `.codex/handoffs/`에서 **공유 `.handoffs/` + 작성자 prefix**(`claude-handoff-*` / `codex-handoff-*`)로 통일. make-plan의 cross-agent 방식과 일관되며, 한 도구가 남긴 handoff를 다른 도구가 자연스럽게 발견할 수 있다.
- handoff 템플릿을 `references/handoff-template.md`로 분리하고, 템플릿을 반복하던 Real Example(~100줄)과 중복 bash 절차를 제거 (289줄 → ~50줄).
- load-handoff에 "handoff 내용을 현재 코드와 대조 검증" 단계 추가 (make-plan의 검증 규칙과 같은 정신).

### git-commit-guidelines(rule) + git-commit-message(skill) — 역할 분리

처음에는 skill을 통째로 rule로 흡수했으나, skill의 원래 가치가 "staged diff 보고 설명+메시지 제안"을 한 번에 부르는 **호출형 단축키**였음이 확인되어 같은 날 역할을 분리했다:

- **rule** (상시 적용): conventional format, 50자 제목, **영어 강제**, AI attribution 금지, 명시적 승인 없이 commit 금지. skill을 안 거치는 커밋에도 적용된다.
- **skill** (호출 시): staged 변경 우선 분석(없으면 전체 working tree를 보고 유연하게 판단) → 파일별 설명 → 메시지 제안 → 승인 대기 워크플로와 한국어 응답 템플릿. 형식 규칙은 rule을 참조해 중복을 없앴다.
- **2026-06-23**: ① Codex 전역 규칙을 `rules/dev-tools/`에서 `AGENTS.md`로 통합(`git-commit-guidelines` 포함). ② 이후 `git-commit-message` skill을 **자기완결형**으로 전환 — 형식·언어·승인 규칙을 SKILL.md에 직접 인라인(claude/codex 동일). 전역 규칙(codex `AGENTS.md` / claude `rules/git-commit-guidelines.md`)은 skill 안 거친 **직접 커밋**용으로 유지. skill 단독 사용·전역 설정 상이 케이스를 위해 독립성을 택한 결정이라 **의도적 중복**이며, 두 사본은 수동으로 동기 유지한다.

### write-review / read-review — 2026-06-12 추가

- `write-review`는 다른 agent/session의 산출물을 독립적으로 검토해 `.reviews/` 아래 review file만 남긴다. 코드·문서 수정, state-mutating git command, 기존 `.reviews/` 무단 읽기를 금지한다.
- `read-review`는 author 쪽 agent가 review file을 읽고 각 finding을 재검증해 accept/dispute/discuss verdict와 response file만 남긴다. 실제 수정은 사용자가 이후 명시적으로 지시할 때만 수행한다.
- Claude/Codex 차이는 작성자 prefix(`claude-review-*` / `codex-review-*`, `claude-response-*` / `codex-response-*`)와 meta author/reviewer 값뿐이다. 교차 agent 예시는 의도적으로 반대 prefix를 남긴다.

### make-plan — 2026-06-04 드리프트 2건 수정

포팅 과정에서 들어간 실수 2건을 발견·수정했다(둘 다 `codex-config/home/skills/make-plan/SKILL.md`):

1. **중복 문구**: "... referenced by future Codex sessions or other coding agents **or other coding agents**." → 중복 절 제거(Claude 버전은 한 번만 등장).
2. **버전 예시 의미 붕괴**: "다른 에이전트의 플랜을 이어받는" 예시가 `claude-`→`codex-` 일괄치환으로 **소스 파일명까지** 바뀌어 위 줄과 중복·자기모순(`continues from codex`)이 됨. → 소스를 `claude-plan.md` / `(continues from claude)`로 복원해 교차 에이전트 예시의 의미를 되살림.

## 검증 이력

### 2026-06-23 — agent 지시 rule 추가 · Codex rules 로딩 검증 · dev-tools 통합 제거

- **새 rule**: "agent 지시 파일(AGENTS.md / CLAUDE.md) 작성" 전역 지시 추가. AGENTS.md는 개발 내용 위주로 간결하게(상세는 다른 문서 가리키기), CLAUDE.md는 `@AGENTS.md` import로 단일 소스화. 기본값이며 사용자의 명시적 지시가 우선. 배치: claude `home/rules/agent-instruction-files.md`(영어) + codex `home/AGENTS.md` 섹션(영어, 2026-06-23 영문화).
- **Codex rules 로딩 검증**: 공식 문서 확인 결과 Codex 전역 지시문은 `~/.codex/AGENTS.override.md`/`AGENTS.md` **단일 파일**만 로드하고 `~/.codex/rules/*.md`는 지시문으로 자동 로드하지 않는다(rules-디렉터리 auto-load는 미구현 — [agents-md 가이드](https://developers.openai.com/codex/guides/agents-md), [#23788](https://github.com/openai/codex/issues/23788)).
- **dev-tools 통합 제거**: 위 결론에 따라 `codex-config/home/rules/dev-tools/`(4개)를 삭제하고 Codex 전역 규칙을 `home/AGENTS.md` 단일 소스로 통합. response-format·tool-usage·python은 이미 AGENTS.md 섹션에 있던 죽은 사본이었고, `git-commit-guidelines`는 AGENTS.md 'Git 커밋' 섹션으로 이전해 기존 미적용 갭을 해소. `git-commit-message` skill의 참조 경로도 AGENTS.md로 수정. response-format의 §4(Plan Mode Output Language)는 Codex에 plan mode가 없어 통합 시 의도적으로 누락(Claude 전용).
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
