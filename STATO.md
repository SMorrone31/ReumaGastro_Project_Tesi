# Stato attuale (al 9 ottobre 2026)

Aggiorna questo file quando lo stato cambia, dopo avermelo detto o chiesto conferma.

## Dove siamo
- Il collaboratore Simone ha aggiunto nella cartella condivisa di Reuma due Lab Worksheet, uno per il plate H e uno per il plate F. Secondo lui contengono "92 e 95 campioni" (187 in totale, come le righe Reuma dell'Excel). L'abbinamento file-piastra e il numero per file non sono verificati: i DCC sono 96 (H) e 93 (F). Non assumere quale file sia quale piastra né quanti campioni abbia: lo dice il Plate_ID scritto nei file.
- Io li ho scaricati ma non sono ancora nel progetto né controllati. Finché non lo sono, nulla del loro contenuto è verificato.
- Gastro: il gruppo Gremese non l'ha prodotto e il suo Lab Worksheet non è stato condiviso. Resta fuori.
- Le domande sull'Excel (pazienti, segmenti incoerenti, codici Gastro, significato delle colonne, se riformulare il file) andranno a Barbara e al suo collaboratore. Il collaboratore non ha i FASTQ degli otto campioni e non sa se ci siano pozzetti di controllo.
- Ho preparato una risposta di attesa ("controllo e ti scrivo"). Resta da rispondere alla sua domanda "i file vanno bene?" dopo i controlli.

## Quando i Lab Worksheet sono nel progetto: controlli, in quest'ordine
1. Leggili come file originali, senza convertirli né risalvarli. Dì quali sono il formato e le colonne, senza assumere i nomi: cerca quella che identifica il DCC, pozzetto, Plate_ID, area, nuclei, ROI, segmento, scan, e se compare un pozzetto di controllo.
2. Plate_ID di ogni file contro quelli dei DCC (1001660015486 per H, 1001660015484 per F).
3. Insieme dei pozzetti del file contro quelli dei DCC della piastra: quali DCC restano senza riga, quali righe senza DCC. Controlla E05 e E10 del plate F.
4. Righe dei worksheet contro le 187 righe Reuma dell'Excel (gruppi 25072024 e 29022024), per scan, ROI e segmento: corrispondenza uno a uno, doppioni, righe in più o in meno.
5. Se i worksheet dicono qualcosa sul pozzetto di controllo (NTC).
6. Confronto con la ricostruzione di `00c`, pozzetto per pozzetto, dicendo cosa coincide e cosa no.
7. Riporta solo ciò che hai verificato. Se un file è diverso da come lo ha descritto il collaboratore, dillo in modo neutro. Se contengono cognomi di pazienti, non stamparli.
8. Alla fine: un riepilogo dei risultati e una risposta breve al collaboratore, in prima persona, che dica se i file vanno bene e cosa devo eventualmente chiedergli.

## Prossimi passi
1. Controllare i Lab Worksheet (sopra).
2. Costruire `ns_object` per Reuma passo per passo, testando come la funzione tratta i DCC vuoti, le righe NTC, il nome del DCC con o senza `.dcc`, e quale colonna di appaiamento accetta.
3. QC e analisi.
4. Prima dei risultati vanno decise per scritto le regole per i segmenti segnalati (escluderli o analizzare con e senza). Piastra, corsa di sequenziamento e gruppo di scan coincidono: tenerne conto nel disegno.
