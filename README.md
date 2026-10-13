# Singlepage Site Generator

TESTATO con gpt-6-astra

Guida rapida per installare e usare la skill con Codex su Windows.

## 1. Installazione della skill

1. Estrai il file `singlepage-site-generator.zip`.
2. Copia la cartella estratta `singlepage-site-generator` in:

   ```text
   C:\Users\NOME_UTENTE\.agents\skills\
   ```

3. Verifica che il file principale si trovi esattamente qui:

   ```text
   C:\Users\NOME_UTENTE\.agents\skills\singlepage-site-generator\SKILL.md
   ```

La struttura deve risultare così:

```text
C:\Users\NOME_UTENTE\.agents\skills\
└── singlepage-site-generator\
    ├── SKILL.md
    └── references\
        ├── design-analysis.md
        ├── form-and-security.md
        ├── qa-checklist.md
        └── seo-accessibility.md
```

Evita una doppia cartella come:

```text
singlepage-site-generator\singlepage-site-generator\SKILL.md
```

Codex rileva normalmente le skill in automatico. Nella CLI o nell’estensione IDE puoi controllare le skill disponibili con `/skills` oppure richiamare direttamente questa skill scrivendo `$singlepage-site-generator` nel prompt.

### Installazione valida solo per un progetto

Se non vuoi rendere la skill disponibile in tutti i progetti, puoi inserirla nella cartella del singolo progetto:

```text
C:\laragon\www\nome-progetto\.agents\skills\singlepage-site-generator\
```

Per un utilizzo abituale su più siti è consigliata l’installazione globale nella cartella dell’utente.

## 2. Preparazione del progetto locale

La skill genera un sito basato su `index.php`, quindi è consigliato lavorare tramite un server PHP locale. Su Windows puoi usare **Laragon**.

1. Crea una cartella per il progetto, per esempio:

   ```text
   C:\laragon\www\mio-sito\
   ```

2. Prepara una cartella con le immagini autorizzate del cliente, per esempio:

   ```text
   C:\laragon\www\mio-sito\immagini-sorgente\
   ```

3. Avvia Laragon e attiva Apache e PHP.
4. Apri il terminale nella cartella del progetto e avvia Codex.

Non aprire `index.php` con un doppio clic: PHP deve essere eseguito dal server locale.

## 3. Avvio della skill in Codex

Nel prompt di Codex scrivi:

```text
$singlepage-site-generator

Crea un nuovo sito singlepage. Prima di iniziare, chiedimi tutti gli input obbligatori mancanti.
```

La skill richiede quattro informazioni obbligatorie:

1. URL del sito da usare come riferimento grafico;
2. URL del sito da cui recuperare i contenuti;
3. percorso locale della cartella immagini;
4. colori o palette del nuovo sito.

Puoi fornire tutto in un unico prompt:

```text
$singlepage-site-generator

Crea il sito nella cartella corrente.

- Riferimento grafico: https://demo.templatemonster.com/demo/494330.html
- Fonte contenuti: https://www.esempio-cliente.it
- Immagini: C:\laragon\www\mio-sito\immagini-sorgente
- Palette: verde salvia #7A8F72, beige #F3EEE4 e antracite #252525
- Server di destinazione: Apache
- Lingua: italiano
```

Se il riferimento è una preview di TemplateMonster, ThemeForest o Envato, la skill deve individuare e analizzare il sito reale caricato nell’iframe, non l’interfaccia del marketplace.

## 4. File generati

Il progetto utilizza normalmente questa struttura:

```text
mio-sito\
├── index.php
├── .htaccess
├── config\
│   └── config.php
├── css\
│   └── style.css
├── js\
│   └── script.js
├── php\
│   ├── form-handler.php
│   └── PHPMailer\
└── img\
```

La skill mantiene leggibili e non minificati HTML, PHP, CSS e JavaScript. Inserisce inoltre:

- link separati a Privacy Policy e Cookie Policy nel footer;
- metadati Open Graph e Twitter Card;
- un’immagine social scelta automaticamente tra quelle disponibili, salvo indicazione diversa;
- `.htaccess` con predisposizione HTTPS per Apache o LiteSpeed;
- form di contatto PHP con PHPMailer e protezioni anti-spam.

Su Nginx il file `.htaccess` non viene usato: il redirect HTTPS deve essere configurato nel virtual host o nel pannello del server.

## 5. Test con Laragon

Apri il sito tramite uno di questi indirizzi, in base alla configurazione di Laragon:

```text
http://mio-sito.test
```

oppure:

```text
http://localhost/mio-sito
```

Controlla almeno:

- visualizzazione desktop e mobile;
- menu mobile e pulsanti;
- testi, immagini e contatti;
- link Privacy Policy e Cookie Policy;
- assenza di errori nella console del browser;
- invio del form, dopo aver configurato SMTP e protezione anti-bot.

In locale il redirect HTTPS può non essere attivo ed è normale. Il certificato SSL e il redirect definitivo devono essere verificati dopo la pubblicazione sul server reale.

## 6. Protezione anti-bot del form

Il form di contatto ha sempre un **honeypot** (campo nascosto che i bot compilano). In aggiunta puoi scegliere un servizio anti-bot indicandolo nel prompt:

| Opzione | Servizio | Come funziona | Quando sceglierla |
|---------|----------|---------------|-------------------|
| `recaptcha` | Google reCAPTCHA v3 | Invisibile, assegna un punteggio da 0.0 (bot) a 1.0 (umano) | Se il cliente usa già servizi Google |
| `turnstile` | Cloudflare Turnstile | Widget leggero, nessun puzzle per l'utente | Alternativa più rispettosa della privacy, gratuita |
| `none` | Solo honeypot | Nessuno script di terze parti | Default, se non scegli nulla |

Esempio di prompt:

```text
$singlepage-site-generator

...
- Anti-bot: Cloudflare Turnstile
```

### Ottenere le chiavi

- **reCAPTCHA v3:** dalla [console reCAPTCHA](https://www.google.com/recaptcha/admin) crea un sito di tipo *reCAPTCHA v3* e aggiungi i domini (anche `localhost` o `nome-sito.test` per i test).
- **Turnstile:** dalla dashboard Cloudflare, sezione *Turnstile*, aggiungi un sito con il dominio. Non serve che il dominio usi Cloudflare come DNS.

Ottieni sempre due chiavi: la **site key** (pubblica, usata nella pagina) e la **secret key** (privata, solo server).

### Configurazione in `config/config.php`

```php
'anti_bot' => [
    'provider'   => 'turnstile',   // 'recaptcha' | 'turnstile' | 'none'
    'site_key'   => 'LA_TUA_SITE_KEY',
    'secret_key' => 'LA_TUA_SECRET_KEY',
    'min_score'  => 0.5,           // solo reCAPTCHA v3
    'action'     => 'contact',     // nome azione, uguale tra frontend e server
    'theme'      => 'auto',        // solo Turnstile: auto | light | dark
],
```

| Chiave | Descrizione |
|--------|-------------|
| `provider` | Servizio attivo. Per cambiarlo basta modificare questo valore: viene caricato solo lo script del provider scelto. |
| `site_key` | Chiave pubblica, inserita nella pagina. |
| `secret_key` | Chiave privata, usata solo da `form-handler.php`. Non va mai pubblicata. |
| `min_score` | Soglia reCAPTCHA v3. Sotto questo punteggio l'invio è rifiutato. 0.5 è un buon punto di partenza; alzala se arriva ancora spam. |
| `action` | Etichetta dell'azione, controllata anche in verifica. |
| `theme` | Aspetto del widget Turnstile. |

### Comportamento

- Il token è verificato **lato server** prima dell'invio dell'email; il controllo nel browser da solo non basta.
- Se le chiavi mancano o il servizio non risponde, il form **rifiuta l'invio** (nessuna email parte) e mostra un errore generico.
- Il token viene rigenerato a ogni tentativo di invio.

### Test in locale

Per Turnstile puoi usare le chiavi di test Cloudflare: site key `1x00000000000000000000AA`, secret `1x0000000000000000000000000000000AA` (passa sempre). Per reCAPTCHA servono chiavi reali con `localhost` o il dominio `.test` tra i domini consentiti. **Sostituisci le chiavi di test con quelle reali prima della pubblicazione.**

### Privacy

Entrambi i servizi sono di terze parti: aggiorna Privacy Policy e Cookie Policy indicando quello scelto. Con reCAPTCHA il badge Google deve restare visibile oppure va aggiunto il testo di attribuzione richiesto da Google vicino al form. I testi legali non sono generati dalla skill.

## 7. Pubblicazione sul server

1. Carica tutti i file generati, incluso `.htaccess` se il server usa Apache o LiteSpeed.
2. Configura dominio, email, SMTP ed eventuali chiavi anti-bot (vedi sezione 6) in `config/config.php`.
3. Installa o attiva il certificato SSL dal pannello hosting.
4. Verifica il passaggio da HTTP a HTTPS senza loop.
5. Controlla che canonical, Open Graph e Twitter usino URL HTTPS assoluti.
6. Prova nuovamente il form dal sito online.

Non pubblicare credenziali SMTP o chiavi private in repository pubblici.

## Riferimento Codex

Documentazione ufficiale: [Build skills in Codex](https://developers.openai.com/codex/skills)
