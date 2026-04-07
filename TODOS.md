# TODOS

Deferred items from CEO review. Revisit triggers noted per item.

---

## Monetization model

Freemium design: paywall architecture, premium tier features, pricing in KRW, Toss Payments
or Stripe integration.

**Revisit trigger:** 5+ operators using the digest without being asked, with unprompted
positive feedback. Do not design the paywall before this signal exists.

**Notes:** 12-month subscriber target revised to 20-30 (not 50-100). Adjacent buyer pool
(bank compliance teams, law firm fintech practices, VC crypto desks) expands the TAM beyond
the ~10 serious KRW stablecoin issuers operating in 2026. Map adjacent buyers before
designing tier structure.

---

## Adjacent audience research

Map the buyer pool beyond stablecoin issuers: bank fintech compliance teams, law firms with
crypto practices, VC funds with KRW stablecoin portfolio companies.

**Revisit trigger:** Before Phase 2 public launch. This shapes the email signup form copy
and the web archive SEO keyword targeting.

---

## Source credibility tracking

Instrument which sources pilot users cite back in conversation or email replies. Surface
high-credibility sources more prominently in the digest sort order.

**Revisit trigger:** Post-Phase 2, once Resend webhook tracking is active.

**Notes:** Enable Resend open/click tracking from Day 1 (zero additional engineering cost
at setup time) — this captures the data needed for post-Phase-2 credibility analysis.

---

## Archive search

Full-text search across all processed articles. Likely Supabase FTS or Algolia integration.

**Revisit trigger:** Post-Phase 2, after web archive has at least 30 days of content.

---

## Competitive intelligence dashboard

Track which companies are mentioned in stablecoin news, how often, and in what context.
Surface for operator audience as a "who's moving" signal.

**Revisit trigger:** Post-Phase 3. Requires at least 60 days of structured article data.

---

## KakaoTalk Business API

Individual messaging via KakaoTalk requires Business API (4-8 weeks approval, Korean
business entity required). Do not attempt without a registered legal entity.

**Revisit trigger:** If/when a Korean business entity is registered.

---

## Expert/beginner classification in web archive

The `expert_level` field (전문 / 입문) is in the schema but hidden in Phase 2 UI. Build
the classification prompt and show the tag in Phase 3.

**Revisit trigger:** Phase 3 — before this ships, draft and eval the `입문/전문`
discrimination prompt with 20 test articles.

---

## Weekly deep-dive publish UX

The "Publish" link delivered via Resend needs: (a) time-limited signed URL (24h expiry),
(b) inline edit capability before publish, (c) preview of the email as it will appear to
subscribers.

**Revisit trigger:** Phase 3C engineering — spec the full publish flow before building.

---

## Alert threshold calibration

After Phase 3A launches and 10+ SMS alerts have been sent, review the urgency_score distribution in `alert_log` and interview pilot recipients on false-positive rate.

**Current default:** `urgency_alert_threshold = 0.8` in the `config` table (tunable without code deploy).

**Context:** Claude Haiku scores are not empirically grounded. 0.8 may be too loose (too many SMS) or too tight (misses real regulatory signals). C-level operators who receive 3+ false positives will opt out. Tune before expanding beyond the pilot cohort.

**Revisit trigger:** 10+ alert_log entries exist for a given recipient. Pull score distribution from alert_log. Interview pilots. Adjust config table. No code deploy needed.

---

## Service name A/B test format

Test 3 name candidates with 2-3 pilot operators. Standardize the test: show each name with
a one-sentence positioning statement, ask two questions: (1) "Does this feel like intelligence
or news?" and (2) "Would you forward a link with this name to your board?".

Current front-runners: 스테이블레이더, 코어 브리핑, 스테이블넷, 원화인텔.

**Revisit trigger:** During Week 1, in parallel with Phase 1 build. Domain reserved before
Phase 2 goes live (not before Phase 2 build starts).
