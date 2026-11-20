# Busta paga 2.0

Un plugin per Claude che trasforma la tua busta paga italiana (PDF) in un PDF di 4 pagine, chiaro e di design:

1. **La busta paga riorganizzata**, che parte da quanto costi davvero all'azienda (costo aziendale) e arriva al netto, con contributi, tasse, TFR, ferie e permessi spiegati in modo semplice, la tua posizione rispetto ai lavoratori dipendenti italiani e quanto dovresti ricevere in più per reggere l'inflazione.
2. **Una pagina per bambini**: un grafico a torta che spiega dove vanno i soldi dello stipendio.
3. **INPS e tasse dove vanno?**: la spesa dello Stato divisa per voci (pensioni, assistenza, sanità, istruzione, interessi, difesa…) e le pensioni INPS per fascia di importo.
4. **Consigli sul tuo netto**: come dividere lo stipendio con la regola 50/30/20 tra spese fisse, spese personali, fondo emergenze e PAC.

Ogni volta crea **due PDF**: uno completo e uno **condivisibile**, identico ma con tutti gli importi in euro coperti da rettangolini neri, da mostrare ad altri senza rivelare quanto guadagni.

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

Claude ti chiederà **in quante mensilità** ricevi la RAL (12, 13 o 14) e **a quanto ammonta la tua RAL** (ed eventualmente regione e comune). Poi ti restituisce i due PDF.

## Aggiornamenti

```bash
claude plugin update busta-paga-2-0@gabriele-marketplace
```

## Avvertenze

Il plugin chiede sempre mensilità e RAL (e regione e comune se non li trova in busta paga) e, se Claude ha un browser, prende da [stipendee.it](https://www.stipendee.it) solo il costo aziendale e la classifica dei redditi; tutto il resto viene dalla busta paga. I dati pubblici vengono da [rivaluta.it](https://www.rivaluta.it/inflazione/inflazione-ultimi-12-mesi.asp) (inflazione ISTAT) e dalle dashboard di Nazareno Lecis su [bilancio pubblico](https://www.nazarenolecis.com/dashboard/bilancio-pubblico/index.html) e [pensioni](https://www.nazarenolecis.com/dashboard/pensioni-italia/index.html). Senza browser stima il costo aziendale al 40% in più del lordo. Costo aziendale e scatto netto restano comunque **stime**. Fa sempre fede la busta paga originale. I consigli sul netto sono una regola generale, non una consulenza fiscale o finanziaria personalizzata.
