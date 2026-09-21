# SEO on-page e accessibilità

## SEO obbligatoria

- `<title>` univoco e descrittivo, indicativamente 50–60 caratteri.
- `<meta name="description">`, indicativamente 140–160 caratteri.
- Un solo `<h1>`.
- Gerarchia H2/H3 coerente.
- `<meta name="viewport" content="width=device-width, initial-scale=1">`.
- Attributo `lang` corretto su `<html>`.
- Open Graph: `og:title`, `og:description`, `og:type`, `og:url`, `og:image` e `og:image:alt`.
- Twitter Card: `twitter:card` con valore `summary_large_image`, `twitter:title`, `twitter:description`, `twitter:image` e `twitter:image:alt`.
- JSON-LD `LocalBusiness` o `Organization` in base al contesto reale.
- Alt text descrittivo per immagini informative.
- `loading="lazy"` per immagini non above-the-fold.
- Non usare lazy-load sulla principale immagine hero/LCP se ciò ne ritarda il caricamento.
- Link `target="_blank"` con `rel="noopener"`.

Non inventare coordinate, P.IVA, social, recensioni, rating, prezzi o altri dati strutturati non presenti nelle fonti autorizzate.

## Immagini social Open Graph e Twitter

- Non omettere `og:image` o `twitter:image` e non lasciarli vuoti.
- Usa un'immagine reale, autorizzata e presente nella cartella `/img` del progetto; privilegia un'immagine social dedicata oppure, in mancanza, l'immagine hero più rappresentativa.
- Open Graph e Twitter possono usare lo stesso file immagine.
- Preferisci un'immagine orizzontale da circa `1200 × 630 px` quando disponibile, senza deformarla.
- Aggiungi un testo alternativo coerente tramite `og:image:alt` e `twitter:image:alt`.
- Sul sito pubblicato, `og:image`, `twitter:image` e `og:url` devono usare URL assoluti HTTPS. Costruisci gli URL dal dominio configurato in `config.php`; se il dominio finale non è ancora disponibile, mantieni il riferimento all'immagine ma segnala chiaramente il dominio come dato da completare.
- Verifica che il file indicato esista davvero e sia raggiungibile; non usare placeholder generici, URL inventati o immagini mancanti.

## Canonical e indicizzazione

Se il dominio finale è noto, inserisci canonical coerente.

Se il progetto è ancora su dominio temporaneo/staging, non inventare il dominio finale. Segnala il canonical come dato da completare oppure usa il comportamento concordato con l'utente.

Non aggiungere `noindex` al sito finale senza richiesta esplicita.

## Accessibilità

- Contrasto testo/sfondo adeguato almeno a WCAG AA quando ragionevolmente verificabile.
- Focus state visibile per link, bottoni e campi.
- Navigazione da tastiera.
- `label for` / `id` corretti nei form.
- Checkbox privacy con label cliccabile.
- Non affidare informazioni solo al colore.
- Usa elementi semantici (`header`, `nav`, `main`, `section`, `footer`) quando appropriati.
- Per menu mobile usa attributi ARIA coerenti, ad esempio `aria-expanded` e `aria-controls`.
- Mantieni ordine DOM logico.
- Immagini puramente decorative possono usare `alt=""`; immagini informative devono avere alt descrittivo.

## Motion

Le animazioni reveal devono rispettare:

```css
@media (prefers-reduced-motion: reduce) {
  /* rimuovi transizioni/trasformazioni non essenziali */
}
```

Il contenuto non deve dipendere dall'animazione per diventare accessibile o leggibile.
