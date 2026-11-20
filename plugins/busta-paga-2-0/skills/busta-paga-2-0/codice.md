# Codice di riferimento (Busta paga 2.0)

Funzioni già collaudate, da copiare nello script di generazione (Python + WeasyPrint).
Nei blocchi HTML/CSS dentro f-string le graffe CSS vanno raddoppiate (`{{ }}`).
I numeri nei commenti sono solo esempi: sostituiscili con quelli letti dalla busta paga e dai siti.

## Formato importi

```python
import math
from weasyprint import HTML

def eur(v):
    s = f"{v:,.2f}".replace(",", "X").replace(".", ",").replace("X", ".")
    return s + "\u00a0€"

def mld(v, d=1):
    s = f"{v:,.{d}f}".replace(",", "X").replace(".", ",").replace("X", ".")
    return s + " mld"
```

## Torta SVG (pagine 2 e 4)

`slices` = lista di tuple `(nome, valore, colore, ...)`. Etichette percentuali dentro le fette >= 4%.
Non riusare il nome della variabile che contiene la torta (es. `big`) per altre cose: la torta sparirebbe.

```python
def pie(slices, total, r=150, cx=170, cy=170, labels=True, donut=0):
    out = []
    a0 = -math.pi / 2
    for name, v, col, *_ in slices:
        a1 = a0 + 2 * math.pi * v / total
        large = 1 if a1 - a0 > math.pi else 0
        x0, y0 = cx + r * math.cos(a0), cy + r * math.sin(a0)
        x1, y1 = cx + r * math.cos(a1), cy + r * math.sin(a1)
        out.append(f'<path d="M{cx},{cy} L{x0:.2f},{y0:.2f} A{r},{r} 0 {large} 1 {x1:.2f},{y1:.2f} Z" fill="{col}" stroke="#fff" stroke-width="5" stroke-linejoin="round"/>')
        if labels and v / total >= 0.04:
            am = (a0 + a1) / 2
            lr = r * (0.78 if v / total < 0.08 else 0.62)
            lx, ly = cx + lr * math.cos(am), cy + lr * math.sin(am)
            pct = round(v / total * 100)
            out.append(f'<text x="{lx:.1f}" y="{ly + 10:.1f}" text-anchor="middle" font-family="Poppins" font-weight="700" font-size="{30 if pct > 15 else (22 if pct > 8 else 16)}" fill="#fff">{pct}%</text>')
        a0 = a1
    if donut:
        out.append(f'<circle cx="{cx}" cy="{cy}" r="{donut}" fill="#fff"/>')
    return f'<svg viewBox="0 0 {2*cx} {2*cy}" xmlns="http://www.w3.org/2000/svg">{"".join(out)}</svg>'
```

## Grafico a quadrati (pagina 3)

`spese` = lista `(nome, miliardi, colore)` con le voci Pensioni, Assistenza, Sanità, Istruzione,
Interessi sul debito, Difesa, Altro. `TW, TH = 184, 96` (mm). `altro_list` e `assist_list` sono tabelle
HTML `<table class="alist">` con le sottovoci; `spese_sub` contiene i sottotitoli brevi
(`{"Pensioni": "vecchiaia e reversibilità", "Altro": "__LIST__"}`).

```python
# Layout fisso: [Pensioni sopra / Assistenza sotto] | [Sanita' / Istruzione / Interessi+Difesa] | [Altro]
# Prima si garantisce a ogni quadrato lo spazio minimo per le etichette, poi si distribuisce il resto in proporzione.
def split(total, vals, mins):
    out = [None] * len(vals); free = list(range(len(vals))); room = total
    while True:
        tv = sum(vals[i] for i in free)
        bad = [i for i in free if room * vals[i] / tv < mins[i]]
        if not bad: break
        for i in bad: out[i] = mins[i]; room -= mins[i]; free.remove(i)
    for i in free: out[i] = room * vals[i] / tv
    return out
S = {n: (n, v, c) for n, v, c in spese}
V = lambda *ns: sum(S[n][1] for n in ns)
w1, w2, w3 = split(TW, [V("Pensioni", "Assistenza"), V("Sanità", "Istruzione", "Interessi sul debito", "Difesa"), V("Altro")], [60, 60, 52])
hp, ha = split(TH, [V("Pensioni"), V("Assistenza")], [34, 34])
hs, hi, hb = split(TH, [V("Sanità"), V("Istruzione"), V("Interessi sul debito", "Difesa")], [26, 24, 28])
wi, wd = split(w2, [V("Interessi sul debito"), V("Difesa")], [33, 25])
layout = [
    ("Pensioni", (0, 0, w1, hp)), ("Assistenza", (0, hp, w1, ha)),
    ("Sanità", (w1, 0, w2, hs)), ("Istruzione", (w1, hs, w2, hi)),
    ("Interessi sul debito", (w1, hs + hi, wi, hb)), ("Difesa", (w1 + wi, hs + hi, wd, hb)),
    ("Altro", (w1 + w2, 0, w3, TH)),
]
spese = [S[n] for n, _ in layout]; rects = [r for _, r in layout]

tiles = []
for (n, v, c), (x, y, w, h) in zip(spese, rects):
    tbig = w * h > 1500 and h > 30
    sub = spese_sub.get(n, "") if w > 30 else ""
    head = (f'<div class="tn{" big" if tbig else ""}">{n}</div>'
            f'<div class="tp{" big" if tbig else ""}">{str(round(v/USCITE_TOT*100,1)).replace(".",",")}%</div>'
            f'<div class="tm">{mld(v)}</div>')
    style = f'left:{x:.2f}mm;top:{y:.2f}mm;width:{w:.2f}mm;height:{h:.2f}mm;background:{c}'
    if n == "Assistenza":   # titolo a sinistra, elenco a destra
        body = f'<div class="tsplit"><div class="tleft">{head}</div><div class="tright">{assist_list}</div></div>'
    elif sub == "__LIST__":
        body = head + altro_list
    else:
        body = head + (f'<div class="ts">{sub}</div>' if sub else "")
    tiles.append(f'<div class="tile" style="{style}">{body}</div>')
treemap = f'<div class="treemap" style="width:{TW}mm;height:{TH}mm">{"".join(tiles)}</div>'
```

CSS della pagina 3:

```css
/* page 3 */
.p3 { page-break-before: always; }
.p3 h1 { font-size: 24pt; text-align: center; }
.p3 .sub3 { font-size: 12.5pt; text-align: center; margin: 6px 0 2px; }
.p3 .sub3 b { font-family: Poppins; }
.p3 .yr { font-size: 8pt; color: #6b7385; text-align: center; margin-bottom: 5mm; }
.treemap { position: relative; margin: 0 auto; }
.tile { position: absolute; border: 2px solid #fff; border-radius: 8px; color: #fff; padding: 3.2mm 3.6mm; overflow: hidden; }
.tn { font-family: Poppins; font-weight: 700; font-size: 9pt; line-height: 1.1; }
.tn.big { font-size: 13pt; }
.tp { font-family: Poppins; font-weight: 700; font-size: 12pt; line-height: 1.15; margin-top: 1mm; }
.tp.big { font-size: 22pt; }
.tm { font-size: 9pt; font-weight: 600; opacity: .95; }
.alist { width: 100%; margin-top: 2.2mm; border-collapse: collapse; }
.alist td { font-size: 7pt; line-height: 1.2; padding: 0.3mm 0; border-bottom: 1px solid rgba(255,255,255,.22); color: #fff; }
.alist td:last-child { text-align: right; font-weight: 600; padding-left: 1.5mm; white-space: nowrap; }
.ts { font-size: 7.6pt; opacity: .85; margin-top: 1mm; }
.p3 h2 { font-size: 13.5pt; text-align: center; margin: 9mm 0 1mm; line-height: 1.3; }
.p3 h2 .g { color: #1a9a5c; } .p3 h2 .o { color: #e06d1f; }
.hole { font-size: 9.5pt; text-align: center; margin: 1mm -4mm 1.5mm; white-space: nowrap; }
.pm { font-family: Inter; font-size: 9pt; font-weight: 400; color: #6b7385; }
.p4 .intro { font-size: 11pt; }
.p4 .bigpie { width: 80mm; height: 80mm; margin: 5mm auto 5mm; }
.disc { font-size: 7.6pt; color: #6b7385; margin-top: 4mm; }
.infl { font-size: 8.4pt; margin-top: 2px; }
.infl b { color: #e06d1f; }
.infl span { color: #6b7385; font-size: 7.4pt; }
.tsplit { display: flex; gap: 3mm; }
.tleft { flex: none; }
.tright { flex: 1; }
.tright .alist { margin-top: 0; }
.tright .alist td { font-size: 6.7pt; white-space: nowrap; }
.bars { width: 184mm; margin: 4mm auto 0; }
.src { font-size: 7.4pt; color: #6b7385; margin-top: 6mm; border-top: 1px solid #eef0f4; padding-top: 5px; }
```

## Grafico a barre delle pensioni (pagina 3)

`fasce` = lista `(etichetta asse X, miliardi, milioni di pensionati)`, es. `("Da 2000€ lordi", 56.68, 2.11)`.

```python
def mil(v): return (f"{v:.1f}".replace(".", ",")) + "M"
BW, BH, top, base = 700, 360, 62, 328
slot = BW / len(fasce); mx = max(f[1] for f in fasce)
bars = []
for i, (lab, cost, num) in enumerate(fasce):
    h = (base - top) * cost / mx; bx = i * slot + 14; bw = slot - 28; by = base - h
    cx = i * slot + slot / 2
    bars.append(f'<rect x="{bx:.1f}" y="{by:.1f}" width="{bw:.1f}" height="{h:.1f}" rx="5" fill="#5B8DEF"/>')
    ty = by - 34
    bars.append(f'<text x="{cx:.1f}" y="{ty:.1f}" text-anchor="middle" font-size="11" fill="#1d2433">'
                f'<tspan x="{cx:.1f}" font-weight="700">Costo: {mld(cost, 1 if cost < 10 else 0)}</tspan>'
                f'<tspan x="{cx:.1f}" dy="12.5">per {mil(num)} di</tspan>'
                f'<tspan x="{cx:.1f}" dy="12.5">pensionati</tspan></text>')
    bars.append(f'<text x="{cx:.1f}" y="{base+17}" text-anchor="middle" font-size="11.5" font-weight="600" fill="#1d2433">{lab}</text>')
bars.append(f'<line x1="0" x2="{BW}" y1="{base}" y2="{base}" stroke="#c9ceda" stroke-width="1"/>')
barchart = f'<svg viewBox="0 0 {BW} {BH-10}" xmlns="http://www.w3.org/2000/svg" font-family="Inter">{"".join(bars)}</svg>'
```

## Versione condivisibile

Le pagine sono `<div>` con classi: pagina 2 `p2`, pagina 3 `p3`, pagina 4 `p2 p4`.
Si oscurano gli importi in euro delle pagine 1, 2 e 4; la pagina 3 resta com'è.

```python
# --- versione condivisibile: ogni importo in euro diventa un rettangolo nero.
# Le cifre vere vengono sostituite (non solo coperte), cosi' non restano nel PDF.
import re
MONEY = re.compile(r'[+−-]?\s?\d{1,3}(?:\.\d{3})*(?:,\d{2})?(?:\u00a0|\s)?€')
def _box(m):
    width = len(m.group(0))
    return f'<span class="red">{"0" * max(width - 2, 3)}</span>'
head, body = html.split('<body>', 1)
priv, rest = body.split('<div class="p3">', 1)
pub, p4 = rest.split('<div class="p2 p4">', 1)
body_red = MONEY.sub(_box, priv) + '<div class="p3">' + pub + '<div class="p2 p4">' + MONEY.sub(_box, p4)
head = head.replace('</style>', '.red {{ display: inline-block !important; white-space: nowrap; background: #111; color: #111; border-radius: 2px; padding: 0 2px; letter-spacing: 0; text-transform: none; font-size: inherit; }}\n</style>'.replace('{{','{').replace('}}','}'))
html_red = head + '<body>' + body_red
HTML(string=html_red).write_pdf(f"Busta_paga_{MESE}_{ANNO}_condivisibile.pdf")


```
