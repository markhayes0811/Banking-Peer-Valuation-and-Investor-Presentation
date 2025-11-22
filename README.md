# Peer Valuation Analysis — Banks & Fintech (USB, WFC, Block/SQ)

**Deliverables:**  
- `model/Equity_Valuation_Model_v2-Hayes.xlsx` — Excel model with Inputs, Comps, DCF (SQ), and WACC×g sensitivity.  
- `deck/Peer Valuation Analysis.pptx` — Investor-style deck summarizing assumptions and valuation ranges.  

> Presenter: **Mark Hayes**

---

## Why these companies?
I chose **two large U.S. banks** (USB, WFC) and **a scaled fintech** (Block/SQ) to compare traditional banking economics (best viewed with **P/B & ROTCE**) against a payments/fintech model (best viewed with **DCF** and **EV/EBITDA**). The contrast helps illustrate how conclusions change depending on the lens.

---

## What’s in the project
### Excel model
- **Inputs**: Price, Shares, Market Cap, Debt, Cash, **EV**, Equity.  
- **Comps**: P/E, P/B, EV/EBITDA with peer medians.  
- **DCF (SQ)**: 5‑year explicit forecast (Revenue, NI, D&A, CapEx, ΔWC), **PV of FCFs** + **PV(TV)**.  
- **Sensitivity**: Two‑way **WACC × Terminal growth** table (EV or Value/Share).

### Deck
- Executive summary, inputs snapshot, comparable multiples, **DCF for Block**, **WACC×g sensitivity heatmap**, and takeaways.  
- Baseline deck assumptions: **WACC = 9%** and **g = 2.5%** for the SQ DCF sensitivity. fileciteturn5file0

---

## Methodology 
**Banks (USB, WFC)**  
- Focus on **P/B & ROTCE**; DCF with **ΔWC ≈ 0** (loans/deposits are the operating book).  
- Use comps and benchmarking for relative value.

**Fintech (Block / SQ)**  
- **DCF** with Gordon terminal value:  
  - `FCF = NI + D&A − CapEx − ΔWC`  
  - `EV = Σ PV(FCFs) + PV(TV)`; `Equity = EV − NetDebt`; `Price = Equity / Shares`  
- **EV/EBITDA** complements the DCF.  
- Sensitivity: vary **WACC** and **g** to show a valuation range.

> Keep **g < WACC**; avoid TV dominating (>70–80%).

---

## Repository structure
```
.
├── README.md
├── deck/
│   └── Peer Valuation Analysis.pptx
├── model/
│   └── Equity_Valuation_Model_v2-Hayes.xlsx
├── src/
│   └── fill_from_polygon.py         # optional helper to refresh prices/financials
├── notebooks/
│   └── 01_exploration.ipynb         # optional visuals/EDA
├── requirements.txt                  # deps for src/ and notebooks/
└── .gitignore                        # ignore .env, __pycache__/, etc.
```

---

## Reproducibility & setup
**Excel‑only**
1) Open `model/Equity_Valuation_Model_v2-Hayes.xlsx` and refresh inputs if needed.  
2) Export tables/charts to the deck.

**With Python (optional)**
1) `pip install -r requirements.txt`  
2) Set your Polygon key (don’t commit secrets):  
   - macOS/Linux: `export POLYGON_API_KEY="your_key"`  
   - Windows (Powershell): `$Env:POLYGON_API_KEY="your_key"`  
3) Run the helper script:  
```bash
python src/fill_from_polygon.py --excel model/Equity_Valuation_Model_v2-Hayes.xlsx
```

**Example `requirements.txt`:**
```
openpyxl==3.1.5
requests>=2.31.0
pandas>=2.0.0
python-dotenv>=1.0.0
```

---


**Minimal script example**:
```python
# src/fill_from_polygon.py
import os, requests, argparse
from openpyxl import load_workbook

API = os.environ.get("POLYGON_API_KEY")

def last_close(ticker):
    import datetime as dt
    end = dt.date.today(); start = end - dt.timedelta(days=30)
    url = f"https://api.polygon.io/v2/aggs/ticker/{ticker}/range/1/day/{start}/{end}?adjusted=true&sort=desc&limit=1&apiKey={API}"
    data = requests.get(url, timeout=15).json()
    return (data.get('results') or [{}])[0].get('c')

def update_excel(path):
    wb = load_workbook(path); ws = wb['Inputs']
    row_by_ticker = {ws.cell(r,2).value: r for r in range(2, ws.max_row+1)}
    for t in ('USB','WFC','SQ'):
        px = last_close(t)
        if px and t in row_by_ticker:
            ws.cell(row_by_ticker[t], 4).value = float(px)  # Stock Price
    wb.save(path); print('Updated', path)

if __name__ == '__main__':
    ap = argparse.ArgumentParser(); ap.add_argument('--excel', default='model/Equity_Valuation_Model_v2-Hayes.xlsx')
    args = ap.parse_args()
    assert API, 'Set POLYGON_API_KEY in your environment.'
    update_excel(args.excel)
```


---

## Figures
Sensitivity heatmap (WACC × g):

![Sensitivity](assets/sensitivity_heatmap.png)
