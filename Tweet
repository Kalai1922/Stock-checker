import streamlit as st
import yfinance as yf
import pandas as pd
from datetime import date

st.set_page_config(layout="wide", page_title="Tweet Stock Checker")
st.title("🔎 Tweet Stock Checker — 4-Gate Study Framework")
st.caption("Tweets are a sourcing method, never a signal. Study first. Buy only if the system says so.")

# ---------------- Universe ----------------
V40 = """BAJAJHLDNG ABBOTINDIA AXISBANK PFIZER BERGEPAINT TITAN HINDUNILVR BATAINDIA LT RELIANCE MARICO BAJAJ-AUTO
KOTAKBANK TCS DABUR SBIN VOLTAS PGHH ITC BAJFINANCE ICICIBANK HCLTECH HDFCBANK HDFCLIFE GILLETTE HAVELLS COLPAL
PIDILITIND MARUTI HDFCAMC NESTLEIND ICICIPRULI ICICIGI ASIANPAINT GLAXO DMART PAGEIND INFY BAJAJFINSV""".split()

V40_NEXT = """CDSL BSE JIOFIN ANGELONE CAMS MCX ULTRACEMCO ACC TEAMLEASE ASTRAZEN CIPLA ERIS LALPATHLAB APOLLOHOSP MEDANTA
FORTIS ADANIPORTS JSWINFRA AWL GODREJCP DIXON KAJARIACER HONAUT DMART RELAXO BLUESTARCO BOSCHLTD EICHERMOT MRF M&M
TATAMOTORS HYUNDAI INDHOTEL ITCHOTELS UNITDSPR RADICO UBL VBL""".split()

V200 = """LTM PGHH WAAREEINDO TIPSMUSIC ICICIAMC COLPAL GILLETTE SANOFICONR WAAREERTL NESTLEIND PGHL GVPIL GVT&D MCX IGIL
ENRIN ESABINDIA PAGEIND JPOLYINVST WEBELSOLAR TCS GLAXO TENNIND CASTROLIND BSE HBLENGINE SANOFI ANANDRATHI INGERRAND
CRIZAC IEX 3MINDIA CAMS MARICO IRCTC OFSS ATLANTAELE EMMVEE ABBOTINDIA NAM-INDIA GRSE HDFCAMC HINDCOPPER TRAVELFOOD
DIXON GKENERGY CRAMC INFY GLENMARK NATIONALUM CUMMINSIND ITC MSUMI WAAREEENER HYUNDAI OSWALPUMPS SOLARINDS PRUDENT
GROWW BEL FORCEMOT MAZDOCK SHARDAMOTR TRITURBINE HEROMOTOCO SUZLON COALINDIA CHENNPETRO ECLERX AJANTPHARM PERSISTENT
TDPOWERSYS INOXINDIA POLYCAB BBTC CRISIL LGEINDIA ABSLAMC CDSL HAL ACE APLAPOLLO ACUTAAS APARINDS PIDILITIND
DDEVPLSTIK NBCC ENGINERSIN VIKRAMSOLR EICHERMOT HCLTECH ANTHEM KIRLPNU MSTCLTD GODFRYPHLP SHARDACROP HEXT TATAELXSI
ABB SKFINDIA LTIM POWERINDIA FIEMIND BLS KFINTECH BAYERCROP JYOTHYLAB CPPLUS HINDUNILVR RUBICON VSTIND RRKABEL
EMAMILTD GPPL INDIAMART LALPATHLAB STYL SCHAEFFLER NMDC JAMNAAUTO CGPOWER LTTS ASHOKA BLUEJET NEULANDLAB UNITDSPR
ASIANPAINT TANLA KPITTECH GABRIEL CHAMBLFERT SUPRIYA NEWGEN HAVELLS KSB CAPLIPOINT AVANTIFEED DOMS RADICO PFIZER
QUESS AJAXENGG ALIVUS DHANUKA MANYAVAR VOLTAMP COFORGE SUMICHEM KAJARIACER NSDL.BO TECHM RAILTEL ZENSARTECH
PETRONET JSWDULUX BALUFORGE REFEX MISHTANN.BO HSCL MPHASIS ELGIEQUIP COROMANDEL RITES BIKAJI DIVISLAB DATAPATTNS
ICICIGI BERGEPAINT BOSCHLTD FINEORG SIEMENS VESUVIUS VINATIORGA WABAG BLUESTARCO ALKEM GRINDWELL BSOFT LOTUSDEV
AIAENG TATATECH ELECON SUPREMEIND EIHOTEL CLEAN NIITMTS SUNPHARMA AHLUCONT GPIL KIRLOSBROS DABUR KEI
MUTHOOTFIN BAJFINANCE CHOLAFIN SHRIRAMFIN SBICARD SUNDARMFIN FIVESTAR""".split()

V50 = """COLPAL WAAREERTL GILLETTE GVT&D IGIL TCS BSE CASTROLIND IEX CAMS ABBOTINDIA BLS MSUMI INFY GRSE FORCEMOT MAZDOCK
ECLERX COALINDIA SHARDAMOTR CDSL CRISIL NBCC HCLTECH BAYERCROP TATAELXSI NMDC ACE ASIANPAINT LTTS TANLA GPPL
INDIAMART NEWGEN MUTHOOTFIN""".split()

def norm(t):
    t = t.strip().upper()
    return t if "." in t else t + ".NS"

def base(t):
    return t.replace(".NS", "")

def membership(sym):
    b = base(sym)
    tags = [n for n, l in [("V40", V40), ("V40 Next", V40_NEXT), ("V200", V200), ("V50 Verified", V50)]
            if b in l or sym in l]
    return tags

# ---------------- Pattern helpers ----------------
def pivots(df, w):
    h, l, n = df["High"].values, df["Low"].values, len(df)
    ph, pl = [], []
    for i in range(w, n - w):
        if h[i] == h[i - w:i + w + 1].max(): ph.append(i)
        if l[i] == l[i - w:i + w + 1].min(): pl.append(i)
    return ph, pl

def cluster(idx, prices, tol):
    if not idx: return []
    pts = sorted([(i, prices[i]) for i in idx], key=lambda x: x[1])
    groups, cur = [], [pts[0]]
    for p in pts[1:]:
        if abs(p[1] - cur[-1][1]) / cur[-1][1] <= tol: cur.append(p)
        else: groups.append(cur); cur = [p]
    groups.append(cur)
    return [{"p": sum(x[1] for x in g) / len(g), "i": sorted(x[0] for x in g)} for g in groups]

def touches(idxs, gap):
    kept = []
    for i in sorted(idxs):
        if not kept or i - kept[-1] >= gap: kept.append(i)
    return kept

def reversal(df):
    if len(df) < 3: return False
    h, l = df["High"].values[-3:], df["Low"].values[-3:]
    return h[-1] > h[-2] and l[-1] > l[-2]

# ---------------- The four strategies ----------------
def check_lth(df_max):
    ath = df_max["High"].max(); px = df_max["Close"].iloc[-1]
    fall = (ath - px) / ath * 100
    first = ath * 0.7
    if fall < 30: return False, f"Only {fall:.1f}% below ATH (needs 30%+)", {}
    stage = "2nd buy zone" if px <= first * 0.9 else "1st buy zone"
    return True, f"{fall:.1f}% below ATH — {stage}", {"1st buy": round(first, 2), "2nd buy": round(first * 0.9, 2), "Target": round(first * 1.2, 2)}

def check_v20(df):
    d = df.copy(); d["S"] = d["Close"].rolling(200).mean()
    dma, px = d["S"].iloc[-1], d["Close"].iloc[-1]
    if pd.isna(dma): return False, "Not enough history for 200 DMA", {}
    if px >= dma: return False, "Price is above 200 DMA (V20 needs below)", {}
    g = (d["Close"] >= d["Open"]).values; runs, s = [], None
    for i, x in enumerate(g):
        if x and s is None: s = i
        elif not x and s is not None: runs.append((s, i - 1)); s = None
    if s is not None: runs.append((s, len(d) - 1))
    for a, b in reversed(runs):
        zl, zh = d["Low"].iloc[a:b + 1].min(), d["High"].iloc[a:b + 1].max()
        if zl <= 0 or (zh - zl) / zl * 100 < 20 or b >= len(d) - 1: continue
        if (d["Close"].iloc[b + 1:] < zl).any(): continue
        if zl <= px <= zh:
            return True, f"Pulled back inside an unbroken 20%+ green-run zone, below 200 DMA", {
                "Zone low (exit if close below)": round(zl, 2), "Zone high (target)": round(zh, 2)}
        return False, "Zone found but price is outside it", {}
    return False, "No fresh, unbroken 20%+ green-run zone", {}

def check_cwh(df, fund_ok):
    d = df.reset_index(drop=True); h, l, n = d["High"].values, d["Low"].values, len(d)
    ph, _ = pivots(d, 5)
    for a in sorted(ph, reverse=True):
        lr = h[a]; end = min(n - 1, a + 260)
        if end - a < 20: continue
        seg = l[a + 1:end + 1]
        if len(seg) == 0: continue
        b = a + 1 + seg.argmin(); cb = l[b]
        depth = (lr - cb) / lr * 100
        if not (12 <= depth <= 50): continue
        seg2 = h[b + 1:end + 1]
        if len(seg2) == 0: continue
        c = b + 1 + seg2.argmax(); rr = h[c]
        if abs(rr - lr) / lr > 0.10 or c - a < 20 or c >= n - 1: continue
        hs = l[c + 1:min(n - 1, c + 25) + 1]
        if len(hs) == 0: continue
        hl = hs.min(); hd = (rr - hl) / rr
        if hd > 0.15 or hd > depth / 100 * 0.5: continue
        px = d["Close"].iloc[-1]; tgt = rr + (lr - cb)
        info = {"Handle high": round(rr, 2), "Handle low (weekly close below = exit)": round(hl, 2), "Target": round(tgt, 2)}
        if px > rr * 1.005: return True, "Confirmed Entry — broke above handle high", info
        if hl <= px <= rr and reversal(d):
            if fund_ok is True: return True, "Early Entry — handle base turning up (profit check passed, verify on Screener)", info
            return False, "Handle forming but Early Entry needs highest-ever profit (not confirmed)", info
        return False, "Cup+handle exists but no live trigger", info
    return False, "No valid cup with handle", {}

def check_rbs(df):
    d = df.reset_index(drop=True); ph, pl = pivots(d, 5)
    sup = cluster(pl, d["Low"].values, 0.03); res = cluster(ph, d["High"].values, 0.03)
    px = d["Close"].iloc[-1]; best = None
    for s in sup:
        for r in res:
            if r["p"] <= s["p"]: continue
            w = (r["p"] - s["p"]) / s["p"] * 100
            if w < 20: continue
            st_, rt = touches(s["i"], 5), touches(r["i"], 5)
            if len(st_) < 2 or len(rt) < 2: continue
            col = []
            for _, sd in sorted([(i, "S") for i in st_] + [(i, "R") for i in rt]):
                if not col or col[-1] != sd: col.append(sd)
            if col.count("S") < 2 or col.count("R") < 2: continue
            if best is None or w > best["w"]: best = {"s": s["p"], "r": r["p"], "w": w}
    if not best: return False, "No 20%+ zig-zag range found", {}
    near = best["s"] * 0.97 <= px <= best["s"] * 1.03
    info = {"Support (exit if decisive daily close below)": round(best["s"], 2), "Resistance (target)": round(best["r"], 2)}
    if near and reversal(d): return True, f"Range {best['w']:.0f}% wide, at support with reversal forming", info
    return False, f"Range {best['w']:.0f}% wide but price not at support with reversal", info

# ---------------- Fundamentals ----------------
def fundamentals(stock):
    out = {"fin": False, "yoy_q": None, "ann": None, "hi": None, "note": "", "pat_cr": None, "de": None, "roe": None, "roce": None}
    try:
        info = stock.info
        out["fin"] = info.get("sector") == "Financial Services"
        de = info.get("debtToEquity"); out["de"] = de / 100 if de is not None else None
        out["roe"] = info.get("returnOnEquity")
    except Exception: pass
    try:
        q = stock.quarterly_financials
        row = next((i for i in q.index if "Net Income" in i), None)
        if row:
            s = q.loc[row].dropna()
            if len(s) >= 5 and s.iloc[4] > 0: out["yoy_q"] = (s.iloc[0] - s.iloc[4]) / s.iloc[4] * 100
            if len(s) >= 4: out["pat_cr"] = s.iloc[:4].sum() / 1e7
            if len(s) >= 2 and s.nunique() > 1:
                out["hi"] = bool(s.iloc[0] >= s.max())
                out["note"] = "Quarters (latest→oldest, ₹ cr): " + ", ".join(f"{v/1e7:,.0f}" for v in s.values)
    except Exception: pass
    try:
        a = stock.financials
        row = next((i for i in a.index if "Net Income" in i), None)
        if row:
            s = a.loc[row].dropna()
            if len(s) >= 2 and s.iloc[1] > 0: out["ann"] = (s.iloc[0] - s.iloc[1]) / s.iloc[1] * 100
            ebit = next((i for i in a.index if i == "EBIT"), None)
            bs = stock.balance_sheet
            if ebit and not bs.empty and "Total Assets" in bs.index and "Current Liabilities" in bs.index:
                ce = bs.loc["Total Assets"].iloc[0] - bs.loc["Current Liabilities"].iloc[0]
                out["roce"] = a.loc[ebit].iloc[0] / ce
    except Exception: pass
    return out

def icon(v):
    return "🟡" if v is None else ("✅" if v else "❌")

# ---------------- UI ----------------
if "log" not in st.session_state: st.session_state.log = []

raw = st.text_input("Ticker(s) from today's tweets (comma separated, e.g. KAYNES, TRENT, IEX)", "")
source = st.text_input("Source (optional — who tweeted it)", "")

if st.button("Run 4-Gate Check") and raw.strip():
    for sym in [norm(x) for x in raw.split(",") if x.strip()]:
        st.divider(); st.subheader(f"📌 {base(sym)}")
        try:
            stock = yf.Ticker(sym); dmax = stock.history(period="max")
            if dmax.empty or len(dmax) < 60:
                st.warning("Not enough price data. Check the ticker symbol."); continue
            df2 = dmax.iloc[-504:]; px = dmax["Close"].iloc[-1]
            ath = dmax["High"].max(); f = fundamentals(stock); tags = membership(sym)

            # Gate 1
            if tags: g1, g1n = True, "In list: " + ", ".join(tags)
            else:
                if f["fin"]:
                    chk = [f["roe"] is not None and f["roe"] > 0.10, f["pat_cr"] is not None and f["pat_cr"] > 1000]
                    g1n = f"Not in any list. Financial proxy → ROE>10%: {icon(chk[0])}, PAT>₹1000cr: {icon(chk[1])}"
                else:
                    chk = [f["de"] is not None and f["de"] < 0.25, f["roce"] is not None and f["roce"] > 0.20,
                           f["pat_cr"] is not None and f["pat_cr"] > 200]
                    g1n = f"Not in any list. Proxy → D/E<0.25: {icon(chk[0])}, ROCE>20%: {icon(chk[1])}, PAT>₹200cr: {icon(chk[2])}"
                g1 = all(chk)
                g1n += " (proxy only — confirm on Screener; also check PSU / retail holding >30%)"
            st.markdown(f"**Gate 1 — Universe** {icon(g1)}  \n{g1n}")

            # Gate 2
            pos = [x for x in (f["yoy_q"], f["ann"]) if x is not None]
            g2 = None if not pos else all(x > 0 for x in pos)
            q_txt = f"{f['yoy_q']:.1f}%" if f["yoy_q"] is not None else "n/a"
            a_txt = f"{f['ann']:.1f}%" if f["ann"] is not None else "n/a"
            st.markdown(f"**Gate 2 — Business quality** {icon(g2)}  \nQuarterly PAT YoY: {q_txt} | Annual PAT growth: {a_txt}  \n"
                        f"{f['note']}  \n⚠️ Confirm highest-ever profit + pledge % + cash flow on Screener (10-yr view).")

            # Gate 3
            fall = (ath - px) / ath * 100
            below = px < 0.8 * ath
            st.markdown(f"**Gate 3 — Strategy trigger**  \nPrice ₹{px:.2f} | ATH ₹{ath:.2f} | {fall:.1f}% below ATH — "
                        f"{'✅ below 0.8×ATH' if below else '❌ above 0.8×ATH, most targets would exceed ATH'}")
            hits = []
            for name, fn in [("LTH", lambda: check_lth(dmax)), ("V20", lambda: check_v20(df2)),
                             ("CWH", lambda: check_cwh(df2, f["hi"])), ("RBS", lambda: check_rbs(df2))]:
                try: ok, msg, info = fn()
                except Exception as e: ok, msg, info = False, f"error ({e})", {}
                if ok: hits.append(name)
                st.markdown(f"- {'✅' if ok else '▫️'} **{name}** — {msg}")
                if ok and info: st.json(info)

            # Verdict
            if not g1: verdict = "SKIP — Not our universe"
            elif g2 is False: verdict = "SKIP — Profits not improving"
            elif hits: verdict = "STUDY-READY — " + ", ".join(hits)
            else: verdict = "WATCH — Quality passes, no setup yet"
            (st.success if verdict.startswith("STUDY") else st.info if verdict.startswith("WATCH") else st.error)(f"Gate 4 — Verdict: **{verdict}**")
            if verdict.startswith("STUDY"):
                st.caption("Before any entry: strategy confluence ✔ · ATH/support verified on TradingView ✔ · target upside meaningful ✔ · "
                           "Confirmed Entry preferred ✔ · business verified on Screener ✔ · one clean pick over several average ones ✔")

            st.session_state.log.append({"Date": date.today().isoformat(), "Stock": base(sym), "Source": source,
                                         "Universe": "Y" if g1 else "N", "Profit trend": icon(g2),
                                         "Setup": ", ".join(hits) or "—", "Verdict": verdict})
        except Exception as e:
            st.warning(f"Could not process {sym}: {e}")

if st.session_state.log:
    st.divider(); st.subheader("📒 Today's study log (paste into your Master Ledger)")
    log = pd.DataFrame(st.session_state.log)
    st.dataframe(log, use_container_width=True)
    st.download_button("Download log CSV", log.to_csv(index=False), "tweet_study_log.csv", "text/csv")
