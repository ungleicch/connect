# Connect — Product Write-up

**Version:** 0.1 (planning)  
**Owner:** Tom Malter / Ungleicch  
**Linear:** https://linear.app/gorkbot/project/connect-2264dc8e786f  
**Repo:** https://github.com/ungleicch/connect

---

## 1. One-liner

Connect pairs you with a **random person somewhere in the world each day** for a **text-only** conversation. You know their **country** and optional taste signals (favorite **book**, **music**, **movie**). After about a day you can exchange socials and keep talking—or walk away with a cultural spark you would not have found otherwise.

---

## 2. Problem & insight

Cross-border friendship is usually filtered by language apps, dating apps, or algorithmically similar social graphs. There is little room for a **low-stakes, time-boxed, text-first** encounter with someone you would never meet—without photos, status games, or infinite scroll.

Connect is deliberately **not** a dating app: no profiles-as-ads, no swipes, no image-first first impression. The scarce resource is **attention for one day**.

---

## 3. Product principles

1. **Text only** — no images, voice, video, or files in-chat (v1).
2. **Scarce identity** — country (+ optional tastes); no real name required.
3. **Time-boxed** — ~24h match window; continuation is an explicit choice.
4. **Pause is first-class** — opt out for a period without losing the account.
5. **Culture over content** — books/music/movies as bridges, not content feeds.
6. **Reuse trust infrastructure** — prefer Telegram (or similar) for delivery and spam tooling rather than reinventing chat.

---

## 4. Core user journey (MVP)

1. **Onboard** via Telegram bot / Mini App (or chosen transport); confirm country; optionally set tastes.
2. **Daily match** — at a local-friendly time (or UTC batch), receive today's partner: country + tastes.
3. **Chat** text-only for ~24 hours; optional **AI translation** (original always available).
4. **End of day** — prompt: exchange social handles **or** close and keep the cultural tip (book/song/movie).
5. **Pause anytime** — skip matching for N days/weeks.

---

## 5. Feature catalog (with difficulty)

Difficulty scale used in Linear labels: **Easy · Medium · Hard · Very Hard**.

### 5.1 Matching & identity

| Feature | Diff | Notes |
|---------|------|-------|
| Country capture + display | Easy | IP geo + manual override; only country shown to partner |
| Fav book / music / movie fields | Easy | Normalize strings for theme-day equality |
| Profile + tastes data model | Medium | Pause flag, sub entitlement, match-history exclusions |
| Daily random matching engine | Hard | Fair pairing, timezones, exclusions, odd-one-out handling |
| Daily match notification | Easy | Push via Telegram with deep link |
| Pause / opt-out periods | Easy | Resume cleanly; no match while paused |
| Theme days (same book or song) | Medium | Scheduled matcher mode |
| Report & block | Medium | Exclude from future matches; mod queue |

### 5.2 Chat & transport

| Feature | Diff | Notes |
|---------|------|-------|
| Messaging architecture decision | Medium | Telegram-first vs custom vs Matrix |
| Telegram bot transport scaffold | Hard | Webhooks, privacy mode, media rejection |
| 24h text-only session lifecycle | Hard | Open/close, reminders, soft close |
| Enforce text-only (no media/calls) | Medium | Reject stickers/files/voice |
| Social exchange opt-in | Medium | Mutual consent to share handles |

### 5.3 Translation

| Feature | Diff | Notes |
|---------|------|-------|
| Choose translation API provider | Easy | DeepL vs Google vs OpenAI; cost/latency/privacy |
| Translation API service wrapper | Medium | Detect, translate, cache, rate-limit, meter |
| In-chat translate UX | Medium | Tap or /translate; show original + translation |
| AI in-chat translation (epic) | Medium | End-to-end demo in matched chat |

### 5.4 Safety & trust

| Feature | Diff | Notes |
|---------|------|-------|
| Safety / spam / harmful-content approach | Hard | Policy + platform reliance + Connect controls |
| Harmful-text AI triage queue | Hard | Classifier + human review; false-positive tuning |
| Rate limits & abuse prevention | Hard (part of safety) | Per-user message caps; velocity checks |

### 5.5 Engagement & monetization

| Feature | Diff | Notes |
|---------|------|-------|
| Conversationalist ranking | Hard | Anti-gaming design; peer quality signals |
| $2.99/mo country preference perk | Hard | Entitlement checked by matcher |
| Stripe billing + entitlements | Hard | Checkout, webhooks, subscription state |

---

## 6. Matching rules (draft)

**Default daily mode**
- Pool: users with pause off and profile complete enough (country required).
- Pair randomly worldwide.
- Exclude: blocked pairs, recent partners (e.g. last 90 days), same-user.
- Reveal: country + optional tastes only.

**Theme days**
- Prefer/require shared favorite book **or** song (configurable).
- Fallback to random if pool too thin; log fallback rate.

**Paid country preference**
- Subscribers may set preferred partner country for next match.
- Soft preference (boost) vs hard filter — product decision; hard filter can leave users unmatched.

**Odd counts**
- Hold one user for next batch or allow a "bye" with a short cultural prompt (no partner).

---

## 7. System architecture (sketch)

```
[Telegram users]
      |
      v
[Telegram Bot API / Mini App]  <-- delivery, basic spam tools
      |
      v
[Connect Gateway]
   |-- Auth / identity link (Telegram user id)
   |-- Profile service (country, tastes, pause, sub)
   |-- Matcher (daily jobs + theme modes)
   |-- Session service (24h windows, social-exchange state)
   |-- Translation service (provider wrapper + cache)
   |-- Safety (report/block, triage queue)
   |-- Billing (Stripe webhooks -> entitlements)
   '-- Analytics (reply rates, reports, paid conversion)
```

**Data principles**
- Minimize PII; store social handles only after mutual exchange.
- Chat content: prefer leaving messages on Telegram where possible; if mirrored, encrypt at rest and short retention.
- Taste strings stored normalized for matching; original display form kept.

---

## 8. Translation design

1. **Provider pick** — DeepL (quality), Google (coverage), OpenAI (flexible). Primary + fallback.
2. **Service** — `detect(text)`, `translate(text, target_lang)`, cache by hash+lang, per-user RPM/daily caps, cost ledger.
3. **UX** — default: on-demand translate; optional auto-translate for subscribers later.
4. **Privacy** — strip identifiers before send; do not log full message bodies in analytics.
5. **Failure** — show original + "translation unavailable"; never block chat on translator downtime.

---

## 9. Monetization

**$2.99 / month (working price)**
- Perk v1: choose preferred country for next day's match.
- Future perks: extra theme-day eligibility, ranking badge, higher translation caps.

Free tier must remain fully usable for daily random matches so the product stays discovery-first.

---

## 10. Ranking (later)

Goal: surface great conversationalists without turning into toxicity or popularity contests.

**Candidate signals**
- Mutual "good conversation" votes after day end
- Low report rate
- Reply latency / completion (careful: cultural norms differ)

**Anti-patterns**
- Public leaderboard farming
- Ranking tied to paid status

Ship as **design + stub** after MVP chat works.

---

## 11. MVP scope (ship order)

**M0 — Product & architecture**
- PRD freeze, Telegram-first decision, safety approach, bot scaffold

**M1 — Core matching & chat**
- Profiles, country, tastes, pause, daily matcher, 24h sessions, text-only, translate MVP, report/block, notifications, social-exchange opt-in

**M2 — Engagement & monetization**
- Theme days, Stripe + country perk, ranking design

---

## 12. Success metrics

- Day-1 reply rate (matched pairs with ≥1 message each)
- 24h completion without report
- Social-exchange opt-in rate
- Pause/resume retention
- Paid conversion (if launched)
- Translation usage vs cost

---

## 13. Risks & open questions

| Risk / question | Mitigation / decision needed |
|-----------------|------------------------------|
| Telegram ToS / bot limits | Mini App vs bot-only; review ToS before build |
| Unmatched hard country filters | Prefer soft boost |
| Translation cost at scale | Caching + caps + paid higher limits |
| Abuse / grooming / spam | Text-only + platform tools + report + triage |
| Thin theme-day pools | Fallback + larger windows |
| Ranking gaming | Delay ranking; private metrics first |

---

## 14. Linear issue index (snapshot)

UNG-5 PRD · UNG-6 messaging decision · UNG-7 safety · UNG-8 profile model · UNG-9 matcher · UNG-10 24h session · UNG-11 translation epic · UNG-12 pause · UNG-13 social exchange · UNG-14 theme days · UNG-15 subscription · UNG-16 ranking · UNG-17 provider pick · UNG-18 translate UX · UNG-19 translation wrapper · UNG-20 country · UNG-21 tastes · UNG-22 Telegram scaffold · UNG-23 report/block · UNG-24 text-only enforce · UNG-25 match notify · UNG-26 Stripe · UNG-27 AI triage

(See Linear for live status and labels.)

---

## 15. Next concrete steps

1. Lock Telegram-first (or not) in UNG-6.
2. Pick translation provider (UNG-17) and stub wrapper (UNG-19).
3. Stand up private repo scaffolding + bot hello-world (UNG-22).
4. Spec matcher job inputs/outputs (UNG-9) against profile schema (UNG-8).
