# Climate Transition Tracker

A self-contained, single-file web app that tracks the **climate transition as an investment theme** — 91 names across Europe, China and the US, from Iberdrola and Siemens Energy to BYD and CATL, plus a news feed of the most important climate-sector stories.

No build step. No dependencies. No backend. **No API key** — one HTML file and your browser. (The only external asset is the optional pixel-font stylesheet used by the retro theme; the app works fine without it.)

## Quick start

1. Download `index.html` and open it in any modern browser (double-click is enough).
2. That's it. Prices, charts, index and news load immediately.

Live quotes come from Yahoo Finance's unofficial API through free public CORS proxies, so occasional hiccups are possible — the app retries through a chain of three proxies and caches everything in `localStorage`, so it degrades gracefully rather than breaking.

## The watchlist

| Group | Tickers |
| --- | --- |
| Iberian Energy | IBE.MC, ANA.MC, ELE.MC, NTGY.MC, RED.MC, SLR.MC, GRE.MC, ENG.MC |
| European Utilities & Renewables | ORSTED.CO, RWE.DE, EOAN.DE, ENEL.MI, ENGI.PA, EDPR.LS, VER.VI, FORTUM.HE, NESTE.HE |
| Transitioning Oil & Gas | SHEL, TTE, EQNR, BP, REP.MC |
| Nuclear | CEG, VST, OKLO, SMR, BWXT, CEZ.PR |
| Wind & Solar Manufacturing | VWS.CO, NDX1.DE, ENR.DE, FSLR, ENPH, SEDG, 601012.SS, 688599.SS, 002459.SZ, 600438.SS, 002202.SZ, 601615.SS |
| Grid & Electrification | SU.PA, ABBN.SW, SIE.DE, ETN, PRY.MI, NEX.PA, LR.PA, NKT.CO, 300274.SZ, 002028.SZ |
| Energy Storage & Industrial Batteries | FLNC, STEM, EOSE, 300014.SZ, 002074.SZ |
| EV & Batteries | 002594.SZ, TSLA, 300750.SZ, 002460.SZ, NIO, XPEV, LI, RNO.PA, VOLV-B.ST, PSNY |
| Heat Pumps & Efficiency | NIBE-B.ST, ROCK-B.CO, KRX.IR, CARR, TT |
| Hydrogen | ITM.L, NEL.OL, NCH2.DE, PLUG, CWR.L, BE |
| Water & Waste | VIE.PA, XYL, AWK, SVT.L, UU.L, PNR, 0371.HK |
| Climate Trackers & Carbon | ICLN, INRG.L, PBW, QCLN, FAN, KRBN, KCCA, 400570.KS |

Every stock is tagged **PURE-PLAY**, **MAJOR** or **TRACKER** — the difference between a company whose whole business is the transition and a diversified giant doing both, visible at a glance.

## Features

- **Live quote cards** — price, day change, and a real sparkline per stock, grouped in tabs. Prices are shown in each stock's own currency: €, $, ¥, kr, CHF, £ (pence), Kč, ₩, HK$.
- **Timeframe switcher (1M / 1W / 1D)** — a segmented control in the header switches every sparkline, the index chart and the index stat between one month of daily closes, one week of 15-minute bars, and one day of 5-minute bars. Day change is always measured against yesterday's official close (not the last bar), and 1D charts anchor their line at yesterday's close so overnight gaps stay visible — a stock that gaps up 30% at the open shows the gap, not just the fade. Each timeframe has its own cache, and switching mid-refresh is safe: the in-flight fetch for the old timeframe silently discards itself.
- **Chart hover** — move the cursor over any chart to read exact values: the index chart shows a crosshair with the date/time and the % of both index lines, and each stock sparkline shows price + time in its caption.
- **News ticker** — the 20 newest headlines scroll continuously across a bar pinned to the top of the page, with a colored dot per source. Hovering pauses the scroll, clicking a headline opens the story, and the feed refreshes itself every 30 minutes.
- **Retro CRT layout** — a `RETRO` toggle in the header switches the whole app into an 80s/90s green-phosphor terminal: Press Start 2P headers, VT323 body text, chunky square borders, scanlines and a soft CRT glow. The charts re-skin to the phosphor palette with thicker lines, but render at full resolution so values and tooltips stay crisp. Your choice is remembered; the pixel fonts load from Google Fonts and fall back to monospace offline.
- **Climate Transition Index** — a market-cap-weighted basket of the 83 stocks (ETFs excluded), plotted against an equal-weighted version of the same basket. The gap between the two lines shows whether the transition trade is concentrated in the giants or broadening out. (Headline finding: Tesla alone is ~32% of the cap-weighted index — the equal-weight line is the counterweight.)
- **News feed** — a 📰 News tab merging five RSS sources: Carbon Brief, Climate Home News, The Guardian (environment), CleanTechnica and PV Magazine. Deduplicated, newest-first, with per-source filter chips.
- **Resilient data fetching** — quotes are batched into multi-symbol requests (7 requests for 91 names) and fetched through a proxy chain (allorigins → codetabs → r.jina.ai) with automatic failover, dead-proxy cooldowns and per-symbol retry. The app kept working through a full-day outage of the primary proxy on 13 Nov 2026.
- **Honest failure states** — no data is ever invented. Missing symbols, dead proxies and market closures are reported explicitly.

## How it works

**Data.** Yahoo Finance's unofficial `spark` and `chart` endpoints, called through free CORS proxies because the browser can't talk to Yahoo directly (no `Access-Control-Allow-Origin`). The multi-symbol `spark` endpoint takes ~15 tickers per call, which is what makes the three-proxy chain viable — 7 requests per full refresh instead of 91.

**Currencies.** The spark endpoint doesn't return a currency, so it's derived from each ticker's exchange suffix (`.MC` → EUR, `.SW` → CHF, `.SZ` → CNY, `.L` → GB pence, …). London-listed prices display as pence; market-cap weights are converted to EUR.

**Index methodology.** Each stock is normalized to its first close in the active window (1M, 1W or 1D), forward-filled to the union of timestamps across exchanges, then blended:

```
index_level = Σ  weightᵢ × (priceᵢ / base_priceᵢ)
```

Cap weights are market caps in EUR (as of 13 Nov 2026, baked in at build time); the index starts at 1.0, so the level reads directly as "the basket is up X% over the active window." Stocks that haven't traded yet in the window — Asian markets open first, then Europe, then the US — are held at their base price rather than excluded, so the basket is complete from the very first point and the line doesn't draw fake staircase steps as each region's market cap joins. The equal-weight line applies `weightᵢ = 1/n`.

**News.** RSS feeds fetched through the same proxy chain. When the request lands on the r.jina.ai reader (which renders feeds as markdown), a second parser handles that format — one code path per proxy type. Refreshes at most every 30 minutes.

**Storage layout** (all in `localStorage`):

| Key | Contents |
| --- | --- |
| `climate_cache_v1_1M` / `_1W` / `_1D` | per-symbol quotes + history per timeframe (survives restarts) |
| `climate_tf` | active timeframe preference |
| `climate_news_v1` | cached news items |

## Disclaimer

Market data from unofficial APIs, delayed and possibly wrong. **Not investment advice.** For study and portfolio watching only.
