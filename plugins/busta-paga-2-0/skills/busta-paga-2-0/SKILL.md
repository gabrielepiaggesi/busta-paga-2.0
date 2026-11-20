---
name: busta-paga-2-0
description: Ricrea una busta paga italiana (PDF) in un PDF più leggibile e di design, partendo dal costo aziendale, con una pagina a torta spiegata per un bambino di 8 anni.
---

# Busta paga 2.0

Prendi una busta paga italiana (Zucchetti, TeamSystem, Inaz, ecc.) e produci un PDF A4 di **2 pagine**:
1. **Pagina 1** – la busta paga riorganizzata, leggibile, che parte dal costo aziendale.
2. **Pagina 2** – un grafico a torta spiegato in modo che lo capisca un bambino di 8 anni.

Rispondi sempre in italiano. Il file finale si chiama `Busta_paga_<Mese>_<Anno>_leggibile.pdf` e va consegnato all'utente.

## 0. Chiedi all'utente (prima di tutto, sempre)

Prima di generare qualsiasi cosa, chiedi all'utente in un unico messaggio (con lo strumento di domande a scelta multipla, se disponibile) tutto ciò che non ti ha già detto:
- **In quante mensilità** riceve la RAL: 12, 13 o 14?
- **Quanto è la sua RAL**, cioè la retribuzione annua lorda, in euro.
- **Regione e comune** di residenza, **solo se** non riesci a ricavarli con certezza dalla busta paga (di solito sono nelle voci delle addizionali regionale e comunale).

Non ricavare mensilità e RAL dalla busta paga al posto dell'utente: chiedile sempre, a meno che non le abbia già scritte lui. Aspetta la risposta prima di andare avanti. Se l'utente non può rispondere (sessione senza nessuno presente), usa la stima della sezione 3 e scrivilo chiaramente nella risposta.

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

Da stipendee.it leggi **solo due dati**: il **costo aziendale annuo** e la **percentuale rispetto ai lavoratori dipendenti italiani**. **Tutti gli altri dati vengono sempre dalla busta paga**, anche se il sito ne mostra altri (netto, IRPEF, INPS, TFR, ecc.): non usarli.

**Procedura nel browser** (Claude in Chrome, browser integrato o simili):
1. Apri `https://www.stipendee.it`. Se compare il banner dei cookie, scegli "Impostazioni" e poi "Rifiuta tutto".
2. Lascia selezionato l'anno corrente e la modalità **RAL**. Scrivi la RAL nel campo €.
3. Seleziona le **mensilità** indicate dall'utente.
4. Seleziona la **regione**, poi scrivi il **comune** nel campo di ricerca e scegli il comune giusto dai suggerimenti.
5. **Togli la spunta a tutte le caselle** (taglio del cuneo fiscale, bonus 100 €, azienda con più di 15 dipendenti, e ogni altra casella spuntata). Verifica che nessuna checkbox resti selezionata.
6. Clicca **Calcola**.
7. Dalla pagina dei risultati leggi con la lettura del testo della pagina (non da screenshot):
   - **Costo azienda** (valore annuo, nel riquadro "Money journey" o "Costi per l'azienda");
   - la prima frase **"Il tuo reddito complessivo è maggiore di quello del X% dei lavoratori dipendenti"**, quella riferita all'**Italia** (ignora quella regionale).

**Costo aziendale del mese:**
- **Costo aziendale del mese** = lordo del mese × (costo aziendale annuo / RAL).
- **Oneri a carico dell'azienda** = costo aziendale del mese − lordo. Scomponili usando i dati della busta paga:
  - **TFR maturato nel mese** = retribuzione utile TFR / 13,5;
  - **Assicurazione INAIL** = imponibile INAIL × tasso della voce tariffa (0,4% se è lavoro d'ufficio o non è indicato);
  - **Contributi pensioni INPS** (a carico azienda) = ciò che resta.
- Arrotonda al centesimo e verifica che le parti tornino al totale. Nella risposta all'utente riporta anche il costo aziendale annuo trovato su stipendee.it.

**Se non c'è un browser** o il sito non funziona: costo aziendale = lordo × **1,40** (circa il 40% in più della RAL), oppure il rapporto costo/RAL indicato dall'utente, e scomponi come sopra. In questo caso non mostrare la classifica dei redditi e dichiaralo nella risposta.

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
   - Subito sotto i rettangoli, una riga: "Il tuo reddito complessivo è maggiore di quello dell'**X%** dei lavoratori dipendenti italiani · fonte stipendee.it" (la percentuale in verde e in grassetto, la fonte in grigio piccolo; niente dato regionale). Ometti la riga se non hai i dati di stipendee.it.
4. **Paragrafi**, ben distanziati (~27px tra uno e l'altro). Ogni titolo ha l'importo allineato a destra:
   - **Costo aziendale** → totale costo; sotto le righe con il segno meno: "Contributi pensioni INPS" e "Assicurazione INAIL". Nessuna nota sotto.
   - **TFR maturato nel mese (accantonato)** → "− importo", come paragrafo a sé senza tabella.
   - **Cosa hai guadagnato** → lordo in verde; righe: voce | riferimento (gg/ore) in grigio | importo.
   - **Cosa viene trattenuto** → "− totale" in arancione; righe raggruppate (contributi, IRPEF, addizionali, altro), con il subtotale nella riga del gruppo su fondo grigio chiaro. Nel gruppo dei contributi, la voce IVS si chiama semplicemente "Contributo pensioni INPS" (senza codice né percentuale). Non scrivere su questa pagina che i contributi pagano le pensioni di oggi: quella spiegazione sta solo nella pagina 2. Sotto, una nota piccola con il calcolo dell'IRPEF (lorda − detrazioni = ritenuta) e l'imponibile IRPEF.
5. **Due riquadri affiancati**, con un po' più di spazio sopra (~36px):
   - **Ferie e permessi**: tabella con colonne Residuo | Maturato | Goduto | Saldo (saldo in grassetto).
   - **Da inizio anno & TFR**: imponibili INPS/INAIL e IRPEF, IRPEF pagata, TFR al 31/12, quota TFR dell'anno · rivalutazione.
6. **Piè di pagina** piccolo: "Versione riformattata per leggibilità della busta paga del <data> (n. …). Fa fede il documento originale." più le posizioni INPS/INAIL.

Formatta gli importi all'italiana (`1.234,56 €`), con spazio non separabile prima di €. Usa cifre tabulari, allineate a destra.

## 6. Pagina 2 – per un bambino di 8 anni

Centrata, tutta su una pagina:
- Titolo grande: "Dove va lo stipendio di <Nome>?". Usa il nome proprio e non "papà": chiedi all'utente se preferisce un'altra parola.
- Introduzione, una sola frase: "A <mese>, per avere <Nome> al lavoro, la sua azienda ha speso **<costo>**." (niente frase sulla "grande torta").
- **Torta SVG** (circa 85–90 mm), disegnata con path calcolati in Python e bordi bianchi spessi. Mostra le percentuali in bianco dentro le fette; per le fette sotto l'8% usa un carattere più piccolo e sposta l'etichetta verso l'esterno; sotto il 4% non scrivere l'etichetta nella torta (la percentuale resta nella scheda). Le fette, che devono sommare al costo aziendale:
  - 🟢 **In tasca a <Nome>** = netto (#2BB673) – "Arrivano sul conto: per casa, spesa, vacanze e giochi!"
  - 🔵 **Ai pensionati di oggi** = contributi INPS del dipendente (IVS + CIGS) + INPS azienda (#5B8DEF) – "Pagano la pensione ai nonni di oggi." Sotto, in piccolo e in grigio: "L'INPS li segna a tuo nome: serviranno a calcolare la tua pensione futura."
  - Non presentare mai i contributi come un salvadanaio o come soldi messi da parte per la pensione del dipendente: il sistema INPS è a ripartizione.
  - 🟠 **Tasse allo Stato** = IRPEF + addizionali (#FF8A3D) – "Per stipendi statali, scuole, ospedali, strade e tanto altro."
  - 🟡 **Assicurazione** = assicurazione INAIL + quota del dipendente al fondo sanitario (es. EST) (#E5B800) – "Protegge <Nome> se si fa male al lavoro o se deve curarsi."
  - 🟣 **TFR** = TFR maturato nel mese (#A66BE8) – "Soldi messi da parte per <Nome>: li riceve quando lascia il lavoro." Sotto, in piccolo e in grigio: "Trattamento di fine rapporto · consigliabile metterlo in un fondo pensione."
- Una scheda per ogni fetta, nell'ordine sopra, con bordo colorato: titolo con la percentuale (se è sotto lo 0,5% scrivi "meno dell'1%"), importo e la frase semplice.
- La pagina **finisce con la scheda del TFR**: niente monete, niente frasi o note dopo.

## 7. Controllo finale

- `pdfinfo` → deve risultare esattamente 2 pagine. Se sono di più, riduci gli spazi o il carattere (testo non sotto ~8,5pt) o la torta. Non togliere dati.
- Renderizza le pagine con `pdftoppm -r 70 -png` e guardale. Controlla che nulla vada a capo dove non deve (rettangoli, riga informazioni), che nulla esca dai margini e che le etichette della torta si leggano.
- Invia il PDF all'utente con una breve spiegazione: il netto rispetto al costo aziendale e come hai stimato oneri e scatto. Proponi di sostituire le stime con i dati reali dell'ufficio del personale.

## Note

- I dati personali (codice fiscale, IBAN) non servono nel PDF riformattato: lasciali fuori, a meno che l'utente non li chieda.
- Se la busta paga è una scansione, usa l'OCR (pytesseract) e ricontrolla tutte le cifre con i totali.
- Se ci sono più mesi, genera un PDF per ogni mese, oppure chiedi all'utente se vuole un unico file.
