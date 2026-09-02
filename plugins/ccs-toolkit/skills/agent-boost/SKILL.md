---
name: agent-boost
description: Claude 자체 성능을 올리는 스킬·플러그인 라우팅표. 사용자가 "어떤 스킬 써야 해", "성능 올리는 방법", "무슨 플러그인 깔까", "너 더 똑똑해지는 법", "스킬 정리해줘", "이 작업에 뭐 쓸래" 를 물을 때, 또는 작업 시작 시 어떤 스킬을 켤지 판단이 필요할 때 사용. Also use when asked which skills/plugins improve agent capability, or to audit the installed skill stack.
---

# Agent Boost : 성능 라우팅표

**결론: 성능 저하의 원인은 스킬이 없어서가 아니라 이미 깔린 스킬이 발동을 안 해서다.**
아래 표대로 작업 시작 시 스킬을 먼저 켠다. 미설치 추천은 §3.

기준일: 2026-07-30 (설치 실측)

---

## 1. 상황 → 켤 스킬 (설치 완료, 지금 쓸 수 있음)

### 코드·개발
| 상황 | 켤 것 | 왜 |
|---|---|---|
| 새 기능/앱 만들기 시작 | `superpowers:brainstorming` → `superpowers:writing-plans` | 요구사항 안 물어보고 짜다가 갈아엎는 것 방지 |
| 버그·장애·이상동작 | `superpowers:systematic-debugging` | 증상만 때우는 패치 방지 (근본원인 강제) |
| 로직 구현 | `superpowers:test-driven-development` | 못 돌려본 코드를 "된다"고 말하는 것 차단 |
| "다 됐다" 말하기 직전 | `superpowers:verification-before-completion` | 실행 근거 없는 완료 보고 차단 |
| 커밋 직전 | `/code-review` (CLAUDE.md 필수 규칙) | |
| 과설계·비대해진 코드 | `ponytail:ponytail-review` / `ponytail:ponytail-audit` | 지울 것만 찾는 리뷰 |
| 같은 버그 2~3회 반복 실패 | `codex:codex-rescue` | 다른 모델 시각 (Claude 혼자 못 푸는 구간) |
| 결제·인증·정산 핵심 코드 | Codex 교차리뷰 (code-review와 이중 금지) | |
| 독립 작업 3개 이상 | `superpowers:dispatching-parallel-agents` | |
| 기존 코드 파악 | `feature-dev:code-explorer` / `claude-mem:smart-explore` | 파일 전체 읽기보다 토큰 절약 |

### 리서치·정보
| 상황 | 켤 것 |
|---|---|
| X·레딧·유튜브·네이버·쿠팡 차단 | `insane-search:insane-search` |
| "저번에 어떻게 했었지?" | `claude-mem:mem-search` |
| 정부 지원사업·공모 조사 | `ir-search` |
| Claude API·모델 관련 코드 | `claude-api` (기억으로 답하지 말 것) |

### 산출물
| 상황 | 켤 것 |
|---|---|
| 프론트·HTML·대시보드 | `impeccable` + `design-taste-frontend` (둘이 가장 두꺼움, 각 22KB). 단 브랜드 규칙이 스킬 기본값을 항상 덮음 |
| 고급/에이전시 톤, 미니멀 톤 | `high-end-visual-design` / `minimalist-ui` / `industrial-brutalist-ui` |
| 기존 사이트 업그레이드 | `redesign-existing-projects` |
| 차트·그래프·대시보드 수치 | `dataviz` |
| 발표덱·교안·제안서 | `slides-grab` (plan→design→export) |
| 인스타 카드뉴스 | <브랜드>은 기존 `cardnews-studio` 유지 |
| 대외 텍스트 (지원서·메일·블로그) | `humanizer` |
| 코드가 중간에 잘림 | `full-output-enforcement` |
| 클립보드 캡처 | `dd` |

### 마케팅 (coreyhaines31/marketingskills 48개, 2026-08-23 설치)
| 상황 | 켤 것 |
|---|---|
| 새 클라 착수 (컨텍스트 먼저) | `product-marketing` → `.agents/product-marketing.md` 생성, 나머지가 참조 |
| 진단·제안서 | `seo-audit` `cro` `competitor-profiling` `marketing-plan` `customer-research` |
| 카피 | `copywriting` `copy-editing` `offers` `marketing-psychology` |
| 콘텐츠·SEO | `content-strategy` `social` `ai-seo`(GEO/AEO) `programmatic-seo` `schema` `site-architecture` |
| 광고 | `ads` `ad-creative` `ab-testing` |
| CRM·리텐션 | `emails` `sms` `referrals` `churn-prevention` `launch` |
| 보고 | `analytics` `attribution` + `dataviz` |
| 영업 | `prospecting` `cold-email` `lead-magnets` `pricing` `sales-enablement` |
| 반복 루프 | `marketing-loops` (+ `/schedule`) |
| 관점 여러 개 | `marketing-council` |

### 반복·자동화
| 상황 | 켤 것 |
|---|---|
| 같은 실수 반복 → 물리적으로 막기 | `hookify:hookify` (훅 = Claude가 아니라 하네스가 강제) |
| 새 스킬 만들기·기존 스킬 개선 | `skill-creator` |
| 주기적 반복 실행 | `/loop` (로컬) / `/schedule` (클라우드 크론) |
| 권한 프롬프트 잦음 | `fewer-permission-prompts` |

---

## 2. 발동 실패 대책

스킬은 description 매칭으로 켜진다. 안 켜졌으면 **작업 문장에 트리거 단어를 넣어 말한다**:

- "이거 만들어줘" (X) → "브레인스토밍부터 하고 만들어줘" (O)
- "안 되는데?" (X) → "체계적으로 디버깅해줘" (O)
- "예쁘게 해줘" (X) → "impeccable로 다듬어줘" (O)
- 아니면 그냥 `/스킬명` 직접 호출

**Claude 쪽 규칙**: 위 표의 상황에 해당하면 사용자가 스킬명을 말하지 않아도 먼저 켜고 "Using [스킬] to [목적]" 한 줄 announce.

---

## 3. 미설치 추천 (성능 실질 상승 순)

`/plugin` 메뉴에서 설치하거나 `/plugin install <이름>@claude-plugins-official`.

| 우선 | 플러그인 | 무엇을 해결하나 |
|---|---|---|
| 1 | **context7** | 라이브러리 최신 문서를 실시간 조회. CLAUDE.md 코드규칙 1번("API·플래그 추측 금지")을 도구로 강제 = 환각 API 억제. 마켓: 별도(Upstash), 공식 목록에 있음 |
| 2 | **pyright-lsp** | 파이썬 타입체크·심볼 탐색을 LSP로. 02-scripts 파이썬 수정 시 존재하지 않는 함수·인자 실수 즉시 감지 |
| 3 | **claude-md-management** | CLAUDE.md 품질 감사 + 세션 학습 자동 적립. 현재 전역+프로젝트 CLAUDE.md가 매우 길어 규칙 충돌·낡은 항목 누적 위험 |
| 4 | **claude-code-setup** | 이 워크스페이스에 맞는 훅·스킬·MCP·서브에이전트를 분석해 추천. 세팅 자체를 자동 튜닝 |
| 5 | **browser-use** | 실제 Chrome 제어. 현재 CDP + 파이썬 스니펫 수작업(슬랙·채널톡·네이버)을 대체 가능. ※ 기존 검증된 CDP 절차가 있으니 신규 사이트에만 시험 적용 |
| 6 | **security-guidance** | 편집 시 패턴 경고 + Stop 시 diff 보안 리뷰. 운영 안전 규칙(권한검사·soft delete)과 결 맞음 |
| 7 | **typescript-lsp** | 01-apps 프론트 작업 시 2번과 동일 효과 |

**설치 안 함 (중복·불필요)**
- `code-simplifier` → `/simplify` + ponytail로 커버됨
- `ralph-loop` → `/loop` 있음
- `pr-review-toolkit` → `code-review` 플러그인과 겹침
- `mattpocock-skills` → superpowers TDD/리뷰와 겹침
- `session-report`·`receipts`·`project-artifact` → 리포팅용, 성능과 무관
- 나머지 260여 개는 SaaS 커넥터(Airtable·Shopify·AWS 등). 그 서비스를 실제로 쓸 때만

---

## 4. 이미 깔렸는데 안 쓰는 것 (아까움)

- `superpowers:subagent-driven-development` — 계획서를 서브에이전트로 병렬 실행. 큰 작업 컨텍스트 보호
- `superpowers:using-git-worktrees` — 격리 작업공간. 멀티디바이스 규칙 9번(기기당 세션 1개)과 궁합
- `claude-mem:learn-codebase` — 낯선 레포 착수 시 전체 정독 프라이밍
- `claude-mem:make-plan` + `claude-mem:do` — 단계별 계획 → 서브에이전트 실행
- `hookify` — "반복 지적"이 메모리에 여러 건 있는데 훅으로 안 막고 있음. 프롬프트보다 훅이 확실

---

## 5. 유지보수

플러그인 추가·제거 시 이 파일과 `<프로젝트>/00-core\04-docs\SKILLS_GUIDE.md` 양쪽 갱신.
실측 방법: `~/.claude/plugins/installed_plugins.json`, `ls ~/.claude/skills/`
