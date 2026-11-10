# Tilla — Product Requirements Document

**Status:** Draft for the W3Node Node Engine hackathon  
**Dates:** 12–14 November 2026, Cape Town  
**Track:** Merchant & Machine Payments  
**Author:** Usman Olukayode  
**Name:** Tilla is a **provisional** working name and has not been cleared for trademark or domain conflicts.

This document specifies what Tilla should do during the hackathon and immediately afterwards. It is the source of truth for scope. The system design is elaborated in [WHITEPAPER.md](WHITEPAPER.md).

---

## 1. Overview

Tilla is a multi-rail merchant till that runs as a progressive web application on an ordinary Android phone. A merchant enters an amount; a customer pays by NFC card or sticker, QR code (USD₮) or a mobile-money prompt (M-Pesa STK Push or MTN MoMo request-to-pay). Settlement can remain in USD₮ or be cashed out to mobile money. Confirmed payments are displayed in large type and spoken aloud. The same till exposes an HTTP 402 / x402-style endpoint so machines and AI agents can pay it.

The product is intended to work worldwide wherever mobile money or stablecoins exist, and is designed first for informal and small merchants in African markets.

## 2. Goals and non-goals

### Goals

- Give a micro-merchant a **single, polished till** that accepts more than one payment rail without a card machine.
- Make confirmation **trustworthy**: the till updates only on a payment-network callback or an on-chain receipt, never on a customer screenshot.
- Let merchants **hold USD₮** as a dollar store of value and cash out when they need local currency.
- Demonstrate **machine and agent payments** on the same till, even if only as a thin x402 path.
- Ship a **publicly deployed**, usable demo within three days that scores on design, technical execution and market fit (see [Section 10](#10-judging-rubric-mapping)).

### Non-goals (hackathon)

- Building or certifying a Visa/Mastercard SoftPOS, or integrating Apple Tap to Pay.
- Becoming a licensed payments institution or holding customer fiat.
- Supporting iOS native NFC (Web NFC is Chrome on Android only).
- A full offline mesh or delayed-broadcast payment protocol (a later roadmap item).
- Automated underwriting or lending against sales history (the sales feed should merely make this plausible later).
- Production-grade key custody, hardware security modules or formal contract audits.
- Every African mobile-money operator. One live sandbox rail is sufficient for the event; a second rail is a should-have.

## 3. Personas

### Mama Chioma — Lagos market trader (primary)

Sells foodstuff from a stall. Accepts cash, bank transfers and occasional mobile-money payments. Has been caught by fake transfer alerts. Feels naira cash savings lose value. Owns an Android phone, not a card terminal. Needs a till she can trust in a noisy, busy stall.

### Thandi — Khayelitsha spaza owner (primary, Southern Africa)

Runs a small shop. Card SoftPOS fees (for example Yoco’s 2.5% plus a payout fee on iPhone Tap to Pay in South Africa) are unattractive on low-margin goods. Customers pay with cash, cards and e-wallets. She wants one screen, a spoken “paid” confirmation she can hear from the fridge, and a daily sales total.

### Ibrahim — solar PAYG or charging-point operator (secondary, machine)

Operates prepaid meters or battery-swap points. Needs a machine-payable endpoint and a single sales ledger, not a separate “IoT product”.

### Ada — customer (secondary)

Sometimes has only a feature phone and an NFC sticker; sometimes has a USD₮ wallet; sometimes has only M-Pesa or MoMo. She will not install a new app if a tap, a QR scan or a prompt will do.

### “Chidi” — AI or software agent (secondary)

A booking or replenishment agent that should pay a merchant or a machine per request, without a human at the till.

## 4. User stories

1. As Chioma, I can sign in with my phone number and open a till in under a minute so I can start taking digital payments today.
2. As Chioma, I can type an amount in local currency and see a USD₮ equivalent so I know what I will hold.
3. As a customer, I can pay by scanning a QR code with a USD₮ wallet so I do not need the merchant’s bank details.
4. As a customer with a Tilla card, I can tap the merchant’s phone, enter a PIN, and pay within a spending cap so I do not need a smartphone at the stall.
5. As a customer with M-Pesa, I can approve an STK Push on my own phone so I can pay the way I already know.
6. As Thandi, I hear and see a confirmed payment so I do not release goods on a screenshot.
7. As Thandi, I can watch a live sales feed and a daily total so I know how the day is going.
8. As Chioma, I can hold USD₮ and cash out to mobile money so I can pay suppliers in local currency.
9. As Chioma, I can set an optional auto-save percentage in USD₮ so part of each sale is kept in dollars.
10. As Ibrahim, I can call a pay-per-request endpoint and have the payment appear on the same till feed so machines share the merchant’s ledger.
11. As Ada, I can understand the till if I have low literacy, because amounts, buttons and voice prompts are large and plain.
12. As a judge or reviewer, I can open a public URL and complete a sandbox payment so the demo does not depend on a local laptop only.

## 5. Functional requirements (MoSCoW)

### Must have

| ID | Requirement |
| --- | --- |
| M1 | Merchant can authenticate (phone or magic link) and reach a till home screen. |
| M2 | Merchant can enter a sale amount, with currency clearly labelled. |
| M3 | Till can display a payment QR for a USD₮ transfer or signed authorisation, using Tether WDK where practical. |
| M4 | Till can read a Tilla NFC tag in Chrome on Android (Web NFC) and initiate a capped card-wallet payment. |
| M5 | At least one live mobile-money sandbox rail: Safaricom Daraja STK Push, with payment result taken from the official callback, not from the customer’s phone screen. |
| M6 | On confirmed payment (callback or on-chain receipt), the till shows a large success state and speaks the amount (and payer name or reference if available). |
| M7 | Failed, cancelled or timed-out payments show a clear failure state and do not speak “paid”. |
| M8 | A live sales feed updates for the merchant without a manual refresh (Supabase Realtime or equivalent). |
| M9 | A simple dashboard: today’s count, today’s volume, and a list of recent sales with rail, status and explorer or receipt link where applicable. |
| M10 | Public deployment of the web application (Vercel). |
| M11 | Environment-specific configuration; secrets never shipped in the client bundle. |

### Should have

| ID | Requirement |
| --- | --- |
| S1 | MTN MoMo request-to-pay sandbox as a second rail behind the same adapter interface. |
| S2 | x402-style machine or AI-agent payment against a checkout or “pay this till” resource, appearing in the same sales feed. |
| S3 | Optional auto-save: a merchant-configured percentage of each sale retained as USD₮. |
| S4 | Cash-out quote and initiation through a mocked or sandbox ramp / mobile-money payout API, clearly labelled if simulated. |
| S5 | PDF or shareable daily sales statement. |
| S6 | Gasless customer payments (ERC-4337 or EIP-7702 via WDK) so the customer need not hold the gas token. |
| S7 | Merchant display language toggle (English plus at least one of Nigerian Pidgin or isiXhosa labels). |

### Could have

| ID | Requirement |
| --- | --- |
| C1 | Offline-signed authorisation that broadcasts when connectivity returns. |
| C2 | Hardware device (ESP32) paying the till as a machine customer. |
| C3 | Multi-staff PINs and a “shop” entity with several tills. |
| C4 | Customer-facing receipt SMS. |
| C5 | Formal accessibility audit and additional languages. |

### Won’t have (this event)

| ID | Requirement |
| --- | --- |
| W1 | Card-scheme (Visa/Mastercard) acquiring or Apple Tap to Pay. |
| W2 | Native iOS NFC. |
| W3 | Holding or safeguarding fiat in Tilla’s own accounts. |
| W4 | Production lending, invoices with legal e-invoicing, or tax filing. |
| W5 | Unbounded, unaudited custody of user funds on cards without a spending cap. |

## 6. Non-functional requirements

### Performance

- Till home and amount entry should feel instant on a mid-range Android phone over a typical 4G link. Aim for first interaction on a warm load in under two seconds; treat this as a design target, not a measured baseline (no lab numbers have been collected yet).
- QR generation and NFC read should start immediately on a user gesture.
- Voice confirmation should begin within a short moment of the confirmed state rendering, without blocking the visual success screen.

### Reliability

- Payment state is driven by rail callbacks and on-chain receipts. Client optimism must be labelled as “pending”.
- Callback handlers must be **idempotent**. Duplicate STK or MoMo notifications must not create duplicate sales.
- If Realtime disconnects, the till should reconnect and recover the latest checkout state from the server.
- A recorded backup demo is part of event reliability, not an afterthought.

### Accessibility

- Large typography and tap targets (minimum 44×44 CSS pixels for primary actions).
- Visible focus states; amount entry usable with a hardware or software keyboard.
- Success and failure must not be colour-only; use text, icon and voice together.
- Voice confirmation is a first-class accessibility feature for noisy stalls and low literacy, not a gimmick.
- Respect the system language and, where possible, the `prefers-reduced-motion` setting.
- Do not autoplay voice on pages other than the active till confirmation, and provide a mute control.

### Security

- HTTPS only in deployed environments.
- Least-privilege Supabase Row Level Security: a merchant sees only their shop’s sales.
- NFC cards use a **spending cap and session keys**; the model is **semi-custodial** and must be described honestly in the product and the pitch (see the white paper).
- Never log PINs, seed phrases or raw card keys.
- Webhook endpoints authenticate the provider or use a shared secret and reject unauthenticated messages.
- Rate-limit checkout creation and cash-out requests.
- Warn clearly when a cash-out path is a mock.

## 7. UX flows and screens

### 7.1 Merchant onboarding

`Splash / sign-in` → phone OTP or magic link → `Create shop` (name, default currency, optional logo) → `Till home`.

Keep onboarding to one or two screens. Avoid wallet jargon on the first screen.

### 7.2 Charge a customer (happy path)

`Till home` (large numeric keypad, currency toggle) → `Collect` (chooser: NFC, QR, M-Pesa / MoMo) → `Pending` (waiting for rail or chain, with a cancel control) → `Paid` (full-screen ✓, amount, payer or reference, spoken confirmation, “New sale”) → item appears at the top of the sales feed.

### 7.3 Failure path

`Pending` → timeout, customer cancel, or rail error → `Not paid` (plain language, try again or change rail). Goods must not be released.

### 7.4 NFC card

Prompt “Hold the card to the back of the phone” → read → PIN pad → pending → paid or declined (cap exceeded, bad PIN, or insufficient balance).

### 7.5 Dashboard

Tabs or a simple second screen: today’s totals, list of sales (rail icon, amount, time, status), optional cash-out and auto-save settings. Statement export lives here if built.

### 7.6 Cash-out (should-have)

`Balance` → amount → quote (USD₮ to local currency, fees shown) → confirm → pending → paid to mobile money, or clearly “demo only” if the payout is simulated.

### 7.7 Machine / x402 (should-have)

A documented `GET/POST` resource returns `402` with payment instructions; after a valid payment the same sale appears on the feed with a “machine” or “agent” badge. A small demo client or curl script is enough for the pitch.

### Visual tone

Calm, high contrast, stall-friendly. Dark mode is desirable for indoor shops. Brand colour should not fight the paid / not-paid states. Avoid cryptocurrency clichés (rockets, neon grids). The till should look like a serious payment tool.

## 8. Success metrics

Hackathon success is qualitative and demo-based. The following are the intended measures; **no baseline conversion rates are claimed**.

| Metric | Target for the event |
| --- | --- |
| End-to-end demo | A judge can complete at least one NFC or QR USD₮ payment and see and hear confirmation |
| Mobile-money path | At least one STK Push sandbox payment completes via callback, or a recorded fallback is shown immediately |
| Deployment | Public HTTPS URL on Vercel, usable from a phone |
| Reliability in the room | Duplicate callbacks do not double-count a sale; a failed payment never speaks “paid” |
| Design completeness | Till, pending, paid, failed and dashboard screens are visually consistent |
| After the event (directional) | Time-to-first-sale for a new merchant; share of sales by rail; cash-out completion; dispute or failure rate — to be instrumented later |

## 9. Dependencies and integrations

| Dependency | Role | Notes |
| --- | --- | --- |
| Vercel | Hosting | Must-have for the public demo |
| Supabase | Auth, Postgres, Realtime | Must-have |
| Tether WDK | Self-custodial / smart-account wallets, USD₮ transfers | Confirm browser versus React Native on day one |
| An EVM chain + paymaster | Gasless USD₮ | Choose at kick-off if a sponsorship quest names a chain |
| Web NFC / Chrome Android | Card read | Physical tags required; no iOS Web NFC |
| Safaricom Daraja sandbox | STK Push and callbacks | Callback URL must be publicly reachable (Vercel or a tunnel) |
| MTN MoMo sandbox | Second rail | Should-have |
| Ramp or payout API (e.g. Kotani Pay, Yellow Card, Fonbnk) | Cash-out | Partner, not a competitor; mock if credentials are not available |
| Browser `speechSynthesis` | Voice confirmation | Quality varies by device and language pack |
| Foundry | Contract tests and deploy | If card-wallet contracts ship |

## 10. Judging rubric mapping

The Merchant & Machine Payments track dashboard lists:

| Pillar | Weight | How Tilla is designed to score |
| --- | --- | --- |
| **Product design** | 33% | Stall-first till: large keypad, obvious rail chooser, full-screen paid / not-paid states, voice as accessibility, live sales feed, dark mode, polish over feature count. Pidgin or isiXhosa labels if time allows. |
| **Technical UX** | 34% | Clean rail-adapter architecture; WDK + account abstraction; Web NFC; real (sandbox) Daraja callbacks; idempotent handlers; Realtime feed; Foundry-tested contracts if used; public Vercel deployment; honesty about semi-custodial cards. |
| **Business / market fit** | 33% | Addresses the documented digital merchant-payment gap, fake-alert pain, NFC rarity, and SoftPOS fees (sources in the README and white paper). Integrates M-Pesa / MoMo rather than pretending to replace them. Cash-out and USD₮ auto-save speak to FX and working-capital needs. Machine payments extend the same till rather than inventing a second product. |

## 11. Risks and mitigations

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Web NFC works only in Chrome on Android | Demo failure on iPhone or desktop | Bring an Android phone and spare tags; QR is the universal fallback; say this in the pitch |
| Daraja or MoMo sandbox is down or delayed | Broken mobile-money path | Record a backup video on day two; keep QR/NFC working independently |
| WDK may not run cleanly in the browser | Wallet integration slips | Spike on morning of day one; fall back to a standard viem/wagmi account or WDK’s React Native path for a filmed mobile demo |
| x402 token / scheme mismatch | Machine path incomplete | Verify token compatibility early; if blocked, ship a well-documented signed-payment webhook instead and label it honestly |
| Semi-custodial NFC cards raise trust questions | Judges push on custody | Caps, session keys, and a clear verbal disclosure; no claim of full self-custody on the card |
| Overlap with PayFlip (ETHCapeTown 2024 winner: local currency in, USDC out) | “We have seen this” | Lead with voice-confirmed multi-rail till, NFC and machine payments, not only “stablecoin settlement” |
| Scope creep (too many rails, hardware, lending) | Unfinished UI | Freeze must-haves by end of day one; design polish is 33% of the score |
| Regulatory and PCI-like expectations | Unrealistic promises | Do not take card PAN data; cash-out only via licensed partners; documentation stays accurate |
| Team is solo or very small | Delivery risk | Vertical slice first (one rail + till UI), then add NFC or x402, not both at once if behind |

## 12. Milestones (Days 1–3)

Conference sessions will run in parallel. Protect build time.

### Day 1 — Thursday 12 November: rails and a vertical slice

- Scaffold Next.js, Supabase schema (shops, checkouts, sales), and environment templates.
- Implement the rail adapter interface and **QR USD₮** payment, plus **one** Daraja STK sandbox path including a public callback.
- Confirm WDK (or fallback) and the target chain, including any sponsorship quest.
- Sketch NFC read in Chrome; do not block the day on card contracts.
- End of day: a pending → paid loop works on at least one rail, deployed or tunnelled.

### Day 2 — Friday 13 November: product and confirmation

- Voice and visual confirmation, sales feed, dashboard totals.
- NFC happy path with a cap (even if the cap is a simple on-chain allowance or a mocked card account).
- Start should-haves only if the must-have path is stable (x402 or MoMo, not both unless ahead).
- Record the backup demo video.
- End of day: a merchant can complete a sale on a phone and see it on the dashboard.

### Day 3 — Saturday 14 November: polish, deploy, pitch

- Visual polish, empty and error states, mute control, copy-editing.
- Final Vercel deployment, explorer links, README quick start filled in where now accurate.
- Three-minute pitch: problem with sourced numbers, live demo, business model, honest limitations.
- Freeze features at least two hours before judging.

## 13. Open questions

1. Which EVM chain and paymaster will sponsorship quests favour at kick-off?
2. Does Tether WDK’s ERC-4337 module run acceptably in a Next.js browser bundle?
3. Which USD₮ contract (native or bridged / USD₮0) is correct for the chosen chain, and does it work with the intended x402 scheme?
4. Can Daraja sandbox credentials and a public callback be approved in time, or must the team rely on a recorded demo plus QR?
5. How should local-currency amounts be quoted into USD₮ — a public FX API, a manual rate, or a ramp quote?
6. For NFC cards, is a merchant-issued, prefunded card acceptable for the demo, or must cards be user-claimed?
7. Will organisers allow a single project to be considered on more than one track if the x402 path is strong?
8. What merchant identity or KYC, if any, do partners require before a real cash-out?
9. Is the working name “Tilla” already in use in payments in a way that would confuse judges?

---

*End of PRD. Related: [README](../README.md), [white paper](WHITEPAPER.md).*
