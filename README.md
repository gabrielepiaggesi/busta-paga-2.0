# Busta paga 2.0

Un plugin per Claude che trasforma la tua busta paga italiana (PDF) in un PDF di 2 pagine, chiaro e di design:

1. **La busta paga riorganizzata**, che parte da quanto costi davvero all'azienda (costo aziendale) e arriva al netto, con contributi, tasse, TFR, ferie e permessi spiegati in modo semplice, più la tua posizione rispetto ai lavoratori dipendenti italiani.
2. **Una pagina per bambini**: un grafico a torta che spiega dove vanno i soldi dello stipendio.

## Come si usa

### Nell'app Claude (web o desktop)
1. Apri **Personalizza → Plugin** (in inglese: *Customize → Plugins*).
2. Clicca **Aggiungi → Aggiungi marketplace** (*Add → Add marketplace*) e incolla: `https://github.com/gabrielepiaggesi/busta-paga-2.0`
3. Installa **Busta paga 2.0**.
4. In una nuova chat allega la tua busta paga in PDF e scrivi: *"Rendimi leggibile questa busta paga"*.

### In Claude Code
```bash
claude plugin marketplace add gabrielepiaggesi/busta-paga-2.0
claude plugin install busta-paga-2-0@gabriele-marketplace
```

Claude ti chiederà **in quante mensilità** ricevi la RAL (12, 13 o 14) e **a quanto ammonta la tua RAL** (ed eventualmente regione e comune). Poi ti restituisce il PDF.

## Aggiornamenti

```bash
claude plugin update busta-paga-2-0@gabriele-marketplace
```

## Avvertenze

Il plugin chiede sempre mensilità e RAL (e regione e comune se non li trova in busta paga) e, se Claude ha un browser, prende da [stipendee.it](https://www.stipendee.it) solo il costo aziendale e la classifica dei redditi; tutto il resto viene dalla busta paga. Senza browser stima il costo aziendale al 40% in più del lordo. Costo aziendale e scatto netto restano comunque **stime**. Fa sempre fede la busta paga originale. Non è una consulenza fiscale o finanziaria.
