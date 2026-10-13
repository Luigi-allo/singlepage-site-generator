# Form, validazione e sicurezza

## Campi standard

- Nome
- Cognome
- Email
- Telefono
- Messaggio
- Checkbox obbligatoria “Ho letto l'informativa privacy”
- Honeypot

Usa `href="#privacy-policy"` come placeholder solo se l'URL reale non è stato fornito e segnala il placeholder nella consegna.

## Metodo e limiti

- Accetta solo richieste `POST`.
- Imposta limiti di lunghezza ragionevoli lato server.
- Rifiuta richieste malformate.
- Non costruire header email direttamente da input non validati.
- Non esporre email admin, credenziali SMTP o secret key nel frontend.

## Validazione PHP

Preferisci validazione esplicita invece di una generica “sanitizzazione di tutto”.

Esempio di principio:

```php
$name    = trim($_POST['nome'] ?? '');
$surname = trim($_POST['cognome'] ?? '');
$email   = trim($_POST['email'] ?? '');
$phone   = trim($_POST['telefono'] ?? '');
$message = trim($_POST['messaggio'] ?? '');

if ($name === '' || $surname === '' || $message === '') {
    // errore
}

if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
    // errore
}
```

Valida lunghezze e formati. Esegui escaping al momento dell'output nel contesto corretto, ad esempio `htmlspecialchars()` per il corpo HTML dell'email.

## Privacy

La checkbox base conferma la presa visione dell'informativa privacy.

Non aggiungere automaticamente consensi marketing, profilazione o newsletter. Quando richiesti:

- separali dalla presa visione privacy;
- non preselezionarli;
- rendili obbligatori solo se giuridicamente e funzionalmente necessari al caso specifico.

Non inventare testi legali completi.

## Honeypot

Attivo di default.

- Campo fuori dal flusso visivo tramite tecnica accessibile/off-screen.
- Non usare solo `display:none` come unica strategia.
- Se valorizzato, termina l'elaborazione senza inviare l'email.

## Anti-bot aggiuntivo

Opzionale e configurato esclusivamente da `config.php` (sezione `anti_bot`). L'honeypot resta sempre attivo, indipendentemente dal provider scelto.

### Scelta del provider

Se l'utente non ha indicato il provider, chiedi quale usare tra le tre opzioni; se non risponde, usa `none` (solo honeypot) e segnalalo nella consegna.

| `provider`   | Servizio              | Tipo di sfida                              | Note |
|--------------|-----------------------|--------------------------------------------|------|
| `recaptcha`  | Google reCAPTCHA v3   | Invisibile, punteggio 0.0–1.0              | Richiede `min_score`; mostra il badge Google (vedi sotto). |
| `turnstile`  | Cloudflare Turnstile  | Widget (managed/invisible), esito pass/fail | Nessun punteggio; `min_score` ignorato. Alternativa più rispettosa della privacy. |
| `none`       | Solo honeypot         | Nessuna                                    | Default. Nessuno script di terze parti. |

Non inserire mai chiavi inventate: se `site_key` o `secret_key` sono vuote, lascia i placeholder, segnalali nella consegna e trattalo come configurazione incompleta (vedi "Fail closed").

### Schema `anti_bot` in `config.php`

```php
'anti_bot' => [
    'provider'   => 'none',      // 'recaptcha' | 'turnstile' | 'none'
    'site_key'   => '',          // chiave pubblica (frontend)
    'secret_key' => '',          // chiave privata (solo server)
    'min_score'  => 0.5,         // solo reCAPTCHA v3: 0.0 (bot) – 1.0 (umano)
    'action'     => 'contact',   // nome azione; deve coincidere tra frontend e verifica
    'theme'      => 'auto',      // solo Turnstile: 'auto' | 'light' | 'dark'
],
```

Regole comuni:

- `index.php` legge `provider` e `site_key` dalla config; `secret_key` non deve mai comparire nell'HTML, nel JS o nelle risposte JSON.
- Carica lo script del provider **solo** se selezionato. Con `none` non deve essere caricato nulla di esterno.
- La verifica avviene sempre lato server in `form-handler.php`, prima di qualsiasi invio email. La sola validazione client non è sufficiente.
- Il token viene inviato nel POST del form insieme agli altri campi (campo nascosto `anti_bot_token`).
- Timeout breve (5–10 s) sulla chiamata di verifica; invia anche `remoteip` (`$_SERVER['REMOTE_ADDR']`) se disponibile.
- Per la chiamata HTTP usa cURL (o `file_get_contents` con stream context) senza librerie esterne.
- **Fail closed:** se la verifica fallisce, il provider è irraggiungibile, la secret è mancante o la risposta non è valida, rifiuta la richiesta (HTTP 400/403 con JSON generico) e non inviare l'email. Non esporre al client i codici errore del provider (`error-codes`).
- Segnala in consegna di aggiornare l'informativa privacy e la cookie policy con il servizio terzo scelto; non generare tu i testi legali.

### reCAPTCHA v3

Frontend (`index.php`, solo se `provider === 'recaptcha'`):

```php
<?php if ($config['anti_bot']['provider'] === 'recaptcha'): ?>
<script src="https://www.google.com/recaptcha/api.js?render=<?= htmlspecialchars($config['anti_bot']['site_key'], ENT_QUOTES) ?>" async defer></script>
<?php endif; ?>
```

Il form contiene `<input type="hidden" name="anti_bot_token" id="anti_bot_token">`. In `script.js`, al submit e prima del `fetch()`:

```js
grecaptcha.ready(function () {
    grecaptcha.execute(SITE_KEY, { action: 'contact' }).then(function (token) {
        form.querySelector('#anti_bot_token').value = token;
        // ... fetch() del form
    });
});
```

Passa `SITE_KEY` e `action` al JS tramite attributi `data-*` sul form generati da `index.php` (non hardcodare la chiave nel JS). Il token scade dopo circa 2 minuti: generalo al momento del submit, non al caricamento pagina, e rigeneralo a ogni nuovo tentativo.

Verifica server (`https://www.google.com/recaptcha/api/siteverify`, POST con `secret`, `response`, `remoteip`):

- `success === true`;
- `score >= min_score`;
- `action` della risposta uguale a `anti_bot.action`;
- `hostname` coerente con il dominio del sito (quando il dominio è noto).

Il badge reCAPTCHA non va nascosto con CSS a meno che sia presente nel form il testo sostitutivo richiesto da Google ("Questo sito è protetto da reCAPTCHA e si applicano le Norme sulla privacy e i Termini di servizio di Google"), con i relativi link.

### Cloudflare Turnstile

Frontend (`index.php`, solo se `provider === 'turnstile'`):

```php
<?php if ($config['anti_bot']['provider'] === 'turnstile'): ?>
<script src="https://challenges.cloudflare.com/turnstile/v0/api.js" async defer></script>
<?php endif; ?>
```

Nel form, prima del pulsante di invio:

```php
<div class="cf-turnstile"
     data-sitekey="<?= htmlspecialchars($config['anti_bot']['site_key'], ENT_QUOTES) ?>"
     data-action="<?= htmlspecialchars($config['anti_bot']['action'], ENT_QUOTES) ?>"
     data-theme="<?= htmlspecialchars($config['anti_bot']['theme'], ENT_QUOTES) ?>"></div>
```

Turnstile inserisce automaticamente un campo `cf-turnstile-response` nel form: `form-handler.php` legge quel campo (con `anti_bot_token` come fallback se si usa il rendering esplicito). Dopo ogni invio, successo o errore, chiama `turnstile.reset()` perché il token è monouso e scade dopo 300 s.

Verifica server (`https://challenges.cloudflare.com/turnstile/v0/siteverify`, POST con `secret`, `response`, `remoteip`):

- `success === true`;
- `action` della risposta uguale a `anti_bot.action` (se restituita);
- `hostname` coerente con il dominio del sito (quando il dominio è noto).

Il widget deve avere spazio riservato nel layout (altezza minima) per evitare layout shift ed essere raggiungibile da tastiera.

### Chiavi di test (solo sviluppo locale)

Per provare il flusso in locale senza chiavi reali:

- **Turnstile:** site key `1x00000000000000000000AA` (passa sempre) e secret `1x0000000000000000000000000000000AA`; per forzare l'errore `2x00000000000000000000AB` / `2x0000000000000000000000000000000AA`.
- **reCAPTCHA v3:** non esistono chiavi di test ufficiali; crea chiavi reali nella console Google con `localhost` (o il dominio `.test` di Laragon) tra i domini consentiti.

Le chiavi di test non devono restare nella configurazione consegnata: sostituiscile con le chiavi reali o segnala che mancano.

### Messaggi di errore

Il client riceve solo messaggi generici, ad esempio "Verifica anti-spam non superata, riprova." Dettagli (score, codici errore) possono essere scritti in `error_log()` lato server senza dati personali dell'utente.

## PHPMailer

PHPMailer è obbligatorio per l'invio; non usare `mail()`.

Struttura prevista:

```text
/php/PHPMailer/src/PHPMailer.php
/php/PHPMailer/src/SMTP.php
/php/PHPMailer/src/Exception.php
```

Non riscrivere o simulare il codice della libreria. Usa una copia PHPMailer autorizzata/approvata per il progetto.

Leggi SMTP da `config.php`:

- host;
- user;
- pass;
- port;
- encryption;
- from_email;
- from_name.

`From` deve usare l'indirizzo configurato/autorizzato per il dominio. `Reply-To` usa l'email validata del mittente del form.

## Email amministratore

Oggetto:

```text
Richiesta di contatto da [Nome] [Cognome]
```

Corpo HTML semplice, con stili inline:

```text
Nuova richiesta di contatto dal sito

Nome:      [Nome]
Cognome:   [Cognome]
Email:     [Email]
Telefono:  [Telefono]

Messaggio:
[Messaggio]

---
Inviato automaticamente dal form di contatto di [nome dominio/brand]
```

Esegui escaping di tutti i valori prima di inserirli nel markup HTML.

## Risposta AJAX

Il frontend usa `fetch()` e mostra un messaggio chiaro di successo/errore senza reload.

Il server restituisce JSON coerente e status HTTP appropriati.

Non restituire dettagli tecnici, stack trace, credenziali o errori SMTP completi all'utente finale.
