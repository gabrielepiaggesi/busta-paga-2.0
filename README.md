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

Il plugin chiede sempre mensilità e RAL (e regione e comune se non li trova in busta paga) e, se Claude ha un browser, prende da [stipendee.it](https://www.stipendee.it) solo il costo aziendale e la classifica dei redditi; tutto il resto viene dalla busta paga. Senza browser stima il costo aziendale al 40% in più del lordo. Costo aziendale e scatto netto restano comunque **stime**. Fa sempre fede la busta paga originale. Se hai le cifre reali dall'ufficio del personale, indicale e il plugin userà quelle.
