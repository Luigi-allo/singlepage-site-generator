# Checklist QA finale

Esegui questi controlli prima della consegna.

## PHP e struttura

- `index.php` carica correttamente `config.php`.
- `form-handler.php` carica lo stesso file di configurazione.
- Nessuna credenziale è duplicata nel frontend.
- Nessun errore PHP evidente o include path rotto.
- Tutti i file referenziati esistono.

## HTTPS e redirect

### Controlli eseguibili in locale

- Il server di destinazione è identificato oppure indicato come dato da confermare.
- Per Apache/LiteSpeed, `.htaccess` è presente nella root e conserva host, percorso e query string.
- Per Nginx, `.htaccess` non viene generato e la configurazione del redirect è segnalata come intervento server.
- La consegna non dichiara SSL o redirect verificati se il sito non è ancora stato caricato online.

### Controlli da eseguire dopo il caricamento

- Il certificato SSL del dominio è attivo e valido prima di rendere operativo il redirect.
- Verifica se il redirect HTTP → HTTPS è già gestito da hosting, pannello server, CDN o proxy ed evita regole duplicate.
- Una richiesta HTTP reale raggiunge la corrispondente URL HTTPS senza loop o catene di redirect evitabili.
- Canonical, Open Graph, Twitter Card, form e risorse interne non riportano URL HTTP residui sul sito pubblicato.

## HTML/CSS/JS

- Nessun errore JavaScript evidente in console.
- `style.css` e `script.js` non sono minificati o compressi: conservano indentazione, ritorni a capo e una struttura facilmente modificabile.
- `index.php` e gli altri file PHP con markup HTML non sono minificati o compressi: tag, sezioni e blocchi PHP conservano indentazione, ritorni a capo e una struttura facilmente modificabile.
- Menu mobile apre/chiude correttamente.
- Il menu mobile aperto usa `position: absolute`, è ancorato all'header/nav e non spinge hero o contenuti verso il basso.
- Il menu si chiude dopo il click su un anchor quando previsto.
- Header scroll state funziona quando previsto.
- Reveal non lascia contenuti permanentemente invisibili.
- `prefers-reduced-motion` disabilita le animazioni non essenziali.
- Nessun overflow orizzontale evidente.
- Nessuna immagine deformata.

## Responsive

Controlla almeno:

- ~375 px mobile;
- ~768 px tablet;
- ~1280 px desktop.

Verifica soprattutto:

- header/menu;
- hero;
- griglie;
- immagini;
- CTA;
- form;
- footer.

## Contenuti

- Se il sito di riferimento era una preview con wrapper o iframe, l'analisi è stata eseguita sulla pagina finale e l'URL effettivo è riportato nella consegna.
- Barre del marketplace, pulsanti di acquisto e controlli della preview non sono stati riprodotti nel nuovo sito.
- Un solo H1.
- H2/H3 in ordine logico.
- Nessuna sezione con testo duplicato senza motivo.
- Orari coerenti con la fonte, se presenti.
- Telefono, email, indirizzo e nome brand coerenti con la fonte o con i dati forniti dall'utente.
- Nessuna informazione fattuale inventata.

## Link e anchor

- Tutti gli anchor del menu puntano a ID esistenti.
- Nessun link `#` residuo salvo placeholder dichiarati.
- Il footer contiene sia il link **Privacy Policy** sia il link **Cookie Policy**.
- Gli URL di Privacy Policy e Cookie Policy sono reali oppure usano placeholder chiaramente segnalati.
- Link esterni `_blank` con `rel="noopener"`.

## Form

- Campi obbligatori marcati correttamente.
- Email validata lato client e server.
- Privacy obbligatoria.
- Honeypot presente.
- Anti-bot aggiuntivo attivo solo se configurato.
- `From` deriva dalla configurazione SMTP.
- `Reply-To` usa l'email validata del mittente.
- La risposta AJAX gestisce successo ed errore.
- Se mancano credenziali SMTP reali, non simulare un invio riuscito: segnala che il test reale resta da completare.

## SEO/accessibilità

- Title presente.
- Meta description presente.
- Viewport presente.
- `lang` corretto.
- Open Graph contiene almeno `og:title`, `og:description`, `og:type`, `og:url`, `og:image` e `og:image:alt`.
- Twitter Card contiene almeno `twitter:card` con valore `summary_large_image`, `twitter:title`, `twitter:description`, `twitter:image` e `twitter:image:alt`.
- `og:image` e `twitter:image` non sono vuoti e puntano a un file immagine realmente presente nel progetto.
- Sul sito pubblicato, `og:image`, `twitter:image` e `og:url` usano URL assoluti HTTPS e risultano raggiungibili.
- JSON-LD sintatticamente coerente e basato solo su dati reali.
- Alt text verificati.
- Focus visibile.
- Label form corrette.
- Contrasto testuale ragionevole.

## Placeholder/TODO

Cerca nel progetto:

```text
TODO
FIXME
example.com
example@
G-XXXX
#privacy-policy
#cookie-policy
YOUR_
CHANGE_ME
```

Ogni placeholder residuo deve essere rimosso oppure dichiarato esplicitamente nella consegna.
