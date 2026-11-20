# gabriele-marketplace

Marketplace personale di plugin per Claude.

## Plugin disponibili

| Plugin | Cosa fa |
|---|---|
| `busta-paga-leggibile` | Ricrea una busta paga italiana (PDF) in un PDF leggibile e di design, partendo dal costo aziendale, con una pagina a torta spiegata per un bambino di 8 anni. |

## Installazione (Claude Code)

```bash
claude plugin marketplace add gabrielepiaggesi/busta-paga-leggibile-claude
claude plugin install busta-paga-leggibile@gabriele-marketplace
```

Oppure, dentro una sessione:

```
/plugin marketplace add gabrielepiaggesi/busta-paga-leggibile-claude
/plugin install busta-paga-leggibile@gabriele-marketplace
```

Poi basta allegare la busta paga in PDF e chiedere: "rendimi leggibile questa busta paga".

## Aggiornamenti

```bash
claude plugin update busta-paga-leggibile@gabriele-marketplace
```

## Avvertenze

Il costo aziendale (di default circa il 40% in più del lordo) e lo scatto netto sono **stime**. Fa sempre fede la busta paga originale. Se hai le cifre reali dall'ufficio del personale, indicale e il plugin userà quelle.
