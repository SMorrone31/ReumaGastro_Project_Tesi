# Tesi: dati GeoMx DSP (Reuma e Gastro)

Sono Simone Morrone, studente di Medical Bioinformatics (UNIVR), relatrice prof.ssa Rosalba Giugno. Analizzo dati GeoMx DSP (RNA, pannello Whole Transcriptome Atlas, lettura NGS). Obiettivo: partire da DCC, PKC e annotazione, costruire l'oggetto `NanoStringGeoMxSet` con `GeomxTools::readNanoStringGeoMxSet()`, poi QC, normalizzazione e analisi, senza fare supposizioni sui dati. Voglio capire ogni passaggio, non solo ottenere codice che funziona. Si parte da Reuma. Gastro è sospeso.

Stato corrente e prossimi passi: @STATO.md

## Come lavorare con me
- Rispondi in italiano, in modo diretto: frasi corte, poco grassetto, niente riepiloghi iniziali. Se il testo suona scritto da un'IA, riscrivilo.
- Distingui sempre ciò che è verificato (e come) da ciò che è dedotto o ignoto. Mai "quasi certo", mai un'ipotesi presentata come fatto. Se sbagli, dillo e correggi anche i file già consegnati.
- Prima di affermare qualcosa sui dati, controllalo sui file veri. Riporta l'output reale dei comandi: non scrivere valori attesi che non hai calcolato.
- Se R è installato, eseguilo con `Rscript`. Se non lo è, o manca un pacchetto, dillo e chiedi: non inventare l'output e non installare pacchetti senza chiedere.
- Se dico "ho il file X" e non lo trovi, dillo e chiedimelo. Non descrivere un file che non hai visto.
- Quando ti incollo un output, confrontalo con ciò che ti aspettavi e dimmi dove coincide e dove no.
- Quando ti chiedo di spiegare il codice: istruzione per istruzione, dicendo a cosa serve per il nostro scopo (non a livello tecnico), con valori reali dei miei dati e interpretazione del risultato. Un passo alla volta. Chiudi con una sola domanda di verifica.
- Documentazione: solo fonti ufficiali (Bruker/NanoString, Bioconductor), con link esatti. Segnala le altre come non ufficiali.
- Documenti ed email per i collaboratori: brevi, professionali, solo problemi e controlli necessari, con link. Non scrivere che qualcosa "va bene" se non l'hai verificato.
- Non modificare mai i file in `data/`. Chiedi conferma prima di sovrascrivere file già prodotti o scrivere in `data_clean/`.

## Privacy
- Una colonna senza intestazione dell'Excel di annotazione contiene cognomi di pazienti. Non stamparne né riportarne i valori, non salvarli in oggetti o file: usa un indice anonimo e rimuovila prima di condividere il file. I codici paziente `PT. xx` si possono usare.
- Se il progetto è in git, `data/` e `data_clean/` non vanno committate.

## Dati (cartella data/)
- PKC standard `Hs_R_NGS_WTA_v1.0.pkc` (l'originale non è stato fornito). È un file JSON con estensione `.pkc`: 18.815 sonde (18.676 endogene, 139 negative).
- 285 DCC, uno per pozzetto. Usare solo le cartelle `DCC_Files`; le altre cartelle di DCC del plate H sono copie. Reuma plate H: 96 DCC, Plate_ID 1001660015486. Reuma plate F: 93 DCC, Plate_ID 1001660015484. Gastro: 96 DCC, Plate_ID 1001660015507. La lettera F compare nel nome sia del plate F di Reuma sia di Gastro: distinguere per Plate_ID e percorso.
- Per ogni run: `DRAGEN/`, `summary.txt`, `logs.zip`, e una cartella `Extra_Files…` con `bsshinput.json` e `preprocessing_output.json`. `fastq_list.csv` contiene link S3 temporanei: non riportarli.
- Excel `Annotation template file GEOMX_29.01.2026_SP-BT-LP.xlsx`: 282 righe (un segmento per riga), 12 scan. Prima riga di istruzioni, intestazione alla riga 2 (`skip = 1`). Chiave unica: `Scan Name` + `ROI (Label)` + `Segment (Name/ Label)`. Il gruppo si ricava dalla data (giorno-mese-anno) in fondo allo `Scan Name`: 03102024 (95 righe, Gastro), 25072024 (95), 29022024 (92). Nessuna colonna con nome del DCC, pozzetto o Plate_ID; nessuna area né nuclei.

## Fatti verificati
- PKC↔DCC: in tutte e tre le run le sonde presenti nei DCC coincidono con quelle del PKC (Jaccard 1). Il confronto va fatto sull'unione dei pozzetti, perché il `Code_Summary` di un DCC elenca solo le sonde con conteggio maggiore di zero.
- Tutti i DCC sono integri (nessun `RTS_ID` duplicato, conteggi interi positivi), stesso software `DRAGEN_GeoMx_3.8.4-96`, un `SeqSetId` per run.
- 10 DCC senza sonde. Otto sono campioni elencati nel file di configurazione per i quali la pipeline non ha ricevuto i FASTQ (`summary.txt`, sezione `<Missing Samples>`, e log di post-elaborazione): Gastro B04, C02, D04, E04, E07, E08, F07 e plate F E05. Sono DCC vuoti da 644 byte, con header non compilato: dato non disponibile, non zero conteggi. Gli altri due sono processati ma quasi senza reads: plate F E10 (`Raw` = 0) e Gastro C04 (`Raw` = 2).
- In ogni piastra A01 ha i conteggi totali più bassi (2, 119 e 4).
- L'Excel ha 282 righe contro 285 DCC: 3 in meno. I due insiemi non si abbinano per posizione.
- Problemi dell'Excel: colonna senza intestazione con `BASELINE` (52 righe, solo Gastro); `ROIComments` duplicata; in Gastro `PATIENT` è vuoto; `RESPONSE` e `RESPONSE_POOLED` con testo `NA` (27 righe Gastro) e celle vuote (3 righe Reuma); etichette non uniformi (`PanCK`/`pANck`, `RESP JAKi`/`RESP JAKI`, `NORMALWEIGHT`/`NORMAL WEIGHT`, doppio spazio in `RESP  ANTI_TNF`); in 25 segmenti Gastro il codice in `Custom Segment Name` non compare nello `Scan Name`; segmenti con paziente incoerente (PT. 09 ROI 006, PT. 14 ROI 013, PT. 13).

## Cosa dice la documentazione ufficiale (verificato online)
- Il readout package si scarica, e si può riscaricare ("Re-download Readout Package"), dal Control Center del DSP. Contiene `Configuration.ini`, Lab Worksheet e lista degli indici. Il Lab Worksheet descrive il contenuto di ogni pozzetto con dettagli di ROI e segmento, nuclei e area (colonna H, MAN-10153 p. 55). Le colonne esatte non sono descritte.
- GeomxTools raccomanda il Lab Worksheet come `phenoDataFile`. Serve una colonna che identifichi ogni DCC (`phenoDataDccColName`, predefinita `Sample_ID`). `configFile` è opzionale. Nell'annotazione l'ordine degli NTC è importante.
- NTC: pozzetto opzionale, scelto nella configurazione dello scan, senza aspirato, con pochi conteggi. Negli esempi ufficiali è A01.
- `DccMetadataEnabled` in `preprocessing_output.json` è l'opzione facoltativa di tracciabilità FASTQ-DCC: non riguarda le annotazioni.
- Fonti: Knowledge Base Bruker (brukerspatialbiology.com/support/knowledgebase/geomx-systems/ e …/geomx-data-analysis/), manuale MAN-10153, manuale e vignette Bioconductor di GeomxTools e GeoMxWorkflows.

## Dedotto o non documentato: non presentarlo come fatto
- La mappa pozzetto→segmento ricostruita in `00c_annotation_reconstruction.Rmd` (dall'espressione dei marcatori) è una deduzione, robusta alle mie scelte ma non confermata. Non va nei documenti per i collaboratori.
- A01 = NTC: non confermato. Quale gruppo di scan corrisponde a quale piastra: non documentato (i nomi dei `.ini` contengono `25072024` per il plate H e `03102024` per Gastro).
- Perché mancano i FASTQ degli otto campioni; il significato di celle gialle, `BASELINE`, `ARTICULAR_INVOLVEMENT` vuoto, `OBESE` rispetto a `OVERWEIGHT_OBESE`.
- Che `RTS_sequences.txt` coincida con la whitelist usata da DRAGEN (`rts_whitelist.txt`, non nella consegna).
- Esistono analisi esplorative interne (cromosoma Y per il sesso, impronta HLA): non fanno parte dei documenti per i collaboratori. Spiegale solo se te lo chiedo, partendo da zero.

## Persone
- Simone (collaboratore con il mio stesso nome): ha condiviso i dati e i Lab Worksheet di Reuma. Barbara ha presentato lo studio e, con il suo collaboratore, risponde alle domande sull'Excel. Il gruppo Gremese ha prodotto Reuma, non Gastro. Humanitas è il tramite per i dati: i collaboratori stretti non possono scaricare altri dati, le richieste vanno girate a Humanitas.

## File di riferimento nel progetto
- `main.tex`: il mio manuale in italiano sul workflow GeoMx.
- `README.md` e gli script R (`01_DGE_analysis.R`, `02_Pseudobulk.R`, …): la pipeline degli autori sul pancreas (Carpenter et al., Cancer Discovery 2023; GeoMx GSE226829, single-cell GSE229413). Manca `src/geomx_utils.R` (con `runQC`).
- I file `.pdf` della documentazione GeomxTools possono essere archivi zip (una pagina per immagine più un `.txt` per pagina): controlla con `file` prima di leggerli.
- Documenti già prodotti: `00a_data_structure.Rmd`, `00b_pkc_dcc_concordance.Rmd`, `00c_annotation_reconstruction.Rmd` (interno), `00_Gastro_Reuma_DataAnalysis.Rmd` (per i collaboratori), `README_dati.txt`. Output in `data_clean/`.
