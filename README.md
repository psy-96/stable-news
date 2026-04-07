# stable-news

> 원화 스테이블코인 + 글로벌 스테이블코인 전문 인텔리전스 서비스

---

## Overview

국내 유일의 스테이블코인 특화 뉴스 서비스. AI 파이프라인이 주요 소스를 자동 수집·요약하고, 전문가와 입문자 모두를 아우르는 콘텐츠를 주 1회 이상 발행한다.

**포지셔닝**: 쟁글·코인니스 같은 종합 크립토 미디어가 아닌, 스테이블코인 이슈만 깊게 다루는 버티컬 인텔리전스 서비스.

---

## Product Scope

### 콘텐츠 축

| 축 | 내용 |
|---|---|
| 원화 스테이블코인 | WEMIX, iST, KRUST 등 국내 발행 KRW 스테이블코인 |
| 글로벌 스테이블코인 | USDC, USDT, DAI, USDe 등 |
| 규제·정책 | FSC, DART, MiCA, 미국 STABLE Act 등 |
| 입문자 콘텐츠 | 용어 해설, 사건 쉬운 설명, 개념 시리즈 |

### 타겟

- 1차: 스테이블코인 프로젝트 팀, 거래소 리스크팀
- 2차 (Phase 3+): 은행 핀테크팀, 법무법인, VC 심사역, 컴플라이언스 담당자

### 12개월 목표

- 유료 구독자 20–30명 (₩200K–300K/월)
- 인접 타겟(컴플라이언스·법무·VC)으로 TAM 자연 확장

---

## Phases

### Phase 1 — 파이프라인 + 이메일 다이제스트 (MVP)

- RSS + 스크래핑 자동 수집
- Claude API로 요약 및 분류
- Supabase 저장
- 주 1회 이메일 다이제스트 자동 발송 (Resend)
- 내부 watchdog + 외부 모니터링 (Cronitor/UptimeRobot)
- **UI 없음** — 이메일 전용

### Phase 2 — 웹 아카이브

- Next.js 뉴스 카드 피드
- 카테고리 필터 (원화 / 글로벌)
- 기사 상세 페이지 (full 요약 + 원문 링크)
- `/plan-design-review` 후 착수

### Phase 3 — 알림 + 딥다이브

- **3A**: 규제 실시간 알림 (FSC 1시간 폴링, DART OpenAPI 15분 폴링, Solapi SMS)
- **3B**: Telegram 공식 Bot API 커버리지 (공식 Bot API only, Telethon 불가)
- **3C**: 주 1회 금요일 딥다이브 (Claude 초안 + 상윤 편집 30분 + Resend 발송)

---

## Architecture

```
[수집]         [처리]              [저장]       [발행]
RSS/Scraping → Claude API 요약  → Supabase  → 이메일 (Phase 1)
               분류·태깅                     → 웹 피드 (Phase 2)
                                             → SMS/Telegram (Phase 3)
```

### 파이프라인 상세

- **스케줄**: Vercel Cron (Pro plan, `maxDuration=300`)
- **중복 방지**: `digest_sends` UNIQUE on `digest_date` — Vercel retry에도 중복 발송 없음
- **watchdog**: `pipeline_runs` 테이블 기반, `last_sent_at` 2시간마다 체크
- **빈 날**: "오늘 수집된 뉴스 없음" 이메일 자동 발송
- **SMS 상한**: 1일 3회, `config` 테이블에서 코드 배포 없이 조정

---

## Tech Stack

| 레이어 | 선택 |
|---|---|
| 프레임워크 | Next.js 15 |
| DB | Supabase (RLS Phase 1부터 적용) |
| AI | Claude API (Sonnet — 요약·분류) |
| 이메일 | Resend |
| SMS | Solapi |
| 배포 | Vercel Pro |
| 모니터링 | Cronitor 또는 UptimeRobot |

---

## Database Schema (Phase 1)

7개 테이블: `sources`, `articles`, `digests`, `subscribers`, `pipeline_runs`, `digest_sends`, `config`

Phase 3 추가: `reports`, `alert_log`, `alert_recipients`

> 전체 DDL: `supabase/migrations/001_phase1_schema.sql`, `002_phase3_schema.sql`

### 설계 원칙

- 모든 테이블에 RLS 정책 필수 (Phase 1부터)
- `ON CONFLICT DO NOTHING` — 첫 번째 분류 결과 우선
- `config` 테이블 — 임계값·상한을 DB에서 런타임 조정
- one-time-use publish token (딥다이브 발행 엔드포인트)

---

## Security

- Supabase RLS: 모든 테이블 Phase 1부터 적용
- Claude API 프롬프트: XML 태그 입력으로 injection 방지
- 딥다이브 발행: one-time-use token (24h signed URL 대신)
- 환경변수: Vercel 환경변수로 관리, 코드에 직접 노출 금지

---

## Constraints & Rules

- Telegram: 공식 Bot API only (Telethon 사용 금지 — ToS 위반)
- DART API: 1일 ~800콜 상한, 1,000 근접 시 circuit breaker 작동
- SMS: "긴급 알림" 문구 사용 ("실시간" 금지 — 30–120s 지연 존재)
- main 브랜치 직접 push 금지
- RLS 없이 테이블 생성 금지

---

## Operational Cadence

| 주기 | 내용 | 담당 |
|---|---|---|
| 자동 (주 1회+) | 뉴스 수집·요약·이메일 발송 | 파이프라인 |
| 매주 금요일 30분 | 딥다이브 초안 검토 및 발행 | 상윤 |
| 수시 | 소스 추가/제거 (`sources` 테이블) | 상윤 |

---

## Development Workflow

```
GitHub 세팅 완료
    ↓
CLAUDE.md 작성 (Harness Engineering 적용)
    ↓
Day 0 소스 오딧 (크롤링 가능 소스 확정)
    ↓
/ship — Phase 1 구현
    ↓
Phase 2 착수 전 /plan-design-review
```

---

## TODOS

> 상세 내용: `TODOS.md`

- [ ] 서비스명 확정 (현재 placeholder: `stablecoin-news`)
- [ ] 수익화 모델 설계
- [ ] 인접 타겟(컴플라이언스·VC) 리서치
- [ ] 소스 신뢰도 트래킹
- [ ] 아카이브 검색 기능
- [ ] 경쟁 인텔리전스 대시보드
- [ ] KakaoTalk Business API 검토
- [ ] 전문/입문 태그 UI (Phase 3에서 활성화)
- [ ] 딥다이브 발행 UX 스펙
- [ ] 서비스명 A/B 테스트 포맷

---

## Review History

| 날짜 | 리뷰 | 결과 |
|---|---|---|
| 2026-04-07 | CEO Review | APPROVED with revisions |
| 2026-04-07 | Eng Review | APPROVED — Phase 1 블로커 0 |

---

*placeholder name: stablecoin-news | 서비스명 확정 시 find-replace (~30분)*