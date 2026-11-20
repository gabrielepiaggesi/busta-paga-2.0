---
name: busta-paga-leggibile
description: Ricrea una busta paga italiana (PDF) in un PDF più leggibile e di design, partendo dal costo aziendale, con una pagina a torta spiegata per un bambino di 8 anni.
---

# Busta paga leggibile

Prendi una busta paga italiana (Zucchetti, TeamSystem, Inaz, ecc.) e produci un PDF A4 di **2 pagine**:
1. **Pagina 1** – la busta paga riorganizzata, leggibile, che parte dal costo aziendale.
2. **Pagina 2** – un grafico a torta spiegato in modo che lo capisca un bambino di 8 anni.

Rispondi sempre in italiano. Il file finale si chiama `Busta_paga_<Mese>_<Anno>_leggibile.pdf` e va consegnato all'utente.

## 0. Chiedi all'utente (prima di tutto)

Fai queste domande in un unico messaggio (con lo strumento di domande a scelta multipla, se disponibile):
- **Quante mensilità** ha il contratto: 12, 13 o 14? (Se la busta paga mostra ratei di 13ª e 14ª, proponi 14 come risposta predefinita.)
- **RAL**, cioè la retribuzione annua lorda, in euro.
- Conferma **regione e comune** di residenza: ricavali dalle voci delle addizionali regionale e comunale in busta paga, ma chiedi conferma.

Se l'utente non risponde, usa: mensilità ricavate dalla busta paga, RAL = (retribuzione mensile senza ratei) × mensilità, regione e comune dalle addizionali. Dichiara questa scelta nella risposta finale.

## 1. Estrai i dati

- Copia il PDF nella cartella di lavoro ed estrai il testo con `pdftotext -layout`.
- Raccogli:
  - **Azienda**: ragione sociale, indirizzo, codice fiscale/P.IVA.
  - **Dipendente**: nome, ruolo, livello, data di assunzione, giorni e ore lavorate, data del prossimo scatto, importo dello scatto attuale (voce SCATTI).
  - **Competenze** (voci a credito): retribuzione, festività, ratei 13ª/14ª, straordinari, ecc. Ognuna con il suo riferimento (giorni/ore).
  - **Trattenute del dipendente**, raggruppate in:
    - Contributi previdenziali (IVS/INPS, CIGS, fondi sanitari tipo EST)
    - IRPEF (ordinaria + tassazione separata)
    - Addizionali regionali e comunali (anche gli acconti)
    - Altro (arrotondamenti del mese precedente)
  - **Totali**: totale competenze (lordo), totale trattenute, arrotondamento del mese, netto.
  - **Retribuzione utile TFR**, imponibile IRPEF, IRPEF lorda e detrazioni.
  - **Ferie e permessi**: residuo anno precedente, maturato, goduto, saldo.
  - **Progressivi**: imponibile INPS/INAIL e IRPEF, IRPEF pagata, TFR al 31/12, quota TFR dell'anno, rivalutazione.

## 2. Verifica i conti (obbligatorio)

Con `assert` in Python, prima di generare il PDF:
- la somma delle competenze è uguale al totale competenze;
- la somma delle trattenute (arrotondamento del mese precedente incluso) è uguale al totale trattenute;
- lordo − trattenute ± arrotondamento del mese = netto.

Se un controllo non torna, rileggi il PDF. Non forzare i numeri.

## 3. Costo aziendale e classifica dei redditi (da stipendee.it)

La busta paga **non** riporta i costi a carico dell'azienda. Ricavali dal calcolatore di **stipendee.it**, se hai a disposizione un browser (Claude in Chrome, browser integrato o simili).

**Procedura nel browser:**
1. Apri `https://www.stipendee.it`. Se compare il banner dei cookie, scegli "Impostazioni" e poi "Rifiuta tutto".
2. Lascia selezionato l'anno corrente e la modalità **RAL**. Scrivi la RAL nel campo €.
3. Seleziona le **mensilità** indicate dall'utente.
4. Seleziona la **regione**, poi scrivi il **comune** nel campo di ricerca e scegli il comune giusto dai suggerimenti.
5. **Togli la spunta a tutte le caselle** (taglio del cuneo fiscale, bonus 100 €, azienda con più di 15 dipendenti, e ogni altra casella spuntata). Verifica che nessuna checkbox resti selezionata.
6. Clicca **Calcola**.
7. Dalla pagina dei risultati leggi (con la lettura del testo della pagina, non da screenshot):
   - nel riquadro "Money journey": **Costo azienda**, **RAL**, **INPS azienda**, **TFR**, **Oneri assicurativi** (valori annui);
   - l'aliquota INPS a carico del datore usata dal calcolo (campo "alinpsDat", di solito 23,81% ma può cambiare);
   - nella sezione "Distribuzione dei redditi" le due frasi **"Il tuo reddito complessivo è maggiore di quello del X% dei lavoratori dipendenti"**: la prima è per l'Italia, la seconda per la regione.

**Riporta i valori annui al mese della busta paga:**
- **INPS azienda del mese** = imponibile INPS del mese × aliquota INPS datore.
- **TFR del mese** = retribuzione utile TFR / 13,5.
- **Assicurazioni INAIL e altri oneri del mese** = lordo del mese × (oneri assicurativi annui / RAL).
- **Costo aziendale del mese** = lordo + INPS azienda + TFR + assicurazioni. Arrotonda al centesimo e verifica che le parti tornino al totale.
- Nella risposta all'utente riporta anche il costo aziendale annuo trovato su stipendee.it.

**Se non c'è un browser** o il sito non funziona: costo aziendale = lordo × **1,40** (circa il 40% in più della RAL), oppure il rapporto costo/RAL indicato dall'utente. Scomponi in TFR (retribuzione utile / 13,5), assicurazioni (≈ 3% del lordo) e INPS azienda (il resto). In questo caso non mostrare la classifica dei redditi e dichiaralo nella risposta.

## 4. Stima del prossimo scatto netto

- Lordo mensile dello scatto = importo di uno scatto × mensilità / 12 (14 mensilità per il commercio, 13 per altri contratti collettivi).
- Netto ≈ lordo × (1 − contributi del dipendente ~9,49%) × (1 − aliquota IRPEF marginale − addizionale regionale − addizionale comunale).
  - Aliquote IRPEF: 23% fino a 28.000 €, 33% da 28.000 a 50.000 €, 43% oltre. Ricava lo scaglione dall'imponibile annuo.
- Mostralo arrotondato all'euro, accanto alla data: `09/2028 ≈ +15 € netti/mese`.
- Nella risposta all'utente spiega il calcolo in una riga.

## 5. Pagina 1 – layout

Genera in HTML e converti con **WeasyPrint** (`pip install --break-system-packages weasyprint`). Font: Poppins per i titoli, Inter per il testo. Usa A4 con margini di circa 11–13 mm. Tutto deve stare su **una sola pagina**.

Dall'alto in basso:

1. **Intestazione**
   - A sinistra: il mese/anno in verde (#2BB673), sotto "Busta paga <Nome Cognome>" (~15pt).
   - A destra: ragione sociale, indirizzo e P.IVA.
   - Sotto, una linea scura.
2. **Riga informazioni in 4 colonne**, senza a capo: Ruolo · Livello | Assunto il | Giorni / ore lavorate | Prossimo scatto + "≈ +X € netti/mese" in verde, sulla stessa riga.
3. **5 rettangoli** su una sola riga, tutti bianchi e uguali (bordo grigio chiaro, angoli arrotondati, nessun grafico dentro). Per ciascuno: etichetta in maiuscoletto, cifra e sottotitolo breve, che non deve andare a capo.
   1. Costo azienda (cifra scura) – "la spesa totale per te"
   2. Trattenute aziendali (cifra arancione #e06d1f) – "contributi e TFR"
   3. Lordo del mese (cifra scura) – "lo stipendio \"ufficiale\""
   4. Trattenute (cifra arancione) – "tasse e contributi"
   5. Netto in busta (cifra verde) – "XX% del costo azienda"
   - Niente legende o quadratini colorati sotto i rettangoli.
   - Subito sotto i rettangoli, una riga: "Il tuo reddito complessivo è maggiore di quello dell'**X%** dei lavoratori dipendenti italiani · Y% nel <Regione> · fonte stipendee.it" (la percentuale italiana in verde e in grassetto, il resto in grigio piccolo). Ometti la riga se non hai i dati di stipendee.it.
4. **Paragrafi**, ben distanziati (~27px tra uno e l'altro). Ogni titolo ha l'importo allineato a destra:
   - **Costo aziendale** → totale costo; sotto le righe con il segno meno: "Contributi INPS a carico azienda (XX,XX%)" e "Assicurazioni INAIL e altri oneri (~X%)". Nessuna nota sulla stima, ma sotto le righe aggiungi in piccolo e in grigio: "I contributi INPS (tuoi e dell'azienda) non vengono messi da parte per te: pagano le pensioni di chi è in pensione oggi. Vengono però registrati a tuo nome e servono a calcolare la tua pensione futura."
   - **TFR maturato nel mese (accantonato)** → "− importo", come paragrafo a sé senza tabella.
   - **Cosa hai guadagnato** → lordo in verde; righe: voce | riferimento (gg/ore) in grigio | importo.
   - **Cosa viene trattenuto** → "− totale" in arancione; righe raggruppate (contributi, IRPEF, addizionali, altro), con il subtotale nella riga del gruppo su fondo grigio chiaro. Nella riga del gruppo dei contributi aggiungi, in grigio e non in grassetto: "· pagano le pensioni di chi è in pensione oggi". Sotto, una nota piccola con il calcolo dell'IRPEF (lorda − detrazioni = ritenuta) e l'imponibile IRPEF.
5. **Due riquadri affiancati**, con un po' più di spazio sopra (~36px):
   - **Ferie e permessi**: tabella con colonne Residuo | Maturato | Goduto | Saldo (saldo in grassetto).
   - **Da inizio anno & TFR**: imponibili INPS/INAIL e IRPEF, IRPEF pagata, TFR al 31/12, quota TFR dell'anno · rivalutazione.
6. **Piè di pagina** piccolo: "Versione riformattata per leggibilità della busta paga del <data> (n. …). Fa fede il documento originale." più le posizioni INPS/INAIL.

Formatta gli importi all'italiana (`1.234,56 €`), con spazio non separabile prima di €. Usa cifre tabulari, allineate a destra.

## 6. Pagina 2 – per un bambino di 8 anni

Centrata, tutta su una pagina:
- Titolo grande: "Dove va lo stipendio di <Nome>?". Usa il nome proprio e non "papà": chiedi all'utente se preferisce un'altra parola.
- Introduzione, una sola frase: "A <mese>, per avere <Nome> al lavoro, la sua azienda ha speso **<costo>**." (niente frase sulla "grande torta").
- **Torta SVG** (circa 80 mm), disegnata con path calcolati in Python e bordi bianchi spessi. Mostra le percentuali in bianco dentro le fette; per le fette sotto l'8% usa un carattere più piccolo e sposta l'etichetta verso l'esterno; sotto il 4% non scrivere l'etichetta nella torta (la percentuale resta nella scheda). Le fette, che devono sommare al costo aziendale:
  - 🟢 **In tasca a <Nome>** = netto (#2BB673) – "Arrivano sul conto: per casa, spesa, vacanze e giochi!"
  - 🔵 **Ai pensionati di oggi** = contributi INPS del dipendente (IVS + CIGS) + INPS azienda (#5B8DEF) – "Pagano la pensione ai nonni di oggi." Sotto, in piccolo e in grigio: "L'INPS li segna a nome di <Nome>: serviranno a calcolare la sua pensione futura."
  - Non presentare mai i contributi come un salvadanaio o come soldi messi da parte per la pensione del dipendente: il sistema INPS è a ripartizione.
  - 🟠 **Tasse allo Stato** = IRPEF + addizionali (#FF8A3D) – "Pagano scuole, ospedali, strade e parchi per tutti."
  - 🟡 **Assicurazione** = assicurazioni INAIL e altri oneri + quota del dipendente al fondo sanitario (es. EST) (#E5B800) – "Protegge <Nome> se si fa male al lavoro o se deve curarsi."
  - 🟣 **Premio di fine lavoro** = TFR (#A66BE8) – "Il TFR: soldi che <Nome> riceverà tutti insieme quando cambierà lavoro."
- Una scheda per ogni fetta, con bordo colorato: titolo con la percentuale, importo e la frase semplice.
- **"Se l'azienda spende 10 monete per <Nome>…"**: 10 cerchi colorati con il simbolo €, divisi secondo le percentuali arrotondate (la somma deve fare 10), più una frase che le spiega (per la fetta blu: "pagano le pensioni dei nonni di oggi"). Usa il metodo dei resti maggiori per arrivare esattamente a 10. Se una fetta prende 0 monete (di solito l'assicurazione), aggiungi sotto in piccolo: "L'assicurazione è così piccola che non arriva neanche a una moneta!"

## 7. Controllo finale

- `pdfinfo` → deve risultare esattamente 2 pagine. Se sono di più, riduci gli spazi o il carattere (testo non sotto ~8,5pt) o la torta. Non togliere dati.
- Renderizza le pagine con `pdftoppm -r 70 -png` e guardale. Controlla che nulla vada a capo dove non deve (rettangoli, riga informazioni), che nulla esca dai margini e che le etichette della torta si leggano.
- Invia il PDF all'utente con una breve spiegazione: il netto rispetto al costo aziendale e come hai stimato oneri e scatto. Proponi di sostituire le stime con i dati reali dell'ufficio del personale.

## Note

- I dati personali (codice fiscale, IBAN) non servono nel PDF riformattato: lasciali fuori, a meno che l'utente non li chieda.
- Se la busta paga è una scansione, usa l'OCR (pytesseract) e ricontrolla tutte le cifre con i totali.
- Se ci sono più mesi, genera un PDF per ogni mese, oppure chiedi all'utente se vuole un unico file.
