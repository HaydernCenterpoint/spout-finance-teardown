# Spout Finance beta teardown (Superteam Product Feedback)

Listing: https://superteam.fun/earn/listing/product-feedback-spout-finance/
Deadline: 21 Sep 2026 22:59 UTC
Prizes: 250 / 200 / 150 / 100 / 100 / 100 / 100 USDC (seven spots; 0 submissions when written)

Surfaces used: https://www.spout.finance/ · https://spout.finance/docs/getting-started · https://spout.finance/docs/health-factor · https://spout.finance/llms-full.txt · https://demo.spout.finance/ · https://beta.spout.finance (linked from their own posts)

Wallet/KYC: not completed in this session. This is a docs + public-demo audit, not a funded mainnet trade.

Judge-facing HTML (no iframe): https://hayderncenterpoint.github.io/spout-finance-teardown/

## One-line product

Spout is a Solana DeFi prime-brokerage pitch: tokenized US equities as collateral, 0% stated borrow APR, lender yield from writing covered calls on that collateral.

## What is actually good

- The economic story is clearer than most RWA lending pages. “Borrower does not pay interest; option premium pays lenders” is a real TradFi structure (covered-call overwriting), not a mystery token emission.
- 50% max LTV + a liquidation threshold *above* that LTV is the right shape. Opening Health Factor > 1.00 is honest.
- Path B in the docs (deposit existing xStocks / Ondo, no extra Spout KYC) is the only way a crypto-native tester can actually touch the product without a broker account.
- Proof-of-reserve + Token-2022 transfer hook is the correct compliance primitive if they mean wallet-level KYC on spAssets.
- Senior/Junior split (~9% / ~32% in docs) is more honest than a single “double-digit APY” headline — once you find the docs.

## Bugs and UX friction (ranked)

### 1. “0% interest” is the marketing number, not the borrower cost

FAQ: if a written call expires ITM, the borrower “absorbs the difference between the strike and the market price,” historically “around 0.5% annualized.” That is a borrower cost. It is not interest, but a user comparing Spout to a 5% margin loan will be misled if the UI only prints `Borrow APR 0%`.

**Fix:** show an estimated cycle cost / max ITM give-up next to 0% APR, and a one-line “you sold a call on your shares.” JEPI/QYLD comparison belongs in an advanced drawer; those funds have NAV erosion for a reason.

### 2. Three live hostnames, production 522, one unresolvable URL

Checked 2026-09-09 from this session:

| Time (UTC) | UA | URL | Result |
| --- | --- | --- | --- |
| ~10:37 | default | https://app.spout.finance/ | **HTTP 522** |
| ~10:41 | Python urllib | https://app.spout.finance/ | **HTTP 403** |
| ~10:41 | Chrome | https://app.spout.finance/ | **HTTP 522** |
| ~10:47 | Python urllib | https://app.spout.finance/ | **HTTP 403** |
| ~10:47 | Chrome | https://app.spout.finance/ | **HTTP 522** |
| ~10:37 / 10:47 | both | https://demo.spout.finance/ | HTTP 200, 12 634 bytes |
| ~10:37 / 10:47 | both | https://beta.spout.finance/ | HTTP 200, **same 12 634 bytes as demo** |
| ~10:37 | — | https://https://beta.spout.finance/ | DNS fail (doubled `https://` in llms-full) |
| ~10:47 | Python urllib | https://spout.finance/docs/getting-started | **HTTP 403** |
| ~10:47 | Chrome | https://spout.finance/docs/getting-started | HTTP 200, 8 744 bytes |
| ~10:47 | both | https://www.spout.finance/ | HTTP 200, 15 273 bytes |

Docs tell testers to “Visit app.spout.finance”. That host flaps 522/403. Demo and beta are the same static bundle. Getting-started is **UA-gated** (403 to urllib, 200 to Chrome), so Path B is invisible to agents and scripted testers. `llms-full.txt` (219 569 bytes) still contains `https://https://beta.spout.finance/`; 7× `beta.spout.finance`, 0× `app.spout.finance`, 0× `demo.spout.finance`.

**Fix the origin**, the double-https, the docs WAF, and put one canonical “start here” URL on the listing and the homepage.

### 3. KYC story disagrees with itself

Homepage FAQ “How do I get started?”: connect wallet, **complete KYC**, then deposit.

Docs getting-started Path B: deposit existing tokenized equities, **no KYC through Spout**.

If Path B is real, the homepage buries the only path Superteam testers can use. If Path B is stale, the docs are lying. Pick one.

### 4. Lockups vs “withdraw whenever”

Homepage: “There are no lockups on either side.”
Getting started: lock collateral to enroll in the **weekly options cycle**; lenders get distributions **every Monday**.

Covered-call overwriting has a cycle. Either withdrawals mid-cycle are delayed / haircut, or they are not writing the calls they claim. The UI must say what happens if a borrower unlocks Tuesday.

### 5. Health Factor copy is dangerous

Docs: “There is no cost to being close to the line, and no cost to being far from it.”

That is false in a gap-down. Close to HF 1.00 means a smaller buffer before partial liquidation. If notifications exist, the empty demo does not show them. **Fix:** show HF, liquidation price, and “buffer vs last print” on the position card, not a sentence that closeness is free.

### 6. Demo buy flow is a brochure, not a protocol

`demo.spout.finance` shows NVDA at a round `$170.00`, 2.79%, 1x–2x leverage chips, and “Connect Wallet.” There is no live oracle timestamp, no proof-of-reserve number on the first screen (the buy page later says “Reserves 100.2%” with no tx or slot), no testnet faucet, no invite-code field. A Superteam “actual product usage” judge cannot tell demo from production.

### 7. Leverage chips vs 50% LTV

Demo offers 1.25x–2x. 2x on a long is 50% margin, which matches 50% LTV — but “leverage” on a buy ticket is not the same mental model as “borrow 50% against shares you already hold.” New users will think they are perping NVDA. Label it “borrow against this buy” or drop the 2x chip from the purchase ticket.

### 8. Senior APY vs “double-digit”

Homepage: lenders earn “double-digit yield.”
Docs: Senior ~9% (single digit), Junior ~32% first-loss.

If most deposits go Senior, the headline is wrong. If Junior is the default, the risk is under-disclosed.

### 9. Transfer-hook KYC vs Path B

llms.txt: spAssets use Token-2022 transfer hooks that enforce wallet-level KYC.
Path B: xStocks/Ondo deposits need no Spout KYC.

If a transfer hook fires on *every* spAsset move, Path B still needs the destination wallet on a KYC allowlist. Document that, or testers will report “deposit failed: hook” as a bug.

### 10. Un-audited Monte Carlo as a FAQ answer

“Positive net returns in over 99% of simulated years” in a crash FAQ, with no notebook, no seed, no period, no assumption on IV crush. Cut it from the FAQ or link the model.

## DeFi / tokenization analysis

Spout is not competing with Aave USDC supply (~3.3%) against T-bills (~3.7%). It is competing with (a) selling the stock, (b) a broker margin loan, (c) Kamino/Jupiter xStocks lending, (d) JEPI-style overwriting ETFs.

The only edge that is actually new on Solana is **0% cash interest funded by selling calls on the borrower’s equity**, with Token-2022 KYC. That edge dies if:

- option assignment is not explained as a cost,
- weekly cycle lock is hidden,
- oracle/liquidation path is a TWAP that can be dumped (they themselves wrote up Morpho’s $36M TWAP cascade in their roundup).

For tokenized equities specifically: 50% LTV on NVDA-like names is aggressive if the oracle is a CEX print plus weekend gap. US equities do not trade 24/7; Solana does. The protocol must say which price it uses Saturday 03:00 UTC and whether liquidations run then.

Covered-call premium is **not** free lunch. Implied > realized on average, until it is not (2020, 2022, Aug 2024-style vol spikes). Junior 32% APY is the first-loss slice of that bet. Call it that.

## Product recommendations (priority)

1. One URL. Fix `https://https://beta.spout.finance/`.
2. Stop UA-gating docs. Path B is unreadable to non-browser clients.
3. Replace “0% interest” on the primary ticket with “0% cash APR · options overwrite · est. cycle cost.”
4. Put Path B (existing xStocks, no extra KYC) on the homepage for testers.
5. Show cycle end, unlock rules, and HF + liquidation price on the position.
6. Default Senior for lenders; Junior behind a first-loss warning.
7. Demo: faucet, invite code, oracle age, PoR link, testnet badge. Right now it looks like a static marketing site with a Connect button.
8. Spell out weekend oracle + liquidation hours.

## What I could not verify (honest)

- Did not complete KYC or a testnet borrow.
- Did not inspect the Token-2022 mint or transfer-hook program id.
- Did not see a live covered-call fill or Monday distribution.
- Invite-gated beta (`beta.spout.finance`) was not opened in this pass.

Those are the next things I would do with an invite code and a throwaway Phantom on testnet.

## Suggested Superteam answers

- **Major challenge:** gated beta + KYC Path A. Public demo does not prove a testnet borrow. Docs 403 to non-browser UAs.
- **Public content:** this file, published at the GitHub URL below.
