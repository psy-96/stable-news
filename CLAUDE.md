# CLAUDE.md — stable-news

## Project Overview
스테이블코인 전문 뉴스 인텔리전스 서비스.
파이프라인: RSS/스크래핑 → Claude API → Supabase → 이메일/웹/SMS

---

## Branch Strategy
- `main` — 직접 push 금지. PR + 리뷰 후 머지만 허용
- `feat/*` — 기능 개발
- `fix/*` — 버그 수정
- `chore/*` — 설정, 의존성, 문서

---

## Absolute Rules (위반 금지)

1. **RLS 없이 테이블 생성 금지** — 모든 Supabase 테이블은 RLS 정책 필수
2. **main 직접 push 금지** — 반드시 PR 경유
3. **API 키·시크릿 코드에 직접 삽입 금지** — Vercel 환경변수 또는 `.env.local`만 사용
4. **Telethon 사용 금지** — Telegram은 공식 Bot API only
5. **스키마 변경 시 반드시 migration 파일 생성** — `supabase/migrations/` 경로

---

## Pipeline Rules

### Claude API 호출
- 모든 프롬프트 입력은 XML 태그로 감싸기 (injection 방지)
  ```
  <article>...</article>
  <instruction>...</instruction>
  ```
- 모델: `claude-sonnet-4-20250514` 고정 (임의 변경 금지)
- 프롬프트 파일 위치: `prompts/` 디렉토리

### Supabase
- `ON CONFLICT DO NOTHING` — 첫 번째 분류 결과 우선
- `config` 테이블로 임계값 관리 — 하드코딩 금지
- 스키마 변경 = migration 파일 필수

### Vercel Cron
- cron 라우트에 `maxDuration = 300` 필수
- 중복 발송 방지: `digest_sends` UNIQUE on `digest_date` 활용

### DART API
- 1일 콜 상한: ~800
- 900 근접 시 circuit breaker 작동, 이후 콜 중단

### SMS (Solapi)
- 문구: "긴급 알림" 사용 ("실시간" 금지 — 30–120s 지연 존재)
- 1일 최대 3회 (`config` 테이블에서 조정)

---

## Error Handling
- 파이프라인 실패 시 `pipeline_runs` 테이블에 에러 기록
- watchdog: `last_sent_at` 25시간 초과 시 빌더에게 알림
- 수집 기사 0건: "오늘 수집된 뉴스 없음" 이메일 자동 발송

---

## Test
- Claude 분류 품질: `eval/classify_article_eval.ts` (테스트 20개) Phase 1 포함
- 새 소스 추가 시 eval 케이스 추가 필수

---

## .env.example 필수 항목
```
ANTHROPIC_API_KEY=
SUPABASE_URL=
SUPABASE_SERVICE_ROLE_KEY=
RESEND_API_KEY=
SOLAPI_API_KEY=
DART_API_KEY=
TELEGRAM_BOT_TOKEN=
CRONITOR_API_KEY=
```