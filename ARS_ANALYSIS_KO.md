# Academic Research Skills (ARS) 전수조사 분석 및 활용·수익화 정리

> 작성일: 2026-09-19
> 대상 저장소(포크): https://github.com/bmshin94/academic-research-skills
> 원본 저장소(upstream): https://github.com/Imbad0202/academic-research-skills
> 분석 시점 버전: **v3.22.0** (2026-09-16)
> 라이선스: **CC BY-NC 4.0** (비상업적 용도만 / 오픈소스 라이선스 아님)
> DOI: [10.5281/zenodo.20696614](https://doi.org/10.5281/zenodo.20696614)

---

## 1. 한 줄 정의

> **Claude Code 위에서 동작하는, 논문 한 편을 처음부터 끝까지 사람과 함께 쓰는 AI 연구 파이프라인 스킬 묶음.**

제작자는 대만 연구자 **Cheng-I Wu (吳政宜)**. "AI는 조종사가 아니라 부조종사(copilot, not pilot)"라는 명시적 철학 위에 설계되어 있으며, 완전 자율 논문 생성은 **의도적으로 거부된 기능**으로 문서에 기록되어 있다.

---

## 2. 저장소 규모 (실측)

| 항목 | 수치 |
|---|---|
| 전체 추적 파일 | 2,742개 |
| Markdown (프롬프트/문서) | 1,087개 / 194,501줄 |
| Python (검증 스크립트·테스트) | 444개 / 266,842줄 |
| `scripts/` 파일 수 | 432개 |
| 스킬 | 4개 |
| 모드 | 27개 |
| 프롬프트 역할(에이전트) | 39개 (판단형 26 / 실행형 13) |
| 슬래시 커맨드 | 16개 (`/ars-*`) |
| CI 워크플로 | 14개 |
| 다국어 README | 6개 (EN / 繁中 / 简中 / 日 / **한국어** / ES) |

> 핵심: 코드량이 20만 줄을 넘지만 **실행 로직은 소수**다. 대부분은 "AI에게 주는 지시문(프롬프트)"과 "그 지시문이 드리프트하지 않았는지 검사하는 린트"다. 즉 **프롬프트를 소프트웨어처럼 엔지니어링한 프로젝트**다.

---

## 3. 구조: 4개 스킬

```
deep-research (v2.12.1)           연구 엔진        에이전트 13 / 모드 8
        ↓
academic-paper (v3.3.1)           집필 엔진        에이전트 12 / 모드 11
        ↓
academic-paper-reviewer (v1.11.1) 모의 피어리뷰     에이전트 7  / 모드 6
        ↓
academic-pipeline (v3.22.0)       총괄 오케스트레이터 에이전트 5  / 10단계
```

### 3.1 deep-research — 연구 설계 & 문헌조사
- 13개 에이전트: 연구질문 설계, **소크라테스 멘토**, 방법론 설계, 서지, 출처 검증, 종합, 편향 위험(RoB), 메타분석, 보고서 컴파일, 편집장, **악마의 변호인**, 윤리 심사, 문헌 모니터링
- 주요 모드: `full`(APA 7.0 보고서 3,000~8,000단어), `socratic`(답을 주지 않고 질문으로 연구질문을 다듬음), `systematic-review`(PRISMA 2020), `fact-check`, `three-way-scan`, `quick`, `review`, `lit-review`

### 3.2 academic-paper — 논문 작성
- 12개 에이전트: 인테이크, 문헌 전략, 구조 설계, 초안 작성, 논증 구축, 시각화, 인용 준수, 이중언어 초록, 포매터, 동료심사, 개정 코치, 소크라테스 멘토
- 11개 모드: `full`, `plan`, `outline-only`, `revision`, `revision-coach`, `abstract-only`, `lit-review`, `format-convert`(LaTeX/DOCX/PDF), `citation-check`, `disclosure`(venue별 AI 사용 고지문), `rebuttal-audit`
- **Style Calibration**: 사용자의 과거 논문 3편 이상을 주면 문체(문장 리듬, 어휘 선호, 인용 통합 방식)를 학습해 소프트 가이드로 적용

### 3.3 academic-paper-reviewer — 모의 피어리뷰
- **5인 패널**: 저널 적합성 심사위원 + 분야 자동 감지 기반 동적 심사위원 3명 + **고정 악마의 변호인**
- 산출물: 심사보고서 5부 + Editorial Decision Letter(Accept/Minor/Major/Reject) + Revision Roadmap
- `calibration` 모드: 심사기 자신의 **FNR/FPR·균형정확도를 측정**하는 자기검증 모드

### 3.4 academic-pipeline — 10단계 총괄
```
1.연구 → 2.집필 → [2.5 무결성 게이트] → 3.심사 → 3→4 개정코칭
 → 4.수정 → 3'.재심사 → 3'→4' 잔여이슈 코칭 → 4'.재수정
 → [4.5 최종 무결성 게이트] → 5.포맷 확정 → 6.연구과정 기록서
```
- **결정형 체크포인트 10개 + 승인형 체크포인트 2개.** AI가 혼자 다음 단계로 진행 불가.
- 단계 간 상태는 **Material Passport**(YAML 원장)로 전달 → 세션이 끊겨도 `resume_from_passport=<hash>`로 재개 가능

---

## 4. 차별점의 핵심 — 무결성 게이트

배경: Zhao et al.(2026, arXiv:2605.07723)이 arXiv/bioRxiv/SSRN/PMC의 250만 편·참고문헌 1억 1,100만 건을 감사한 결과, **2025년 한 해에만 환각 인용 약 146,932건**(보수 추정), bioRxiv→PMC 쌍에서 **프리프린트 환각의 85.3%가 정식 출판까지 존속**.

ARS의 대응:

1. **4개 인덱스 교차검증** — Semantic Scholar + OpenAlex + Crossref + arXiv에 **실제 API 호출**로 존재 여부 확인 (LLM에게 묻지 않는 **결정론적** 게이트)
2. **삼각/사각 검증 매트릭스** — 불일치 인덱스 수 k=0~4에 따라 `CONTAMINATED-COVERAGE-NOISE` / `-PARTIAL-UNMATCH` / `-TRIANGULATION-UNMATCHED` / `-QUADRANGULATION-UNMATCHED` 자문 신호 부여
3. **3층 인용 앵커** — 모든 인용에 `<!--ref:slug-->` + `<!--anchor:page|quote|section:value-->`를 강제. 로케이터 없는 인용은 하드 게이트에서 거부
4. **주장–출처 정합성 감사** (`ARS_CLAIM_AUDIT=1`) — 인용이 실제로 그 주장을 뒷받침하는지 별도 검사, HIGH-WARN 5개 클래스는 출력 거부
5. **7가지 AI 연구 실패모드 체크리스트** — 구현 버그, 환각 결과, 지름길 의존, 버그의 인사이트化, 방법론 날조, 프레임 고착, 인용 환각을 Stage 2.5/4.5에서 차단
6. **반아첨(anti-sycophancy)** — 악마의 변호인이 사용자 반박을 1~5점으로 채점, **4점 미만이면 양보 금지**; 연속 양보 금지 + 양보율 추적
7. **정책 계층** — `advisory`(기본) / `strict` / `strict_articles_only` 옵트인. 기본값은 이전 버전과 바이트 동등

### 실제 실행 결과 (examples/showcase/)
| 산출물 | 내용 |
|---|---|
| Integrity Report Stage 2.5 | **날조 참고문헌 15건 + 통계 오류 3건** 검출 |
| Integrity Report Stage 4.5 | 회귀 0건 확인 |
| Post-Publication Audit | **3라운드 무결성 체크가 놓친 21/68건**을 사후 감사에서 발견 (자기 한계 공개) |

---

## 5. 명시적으로 거부한 기능 (POSITIONING.md)

- 종단간 완전 자율 연구 파이프라인
- AI의 자율적 연구 아이디어/가설 생성
- 논문 → 슬라이드/포스터/영상 자동 변환 (Paper2X)
- AI의 자율 실험 실행/코딩
- 물리적 실험실 자동화 API
- **가상 IRB(연구윤리위원회) 시뮬레이션**
- **"논문 편수"를 성과 지표로 삼는 것**

또한 "실행 검증의 한계"를 직접 명시한다:
> *"일관되게 보고된 조작 데이터는 ARS의 모든 게이트를 통과할 수 있다."*

근거로 인용된 연구: Lu et al.(2026, *Nature* 651:914-919), Zhao et al.(2026), Kong et al.(2026), Ren et al.(2026), Gartenberg et al.(2026, *Organization Science* 37(3)), Wang & Li et al.(2026), Brodeur et al.(2026, *PNAS* 123(22)) — **AI 주도 재현 37% vs 인간/AI보조 94%·91%**.

---

## 6. 설치 및 사용법

### 방법 0 — 플러그인 (권장, v3.7.0+)
```text
/plugin marketplace add Imbad0202/academic-research-skills
/plugin install academic-research-skills
```

### 방법 1 — 심볼릭 링크 (전통 방식)
```bash
git clone https://github.com/Imbad0202/academic-research-skills.git ~/academic-research-skills
cd /path/to/your/project
mkdir -p .claude/skills
ln -s ~/academic-research-skills/deep-research           .claude/skills/deep-research
ln -s ~/academic-research-skills/academic-paper          .claude/skills/academic-paper
ln -s ~/academic-research-skills/academic-paper-reviewer .claude/skills/academic-paper-reviewer
ln -s ~/academic-research-skills/academic-pipeline       .claude/skills/academic-pipeline
```

### 기타 설치 경로
| 방법 | 대상 |
|---|---|
| Method 2 | 저장소 자체를 프로젝트로 사용 |
| Method 3 | Claude Cowork (데스크톱, zip 업로드) |
| Method 4 | claude.ai 웹 Project 업로드 |
| Method 5 | Claude Science — Skills → Import from GitHub |
| Pi | `pi install git:github.com/Imbad0202/academic-research-skills` |
| Codex CLI | 별도 배포판 `academic-research-skills-codex` |

### 선택적 준비물
| 목적 | 필요 |
|---|---|
| Markdown 출력만 | 없음 |
| DOCX | Pandoc |
| PDF (APA 7.0) | tectonic + Source Han Serif TC |
| 쓰기범위 가드 훅·일부 커맨드 | 실제 Python (Windows는 python.org 버전) |
| Windows | **Git Bash 필수** |

### 사용법
자연어로 요청하면 스킬/모드가 자동 라우팅된다. 한국어 트리거 키워드 정식 지원("심층 연구", "논문 심사", "연구 방향을 잡아줘", "심사 의견을 받았어" 등).
슬래시 커맨드 16개: `/ars-full`, `/ars-plan`, `/ars-outline`, `/ars-reviewer`, `/ars-lit-review`, `/ars-abstract`, `/ars-citation-check`, `/ars-revision`, `/ars-revision-coach`, `/ars-rebuttal-audit`, `/ars-disclosure`, `/ars-format-convert`, `/ars-3w`, `/ars-mark-read`, `/ars-unmark-read`, `/ars-cache-invalidate`

한국어 README: [`README.ko-KR.md`](README.ko-KR.md)

---

## 7. 플러그인 / 스킬 / MCP — 정체 구분

| 형태 | 해당 | 근거 |
|---|---|---|
| **Skill** | O (본질) | 4개 디렉터리 각각 `SKILL.md` 보유 |
| **Plugin** | O (포장) | `.claude-plugin/plugin.json` + `marketplace.json` |
| **MCP** | **X** | MCP 서버·프로토콜 구현 없음 |
| Subagent | 일부 | `agents/` 3개만 플러그인 노출, 나머지 36개는 인라인 |
| Hooks | O | `SessionStart`(로드 안내), `PreToolUse`(쓰기범위 가드) |

**결론: "스킬 4개를 플러그인으로 포장한 것". MCP와는 계층이 다르다.** (MCP=외부 도구 연결 프로토콜, Skill=작업 수행 지시문)

---

## 8. API 토큰 요구사항

### 필수
| 토큰 | 용도 |
|---|---|
| `ANTHROPIC_API_KEY` | Claude Code 구동 (Claude Pro/Max 구독 로그인으로 대체 가능) |

### 논문 DB 검증 — 전부 키 불필요
| 리졸버 | 엔드포인트 | 키 |
|---|---|---|
| Semantic Scholar | `api.semanticscholar.org` | 선택(`S2_API_KEY`, 레이트리밋 완화) |
| OpenAlex | `api.openalex.org` | 선택(`OPENALEX_API_KEY`) |
| Crossref | `api.crossref.org` | 불필요 |
| arXiv | `export.arxiv.org` | 불필요 |

> 문서 명시: 게이트를 키 없이 유지하는 것은 **의도적인 재현성 선택**.

### 선택 (기본 OFF)
| 토큰 | 용도 |
|---|---|
| `OPENAI_API_KEY` | 교차모델 검증 (`ARS_CROSS_MODEL`) |
| `GOOGLE_AI_API_KEY` | Gemini 교차검증 |
| `ARS_OPENAI_COMPAT_API_KEY` | DeepSeek/MiMo 등 OpenAI 호환 엔드포인트 |
| Codex CLI 로그인 | ChatGPT 구독 기반 인용 검증 전송로 |

### 비용 추정 (docs/PERFORMANCE.md)
| 모드 | 추정 비용 |
|---|---|
| reviewer quick | ~$0.30 |
| deep-research full | ~$1.20 |
| academic-paper full | ~$1.80 |
| **풀 파이프라인 10단계** | **~$4–6** (Opus 4.x 기준) / **~$7** (Fable 5.1 기준 재산출) |
| + 교차모델 검증 | +$0.60~1.10 |

### 프라이버시
- 텔레메트리 0 / 애널리틱스 0 / ARS 계정 불필요
- 외부 전송: DOI·arXiv ID·제목 쿼리 문자열 (저자·연도는 로컬 매칭)
- **원고 본문 전송은 교차모델 검증을 켜고 세션별 명시 동의했을 때만**
- 로컬 캐시: `~/.cache/ars/verification.db` (SQLite, 90일 TTL)
- 전체 네트워크 지도 + 차단 방법: `docs/DATA_FLOWS.md`

---

## 9. GitHub에서 유명한 이유

> 주: 본 세션에서는 GitHub API 접근이 차단되어 별 개수를 직접 확인하지 못했다. 외부 집계 사이트 기준 **약 45.9k~48.6k ⭐ / 글로벌 랭크 #574** 수준으로 보고된다 (원본 저장소 기준, 미검증 수치).

1. **타이밍** — Claude Code Skills/Plugin 생태계 초기에 "스킬을 이 정도 깊이로 만들 수 있다"를 증명한 레퍼런스
2. **시장 규모** — 전 세계 대학원생·연구자·교수
3. **정직함** — "이건 네 논문을 대신 써주지 않는다"로 시작. 자기가 놓친 21건 공개, `NOT_CALIBRATED`/`NOT_RUN` 라벨링
4. **학술적 근거 무장** — *Nature*, *PNAS*, *Organization Science*, arXiv 다수 인용 + Zenodo DOI 발급
5. **엔지니어링 완성도** — CI 14개, 스펙 린트 수백 개, 프롬프트 파일 sha256 콘텐츠 락, 뮤테이션 테스트, ISO/IEC 42001 정신 갭 진단·리스크 레지스터·거버넌스 문서
6. **국제화** — README 6개 언어 + 다국어 트리거 키워드
7. **유지보수 속도** — CHANGELOG 75만 바이트, v3.0→v3.22 수 주 단위 릴리스

---

## 10. 로컬 에이전트 구축에 주는 가치

"코드 재사용"이 아니라 **"설계 패턴 학습"** 관점에서 매우 높다.

| # | 패턴 | 위치 | 활용 |
|---|---|---|---|
| 1 | 핸드오프 스키마 | `shared/handoff_schemas.md`, `shared/contracts/**` | 에이전트 간 전달을 JSON Schema로 고정 |
| 2 | Material Passport | `academic-pipeline/references/passport_as_reset_boundary.md` | 체크포인트/재개 설계, SHA-256 해시 체인 |
| 3 | Degradation Registry | `shared/contracts/degradation_registry.json` | 실패클래스→저하상태→진단마커→소비자 인덱싱 |
| 4 | 3단계 정책 게이트 | advisory / strict / strict_articles_only | 새 검사 추가 시 안전한 롤아웃 전략 |
| 5 | 교차모델 핸드오프 봉투 | `scripts/cross_model_handoff.py` | 블라인드 판단 전달 + 일치/불일치 라우팅 |
| 6 | 쓰기범위 가드 훅 | `hooks/run_guard.sh`, `scripts/ars_write_scope_guard.py` | 서브에이전트 샌드박싱, graceful no-op |
| 7 | 프롬프트 드리프트 CI | `scripts/check_*.py` (432개) | 프롬프트 verbatim 핀 고정, 중복 규칙 충돌 탐지 |
| 8 | 모델 티어링 | `shared/model_tiering.md` | 판단형/실행형 분리 후 모델 티어 라우팅(비용 최적화) |

### 한계
- Claude Code 전용 구조 — LangGraph/CrewAI/AutoGen 이식 시 오케스트레이션 재작성 필요
- 19만 줄 프롬프트는 토큰 소모가 크고, 로컬 소형 모델은 지시문 준수율이 떨어질 가능성
- **CC BY-NC 4.0** — 상업적 활용 제약

---

## 11. React / PHP 구현 가능성

### 코드로 구현 가능한 부분 (약 40%)
| 기능 | 난이도 |
|---|---|
| 4개 DB 인용 검증 게이트 | 쉬움 (REST 호출) |
| 삼각/사각 검증 매트릭스 (k=0~4) | 쉬움 |
| SQLite 검증 캐시(90일 TTL) | 쉬움 |
| JSON Schema 계약 검증 | 쉬움 (`ajv` / `opis/json-schema`) |
| Material Passport YAML 파서 | 쉬움 |
| 10단계 상태머신 + 체크포인트 UI | 중간 (React가 오히려 우위) |
| 인용 포맷 변환 | 중간 |
| PDF/DOCX 내보내기 | 중간 (Pandoc/tectonic 셸아웃) |

### 불가능한 부분 (약 60%)
19만 줄 프롬프트는 **LLM이 있어야 의미가 있다.** 반드시 LLM API(Anthropic/OpenAI/로컬 Ollama 등) 연동 필요.

### 권장 아키텍처
```
React(Next.js)  ──REST/SSE──  Node(NestJS) 또는 Python(FastAPI)
  · 파이프라인 진행 UI              · 오케스트레이터(상태머신)
  · 체크포인트 승인 화면            · LLM 호출 래퍼
  · 인용 검증 대시보드              · 인용 검증 게이트(순수 코드)
  · 심사보고서 뷰어/diff            · 큐(BullMQ/Laravel Queue)
  · Passport 시각화                · Postgres + Redis
                                        │
                              LLM API + 4개 논문 DB API
```

| 스택 | 추천도 | 비고 |
|---|---|---|
| React + Node/TS | ★★★★★ | AI SDK·스트리밍·JSON Schema 생태계 우수 |
| React + Python(FastAPI) | ★★★★★ | ARS의 스크립트 자산 재활용 가능 |
| React + PHP(Laravel) | ★★★ | 가능하나 LLM/스트리밍 생태계 약함 |
| PHP 단독 | ★ | 비권장 |

---

## 12. 수익화

### 12.1 법적 경계선 (최우선)

`LICENSE` + `POSITIONING.md` 기준 **금지 용도**:
- ARS 기반 상업 SaaS / 호스팅 서비스
- ARS를 유료 상품으로 패키징한 컨설팅·프리랜싱
- 별도 라이선스 없는 기업/기관 유료 배포

**CC BY-NC 4.0** = 공유·개작 자유, 출처 표기 필수, **비상업적 용도만**. 오픈소스 라이선스가 아니다.

합법적 경로 3가지:
| 경로 | 설명 |
|---|---|
| A. 패턴만 학습 후 독자 구축 | 아이디어·구조는 저작권 대상이 아님. 프롬프트/코드 복사 없이 신규 작성 |
| B. 저자와 상업 라이선스 협의 | `GOVERNANCE.md`의 문의 경로 활용 |
| C. ARS를 판매하지 않는 사업 | 교육, 서비스, 주변 도구 |

### 12.2 아이디어 10선

#### 티어 1 — 즉시 가능, 라이선스 안전
| # | 아이디어 | 요지 | 가격 | MVP |
|---|---|---|---|---|
| 1 | **환각 인용 검증 SaaS (독자 구축)** | 원고 업로드 → 4개 DB 대조 → 미존재 인용 리포트. **API 키 불필요 = 원가 거의 0** | 건당 $5~20 / 기관 월 $200~ | 2~3주 |
| 2 | 교육 콘텐츠·강의 | ARS를 *교재로 소개*하는 것은 합법(BY 준수) | 강의 $50~200 / 워크샵 100~300만원 | 즉시 |
| 3 | 연구 워크플로 자문 | "ARS 패키징 판매"가 아닌 **자문·교육비 수취** 구조로 설계 필요(법률 검토 권장) | 프로젝트당 500~2,000만원 | 즉시 |

#### 티어 2 — 독자 구축 필요, 수익성 높음
| # | 아이디어 | 요지 | 가격 |
|---|---|---|---|
| 4 | **한국형 논문 파이프라인 SaaS** | ARS는 KCI/RISS/DBpia/ScienceON 미지원 → **시장 빈틈**. 한국어 인용 규칙·학위논문 양식 포함 | 학생 월 9,900원 / 연구실 월 99,000원 |
| 5 | AI 심사위원 서비스 | 투고 전 모의 피어리뷰 5인 패널. **자체 FNR/FPR 공개가 신뢰 무기** | 건당 $30~100 |
| 6 | **학위논문 양식 자동화** | ARS가 의도적으로 거부한 영역(기관별 format_profile) = **경쟁 없는 시장**. 국내 대학 양식 DB 구축 | 건당 3~5만원 / 대학 연간 라이선스 |

#### 티어 3 — 장기/고위험
| # | 아이디어 | 요지 |
|---|---|---|
| 7 | 연구 무결성 감사 B2B | 저널·출판사·연구윤리위 대상, 저널당 연 1,000만원~ |
| 8 | Claude Code 스킬 마켓플레이스 | ARS 자체 유료 재배포는 위반. 무료 디렉터리 + 유료 부가기능 구조 |
| 9 | 에이전트 신뢰성 툴킷 (오픈코어) | ARS 패턴을 범용 라이브러리로 **독자 구현**. 코어 MIT + 엔터프라이즈 유료, 시트당 월 $50~ |
| 10 | 저자와 상업 라이선스 파트너십 | 성사 시 위 아이디어 합법화 + 브랜드 활용 |

### 12.3 권장 로드맵
```
1개월차   #1 인용 검증 SaaS MVP      (원가 0, 라이선스 안전, 수요 증명됨)
3개월차   #2 교육 콘텐츠로 브랜딩·초기 유저 확보
6개월차   #6 학위논문 양식 자동화 추가 (경쟁 없는 시장)
12개월차  #4 한국형 풀 파이프라인으로 확장
병행      #10 저자와 라이선스 협의 (안전판)
```

### 12.4 핵심 통찰
1. **ARS가 "하지 않겠다"고 선언한 영역이 오히려 사업 기회다** — 기관별 양식, 사후 출판 추적, 논문 간 메모리, 자동화
2. **한국 학술 DB 미지원은 거대한 빈틈이다**
3. **"정직함"이 이 시장의 차별화 무기다** — ARS가 높은 스타를 받은 실질적 이유. 오탐률 공개가 곧 마케팅이 된다

---

## 13. 참고 링크

| 문서 | 경로 |
|---|---|
| 포크 저장소 | https://github.com/bmshin94/academic-research-skills |
| 원본 저장소 | https://github.com/Imbad0202/academic-research-skills |
| Codex 배포판 | https://github.com/Imbad0202/academic-research-skills-codex |
| 한국어 README | `README.ko-KR.md` |
| 빠른 시작 | `QUICKSTART.md` |
| 설치 전체 가이드 | `docs/SETUP.md` |
| 아키텍처 | `docs/ARCHITECTURE.md` |
| 데이터 흐름(네트워크/저장소) | `docs/DATA_FLOWS.md` |
| 리스크 레지스터 | `docs/RISK_REGISTER.md` |
| 성능·비용 | `docs/PERFORMANCE.md` |
| 모드 레지스트리 | `MODE_REGISTRY.md` |
| 포지셔닝(거부 기능·허용/금지 용도) | `POSITIONING.md` |
| 거버넌스(라이선스 문의 포함) | `GOVERNANCE.md` |
| 실행 결과 예시 | `examples/showcase/` |

---

*본 문서는 저장소 전수조사(파일 트리, 매니페스트, 4개 SKILL.md, 에이전트·스크립트·훅·CI·문서)를 근거로 작성되었다. 별 개수 등 외부 지표와 비용 추정치는 각 항목에 명시한 출처·한계를 따른다.*
