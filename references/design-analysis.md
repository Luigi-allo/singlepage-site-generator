# Analisi del sito di riferimento

Usa questo documento durante il punto 1 del workflow principale.

## Procedura

1. Recupera la pagina principale e le pagine/sezioni necessarie a comprendere il linguaggio visivo.
2. Individua i fogli CSS collegati pubblicamente raggiungibili.
3. Leggi i valori reali quando disponibili invece di stimarli visivamente.
4. Se il sito usa iframe o wrapper di preview, raggiungi e analizza la pagina reale con la procedura seguente.
5. Se il sito è fortemente client-rendered e gli stili non sono ricavabili dal sorgente, usa un browser con rendering JavaScript e gli strumenti visivi disponibili.
6. Se dettagli essenziali non sono verificabili, chiedi l'URL diretto o screenshot completi; non sostituirli con supposizioni generiche.

## Anteprime, wrapper e iframe

Applica questa procedura ai marketplace e servizi di preview, inclusi TemplateMonster, ThemeForest ed Envato:

1. Renderizza la pagina contenitore eseguendo JavaScript e attendi il caricamento del DOM.
2. Cerca `iframe[src]`, `data-src`, `srcdoc`, iframe creati dinamicamente e redirect verso la preview.
3. Individua l'iframe principale in base a dimensione, posizione e contenuto. Ignora iframe pubblicitari, analytics, chat, video e altri elementi di servizio.
4. Risolvi URL relativi e URL con protocollo `//` rispetto alla pagina contenitore.
5. Apri direttamente l'URL finale dell'iframe o della preview in una nuova navigazione; non tentare di analizzarlo soltanto attraverso il wrapper.
6. Se la pagina raggiunta contiene un altro iframe principale, ripeti l'operazione fino a tre livelli, mantenendo l'elenco degli URL visitati per evitare cicli.
7. Analizza HTML, CSS, font, immagini, layout, breakpoint e comportamento visivo esclusivamente sulla pagina finale.
8. Acquisisci, quando possibile, schermate desktop e mobile della pagina finale renderizzata.
9. Escludi dall'analisi barra del marketplace, pulsanti di acquisto, selettori responsive e controlli appartenenti al contenitore.
10. Annota l'URL finale effettivamente analizzato.

Se il recupero HTML restituisce una pagina vuota, incompleta o limitata all'interfaccia del marketplace, non fermarti al fetch statico: usa il DOM renderizzato. Se autenticazione, protezioni anti-bot o restrizioni tecniche impediscono l'accesso, chiedi all'utente l'URL diretto oppure screenshot completi e non produrre un layout generico basato su supposizioni.

## Elementi da estrarre

### Layout

- larghezza massima dei container;
- full-width vs boxed;
- struttura hero;
- numero e ordine delle sezioni;
- griglie e colonne;
- allineamenti;
- spazi verticali;
- rapporto testo/immagine.

### Tipografia

- `font-family` se verificabile;
- fallback stack;
- pesi;
- dimensioni H1/H2/H3/body;
- line-height;
- letter-spacing;
- uppercase/lowercase;
- comportamento responsive.

Non scaricare o redistribuire font proprietari dal sito di riferimento.

### Colori

Rileva dai CSS, quando disponibili:

- background principali;
- colori testo;
- accent;
- bottoni;
- bordi;
- overlay;
- gradienti.

Se l'utente fornisce una palette, usa quella come vincolo principale e usa il sito di riferimento solo per i rapporti visivi tra i colori.

### UI

Analizza:

- bottoni;
- nav;
- card;
- badge;
- icone;
- bordi/radius;
- shadow;
- divider;
- form;
- CTA.

### Responsive

Quando verificabile annota:

- breakpoint;
- trasformazione griglia → stack;
- comportamento menu;
- riduzione tipografica;
- padding mobile;
- immagini crop/contain.

### Decorazioni

Se il sito usa blob, ellissi, pattern, linee, gradienti, pseudo-elementi o SVG decorativi:

- non copiare l'asset originale;
- ricrea un effetto originale con CSS/SVG generati da zero;
- usa la palette del progetto;
- mantieni solo il ruolo compositivo generale.

### Header trasparente

Se l'header è overlay sulla hero:

- controlla contrasto logo/nav;
- verifica lo stato `is-scrolled` o equivalente;
- se manca un logo negativo autorizzato, non crearne uno automaticamente;
- usa un fallback di header non trasparente oppure segnala la necessità dell'asset.
