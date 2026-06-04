# 스킬·룰 동기화 검증 현황

`claude-config/home/`과 `codex-config/home/`의 같은 항목(skill·rule)이 어떤 상태인지 항목별로 기록한다. 각 항목에 대해 (1) 두 도구 버전이 **동일/변환/재작성** 중 무엇인지, (2) 변환됐다면 **무엇이·왜** 다른지, (3) **검증 상태**를 남긴다.

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
| git-commit-message | 변환 | `Co-Authored-By: Claude`→`Codex` 트레일러, "🤖 Generated with Claude/Codex" 푸터 문구 | ✅ | 2026-06-04 |
| handoff | 변환 | 경로 `.claude/handoffs`↔`.codex/handoffs`, `~/.claude`↔`~/.codex`, "Claude session"→"Codex session" 전반 | ✅ | 2026-06-04 |
| load-handoff | 변환 | handoff와 같은 계열의 경로·명칭 치환(`.claude`↔`.codex`, 세션 명칭) | ✅ | 2026-06-04 |
| make-plan | 변환 | 플랜 파일 prefix `claude-plan-`→`codex-plan-`, 세션 명칭. **2026-06-04 드리프트 2건 수정**(아래 메모) | ✅ | 2026-06-04 |
| review-pr | 동일 | 변환 불필요(tool-neutral). `SKILL.md` + `references/` 4종 전부 byte 동일 | ✅ | 2026-06-04 |
| setup-team-agents | 재작성 | Claude 전용 `TeamCreate`/`TaskCreate`/`SendMessage` 공유 task board 의존 → Codex `spawn_agent`/`send_input`/`wait_agent` + 로컬 오케스트레이션으로 재설계. frontmatter `compatibility`에 명시 | ✅ | 2026-06-04 |

## Rules

경로: `claude-config/home/rules/<f>` ↔ `codex-config/home/rules/dev-tools/<f>`

| Rule | 차이 유형 | 차이 내용 / 사유 | 검증 | 최종 점검 |
|---|---|---|---|---|
| response-format | 변환 | Codex 버전에 "Default Response Language(한국어 응답)" 섹션이 **추가**되고 이후 섹션 번호가 +1씩 밀림. 나머지 항목은 동일 | ✅ | 2026-06-04 |
| tool-usage | 변환 | 도구명·예시를 Codex 셸 도구로 치환: `Read/Edit/Write/Grep/Glob/Bash` → `rg/sed/find/git/apply_patch`, `Read("./..")` → `sed -n '..' ./..` | ✅ | 2026-06-04 |
| python-guidelines | 동일 | 변환 불필요. byte 동일 | ✅ | 2026-06-04 |

## 항목별 메모

### make-plan — 2026-06-04 드리프트 2건 수정

포팅 과정에서 들어간 실수 2건을 발견·수정했다(둘 다 `codex-config/home/skills/make-plan/SKILL.md`):

1. **중복 문구**: "... referenced by future Codex sessions or other coding agents **or other coding agents**." → 중복 절 제거(Claude 버전은 한 번만 등장).
2. **버전 예시 의미 붕괴**: "다른 에이전트의 플랜을 이어받는" 예시가 `claude-`→`codex-` 일괄치환으로 **소스 파일명까지** 바뀌어 위 줄과 중복·자기모순(`continues from codex`)이 됨. → 소스를 `claude-plan.md` / `(continues from claude)`로 복원해 교차 에이전트 예시의 의미를 되살림.

### setup-team-agents — 단순 치환이 아님

이 항목만 두 버전의 **로직·구조 자체가 다르다**. Codex에는 Claude의 공유 TeamCreate/TaskCreate board와 inter-agent 메시징이 없어 그대로 옮기면 깨진다. 따라서 의도적으로 재설계된 것이며 "드리프트"가 아니다.

## 검증 이력

### 2026-06-04 — 전체 항목 정합성 감사

- **범위**: skill 6종 + rule 3종. 각 항목의 claude/codex 버전을 line-by-line 비교.
- **방법**: 항목별 병렬 감사로 "의도된 변환 vs 실수(drift)"를 구분하고, 발견 항목은 별도로 **적대적 재검증**(intentional 변환이면 기각). `review-pr`의 references 4종도 포함.
- **결과**: 확정 드리프트 **2건**(둘 다 make-plan) 수정, 기각 0건, 나머지 **8건은 동일 또는 의도된 변환**으로 확인. 수정분은 `codex-diff-with-home` 확인 후 `codex-sync-to-home`으로 `~/.codex/`에 반영(사후 diff "No differences found").
- **재현**: `diff claude-config/home/<path> codex-config/home/<path>`. 단, 도구명·경로·기능 치환은 **의도된 변환**이라 버그가 아니다. 버그 신호 = 중복 단어, 절반만 끝난 치환, 역방향 치환(Codex 파일에 남은 "Claude" 지시), 깨진 문법, 의도치 않은 로직 분기.
