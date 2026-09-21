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

Opzionale e configurato da `config.php`.

### reCAPTCHA v3

- Carica lo script frontend solo se `provider === 'recaptcha'`.
- Genera il token prima del submit.
- Verifica il token lato server tramite endpoint ufficiale.
- Controlla almeno esito e score rispetto a `min_score`.
- Non considerare sufficiente la sola validazione client.

### Cloudflare Turnstile

- Carica lo script solo se `provider === 'turnstile'`.
- Verifica il token lato server tramite endpoint ufficiale.
- Non considerare sufficiente la sola presenza del token nel POST.

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
