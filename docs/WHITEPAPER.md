# Tilla: Multi-Rail Merchant and Machine Payments with Stablecoin Settlement

**A technical white paper (draft)**  
W3Node Node Engine, Merchant & Machine Payments track  
Cape Town, 12–14 November 2026  

**Author:** Usman Olukayode  
**Working name:** Tilla is provisional and has not been cleared for trademark or domain conflicts.  
**Licence:** MIT © 2026 Usman Olukayode. This paper describes a system under construction; it is not an offer of payment services.

---

## Abstract

Digital person-to-person payments are widespread in Sub-Saharan Africa, yet digital **merchant** payments remain comparatively rare. Merchants still lose money to failed transfers and forged payment alerts, and dedicated card terminals or scheme-based tap-to-pay services are expensive or unavailable in much of the region. Tilla is a software till that runs on an ordinary Android phone and accepts three customer rails — NFC cards or stickers, USD₮ QR (or signed authorisation), and mobile-money request-to-pay — under a single confirmation model: the till marks a sale paid only when a payment network or an on-chain receipt says so. Balances can be held in USD₮ and cashed out to mobile money through licensed partners. The same till exposes an HTTP 402 / x402-style interface so machines and agents pay into the same ledger. This paper sets out the market context (using only previously sourced figures), the problem, the architecture, the payment-rail abstraction, the NFC card-wallet model (spending caps and session keys), machine payments, security and custody, economics, go-to-market, and limitations.

## 1. Market context

Sub-Saharan Africa is the centre of global mobile money. That success has not automatically digitised **in-store** commerce.

Global Findex-based analysis published by CGAP in July 2025 reports that only **20% of adults in Sub-Saharan Africa made a digital merchant payment in 2024**, compared with **42% of adults globally**. The same note records that in Nigeria, **42% of businesses name payment failures as a key concern**, and that even a **1% merchant discount rate can feel punitive** for smaller merchants. It also notes wealth, urban–rural and gender gaps (for example, 13% of adults in the poorest 40% of households in Sub-Saharan Africa made digital purchases in 2024). CGAP’s broader argument is that person-to-business payments lag person-to-person payments in several digitally mature African markets, so everyday commerce remains cash-heavy.[1]

The GSMA’s *State of the Industry Report on Mobile Money 2025* (regional cuts) describes how merchants are actually charged in practice. In a 2024 survey of proximity-payment mechanisms, **USSD was accepted by 62%** of participating providers in Sub-Saharan Africa, **in-app payments by 56%**, **QR codes by 37%**, and **NFC by only 6%**. The same section states that mobile-money merchant payments processed about **US$5.8 billion per month** in 2024, and about **576 million transactions per month**. The report also records **17 million registered agents** in Sub-Saharan Africa in 2024 (38% active monthly) and regional mobile-money transaction value of **US$1.1 trillion** in 2024.[2]

Card-based SoftPOS is not a universal substitute. Reporting in May 2026 states that Apple Tap to Pay on iPhone launched in South Africa with Yoco and iStore, with Yoco transaction fees **starting at 2.5% plus a flat R2.99 payout fee**, and that **Kenya has neither Apple Pay nor Tap to Pay**, with Safaricom’s planned M-Pesa tap-to-pay described as Android-only.[3]

Meanwhile, on-chain dollar instruments are already used at scale in the region. Chainalysis reports that Sub-Saharan Africa received **more than US$205 billion** in on-chain value between July 2024 and June 2025, and commentary on the same research cycle cites **stablecoins at 43%** of regional crypto transaction volume in the prior period.[4] Tilla treats USD₮ as a settlement and savings asset for the merchant, not as a requirement imposed on every shopper.

These figures describe a market in which the rails exist (mobile money, QR, emerging NFC, stablecoins) but are fragmented at the counter.

## 2. Problem analysis

Informal and small merchants face a bundle of problems that a single-network till number does not solve.

**Fragmentation.** A stall may be asked to accept cash, a bank transfer, one mobile-money operator, and occasionally a card. Each rail has a different confirmation ritual. NFC, the most convenient physical gesture, is rarely offered on mobile-money rails.[2]

**Unreliable confirmation.** Where customers show screenshots or SMS alerts, merchants bear the fraud. Payment **failures** themselves are a documented business concern in Nigeria.[1] A till that can speak “paid” only on authoritative confirmation is a product requirement, not a flourish.

**Cost and hardware.** Scheme tap-to-pay, where it exists, may price out low-margin goods.[3] Dedicated terminals require capital and a bank relationship many informal merchants lack.

**Currency risk.** Merchants who accumulate local currency overnight take depreciation risk. A dollar-denominated balance, with an optional auto-save and a path back to mobile money, addresses a treasury problem rather than a payments-fashion problem.

**Mis-tagged commerce.** CGAP notes that when customers pay merchants with a P2P push, providers may classify the payment as a remittance, weakening business data and consumer protections.[1] A till that opens a proper merchant checkout, even if the funding source is a P2P wallet, is a step toward a cleaner record.

**Machines.** Prepaid electricity, solar pay-as-you-go, charging and software agents need to pay a merchant without standing at a keypad. If that path is a second product, it will not ship. If it is another rail on the same till, it can.

Prior art is acknowledged. PayFlip, the overall winner at ETHCapeTown 2024 (which ran alongside W3Node), collected local currency (including M-Pesa via Peach Payments) and settled merchants in USDC on Base.[5] Kotani Pay, Yellow Card and Fonbnk offer business APIs for collection and payout.[6] MiniPay has scaled a consumer stablecoin wallet, including mobile-money on-ramps in selected markets.[7] Tilla is not a replacement for those APIs or consumer wallets. It is the **merchant surface**: multi-rail confirmation, voice-assisted UX, NFC cards, USD₮ treasury, and machine payments in one place.

## 3. System design

Tilla is a three-layer system: a merchant-facing progressive web application; an application back end that records checkouts and sales; and a set of **payment-rail adapters** that speak to mobile-money providers and to an EVM chain.

**Clients.** The till is a Next.js application deployed on Vercel, used full-screen on Android Chrome. Customers may use any compatible USD₮ wallet, a Tilla NFC card, or their existing mobile-money app. Machines call an HTTP API.

**Application layer.** Supabase provides authentication, Postgres and Realtime. A checkout row is created in `pending` when the merchant enters an amount. Rail-specific identifiers (STK checkout request ID, on-chain intent hash, x402 payment payload) are stored against it. When an adapter reports success, the row becomes `paid`, the sales feed updates, and the till speaks. Adapters must be idempotent: the same provider transaction cannot pay two checkouts, and a replay cannot create two sales.

**Wallet layer.** Tether’s Wallet Development Kit supplies account and transfer primitives, including account-abstraction modules that can pay gas in USD₮ via a paymaster.[8] That matters at a stall: a customer or a card wallet should not fail because it holds no ETH or equivalent.

**Chain choice** is deliberately unset until hackathon sponsorship quests are announced. The architecture assumes a low-fee EVM chain and a USD₮ (or USD₮0) token contract.

## 4. Protocol and component design

### 4.1 Payment-rails abstraction

Every rail implements a small interface:

- `createPayment(checkoutId, amount, currency, metadata)` → `{ nextAction }` where `nextAction` is “show QR”, “prompt NFC”, “wait for STK”, or “return HTTP 402”.
- `interpretCallback(raw)` or `interpretOnChainEvent(log)` → `{ checkoutId, status, railTxId }`
- `isAuthoritative(proof)` — screenshots always return false.

The till UI never talks to Daraja, MoMo or a bundler directly. Adding a rail should not require a new success screen. This is the principal internal design rule.

Local-currency amounts are converted to a USD₮ charge using a quoted rate with an explicit timestamp and source (manual, public FX, or ramp). The quote is frozen for a short expiry so the merchant and customer see the same number.

### 4.2 NFC card wallet, spending caps and session keys

Web NFC in Chrome on Android can read and write NDEF records on a physical tag, subject to HTTPS and a user gesture.[9] Tilla tags carry an identifier and, optionally, a short public hint; they do not carry a raw private key in a form that a casual tap should reveal.

The intended card model for the hackathon is **semi-custodial and capped**, and will be described as such:

1. Each card corresponds to an on-chain account (or a sub-account) funded in USD₮.
2. A **spending cap** (per transaction and per day) is enforced on-chain or by a session module. A stolen card without the PIN, or a card presented for an unusually large sale, should fail.
3. A **session key** authorises payments within the cap for a limited time or number of uses. The root key is not required at the till.
4. PIN entry happens on the merchant’s phone or a customer confirmation screen; the PIN is not stored in the till log.

This is **not** full self-custody of an uncapped key on a sticker, which would be irresponsible. It is closer to a stored-value card with an on-chain audit trail. Production systems would add official card manufacturing, secure elements, and a formal issuer policy. The demo may use prefunded tags and a simple allowance or session-key contract tested in Foundry.

If Web NFC is unavailable, the same card identifier can be entered manually or reached via QR printed on the sticker — slower, but the cap still applies.

### 4.3 QR and wallet payments

The default customer-with-smartphone path is a QR encoding a USD₮ transfer to the merchant’s account, or an EIP-2612 / EIP-3009-style (or account-abstraction user-operation) authorisation if gasless flows are available. The till observes the chain (direct receipt or a relayer) and only then confirms. Explorer links are shown on the sale row.

### 4.4 Mobile-money request-to-pay

For M-Pesa, the adapter initiates a Daraja STK Push and waits for the result at a public callback URL.[10] For MTN, the adapter uses request-to-pay and the corresponding status callback.[11] The merchant sees `pending` until the callback (or a documented timeout). This is the path that defeats screenshot fraud: the till does not offer a “mark as paid from photo” control.

Cash-out of a USD₮ balance uses a payout API from a licensed partner when credentials exist, and is otherwise labelled as a simulated quote. Tilla does not hold fiat.

### 4.5 x402 machine payments

HTTP 402 Payment Required is the basis of the emerging “x402” pattern: a resource answers with payment instructions; the client pays on-chain (often with a signed authorisation) and retries with a payment receipt header. A prior Node Engine project, x402-cronos, explored pay-per-request agent access on Cronos.[12]

Tilla exposes, for a given checkout or for a merchant’s “open till” resource, a 402 response that names the asset (USD₮), amount, recipient and validity window. On verification, the server marks the checkout paid and the sale appears in the same feed with a machine or agent badge. Compatibility between the chosen USD₮ token and the x402 facilitator or scheme must be verified before this path is promised in a pitch. If the scheme cannot be wired in time, a narrowly scoped signed-payment webhook may be substituted, and must be labelled honestly.

## 5. Security and trust model

**Authoritative confirmation.** Trust sits with the mobile-money operator (for STK or MoMo) or with the chain (for USD₮ and x402). Tilla’s server is a recorder and notifier, not a substitute for those sources.

**Shop isolation.** Row Level Security ensures that sales and API keys belong to a shop. Staff roles are a later addition.

**Callback authenticity.** Webhooks are checked against provider secrets or signatures. Replays are rejected or folded into the existing sale.

**Card risk.** Loss or skimming of a physical tag is mitigated by caps, session expiry, PIN and the ability to revoke a card’s session. Residual risk remains and is disclosed.

**Client platform.** Web NFC’s narrow browser support is a platform risk, not hidden behind marketing.

**Dependencies.** Compromise of Vercel, Supabase, a paymaster, or a ramp partner can affect availability or payouts. Key material is not stored in the repository or in client source maps.

## 6. Custody and regulatory considerations

Tilla, as specified for the hackathon, is software. It is not a bank, a mobile-money operator or a card acquirer.

- **USD₮ at the merchant** should sit in a wallet the merchant controls (WDK). If a hosted wallet is used for speed in the demo, that must be said aloud.
- **NFC cards are semi-custodial.** The issuer of the cap and session policy (the project, during the demo) has more power than a fully self-custodial wallet. That is a regulatory and consumer-protection issue for any real launch.
- **Fiat legs** go through licensed mobile-money operators and, for cash-out, licensed ramps. Tilla should not take possession of customer naira, shillings or rand.
- **KYC and transaction monitoring** will be required if the system is offered to the public beyond a sandbox. The hackathon build will not pretend to have completed that work.
- **Data protection.** Phone numbers, sale amounts and device identifiers are personal or commercially sensitive data and should be minimised.

This section is a design constraint, not legal advice.

## 7. Economic model and fees

A sustainable model, for discussion rather than as a committed tariff, is:

- A **low ad valorem fee** on each confirmed sale (the working assumption is about **0.5%**, with **low-value sales free**), sensitive to CGAP’s warning about even a 1% merchant discount rate.[1]
- A **spread or partner fee** on cash-out FX, shared with the licensed payout provider.
- Optional **paid statements** or bookkeeping exports for merchants who need a record for suppliers or lenders.
- A **per-call fee** for machine and agent payments.

No revenue, take-rate or volume forecast is stated here; the project has no trading history. Competitors’ published prices (for example Yoco’s 2.5% plus payout fee on a specific SoftPOS product[3]) are used only as context for why a software till might be attractive, not as a claim that Tilla will undercut every rail on day one. Mobile-money operators will still charge their own customer or merchant fees on STK or MoMo legs; Tilla must display those honestly when known, and remain silent when not known.

## 8. Go-to-market

The first users are associations of market traders and spaza retailers, reached through existing agent and fintech communities rather than through a cold consumer campaign. NFC stickers can be distributed through the same agent networks that already serve mobile-money cash-in. Geographic sequencing favours markets where a request-to-pay API is actually available to developers: Kenya (Daraja) first, then MoMo markets (for example Ghana or Uganda), with South Africa as a design market because SoftPOS card fees are visible and the event itself is in Cape Town.

Consumer wallets such as MiniPay, and payout APIs such as Kotani Pay, Yellow Card and Fonbnk, are distribution or cash-out partners, not targets for displacement.[6][7]

The same software can be offered later in any country with an instant-payment rail and a stablecoin on-ramp. The African counter is the design centre, not a limit on where the rails can plug in.

## 9. Roadmap

**Hackathon (three days).** Must-haves: QR and NFC USD₮, one live mobile-money sandbox, voice-confirmed till, dashboard, public deploy. Should-haves: x402, MoMo, auto-save, cash-out quote, sales statement. Details are in the PRD.

**After the event.** Production session-key contracts and card issuance; additional rails; proper FX quoting; licensed payout in at least one country; iOS strategy (likely a native wrapper, because Web NFC will not appear on Safari by wishing); optional credit products **only** with a regulated partner and informed consent.

## 10. Limitations

- Web NFC is Chrome on Android only.[9]
- Sandbox mobile-money APIs are not production operators.
- Semi-custodial cards will not satisfy a self-custody purist, and should not be marketed as such.
- Liquidity, FX and payout licences are not provided by this repository.
- Voice quality and language coverage depend on the device.
- PayFlip and existing APIs already occupy parts of the design space; differentiation must stay visible in the demo (voice, NFC, machines), not only in this paper.
- Figures cited in Section 1 come from public reports and journalism. They were not re-audited from primary microdata for this draft.

## 11. Conclusion

Tilla’s claim is modest and testable: a merchant should be able to take a tap, a QR payment or a mobile-money prompt on one phone, believe the confirmation, and keep or cash out a dollar balance — and a machine should be able to pay that same till. The market numbers justify the problem. The hackathon exists to justify the implementation.

## References

1. Hillary Miller-Wise, Rashmi Pillai and Xavier Faz, “Breaking the Cash Habit: The Urgent Need to Accelerate Digital Merchant Payments,” CGAP, 29 July 2025, [https://www.cgap.org/blog/breaking-cash-habit-urgent-need-to-accelerate-digital-merchant-payments](https://www.cgap.org/blog/breaking-cash-habit-urgent-need-to-accelerate-digital-merchant-payments). Based on Global Findex 2025.
2. GSMA, *The State of the Industry Report on Mobile Money 2025: Sub-Saharan Africa, East Africa & Asia* (regional cuts), 2025, [PDF](https://www.gsma.com/solutions-and-impact/connectivity-for-good/mobile-for-development/wp-content/uploads/2025/10/SOTIR-2025-Regional-Cuts_r-1.pdf). See also the global report landing pages at [https://www.gsma.com/sotir/](https://www.gsma.com/sotir/).
3. “Apple Launches Tap to Pay on iPhone in South Africa. Kenya Still Locked Out,” *tech-ish*, 12 May 2026, [https://tech-ish.com/2026/05/12/apple-tap-to-pay-south-africa-kenya-still-waits/](https://tech-ish.com/2026/05/12/apple-tap-to-pay-south-africa-kenya-still-waits/).
4. Chainalysis, “Sub-Saharan Africa Shows Strong Crypto Retail Activity,” 2025 Geography of Cryptocurrency report coverage, [https://www.chainalysis.com/blog/subsaharan-africa-crypto-adoption-2025/](https://www.chainalysis.com/blog/subsaharan-africa-crypto-adoption-2025/); Cointelegraph, “Sub-Saharan Africa Third-Fastest in Crypto Adoption: Report,” [https://cointelegraph.com/news/sub-saharan-africa-third-fastest-growing-region-crypto-adoption-report](https://cointelegraph.com/news/sub-saharan-africa-third-fastest-growing-region-crypto-adoption-report).
5. Christine Kinya, “PayFlip won 1st place at EthCapeTown,” LinkedIn, September 2024, [https://www.linkedin.com/posts/christine-kinya2020_ethcapetown-hackathon-web3-activity-7244997162709700608-cyfs](https://www.linkedin.com/posts/christine-kinya2020_ethcapetown-hackathon-web3-activity-7244997162709700608-cyfs).
6. Kotani Pay API documentation, [https://documentation.kotanipay.com/v3/overview](https://documentation.kotanipay.com/v3/overview); Yellow Card payments API, [https://yellowcard.io/api](https://yellowcard.io/api); Fonbnk, [https://fonbnk.com/](https://fonbnk.com/).
7. Opera, “MiniPay turns two with 10 million wallets and 270 million transactions,” 16 September 2025, [https://press.opera.com/2025/09/16/minipay-turns-two-10-million-wallets/](https://press.opera.com/2025/09/16/minipay-turns-two-10-million-wallets/).
8. Tether, Wallet Development Kit documentation, [https://docs.wdk.tether.io/](https://docs.wdk.tether.io/); wallet modules including account abstraction, [https://docs.wdk.tether.io/sdk/wallet-modules/](https://docs.wdk.tether.io/sdk/wallet-modules/).
9. Chrome Developers, “Web NFC,” [https://developer.chrome.com/docs/capabilities/nfc](https://developer.chrome.com/docs/capabilities/nfc); MDN, `NDEFReader`, [https://developer.mozilla.org/en-US/docs/Web/API/NDEFReader](https://developer.mozilla.org/en-US/docs/Web/API/NDEFReader).
10. Safaricom Daraja API (developer programme), [https://developer.safaricom.co.ke/](https://developer.safaricom.co.ke/).
11. MoMo Open API sandbox materials, [https://github.com/momoopenapi/momoopenapi_sandbox](https://github.com/momoopenapi/momoopenapi_sandbox).
12. W3Node, Node Engine 2026 track winners announcement (including x402-cronos), [https://www.linkedin.com/posts/w3node-sa_nodeengine2026-web3builders-africabuilds-activity-7420856185957924864-5nF7](https://www.linkedin.com/posts/w3node-sa_nodeengine2026-web3builders-africabuilds-activity-7420856185957924864-5nF7).
13. W3Node, Node Engine tracks, [https://w3node.io/node-engine](https://w3node.io/node-engine); conference site, [https://w3node.io/](https://w3node.io/).

---

*This draft accompanies [PRD.md](PRD.md) and the repository [README](../README.md). It should be revised if primary sources are updated or if implementation choices (chain, custody, x402 scheme) change during the event.*
