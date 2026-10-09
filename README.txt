================================================================================
DATI GeoMx DSP - spondiloartrite e malattia di Crohn
Descrizione del dataset contenuto in data/
================================================================================

Questo file descrive CHE COSA sono questi dati, da dove vengono e come sono
organizzati. E' un documento descrittivo: non contiene lo stato di avanzamento
dell'analisi.

Le informazioni sullo studio clinico provengono dall'incontro con le parti
coinvolte e sono riportate come raccontate; quelle sui file derivano
dall'ispezione diretta del dataset.


--------------------------------------------------------------------------------
1. LO STUDIO
--------------------------------------------------------------------------------
Studio avviato al Policlinico Gemelli e poi spostato a Humanitas.

Coorte complessiva: 120 pazienti, suddivisi in
    60 con spondiloartrite
    60 con malattia di Crohn

Si tratta di pazienti "difficult to treat": il criterio di inclusione e' la
mancata risposta alla terapia convenzionale. Non e' detto che siano stati
arruolati al primo livello biologico, possono esserlo anche al secondo.

Percorso terapeutico di riferimento:
  - terapia di base all'esordio
  - se il paziente non risponde, si passa a una terapia di secondo livello,
    diretta contro una singola citochina
  - esistono circa 15 terapie di secondo livello, sia in ambito reumatologico
    sia gastroenterologico

Gli anti-TNF sono usati per entrambe le patologie, cosi' come i JAK inhibitors.

SCOPO DELLO STUDIO
Individuare biomarcatori o marcatori clinici che si associno alla risposta alla
terapia, per aiutare il clinico nella scelta della terapia di secondo livello.
La risposta clinica si valuta a 12 mesi; i biomarcatori vengono raccolti ogni
3 mesi. Sarebbe il primo dato prodotto su questa coorte.

SOVRAPPOSIZIONE TRA LE DUE PATOLOGIE
Le due condizioni non sono separate nettamente, ed e' una caratteristica
rilevante del disegno:
  - i pazienti con spondiloartrite possono avere interessamento della colonna o
    rigidita' del bacino, ma anche coinvolgimento intestinale
  - i pazienti con malattia infiammatoria intestinale possono sviluppare una
    spondiloartrite secondaria
Vanno quindi considerate tutte le combinazioni, non due gruppi disgiunti.

LIVELLI DI DATO RACCOLTI (oltre alla spatial transcriptomics)
  - circa 40 citochine, oltre ai marcatori classici del laboratorio clinico
  - analisi anatomopatologica di base sul tessuto
  - immunoistochimica
  - campioni di feci per il microbiota (gruppo Rescigno)
  - single cell analysis
  - digital spatial profiling (questi dati)

TESSUTI
  - pazienti reumatologici: tessuto sinoviale delle articolazioni
  - pazienti gastroenterologici: tessuto gastrointestinale

Sottogruppo con digital spatial profiling, come riferito all'incontro:
  12 pazienti con spondiloartrite, 8 con biopsia intestinale.


--------------------------------------------------------------------------------
2. LA TECNOLOGIA: GeoMx DSP
--------------------------------------------------------------------------------
Il GeoMx Digital Spatial Profiler misura l'espressione genica MANTENENDO
L'INFORMAZIONE SPAZIALE all'interno del tessuto. Non e' single cell: il profilo
trascrittomico viene ricostruito per REGIONE, non per cellula.

Flusso di lavoro:

  1. SELEZIONE DELLE REGIONI (fatta dal patologo, a mano)
     Il patologo individua sulla fetta di tessuto le regioni dove ha senso
     studiare l'espressione genica. Essendo una selezione manuale, le regioni
     possono trovarsi ovunque e il loro numero varia: da una fetta si possono
     ricavare piu' zone di interesse, e un paziente puo' avere piu' fette.
     Di conseguenza il numero di regioni per campione NON e' costante.

  2. ROI (Region of Interest)
     La regione selezionata sul tessuto.

  3. AOI (Area of Illumination)
     All'interno di un ROI si identificano sottoregioni corrispondenti alle
     componenti cellulari che si vogliono studiare nel dettaglio: immunitaria,
     epiteliale, tumorale, e cosi' via. Le componenti vengono distinte mediante
     colorazione. L'AOI e' l'unita' effettivamente illuminata e raccolta.

  4. SEQUENZIAMENTO
     Ogni AOI viene sequenziata separatamente. Si ottiene il profilo
     trascrittomico di quella specifica area (epiteliale, immunitaria,
     tumorale...), non della singola cellula.

  5. MATRICE RISULTANTE
     Una tabella GENI x AOI. L'attesa biologica e' di osservare differenze di
     espressione tra componenti diverse, per esempio tra immunitaria e tumorale,
     evidenziabili gia' con una PCA.

La struttura si scala su piu' livelli: piu' AOI per ROI, piu' ROI per campione,
piu' campioni per paziente, piu' pazienti. Questo annidamento e' una
caratteristica intrinseca del dato e va tenuto presente in qualsiasi modello
statistico.

Il formato di output per la matrice di conteggi e' il DCC (uno per AOI).
L'identificazione di una AOI passa per lo Scan Name; il paziente e' un livello
superiore.


--------------------------------------------------------------------------------
3. COMPOSIZIONE DEL DATASET
--------------------------------------------------------------------------------
I dati sono stati prodotti da Humanitas e consegnati tramite cartella Drive
condivisa. Sono divisi in due gruppi, corrispondenti ai due ambiti clinici:

  Reuma   - tessuto sinoviale, pazienti con spondiloartrite
            due piastre di sequenziamento (plate H e plate F)
  Gastro  - tessuto gastrointestinale
            una piastra

Ogni AOI occupa un pozzetto di una piastra da 96 e produce un file .dcc.
Il nome del file codifica piastra e pozzetto, per esempio:

    DSP-1001660015486-H-A01.dcc
        |             | |
        |             | pozzetto (A01 ... H12)
        |             lettera della piastra
        Plate_ID

Contenuto effettivo:

  GRUPPO   PIASTRA   Plate_ID         POZZETTI   NOTE
  Reuma    plate H   1001660015486    96         piastra completa
  Reuma    plate F   1001660015484    93         mancano H10, H11, H12
  Gastro   -         1001660015507    96         piastra completa

  Totale Reuma: 189 pozzetti.

Attenzione: sia Reuma plate F sia Gastro usano la lettera "F" nel nome dei file.
Sono piastre diverse, distinguibili solo dal Plate_ID.

I dati di Reuma provengono da due sedute di sequenziamento distinte
(aprile 2024 per plate F, ottobre 2024 per plate H), con profondita' di
sequenziamento diverse tra le due piastre.


--------------------------------------------------------------------------------
4. IL PANNELLO DI SONDE (PKC)
--------------------------------------------------------------------------------
File: Hs_R_NGS_WTA_v1.0.pkc

Pannello Human NGS Whole Transcriptome Atlas RNA, versione 1.0 (data 210202),
con MinArea 16000 e MinNuclei 200. E' il pannello standard NanoString, non un
pannello personalizzato: Humanitas non ha fornito il PKC originale e si utilizza
quello pubblico, la cui compatibilita' con questi DCC e' stata verificata.

Contenuto: 18.815 sonde, ciascuna con un identificativo RTS_ID univoco e un
barcode RTS_Seq di 12 basi.

    Endogenous01 : 18.676 sonde   geni misurati
    Negative01   :    139 sonde   NegProbe-WTX, sonde negative di controllo

Le sonde negative non corrispondono ad alcun trascritto: misurano il rumore di
fondo e sono la base del calcolo del limite di quantificazione (LOQ).

Il file e' un JSON: la lista Targets contiene i target, ciascuno con una o piu'
sonde (RTS_ID, RTS_Seq, ProbeID, GeneID, Accession, coordinate genomiche).


--------------------------------------------------------------------------------
5. FORMATO DEI FILE DCC
--------------------------------------------------------------------------------
Ogni .dcc e' un file di testo con due parti.

HEADER - coppie "chiave,valore", tra cui:
    Plate_ID, Well, SeqSetId      identificazione del pozzetto
    Raw                           reads grezze
    Trimmed                       reads dopo il trimming
    Stitched                      reads ricostruite
    Aligned                       reads allineate alle sonde
    umiQ30, rtsQ30                qualita' di base

CODE_SUMMARY - blocco delimitato da <Code_Summary> e </Code_Summary>, con righe
"RTS_ID,conteggio".

Caratteristica importante del formato: il Code_Summary elenca SOLO le sonde con
conteggio maggiore di zero. Il numero di righe varia molto tra pozzetti, da 0 a
18.815. Un singolo DCC non contiene quindi l'intero pannello, e questo e'
normale: riflette quante sonde hanno prodotto segnale in quella AOI.

Accanto ai DCC, ogni run contiene:
    summary.txt                   metriche Raw/Trimmed/Stitched/Aligned e
                                  conteggio deduplicato per ogni pozzetto
    DRAGEN/<pozzetto>/            metriche dettagliate per pozzetto prodotte da
                                  DRAGEN (mapping, trimming, FastQC, insert size)
    DRAGEN/Input/                 parametri della pipeline: dragen.cfg,
                                  fastq_list.csv, adapter.R1/R2.fasta e
                                  RTS_sequences.txt, l'elenco dei 18.815 barcode
                                  cercati nelle reads
    Extra_Files.../               file di servizio BaseSpace, tra cui
                                  preprocessing_output.json (elenco dei campioni
                                  processati) e bsshinput.json (parametri e
                                  riferimenti ai FASTQ di input)
    system_files/, logs.zip       log di esecuzione, senza contenuto analitico


--------------------------------------------------------------------------------
6. ORGANIZZAZIONE DELLE CARTELLE
--------------------------------------------------------------------------------
data/
  Hs_R_NGS_WTA_v1.0.pkc                    pannello sonde
  Annotation template file GEOMX_29.01.2026_SP-BT-LP.xlsx
  Reuma/
    Finalizzata_GeoMX_plate_H-32399433/            plate H (ottobre 2024)
    Finalizzata-Gremese-11-04-2024-plateF-30160133/ plate F (aprile 2024)
    GeoMx_NGS_Pipeline_..._DCC_2 seduta Geomx/      copia dei DCC del plate H
  Gastro/
    GeoMx_NGS_Pipeline_09_24_2025_12_03_48-64833845/

Dentro ogni cartella di run (schema identico nei tre casi):
    <run>-ds.<hash>/
        DCC_Files/            i file .dcc
        DRAGEN/               metriche per pozzetto e cartella Input/
        summary.txt
        logs.zip, system_files/
    Extra_Files_from_<run>-ds.<hash>/
        metadata/, bsshinput.json, bsshoutput.json,
        preprocessing_output.json, _manifest.json, _tags.json
    ds.<hash>/                solo system_files

Ridondanze presenti nella consegna: i DCC del plate H compaiono in tre cartelle
distinte, che contengono gli stessi 96 file identici byte per byte, piu' le
rispettive versioni compresse. Anche il PKC e' presente sia come .pkc sia come
.zip.


--------------------------------------------------------------------------------
7. IL FILE DI ANNOTAZIONE
--------------------------------------------------------------------------------
"Annotation template file GEOMX_29.01.2026_SP-BT-LP.xlsx"

Nonostante il nome, e' un file gia' compilato. Contiene 282 righe, una per
segmento (AOI), su 12 scan, con 23 colonne. La prima riga del foglio e' una riga
di istruzioni: l'intestazione effettiva e' la seconda.

Ogni riga e' identificata dalla combinazione Scan Name + ROI + Segment, che
rispecchia esattamente la gerarchia descritta al punto 2.

Le colonne coprono tre livelli:
  - spaziale/tecnico : Scan Name, ROI, Segment, tag di segmento (CD68, CD3,
                       CD20, PanCK...), area e conteggio nuclei
  - paziente         : codice paziente, sesso, BMI, sede
  - clinico          : risposta alla terapia, terapia precedente, grado
                       sinoviale, coinvolgimento intestinale e articolare,
                       guarigione istologica

Le righe si ripartiscono tra scan riferibili al tessuto sinoviale (segmenti
CD68/CD3/CD20, grado sinoviale) e scan riferibili al tessuto intestinale
(PanCK, guarigione intestinale, controlli sani).

Due caratteristiche del file da conoscere:
  - alcune etichette compaiono in varianti non uniformi (differenze di
    maiuscole, spazi doppi), e un paio di colonne hanno nomi irregolari o
    duplicati
  - una colonna contiene cognomi di pazienti; l'identificativo da usare e' il
    codice paziente

Il file non contiene riferimenti ai pozzetti ne' ai nomi dei DCC: il legame tra
un pozzetto di piastra e la AOI che vi e' stata caricata non e' presente in
nessuno dei file ricevuti, e i DCC non incorporano annotazioni
(preprocessing_output.json riporta DccMetadataEnabled: false).
================================================================================
