# The Index Rebalance Playbook

### How passive money mechanically moves stocks — read through three case studies: Tesla (S&P 500), DigitalOcean (Russell 1000), and SpaceX (Nasdaq-100)

> **The one idea:** Being added to (or promoted within) a major index changes a
> stock's **ownership structure and forced order flow** — *not* its intrinsic
> value. The sharp, often violent price moves around a rebalance are **mechanical**
> and frequently **mean-revert**. Learn to separate the flow from the fundamentals.

---

## 1. The foundations (the math that drives everything)

### 1.1 Market cap vs. public float
- **Total market cap** = value of *all* shares (public + locked-up insider/strategic shares).
- **Public float** = value of only the shares *freely tradable* on the exchange.
- Media headlines rank companies by **market cap**. Index providers weight them by
  **float-adjusted market cap** — because a fund can only buy what actually trades.

A company can be a giant by market cap but *small* by float. That gap is the source
of most rebalance surprises.

### 1.2 The low-float multiplier (the detail most explanations miss)
When a company's float is unusually small relative to its size, some index
methodologies apply a **weighting multiplier** (Nasdaq: up to **~3×**) to its float
so a genuine mega-cap isn't absurdly underweighted just because little of it trades.

> ⚠️ **Common misconception:** "weight = float ÷ total index float." For low-float
> names that's wrong — the multiplier is applied *first*. (See the SpaceX case: ~$89B
> raw float becomes ~$267B **effective** weight before the division.)

### 1.3 Passive demand = index weight × tracking AUM
A passive fund is *legally obligated* to mirror its benchmark. So the forced buy/sell
for any name is roughly:

```
forced flow ≈ (stock's weight in the index) × (assets tracking that index)
```

Both terms matter — a big weight in a small-AUM index and a tiny weight in a huge-AUM
index can create very different flows (this asymmetry is the whole DigitalOcean story).

### 1.4 Tracking error — why funds trade at the *exact* close
**Tracking error** is how far a fund's return drifts from its benchmark. To minimize
it, index funds execute their rebalance at the **official closing price on the
effective date**, via **Market-on-Close (MOC)** orders funneled into the
**closing cross / auction** — one auction, one price, matched at 4:00 PM ET. Trading
at that single reference price means the fund's execution ≈ the index's mark → near-zero
tracking error.

### 1.5 Fixed-count vs. tiered indices (who gets kicked out?)
- **Fixed-count** (S&P 500 = 500 seats, Nasdaq-100 = 100 seats): an **add forces a
  delete**, and every other constituent is trimmed pro-rata to fund the newcomer.
- **Whole-universe / tiered** (Russell 1000 large-cap vs Russell 2000 small-cap): names
  **migrate between tiers** as their market cap changes — a "graduation" or "demotion,"
  not a straight in/out.

### 1.6 Sampling — the escape hatch for tiny additions
Big funds don't have to buy every micro-weight addition. Via **sampling/optimization**
they can skip a tiny, volatile new name and instead nudge up a correlated large holding
(Apple, Microsoft…) to replicate the index's *risk* without the trading headache. This
means the **smallest additions receive less forced buying** than a naïve weight × AUM
figure implies.

---

## 2. The two engines behind every rebalance move

### 2.1 The add-vs-delete asymmetry (why "graduations" often fall first)
This is the counter-intuitive core:

| | Stock **leaving** a tier / being **deleted** | Tiny **new addition** at the bottom of a big index |
|---|---|---|
| Weight being changed | **Large** (it was a meaningful holding) | **Microscopic** |
| Tracking error if you *don't* trade | **Big** — you're badly off-benchmark | **Trivial** — a rounding error |
| Fund behavior | **Forced seller of 100%, at any price** | Relaxed buyer; may even *sample* and skip |
| Net pressure | **Urgent, price-insensitive selling** | Weak, unhurried buying |

So when a stock is **promoted from small-cap to large-cap**, the **big weight it loses**
in the small-cap index (forced selling) **outweighs** the **tiny weight it gains** in the
large-cap index (relaxed buying). The mechanical net is **selling into the event**, then
a **rebound** once forced selling clears and active buyers step into the discount.

### 2.2 Why the price sometimes spikes and sometimes fades
The **direction** on the effective day depends on **share availability**, not the
headline:
- **Share shortage → violent spike.** If market makers can't pre-source enough stock,
  forced buyers chase the price up into the cross (the **Tesla** outcome).
- **Ample pre-positioning → flat or fade.** If market makers hoarded enough inventory to
  hand over at 4:00 PM (or the float is deep), the price can drift *down* on inclusion
  day — the classic **"buy the rumor, sell the news."**

Market makers (Citadel, etc.) spend the run-up **pre-buying** to sell into the closing
cross — they are the buffer that determines whether the add pops or flops.

---

## 3. Case study A — Tesla → S&P 500 (Dec 2020): the violent add

- **Setup:** Announced **Nov 16, 2020**; **effective Mon Dec 21, 2020**, replacing
  Apartment Investment & Management (AIV). Rebalance trade executed at the close on
  **Fri Dec 18, 2020** — which was also a **triple-witching** day (options + index
  options + index futures all expiring, amplifying volume).
- **Scale:** The **largest company ever added** to the S&P 500 at the time —
  ~**$624B** market cap, entering at a **1.69% weight** (5th-largest holding). YTD gain
  into inclusion: **+730%**.
- **The finale:** Forced buyers had to own it *at any price* to avoid tracking error.
  Tesla closed at **$695.00 — an all-time high**, up ~**6%** on the day. The session saw
  **200M+ shares** (4×+ normal), and the **closing cross was a record single-stock
  auction (~69M shares ≈ ~$48B notional)**.
- **The lesson:** Maximum forced buying → a spike *into* the close, then Tesla **fell the
  following week**. The inclusion was the *event to sell*, not to chase.

> 🧹 **Fact-check note:** the "69 million shares worth ~$150 billion" figure conflates two
> numbers. 69M × $695 ≈ **$48B** (the record *closing-auction* print). The ~**$148B** was
> **total-market** volume that triple-witching day — not Tesla's auction alone. Also,
> Tesla joined the **S&P 500**, not the Nasdaq-100.

---

## 4. Case study B — DigitalOcean → Russell 1000 (Jun 2026): the graduation that sold off

- **Catalyst:** A **+425%** year (AI-cloud growth + analyst upgrades) pushed DOCN's cap
  to ~**$15.6B**, promoting it from the small-cap **Russell 2000** to the large-cap
  **Russell 1000**.
- **Timeline:** Preliminary lists (the announcement) **Fri May 22**; changes **effective
  after the close Fri Jun 26**; **live Mon Jun 29**.
- **The weight asymmetry:** A **heavy weight** in the Russell 2000 (a top small-cap name)
  vs a **microscopic** weight (~hundredths of a percent) at the bottom of the Russell 1000.
  → Forced R2000 **selling dwarfed** R1000 buying (see §2.1).
- **Into the cross:** **Five straight down days**; **~15.4M shares** (5–6× normal) crossed
  at the **Jun 26** close, bottoming at **$139.28**. Outgoing small-cap funds were purging
  100% of the position.
- **The rebound:** **Mon Jun 29, +7.59% to $149.86** — forced selling gone, active
  managers bought the mechanically-discounted stock.
- **The lesson:** *An index promotion is not a valuation catalyst.* The announcement barely
  moved the stock; the violence was pure flow, and it **mean-reverted**.

---

## 5. Case study C — SpaceX → Nasdaq-100 (Jul 2026): the low-float mega-cap

- **Setup:** After a **~$1.75–2T mega-IPO**, SpaceX now trades on Nasdaq. Under a
  **new Nasdaq methodology (effective May 1, 2026)** — top-40 newly listed names can enter
  after just **15 trading days**, and the minimum-float requirement was removed — it was
  **fast-tracked** into the index.
- **Timeline:** Announced **Fri Jun 26**; **effective before the open Tue Jul 7**; the
  rebalance trade lands at the **close of Mon Jul 6**. The **July 4th holiday weekend**
  (market closed Fri Jul 3) leaves exactly **5 trading days** to prepare.
- **The float twist (the crux):** Only ~**4.3% of shares float (~$89B)**. But a
  ~**3× low-float multiplier** lifts its **effective** index weight to ~**$267B → ~0.6%**
  of the Nasdaq-100. *This is why "raw float ÷ total float" understates the weight.*
- **The forced buy:** ~**0.6%** of Nasdaq-100 passive AUM (QQQ and peers) ≈ a **multi-billion
  (~$4B+) mandatory buy**. This figure **floats upward** all week if SpaceX's price or fund
  assets rise. Because the Nasdaq-100 has a **fixed 100 seats**, one constituent is dropped
  and the rest trimmed pro-rata to fund it.
- **The finale — twin auctions at the Jul 6 close:**
  - **SpaceX (the add):** record-breaking volume. A vertical spike is **not guaranteed** —
    if market makers pre-sourced enough shares it can stay flat or drift down (the fade
    scenario); if there's a share shortage it spikes violently (the Tesla scenario).
  - **The deletion:** index funds simultaneously dump 100% of the removed name → a sharp
    drop into the 4:00 PM bell.

> 🧹 **Fact-check note:** the specific deleted stock should be read from **Nasdaq's official
> notice**, not assumed. In particular, **Walgreens (WBA) is a poor guess** — it was taken
> **private by Sycamore Partners in 2025** and is no longer an index constituent.

---

## 6. Putting the three side by side

| | **Tesla** | **DigitalOcean** | **SpaceX** |
|---|---|---|---|
| Index | S&P 500 | Russell 1000 (from R2000) | Nasdaq-100 |
| Event type | New add (fixed-count) | Tier promotion | New add (fixed-count, low-float) |
| Effective | Dec 21, 2020 | Jun 29, 2026 | Jul 7, 2026 |
| Weight at entry | 1.69% (5th largest) | ~0.0x% (bottom of R1000) | ~0.6% (float-multiplied) |
| Dominant flow | Forced **buying** | Net forced **selling** | Forced **buying** |
| Effective-day move | **Spike** to ATH ($695) | **Sold off** into cross ($139), then rebounded | **TBD** (spike vs fade) |
| Why | Share shortage → chase | Big-weight exit > tiny-weight entry | Depends on MM pre-sourcing |

---

## 7. The unifying takeaways

1. **Index membership ≠ value.** A rebalance rewires *who must own the stock*, not what
   it's worth. Reprice your thesis on fundamentals, not the index headline.
2. **Follow the tracking error.** Whoever faces the **larger tracking error** trades most
   aggressively. Forced sellers of a big-weight deletion are more desperate than relaxed
   buyers of a tiny-weight addition.
3. **The closing cross is where it happens.** The real trade is the MOC auction on the
   effective date — that's where the volume and the price dislocation concentrate.
4. **Direction depends on share availability, not the headline.** Same event ("added to
   the index") → spike (Tesla) or fade (many others), decided by market-maker inventory.
5. **Mechanical dislocations mean-revert.** DigitalOcean's slide-then-pop is the archetype:
   forced flow creates a temporary discount/premium that unwinds once the flow is done.

---

## 8. Glossary

- **Float-adjusted market cap** — value of freely tradable shares; the basis for index weights.
- **Low-float multiplier** — an upward weight adjustment (Nasdaq: up to ~3×) for thin-float names.
- **Tracking error** — deviation of a fund's return from its benchmark; index funds minimize it.
- **Closing cross / MOC** — the 4:00 PM auction that matches Market-on-Close orders at one price.
- **Reconstitution** — a scheduled index membership rebuild (e.g., Russell's annual/semi-annual event).
- **Sampling / optimization** — replicating an index's risk without holding every constituent.
- **Triple witching** — simultaneous expiry of stock options, index options, and index futures; amplifies volume.

---

## Sources

- SpaceX → Nasdaq-100: [Seeking Alpha](https://seekingalpha.com/news/4607865-spacex-to-join-nasdaq-100-effective-july-7-2026) · [CNBC](https://www.cnbc.com/2026/06/26/spacex-added-to-nasdaq-100-on-hold-on-hold-on-hold.html) · [CME Group](https://www.cmegroup.com/articles/2026/the-spacex-mega-ipo-why-index-choice-matters.html) · [Morningstar](https://www.morningstar.com/funds/spacex-ipo-how-index-funds-are-adapting)
- Tesla → S&P 500: [S&P Global](https://www.spglobal.com/en/research-insights/market-insights/tesla-added-to-the-sp-500) · [CNBC](https://www.cnbc.com/2020/12/20/tesla-enters-the-sp-500-with-1point69percent-weighting-in-the-benchmark-fifth-largest.html) · [Bloomberg](https://www.bloomberg.com/news/articles/2020-11-16/tesla-will-join-s-p-500-in-december-as-largest-ever-new-member)
- DigitalOcean → Russell 1000: [Business Wire](https://www.businesswire.com/news/home/20260630757085/en/DigitalOcean-Added-to-the-Russell-1000-Index) · [LSEG / FTSE Russell schedule](https://www.lseg.com/en/media-centre/press-releases/ftse-russell/2026/russell-reconstitution-2026-schedule)

---

*⚠️ Educational only — not financial advice. Figures for forward-looking events (SpaceX)
are estimates that move with price and fund assets; verify specifics (including the exact
deleted constituent) against the index provider's official notices.*
