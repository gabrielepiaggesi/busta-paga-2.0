---
name: busta-paga-2-0
description: Ricrea una busta paga italiana (PDF) in un PDF di 4 pagine leggibile e di design - busta paga dal costo aziendale, torta per un bambino di 8 anni, dove vanno tasse e INPS, consigli sul netto - più una copia condivisibile con gli importi oscurati.
---

# Busta paga 2.0

L'utente carica una busta paga italiana (Zucchetti, TeamSystem, Inaz, ecc.). Tu produci **due PDF A4 di esattamente 4 pagine**:

- `Busta_paga_<Mese>_<Anno>_leggibile.pdf`: la versione completa.
- `Busta_paga_<Mese>_<Anno>_condivisibile.pdf`: identica, ma con ogni importo personale in euro sostituito da un rettangolino nero (sezione 11).

Le quattro pagine sono:

1. **La busta paga riorganizzata**, che parte dal costo aziendale e arriva al netto, con la riga sull'inflazione.
2. **"Dove va lo stipendio di <Nome>?"**: una torta spiegata a un bambino di 8 anni.
3. **"INPS e tasse dove vanno?"**: la spesa dello Stato a quadrati e le pensioni INPS per fascia.
4. **"Consigli sul tuo netto"**: la regola 50/30/20 con torta e schede.

Rispondi sempre in italiano. Tutto deve poter essere rifatto solo caricando la busta paga e rispondendo alle domande della sezione 0. Nella cartella di questa skill, `codice.md` contiene le funzioni già pronte (torta, quadrati, barre, oscuramento): usale.

## 0. Chiedi all'utente (prima di tutto, sempre)

Chiedi in un unico messaggio, con lo strumento di domande a scelta multipla se c'è, solo ciò che non ti ha già detto:

- **In quante mensilità** riceve la RAL: 12, 13 o 14?
- **A quanto ammonta la RAL**, in euro lordi annui.
- **Regione e comune** di residenza, **solo se** non si capiscono con certezza dalla busta paga. Di solito compaiono nelle voci delle addizionali regionale e comunale.

Non ricavare mensilità e RAL dalla busta paga al posto dell'utente. Aspetta la risposta. Se nessuno può rispondere (sessione non presidiata), usa le stime della sezione 3 e dichiaralo nella risposta.

## 1. Estrai i dati dalla busta paga

Copia il PDF nella cartella di lavoro ed estrai il testo con `pdftotext -layout`. Raccogli:

- **Azienda**: ragione sociale, indirizzo, codice fiscale o P.IVA.
- **Dipendente**: nome, ruolo e livello, data di assunzione, giorni e ore lavorate, data del prossimo scatto, importo dello scatto attuale (voce SCATTI).
- **Competenze**, cioè le voci a credito, ognuna con giorni o ore di riferimento.
- **Trattenute del dipendente**, divise in:
  - contributi (IVS/INPS, CIGS, fondi sanitari come EST)
  - IRPEF (ordinaria e tassazione separata)
  - addizionali regionali e comunali, compresi gli acconti
  - altro (arrotondamenti del mese precedente)
- **Totali**: lordo, trattenute, arrotondamento del mese, netto.
- **Retribuzione utile TFR**, imponibile IRPEF, IRPEF lorda e detrazioni.
- **Ferie e permessi**: residuo, maturato, goduto, saldo.
- **Progressivi**: imponibile INPS/INAIL e IRPEF, IRPEF pagata, TFR al 31/12, quota TFR dell'anno, rivalutazione.

## 2. Verifica i conti (obbligatorio)

Prima di generare il PDF, con `assert` in Python:

- la somma delle competenze è uguale al totale competenze;
- la somma delle trattenute, compreso l'arrotondamento del mese precedente, è uguale al totale trattenute;
- lordo − trattenute ± arrotondamento è uguale al netto.

Se un controllo non torna, rileggi il PDF. Non forzare i numeri.

## 3. Costo aziendale e classifica dei redditi (stipendee.it)

Da stipendee.it leggi **solo due dati**: il **costo aziendale annuo** e la **percentuale dei lavoratori dipendenti italiani** con reddito più basso. Tutti gli altri dati vengono sempre dalla busta paga.

Procedura nel browser:

1. Apri `https://www.stipendee.it`. Se compare il banner dei cookie, scegli "Impostazioni" e poi "Rifiuta tutto".
2. Lascia selezionati l'anno corrente e la modalità **RAL**, poi scrivi la RAL.
3. Seleziona le mensilità.
4. Seleziona la regione, scrivi il comune nel campo di ricerca e scegli quello giusto dai suggerimenti.
5. **Togli la spunta da tutte le caselle**: taglio del cuneo fiscale, bonus 100 €, azienda con più di 15 dipendenti e ogni altra.
6. Clicca **Calcola**.
7. Leggi il testo della pagina, non uno screenshot:
   - "Costo azienda" (valore annuo);
   - la frase "Il tuo reddito complessivo è maggiore di quello del X% dei lavoratori dipendenti", nella versione per l'**Italia** (non quella regionale).

Calcoli del mese:

- **Costo aziendale del mese** = lordo del mese × (costo aziendale annuo / RAL).
- **Oneri a carico dell'azienda** = costo aziendale del mese − lordo, divisi così:
  - TFR maturato nel mese = retribuzione utile TFR / 13,5;
  - assicurazione INAIL = imponibile INAIL × 0,4% (lavoro d'ufficio, se la busta non indica altro);
  - contributi pensioni INPS a carico dell'azienda = ciò che resta.

**Senza browser**, o se il sito non funziona: costo aziendale = lordo × 1,40. In quel caso ometti la riga della classifica e dichiaralo nella risposta.

## 4. Prossimo scatto e inflazione

**Scatto netto**:

- lordo mensile = importo di uno scatto × mensilità / 12;
- netto ≈ lordo × (1 − 9,49%) × (1 − aliquota IRPEF marginale − addizionali). Le aliquote IRPEF sono 23% fino a 28.000 €, 33% fino a 50.000 €, 43% oltre.

Scrivilo arrotondato all'euro: `09/2028 ≈ +15 € netti/mese`.

**Inflazione**:

- Fonte: `https://www.rivaluta.it/inflazione/inflazione-ultimi-12-mesi.asp`, scheda **"Italia NIC"** (ISTAT, variazione annua %). Funziona con il semplice recupero della pagina (WebFetch): non serve il browser.
- Salva nel codice tutta la serie degli ultimi 12–13 mesi, in un dizionario `{"AAAA-MM": valore}`.
- Usa il valore del **mese della busta paga**. Se non è ancora uscito, usa l'ultimo mese disponibile e scrivi quale.
- Extra necessario = netto × inflazione / 100.

## 5. Dati pubblici per la pagina 3 (nazarenolecis.com)

Le due dashboard sono pagine dinamiche: usa il **browser** (Claude in Chrome o il browser integrato). Leggi tabelle e testo con la lettura del testo della pagina, e i grafici Plotly con JavaScript, come indicato sotto. Prendi sempre l'**ultimo anno disponibile** e annota l'anno di ogni dato.

### 5a. Bilancio pubblico

Apri `https://www.nazarenolecis.com/dashboard/bilancio-pubblico/index.html#uscite`.

- **Entrate totali** (`ENTRATE_TOT`): riga "Totale" della tabella "Composizione entrate", ultimo anno (per il 2024: 820,8 mld). Sono tasse più contributi sociali.
- **Uscite totali** (`USCITE_TOT`): riga "Totale" della tabella "Composizione spese" (per il 2024: 1.109,2 mld).
- **Macro voci** dalla stessa tabella: Protezione sociale, Sanità, Istruzione, Difesa, oltre a Servizi generali, Affari economici, Ordine pubblico, Ambiente, Cultura e religione, Casa e territorio.
- **Scomposizione di secondo livello**: nella sezione "Scomposizione delle macro voci di spesa", cambia la select "MACRO VOCE" e leggi la tabella sotto. Puoi farlo da JavaScript:

  ```js
  const s=[...document.querySelectorAll('select')].find(x=>[...x.options].some(o=>o.text==='Protezione sociale')&&[...x.options].some(o=>o.text==='Casa e territorio')&&x.options.length<15);
  let c=s; for(let i=0;i<8&&c&&!c.querySelector('table');i++) c=c.parentElement;
  const out={};
  for (const name of ['Protezione sociale','Servizi generali','Affari economici','Ordine pubblico','Ambiente','Cultura e religione','Casa e territorio']) {
    const o=[...s.options].find(o=>o.text===name); s.value=o.value; s.dispatchEvent(new Event('change',{bubbles:true}));
    await new Promise(r=>setTimeout(r,1200)); out[name]=c.querySelector('table').innerText;
  }
  out
  ```

Da qui ricavi:

- **Pensioni** = Vecchiaia + Superstiti (dentro Protezione sociale).
- **Assistenza** = Protezione sociale − Pensioni, con il dettaglio:
  - Malattia e disabilità
  - Figli: Famiglia e figli (assegno unico)
  - NASpI e cassa integrazione: Disoccupazione
  - Assegno di inclusione: Esclusione sociale n.c.a.
  - Altro: Protezione sociale n.c.a. + Abitazione + Ricerca e sviluppo
- **Interessi sul debito** = "Operazioni sul debito pubblico" (dentro Servizi generali).
- **Altro** = Uscite totali − (Pensioni + Assistenza + Sanità + Istruzione + Difesa + Interessi). Per il dettaglio elenca le sottovoci di Servizi generali (senza debito), Affari economici, Ordine pubblico, Ambiente, Cultura e Casa, con nomi brevi e comprensibili:

  | Sottovoce sul sito | Nome da usare |
  |---|---|
  | Trasporti | Trasporti |
  | Organi esecutivi e legislativi… | Governo, fisco, esteri |
  | Servizi di polizia | Polizia |
  | Servizi generali | Uffici e personale PA |
  | Ricerca di base + R&S per affari economici | Ricerca scientifica |
  | Affari economici, commerciali e del lavoro | Lavoro e imprese |
  | Combustibili ed energia | Energia |
  | Attività estrattive… | Industria e costruzioni |
  | Gestione dei rifiuti | Rifiuti |
  | Sviluppo abitativo | Case popolari |
  | Tribunali | Tribunali |
  | Servizi culturali | Cultura |
  | Sviluppo del territorio + Illuminazione stradale | Territorio e strade |
  | Servizi ricreativi e sportivi | Sport e tempo libero |
  | Agricoltura… | Agricoltura e pesca |
  | Servizi antincendio | Vigili del fuoco |
  | Istituti penitenziari | Carceri |

  L'ultima riga è "Altre voci minori" = Altro − la somma delle voci elencate. Controlla che il totale torni.

### 5b. Pensioni italiane

Apri `https://www.nazarenolecis.com/dashboard/pensioni-italia/index.html`.

- **Riquadri in alto** (testo della pagina):
  - "CONTRIBUTI INPS", cioè le entrate contributive complessive (`CONTRIB_INPS`, 2025: 294,2 mld);
  - "REDDITO PENSIONISTICO LORDO … di cui pensionati INPS" (`SPESA_INPS`, 2025: 361,3 mld).
- **Trasferimenti dello Stato all'INPS** (`TRASF_INPS`, 2025: 165,4 mld): serie "Trasferimenti Stato all'INPS" del grafico "Reddito pensionistico, contributi e trasferimenti complessivi all'INPS". Usa **sempre questo dato** come "buco di bilancio" coperto dallo Stato, **non** la differenza tra altri numeri, che spesso sono di anni diversi.
- **IRPEF totale dello stesso anno** (`IRPEF_TOT`, 2025: 231,3 mld): dalla tabella "Tutte le entrate disponibili" della dashboard Bilancio pubblico, riga IRPEF (MEF erariali).
- **Grafico a fasce** "Valore delle pensioni e reddito dei pensionati":
  1. prima select (PERIMETRO) = **Reddito pensionistico: persone**;
  2. seconda select (MISURA) = **Ammontare lordo complessivo per fascia**;
  3. terza select (ANNO) = l'anno **più vicino a quello corrente**. Annota i miliardi lordi di ogni fascia;
  4. seconda select = **Numero di pensionati per fascia di reddito**. Annota i milioni di pensionati per fascia;
  5. annota l'etichetta dell'asse X di ogni fascia (range mensile lordo).

  Gli ID delle select sono `piDistributionScope`, `piDistributionMetric` e `piDistributionYear`. Dopo ogni cambio leggi i dati Plotly:

  ```js
  let c=document.getElementById('piDistributionScope'); for(let i=0;i<8&&c&&!c.querySelector('.js-plotly-plot');i++) c=c.parentElement;
  c.querySelector('.js-plotly-plot').data.map(t=>({x:t.x,y:t.y}))
  ```

  La somma dei miliardi delle fasce deve essere uguale a `SPESA_INPS`. L'ultima fascia (es. "3000,00-3499,99") nel sito comprende **tutti da 3000 € in su**.

**Senza browser**: usa i valori di esempio indicati sopra (aggiornati a novembre 2026), dichiaralo nella risposta e scrivi nel PDF l'anno dei dati.

## 6. Pagina 1: layout

Genera in HTML e converti con **WeasyPrint** (`pip install --break-system-packages weasyprint`). Font: Poppins per i titoli, Inter per il testo. A4 con margini di circa 11–13 mm. **Ogni pagina è una sola pagina**: usa `page-break-before: always` sulle pagine 2, 3 e 4.

Dall'alto in basso:

1. **Intestazione**:
   - a sinistra, il mese e l'anno in verde (#2BB673), con sotto "Busta paga <Nome Cognome>" (~15pt);
   - a destra, ragione sociale, indirizzo e P.IVA;
   - sotto, una linea scura.
2. **Riga informazioni in 4 colonne**, senza andare a capo: Ruolo · Livello | Assunto il | Giorni / ore lavorate | Prossimo scatto + "≈ +X € netti/mese" in verde.
3. **5 riquadri bianchi uguali su una riga**: bordo grigio chiaro, angoli arrotondati, etichetta in maiuscoletto, cifra, sottotitolo breve:
   - Costo azienda: "la spesa totale per te";
   - Trattenute aziendali, in arancione (#e06d1f): "contributi e TFR";
   - Lordo del mese: "lo stipendio \"ufficiale\"";
   - Trattenute, in arancione: "tasse e contributi";
   - Netto in busta, in verde: "XX% del costo azienda".

   Subito sotto: "Il tuo reddito complessivo è maggiore di quello dell'**X%** dei lavoratori dipendenti italiani · fonte stipendee.it". Omettila senza dati di stipendee.it.
4. **Paragrafi** (~14–19px tra uno e l'altro), ognuno con l'importo allineato a destra nel titolo:
   - **Lordo aziendale**: solo il titolo con l'importo.
   - **Trattenute aziendali**: righe con il segno meno: Contributi pensioni INPS, Assicurazione INAIL, TFR maturato nel mese (accantonato).
   - **Lordo personale**: voce | riferimento in grigio | importo.
   - **Trattenute personali**: righe raggruppate con il subtotale su fondo grigio chiaro: contributi previdenziali, IRPEF, addizionali, altro. La voce IVS si chiama "Contributo pensioni INPS". Sotto, una nota piccola con il calcolo dell'IRPEF. **Non** dire su questa pagina che i contributi pagano le pensioni di oggi.
   - **Netto personale**: il titolo con l'importo in verde. Sotto, la **riga dell'inflazione**: "Per reggere l'inflazione di <mese anno> (**+X%** in un anno) dovresti ricevere circa **Y € netti in più** al mese, cioè Z €. · Inflazione ISTAT NIC · fonte rivaluta.it".
5. **Due riquadri affiancati**: Ferie e permessi (Residuo | Maturato | Goduto | Saldo) e Da inizio anno & TFR.
6. **Piè di pagina** piccolo: "Versione riformattata per leggibilità … Fa fede il documento originale." più le posizioni INPS e INAIL.

Formatta gli importi all'italiana (`1.234,56 €`), con uno spazio non separabile prima di €.

## 7. Pagina 2: per un bambino di 8 anni

- **Titolo**: "Dove va lo stipendio di <Nome>?"
- **Introduzione** di una sola frase: "A <mese>, per avere <Nome> al lavoro, la sua azienda ha speso **<costo>**."
- **Torta SVG** di circa 88 mm. Le fette devono sommare al costo aziendale:
  - 🟢 **In tasca a <Nome>** = netto (#2BB673). Testo: "Arrivano sul conto: per casa, spesa, vacanze e giochi!"
  - 🔵 **Ai pensionati di oggi** = contributi INPS del dipendente + INPS a carico dell'azienda (#5B8DEF). Testo: "Pagano la pensione ai nonni di oggi." In piccolo: "L'INPS li segna a tuo nome: serviranno a calcolare la tua pensione futura."
  - 🟠 **Tasse allo Stato** = IRPEF + addizionali (#FF8A3D). Accanto all'importo, in piccolo: "(circa **Q** € per aiutare l'INPS)", dove Q = IRPEF del mese × `TRASF_INPS / IRPEF_TOT`. Le addizionali sono escluse perché vanno a Regione e Comune. Testo: "Per stipendi statali, scuole, ospedali, strade e tanto altro."
  - 🟡 **Assicurazione** = INAIL + fondo sanitario (#E5B800). Testo: "Protegge <Nome> se si fa male al lavoro o se deve curarsi."
  - 🟣 **TFR** (#A66BE8). Testo: "Soldi messi da parte per <Nome>: li riceve quando lascia il lavoro." In piccolo: "Trattamento di fine rapporto · consigliabile metterlo in un fondo pensione."
- **Una scheda per ogni fetta**, con bordo colorato: titolo e percentuale (sotto lo 0,5% scrivi "meno dell'1%"), importo, frase. La pagina **finisce con la scheda del TFR**.

## 8. Pagina 3: "INPS e tasse dove vanno?"

- **Titolo**: "INPS e tasse dove vanno?"
- **Sottotitolo**: "Lo Stato incassa **<ENTRATE_TOT>** e spende **<USCITE_TOT>**" (verde e arancione). Sotto, in piccolo: anno, "entrate = tasse e contributi sociali · uscite = spesa pubblica totale".
- **Grafico a quadrati** (treemap, stile mappa delle azioni), 184 × 96 mm. Ogni quadrato ha nome, percentuale sulle uscite e miliardi. La disposizione è fissa:
  - **colonna sinistra**: **Pensioni** sopra (sottotitolo "vecchiaia e reversibilità") e **Assistenza** sotto. In Assistenza il titolo sta a sinistra e l'elenco a destra (Malattia e disabilità, Figli (assegno unico), NASpI e cassa integraz., Assegno di inclusione, Altro), con i miliardi;
  - **colonna centrale**: Sanità, Istruzione e, sotto, Interessi sul debito e Difesa affiancati;
  - **colonna destra**: **Altro**, a tutta altezza, sempre ultimo a destra, con l'elenco delle sottovoci (sezione 5a).
  - Colori: Pensioni #5B8DEF, Assistenza #3FB8C9, Sanità #2BB673, Istruzione #E5B800, Interessi #e0563b, Difesa #6b7385, Altro #A66BE8.
  - **Prima lo spazio, poi le proporzioni**: ogni quadrato deve avere lo spazio minimo per non tagliare le etichette e tenerle lontane dai bordi (padding di circa 3 mm). Solo dopo dividi il resto in proporzione ai miliardi. Usa `split()` da `codice.md`, con minimi di partenza: colonne 60/60/52 mm, altezze Pensioni/Assistenza 34/34 mm, Sanità/Istruzione/riga bassa 26/24/28 mm, Interessi/Difesa 33/25 mm. Se qualcosa resta tagliato, alza il minimo e rigenera.
- **Titolo delle pensioni** su due righe: "L'INPS incassa dai contributi **<CONTRIB_INPS>** / ma deve pagare **<SPESA_INPS>** di pensioni + **<Assistenza>** di assistenza".
- Sotto, su una riga: "Il buco di bilancio di circa **<TRASF_INPS>** è coperto dallo Stato, che trasferisce all'INPS circa il **<TRASF_INPS/IRPEF_TOT %>** delle tue tasse IRPEF."
- Poi, in piccolo: "Anno <anno> · pensionati divisi per pensione mensile lorda (un pensionato può avere più pensioni, qui sommate)".
- **Grafico a barre** (SVG), una barra per fascia con altezza proporzionale ai miliardi.
  - Asse X con il solo inizio del range e il simbolo dell'euro: "Fino a 499€ lordi" (la prima fascia parte da 0), poi "Da 500€ lordi", "Da 1000€ lordi" … "Da 3000€ lordi".
  - Sopra ogni barra: "**Costo: 56 mld**" / "per 2,7M di" / "pensionati".
- **Fonti** in fondo: le due dashboard di Nazareno Lecis (Eurostat gov_10a_exp e gov_10a_taxag, MEF, INPS Open Data, Appendici statistiche, Rendiconti). Spiega che gli interessi sono la voce "Operazioni sul debito pubblico" e indica gli anni dei dati.

## 9. Pagina 4: "Consigli sul tuo netto"

Stesso stile della pagina 2: titolo, una frase introduttiva, torta e schede.

- **Introduzione**: "Ogni mese ricevi **<netto>**. Con la regola **50/30/20**: metà alle spese fisse, il 30% a quelle personali e il 20% al risparmio, diviso tra fondo emergenze e PAC."
- **Fette e schede** (importo + "al mese"):
  - 🔵 **Spese fisse · 50%**: "Le cose necessarie: affitto o mutuo, bollette, spesa, trasporti, telefono, assicurazioni." In piccolo: "Regola 50/30/20 · se superano il 50%, è la prima voce da rivedere."
  - 🟠 **Spese personali · 30%**: "I desideri: uscite, ristoranti, viaggi, hobby, abbonamenti, shopping." In piccolo: "Spendili senza sensi di colpa: il risparmio è già stato messo da parte."
  - 🟡 **Fondo emergenze · 10%**: "Su un conto deposito, per imprevisti come auto, salute o lavoro." In piccolo: "Obiettivo: 3–6 mesi di spese fisse (<3×> – <6×>). Raggiunto l'obiettivo, sposta questa quota sul PAC."
  - 🟢 **PAC personale · 10%**: "Piano di accumulo: una cifra fissa ogni mese, investita a lungo termine." In piccolo: "Di solito su strumenti diversificati e a basso costo (es. ETF azionari globali), con orizzonte di almeno 10 anni."
- **In fondo**: "Regola generale di educazione finanziaria, non è una consulenza personalizzata. Investire comporta rischi: il valore del PAC può scendere." Non nominare prodotti o emittenti specifici.

## 10. Controllo finale

- `pdfinfo`: entrambi i PDF devono avere **esattamente 4 pagine**. Se sono di più, riduci gli spazi della pagina 1 (`.sec` fino a ~14px), poi il carattere (non sotto ~8,5pt). Non togliere dati.
- Renderizza ogni pagina con `pdftoppm -r 70 -png` e guardala. Controlla che:
  - la torta della pagina 2 si veda. Attenzione a non riusare per altro nel codice il nome della variabile della torta;
  - non ci siano etichette tagliate nei quadrati;
  - le righe non vadano a capo dove non devono;
  - nulla esca dai margini.
- Invia i due PDF all'utente con una breve spiegazione: netto rispetto al costo aziendale, come hai stimato oneri e scatto, mese dell'inflazione usato, anni dei dati pubblici. Proponi di sostituire le stime con i dati reali dell'ufficio del personale.

## 11. Versione condivisibile (privacy)

Genera sempre la seconda copia, `…_condivisibile.pdf`, dallo **stesso HTML**:

- Nelle **pagine 1, 2 e 4** sostituisci ogni importo in euro con `<span class="red">000000</span>`. Usa tanti zeri quanto è lungo l'importo e lo stile CSS `display:inline-block; white-space:nowrap; background:#111; color:#111; border-radius:2px; padding:0 2px`.
- Regex per trovare gli importi: `[+−-]?\s?\d{1,3}(?:\.\d{3})*(?:,\d{2})?(?: |\s)?€`.
- La **pagina 3 resta intatta**: sono dati pubblici. Separa il corpo in pezzi e applica la regex solo alle pagine personali (vedi `codice.md`).
- Le cifre vere vanno **sostituite**, non solo coperte, così nel PDF non restano.
- Verifica con `pdftotext -f 1 -l 2` e `-f 4 -l 4` che nelle pagine personali non resti nessun "€", e che entrambi i PDF abbiano 4 pagine.
- Nella risposta ricorda che nome, azienda e dati anagrafici restano visibili, e offri di oscurarli.

## Note

- I dati personali come codice fiscale e IBAN non servono: lasciali fuori dal PDF.
- Se la busta paga è una scansione, usa l'OCR (pytesseract) e ricontrolla ogni cifra con i totali.
- Se ci sono più mesi, genera un PDF per mese, oppure chiedi se preferisce un unico file.
