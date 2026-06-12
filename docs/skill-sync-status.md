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
| handoff | 변환 | 도구 명칭("Claude/Codex instance"), 작성자 prefix `claude-handoff-`↔`codex-handoff-`, 전역 경로 `~/.claude`↔`~/.codex`. `SKILL.md` + `references/handoff-template.md` 구성 | ✅ | 2026-06-12 |
| load-handoff | 변환 | handoff와 같은 계열: 작성자 prefix·도구 명칭 치환 (교차 agent 예시는 반대 prefix) | ✅ | 2026-06-12 |
| make-plan | 변환 | 플랜 파일 prefix `claude-plan-`→`codex-plan-`, 세션 명칭. 2026-06-04 드리프트 2건 수정(아래 메모) | ✅ | 2026-06-12 |
| review-pr | 동일 | 변환 불필요(tool-neutral). `SKILL.md` + `references/` 4종 전부 byte 동일 | ✅ | 2026-06-12 |

## Rules

경로: `claude-config/home/rules/<f>` ↔ `codex-config/home/rules/dev-tools/<f>`

| Rule | 차이 유형 | 차이 내용 / 사유 | 검증 | 최종 점검 |
|---|---|---|---|---|
| git-commit-guidelines | 동일 | 변환 불필요. 2026-06-12에 skill `git-commit-message`를 흡수해 신설 (아래 메모) | ✅ | 2026-06-12 |
| response-format | 변환 | Codex 버전에 "Default Response Language(한국어 응답)" 섹션이 **추가**되고 이후 섹션 번호가 +1씩 밀림. Claude는 settings.json의 `language`로 같은 효과를 얻으므로 의도된 차이 | ✅ | 2026-06-12 |
| tool-usage | 변환 | 도구명·예시를 Codex 셸 도구로 치환: `Read/Edit/Write/Grep/Glob/Bash` → `rg/sed/find/git/apply_patch`, `Read("./..")` → `sed -n '..' ./..` | ✅ | 2026-06-12 |
| python-guidelines | 동일 | 변환 불필요. byte 동일 | ✅ | 2026-06-12 |

## 퇴역·흡수된 항목

| 항목 | 처리 | 일자 | 비고 |
|---|---|---|---|
| setup-team-agents (skill) | 양쪽 `outdated/skills/`로 이동, sync 제외 | 2026-06-12 | 팀 agent 기능이 도구에 네이티브로 들어오면서 사장. 설계 기록용으로 보관 — 각 submodule `outdated/README.md` 참고 |
| git-commit-message (skill) | 삭제, rule `git-commit-guidelines.md`로 흡수 | 2026-06-12 | skill 본문 대부분이 모델이 이미 아는 내용의 재진술이라, 실질 가치(승인 후 커밋, conventional format, AI attribution 금지)만 rule로 이전 |

## 항목별 메모

### handoff / load-handoff — 2026-06-12 재설계

- 저장 경로를 도구별 `.claude/handoffs/` ↔ `.codex/handoffs/`에서 **공유 `.handoffs/` + 작성자 prefix**(`claude-handoff-*` / `codex-handoff-*`)로 통일. make-plan의 cross-agent 방식과 일관되며, 한 도구가 남긴 handoff를 다른 도구가 자연스럽게 발견할 수 있다.
- handoff 템플릿을 `references/handoff-template.md`로 분리하고, 템플릿을 반복하던 Real Example(~100줄)과 중복 bash 절차를 제거 (289줄 → ~50줄).
- load-handoff에 "handoff 내용을 현재 코드와 대조 검증" 단계 추가 (make-plan의 검증 규칙과 같은 정신).

### git-commit-guidelines — skill에서 rule로

커밋 메시지 생성은 skill 트리거 없이도 일상적으로 일어나는 일이라 상시 적용되는 rule이 더 적합하다. "명시적 승인 없이 commit 금지" 원칙도 함께 명문화했다.

### make-plan — 2026-06-04 드리프트 2건 수정

포팅 과정에서 들어간 실수 2건을 발견·수정했다(둘 다 `codex-config/home/skills/make-plan/SKILL.md`):

1. **중복 문구**: "... referenced by future Codex sessions or other coding agents **or other coding agents**." → 중복 절 제거(Claude 버전은 한 번만 등장).
2. **버전 예시 의미 붕괴**: "다른 에이전트의 플랜을 이어받는" 예시가 `claude-`→`codex-` 일괄치환으로 **소스 파일명까지** 바뀌어 위 줄과 중복·자기모순(`continues from codex`)이 됨. → 소스를 `claude-plan.md` / `(continues from claude)`로 복원해 교차 에이전트 예시의 의미를 되살림.

## 검증 이력

### 2026-06-12 — 구조 개편 및 기계 검증 도입

- **범위**: skill 6종 → 4종(setup-team-agents 퇴역, git-commit-message 흡수), rule 3종 → 4종(git-commit-guidelines 신설), handoff/load-handoff 재설계.
- **방법**: 신설된 `./check-sync-status`로 전 쌍 비교. IDENTICAL 3건(review-pr, python-guidelines, git-commit-guidelines), DIFFERS 항목은 diff로 의도된 변환만 존재함을 확인.
- **결과**: 드리프트 0건. handoff/load-handoff의 차이는 작성자 prefix·도구 명칭·전역 경로 치환뿐임을 line-by-line으로 확인.

### 2026-06-04 — 전체 항목 정합성 감사

- **범위**: skill 6종 + rule 3종. 각 항목의 claude/codex 버전을 line-by-line 비교.
- **방법**: 항목별 병렬 감사로 "의도된 변환 vs 실수(drift)"를 구분하고, 발견 항목은 별도로 **적대적 재검증**(intentional 변환이면 기각). `review-pr`의 references 4종도 포함.
- **결과**: 확정 드리프트 **2건**(둘 다 make-plan) 수정, 기각 0건, 나머지 **8건은 동일 또는 의도된 변환**으로 확인. 수정분은 `codex-diff-with-home` 확인 후 `codex-sync-to-home`으로 `~/.codex/`에 반영(사후 diff "No differences found").
- **재현**: `diff claude-config/home/<path> codex-config/home/<path>`. 단, 도구명·경로·기능 치환은 **의도된 변환**이라 버그가 아니다. 버그 신호 = 중복 단어, 절반만 끝난 치환, 역방향 치환(Codex 파일에 남은 "Claude" 지시), 깨진 문법, 의도치 않은 로직 분기.
