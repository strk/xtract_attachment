# xtract_attachment [![Codacy Badge](https://api.codacy.com/project/badge/Grade/77b81d2d5ae94ca0af67f0c921f9992c)](https://www.codacy.com/app/alexlab2017/xtract_attachment?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=alexlab2017/xtract_attachment&amp;utm_campaign=Badge_Grade)

Questa è un'utility per estrarre eventuali allegati da una fattura in formato "xml".

## Background

In Italia è entrata in vigore una normativa per cui le fatture devono essere trasmesse all'Agenzia delle Entrate secondo un determinato schema, in formato "xml".

Esiste della documentazione ufficiale, si può trovare [qui](https://www.fatturapa.gov.it/it/norme-e-regole/documentazione-fattura-elettronica/).

Non è obbligatorio che sia incluso un allegato ma alcune società aggiungono in un `<tag>` del file "xml" la versione PDF della fattura.

Un amico riceve via email una copia della fattura che viene inviata all'Agenzia delle Entrate e mi ha chiesto un modo facile per poter aprire l'allegato PDF.

Ho cominciato con poche righe di codice, per finire con questo programma.

## Installazione

-   Puoi eseguire questo programma dalla directory in cui si trova.
-   Se usi linux, puoi copiare il programma in `/usr/local/bin/`.

### pyinstaller

Il mio amico usa Windows e non è molto pratico di informatica, quindi ho usato pyinstaller per creare un eseguibile.
Se vuoi farlo anche tu, ricordati di assegnare il valore `True` alla variabile `_OVERRIDE`, in questo modo il programma analizzerà tutti i file "xml" presenti nella directory da cui viene eseguito.

Io ho usato le seguenti opzioni:

`pyinstaller --onefile --noconsole xtract_attachment.py`

NOTA: tieni presente che l'eseguibile generato dipende dal sistema operativo, achitettura e versione di Python come descritto nella nota che trovi in [questa](https://pyinstaller.readthedocs.io/en/stable/operating-mode.html#what-pyinstaller-does-and-how-it-does-it) sezione.

#### Considerazioni sulla sicurezza

La [documentazione](https://pyinstaller.readthedocs.io/en/stable/operating-mode.html#how-the-one-file-program-works) di pyinstaller è chiara sulle possibili implicazioni di usare l'opzione `--onefile`, fare riferimento a quella.

## Usare il programma

Usare il programma è semplice, serve fornire un solo argomento: un file o una directory da analizzare.

### Argomenti facoltativi

`-o OUTDIR, --outdir OUTDIR`

Il programma di default salverà gli allegati nella stessa directory del file di origine, puoi usare questo argomento facoltativo per indicare una diversa directory in cui estrarre gli allegati.

`-s {low,max}, --safety {low,max}`
Di default il programma **non** sovrascriverà un file esistente con lo stesso nome di un allegato. Puoi utilizzare questo argomento facoltativo per sovrascrivere i file con lo stesso nome.

`-q, --quite`
Il programma non scriverà nulla in caso di successo ma scriverà deli avvertimenti nel caso di errori. Puoi impedire qualsiasi messaggio di errore con questo argomento. NOTA: nel caso di un errore di livello `CRITICAL` il messaggio verrà presentato lo stesso.

### Esempi

`./xtract_attachment.py .`

`./xtract_attachment.py "Fattura1.xml"`

`./xtract_attachment.py /home/alex/Download/`

`./xtract_attachment.py /home/alex/Download/ -o /home/alex/Documenti/Allegati/`

`./xtract_attachment.py ../input -o /home/alex/Documenti/Allegati/ -s low`

`./xtract_attachment.py /cartella -o /home/alex/Documenti/Allegati/ -s low -q`

## Note importanti (TODO?)

Secondo la documentazione dell'Agenzia delle Entrate, all'interno di un tag `<Allegati>` sono obbligatori solamente due tag:
-   `<NomeAttachment>`
-   `<Attachment>`

Mentre sono facoltativi:
-   `<AlgoritmoCompressione>`
-   `<FormatoAttachment>`

Non è chiaro però se nel nome sia consentito avere anche l'estensione, che già di per sè dovrebbe indicare il formato. Nell'esempio reale che ha ricevuto il mio amico non c'era l'estensione nel nome del file, mentre era presente il tag `<FormatoAttachment>`.
Il tag `<AlgoritmoCompressione>` mi pare ridondante dato che, se venisse allegato un archivio compresso, sarebbe sufficente indicarlo nel tag `<FormatoAttachment>` anche perché un archivio compresso potrebbe contenere più file al suo interno, di formati diversi.

*Non* avrebbe senso per me qualcosa del genere:
```xml
<Allegati>
    <NomeAttachment>1234567890</NomeAttachment>
    <FormatoAttachment>PDF</FormatoAttachment>
    <AlgoritmoCompressione>ZIP</AlgoritmoCompressione>
</Allegati>
```
Ad ogni modo, dato lo scopo principale di aiutare un amico e la mancanza di documentazione *chiara* e *inequivocabile*, lascio come TODO l'approfondimento di questo aspetto.

## Test

Nella cartella "test" troverai tre file di esempio elencati di seguito, con il relativo output.
NOTA: per quanto riguarda il file "IT01234567890\_FPR03.xml" non vedrai alcun messaggio in quanto nello stile Unix/Linux "nessun output" vuol dire "tutto ok".

-   binary.xml
    -   `WARNING: open and read file "binary.xml" fail: 'utf-8' codec can't decode byte 0xa1 in position 0: invalid start byte`

-   IT01234567890\_FPR03.xml
    -   IT01234567890\_FPR03\_allegato\_3.pdf
    -   IT01234567890\_FPR03\_allegato\_4.txt
    -   myAttachment123\_1.pdf
    -   myAttachment123\_2.txt

-   IT01234567890\_FPR04\_errors.xml
    -   `WARNING: Processing <Allegati> tag n#3 in "IT01234567890\_FPR04\_errors.xml": <Attachment> tag not found.`
    -   `WARNING: Processing <Allegati> tag n#1 in "IT01234567890\_FPR04\_errors.xml": <Attachment> tag empty.`
    -   `WARNING: Processing <Allegati> tag n#2 in "IT01234567890\_FPR04\_errors.xml": <Attachment> tag wrong encoding.`
