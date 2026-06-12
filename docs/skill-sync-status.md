# 스킬·룰 동기화 검증 현황

`claude-config/home/`과 `codex-config/home/`의 같은 항목(skill·rule)이 어떤 상태인지 항목별로 기록한다. 각 항목에 대해 (1) 두 도구 버전이 **동일/변환/재작성** 중 무엇인지, (2) 변환됐다면 **무엇이·왜** 다른지, (3) **검증 상태**를 남긴다.

**기계 검증**: repo 루트의 `./check-sync-status`가 모든 skill·rule 쌍을 비교해 IDENTICAL / DIFFERS / 한쪽에만 존재를 보고한다. "동일" 주장의 byte 동일성은 이 스크립트로 검증하고, 이 문서는 **DIFFERS 항목의 사유 기록**에 집중한다. DIFFERS인데 이 문서에 사유가 없으면 드리프트로 간주한다.

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
| git-commit-message | 변환 | 참조하는 rule 경로 한 줄만 다름: `~/.claude/rules/` ↔ `~/.codex/rules/dev-tools/`. 워크플로(staged 우선 분석·설명·제안)와 한국어 응답 템플릿 담당, 형식 규칙은 rule로 위임 | ✅ | 2026-06-12 |
| handoff | 변환 | 도구 명칭("Claude/Codex instance"), 작성자 prefix `claude-handoff-`↔`codex-handoff-`, 전역 경로 `~/.claude`↔`~/.codex`. `SKILL.md` + `references/handoff-template.md` 구성 | ✅ | 2026-06-12 |
| load-handoff | 변환 | handoff와 같은 계열: 작성자 prefix·도구 명칭 치환 (교차 agent 예시는 반대 prefix) | ✅ | 2026-06-12 |
| load-review | 변환 | peer review 응답자 prefix `claude-response-`↔`codex-response-`, Author 메타데이터, 예시 review 파일 prefix만 도구별로 치환. "검토만 하고 수정 금지", side-effect-free verification, response file 기록 의도는 동일 | ✅ | 2026-06-12 |
| make-plan | 변환 | 플랜 파일 prefix `claude-plan-`→`codex-plan-`, 세션 명칭. 2026-06-04 드리프트 2건 수정(아래 메모) | ✅ | 2026-06-12 |
| review-pr | 동일 | 변환 불필요(tool-neutral). `SKILL.md` + `references/` 4종 전부 byte 동일 | ✅ | 2026-06-12 |
| write-review | 변환 | peer review 작성자 prefix `claude-review-`↔`codex-review-`, Reviewer 메타데이터, 전역 경로 `~/.claude`↔`~/.codex`만 치환. 독립 리뷰, 기존 `.reviews/` 격리, review file만 생성하는 경계는 동일 | ✅ | 2026-06-12 |

## Rules

경로: `claude-config/home/rules/<f>` ↔ `codex-config/home/rules/dev-tools/<f>`

| Rule | 차이 유형 | 차이 내용 / 사유 | 검증 | 최종 점검 |
|---|---|---|---|---|
| git-commit-guidelines | 동일 | 변환 불필요. conventional format, 영어 메시지, AI attribution 금지, 명시적 승인 없이 commit 금지 같은 상시 규칙 담당. `git-commit-message` skill은 호출형 워크플로만 담당 | ✅ | 2026-06-12 |
| response-format | 변환 | Codex 버전에 "Default Response Language(한국어 응답)" 섹션이 **추가**되고 이후 섹션 번호가 +1씩 밀림. Claude는 settings.json의 `language`로 같은 효과를 얻으므로 의도된 차이 | ✅ | 2026-06-12 |
| tool-usage | 변환 | 도구명·예시를 Codex 셸 도구로 치환: `Read/Edit/Write/Grep/Glob/Bash` → `rg/sed/find/git/apply_patch`, `Read("./..")` → `sed -n '..' ./..` | ✅ | 2026-06-12 |
| python-guidelines | 동일 | 변환 불필요. byte 동일 | ✅ | 2026-06-12 |

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

### write-review / load-review — 2026-06-12 추가

- `write-review`는 다른 agent/session의 산출물을 독립적으로 검토해 `.reviews/` 아래 review file만 남긴다. 코드·문서 수정, state-mutating git command, 기존 `.reviews/` 무단 읽기를 금지한다.
- `load-review`는 author 쪽 agent가 review file을 읽고 각 finding을 재검증해 accept/dispute/discuss verdict와 response file만 남긴다. 실제 수정은 사용자가 이후 명시적으로 지시할 때만 수행한다.
- Claude/Codex 차이는 작성자 prefix(`claude-review-*` / `codex-review-*`, `claude-response-*` / `codex-response-*`)와 meta author/reviewer 값뿐이다. 교차 agent 예시는 의도적으로 반대 prefix를 남긴다.

### make-plan — 2026-06-04 드리프트 2건 수정

포팅 과정에서 들어간 실수 2건을 발견·수정했다(둘 다 `codex-config/home/skills/make-plan/SKILL.md`):

1. **중복 문구**: "... referenced by future Codex sessions or other coding agents **or other coding agents**." → 중복 절 제거(Claude 버전은 한 번만 등장).
2. **버전 예시 의미 붕괴**: "다른 에이전트의 플랜을 이어받는" 예시가 `claude-`→`codex-` 일괄치환으로 **소스 파일명까지** 바뀌어 위 줄과 중복·자기모순(`continues from codex`)이 됨. → 소스를 `claude-plan.md` / `(continues from claude)`로 복원해 교차 에이전트 예시의 의미를 되살림.

## 검증 이력

### 2026-06-12 — 구조 개편 및 기계 검증 도입

- **범위**: skill 7종(setup-team-agents 퇴역, git-commit-message 역할 분리 후 유지, write-review/load-review 추가), rule 4종(git-commit-guidelines 신설 포함).
- **방법**: `./check-sync-status`로 전 쌍 비교. IDENTICAL 3건(review-pr, python-guidelines, git-commit-guidelines), DIFFERS 항목은 diff로 의도된 변환만 존재함을 확인.
- **결과**: 드리프트 0건. DIFFERS 항목(git-commit-message, handoff, load-handoff, load-review, make-plan, write-review, response-format, tool-usage)은 이 문서의 표에 사유를 기록했다.

### 2026-06-04 — 전체 항목 정합성 감사

- **범위**: skill 6종 + rule 3종. 각 항목의 claude/codex 버전을 line-by-line 비교.
- **방법**: 항목별 병렬 감사로 "의도된 변환 vs 실수(drift)"를 구분하고, 발견 항목은 별도로 **적대적 재검증**(intentional 변환이면 기각). `review-pr`의 references 4종도 포함.
- **결과**: 확정 드리프트 **2건**(둘 다 make-plan) 수정, 기각 0건, 나머지 **8건은 동일 또는 의도된 변환**으로 확인. 수정분은 `codex-diff-with-home` 확인 후 `codex-sync-to-home`으로 `~/.codex/`에 반영(사후 diff "No differences found").
- **재현**: `diff claude-config/home/<path> codex-config/home/<path>`. 단, 도구명·경로·기능 치환은 **의도된 변환**이라 버그가 아니다. 버그 신호 = 중복 단어, 절반만 끝난 치환, 역방향 치환(Codex 파일에 남은 "Claude" 지시), 깨진 문법, 의도치 않은 로직 분기.
