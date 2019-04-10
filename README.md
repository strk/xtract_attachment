# xtract_attachment

Questa è un'utility per estrarre eventuali allegati da una fattura in formato "xml".

## Background

In Italia è entrata in vigore una normativa per cui le fatture devono essere trasmesse all'Agenzia delle Entrate secondo un determinato schema, in formato "xml".

Esiste della documentazione ufficiale, si può trovare [qui](https://www.fatturapa.gov.it/export/fatturazione/it/normativa/f-2.htm).

Non è obbligatorio che sia incluso un allegato ma alcune società aggiungono in un `<tag>` del file "xml" la versione PDF della fattura.

Un amico riceve via email una copia della fattura che viene inviata all'Agenzia delle Entrate e mi ha chiesto un modo facile per poter aprire l'allegato PDF.

Ho cominciato con poche righe di codice, per finire con questo programma.

## Installazione

-   Puoi eseguire questo programma dalla directory in cui si trova.
-   Se usi linux, puoi copiare il programma in `/usr/local/bin/`.

### pyinstaller

Il mio amico usa Windows e non è molto pratico di informatica, quindi ho usato pyinstaller per creare un eseguibile.
Se vuoi farlo anche tu, ricordati di assegnare il valore `True` alla variabile `_pyinstaller_trick`, in questo modo il programma analizzerà tutti i file "xml" presenti nella directory da cui viene eseguito.

Io ho usato le seguenti opzioni:

`pyinstaller --onefile --noconsole xtract_attachment.py`

NOTA: tieni presente che l'eseguibile generato dipende dal sistema operativo, achitettura e versione di Python come descritto nella nota che trovi in [questa](https://pyinstaller.readthedocs.io/en/stable/operating-mode.html#what-pyinstaller-does-and-how-it-does-it) sezione.

#### Considerazioni sulla sicurezza

La [documentazione](https://pyinstaller.readthedocs.io/en/stable/operating-mode.html#how-the-one-file-program-works) di pyinstaller è chiara sulle possibili implicazioni di usare l'opzione `--onefile`, fare riferimento a quella.

## Usare il programma

Usare il programma è semplice, serve fornire un solo argomento: un file o una directory da analizzare.

Il programma di default salverà gli allegati nella stessa directory del file di origine, puoi usare il seguente argomento facoltativo per indicare una diversa directory in cui estrarre gli allegati.
`-o OUTDIR, --outdir OUTDIR`

Di default il programma *non* sovrascriverà un file esistente con lo stesso nome di un allegato. Puoi utilizzare il seguente argomento facoltativo per cambiare questo comportamento.
`-s {low,max}, --safety {low,max}`

### Esempi

`./xtract_attachment.py .`

`./xtract_attachment.py "Fattura1.xml"`

`./xtract_attachment.py /home/alex/Download/`

`./xtract_attachment.py /home/alex/Download/ -o /home/alex/Documenti/Allegati/`

`./xtract_attachment.py /home/alex/Download/ -o /home/alex/Documenti/Allegati/ -s low`

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

Nella cartella "test" troverai un file di esempio che ho scaricato dalla documentazione dell'Agenzia delle Entrate nel quale ho aggiunto degli allegati. Se eseguirai il programma su quel file, otterrai quattro allegati con al loro interno poche righe che confermano le loro caratteristiche.

-   'IT01234567890\_FPR03\_allegato\_3.pdf'
-   'IT01234567890\_FPR03\_allegato\_4.txt'
-   'myAttachment123\_1.pdf'
-   'myAttachment123\_2.txt'
