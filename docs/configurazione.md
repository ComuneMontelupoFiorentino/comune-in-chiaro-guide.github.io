---
title: Configurazione
nav_order: 4
description: "Tutti i parametri di config/config.php e come personalizzare il portale per un altro ente."
---

# Configurazione
{: .no_toc }

<details open markdown="block">
  <summary>Indice</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Il file `config/config.php`

Tutte le impostazioni specifiche dell'installazione sono nel file `config/config.php`, che restituisce un array PHP:

```php
<?php
declare(strict_types=1);

return [
    'base_url'              => '/comune-in-chiaro/',
    'name_service'          => 'Comune in Chiaro',
    'name_service_sh'       => 'In Chiaro',
    'desc_service'          => 'Gli open data comunali, in chiaro',
    'version_service'       => '1.2.0 (beta)',
    'ente'                  => 'Comune di Montelupo Fiorentino',

    'embed_parent_origin'   => 'https://www.comune.montelupo-fiorentino.fi.it',
    'csp_img_src_extra'     => 'https://www.comune.montelupo-fiorentino.fi.it',
    'csp_connect_src_extra' => 'https://dati.toscana.it',
];
```

{: .importante }
Il portale **non ha valori predefiniti**: se il file manca, non restituisce un array o manca una chiave obbligatoria, ogni pagina risponde con errore **500** e un messaggio breve ("Configurazione mancante", "Configurazione non valida" o "Configurazione incompleta"). Il nome della chiave mancante è scritto nel log degli errori di PHP. È una scelta voluta (*fail-closed*): meglio fermarsi subito in fase di installazione che funzionare con valori sbagliati.

## Parametri

| Chiave | Obbligatoria | Descrizione | Esempio |
| :--- | :---: | :--- | :--- |
| `base_url` | sì | Percorso del portale sul dominio, **con `/` iniziale e finale**. Usato per costruire i link e i percorsi di CSS, JS e immagini. | `/comune-in-chiaro/` |
| `name_service` | sì | Nome del portale: titolo della pagina, testata, piè di pagina. | `Comune in Chiaro` |
| `name_service_sh` | sì | Nome breve, mostrato al posto del nome completo su schermi piccoli. | `In Chiaro` |
| `desc_service` | sì | Sottotitolo in testata e nel piè di pagina. | `Gli open data comunali, in chiaro` |
| `version_service` | sì | Versione mostrata nella pagina "Cos'è". | `1.2.0 (beta)` |
| `ente` | sì | Nome dell'ente: titolo della pagina, testata, copyright. | `Comune di Montelupo Fiorentino` |
| `embed_parent_origin` | sì | Sito autorizzato a [incorporare i servizi](embed) in un `iframe`. Va in `frame-ancestors` della CSP. | `https://www.comune.esempio.it` |
| `csp_img_src_extra` | sì | Domini aggiuntivi da cui caricare immagini (oltre al portale stesso e alle mappe OpenStreetMap). Più domini separati da spazio. | `https://www.comune.esempio.it` |
| `csp_connect_src_extra` | no | Domini aggiuntivi che il browser può interrogare con `fetch()` (per esempio il portale open data, se i dataset vengono letti direttamente). Più domini separati da spazio. Può essere vuoto. | `https://dati.toscana.it` |

{: .attenzione }
`csp_img_src_extra` è obbligatoria e **non può essere una stringa vuota**. Se non ti servono domini aggiuntivi, indica comunque il dominio del portale stesso.

### Come scrivere le origini

Le chiavi `embed_parent_origin`, `csp_img_src_extra` e `csp_connect_src_extra` finiscono dentro l'header `Content-Security-Policy`. Conviene scriverle come **origini**: schema + dominio (+ porta), **senza percorso** e senza `/` finale.

| Corretto | Da evitare |
| :--- | :--- |
| `https://www.comune.esempio.it` | `https://www.comune.esempio.it/pagina/` |
| `https://dati.toscana.it https://geoportale.esempio.it` | `dati.toscana.it, geoportale.esempio.it` (niente virgole) |

### Sovrascrivere l'origine dell'embed in locale

Per provare l'incorporamento dal proprio computer, senza toccare `config.php`, si può impostare la variabile d'ambiente `EMBED_PARENT_ORIGIN`, che ha la precedenza sul valore del file:

```apache
# nel virtual host locale di Apache
SetEnv EMBED_PARENT_ORIGIN "http://localhost:8080"
```

```bash
# con il server integrato di PHP
EMBED_PARENT_ORIGIN="http://localhost:8080" php -S 127.0.0.1:8000 _router.php
```

## Scegliere tra lettura diretta e relay

I dati dei servizi vengono scaricati **dal browser del cittadino**. Per ogni dataset ci sono due strade:

| | Lettura diretta | Tramite relay |
| :--- | :--- | :--- |
| Come | Il browser chiama direttamente `https://dati.toscana.it/...` | Il browser chiama `api/opendata_relay.php`, che scarica il dato lato server |
| Quando funziona | Solo se il portale open data risponde con l'header CORS `Access-Control-Allow-Origin` | Sempre (il relay è sullo stesso dominio del portale) |
| Cosa configurare | Aggiungere il dominio in `csp_connect_src_extra` | Configurare `ALLOWED_HOSTS` nel relay. Non serve toccare la CSP |

Le API CKAN (`/api/3/action/...`) di norma consentono la lettura diretta; i file caricati nel FileStore (`/dataset/.../download/file.json`) spesso no, e vanno letti tramite relay. Dettagli nella pagina [Relay open data](relay).

## Personalizzare per un altro ente

Oltre a `config.php`, alcuni riferimenti al Comune di Montelupo Fiorentino sono scritti direttamente nei template. Chi riusa il portale deve modificarli a mano:

| File | Cosa cambiare |
| :--- | :--- |
| `views/layout/header.php` | Dominio delle favicon (`https://www.comune.montelupo-fiorentino.fi.it...`) e link al sito dell'ente nella barra in alto e nel menu mobile. |
| `views/layout/footer.php` | Indirizzo, codice fiscale, PEC, link all'ufficio, link al logo, dichiarazione di accessibilità, privacy policy. |
| `views/partials/segnalazioni.php` e `segnalazioni_obj.php` | Indirizzo del modulo "Segnala un disservizio". |
| `views/partials/notice_dataset.php` | Link al portale open data dell'ente. |
| `views/main/about.php` | Testi della pagina "Cos'è" e link al portale open data. |
| `assets/dist/logo.png` e `assets/dist/favicon/` | Logo e icone dell'ente. |
| `assets/dist/css/comune-in-chiaro.css` | Colori e stili personalizzati. |

{: .suggerimento }
Se le favicon restano sul dominio del Comune di Montelupo, il browser le blocca comunque perché quel dominio non è in `csp_img_src_extra`: nella console compaiono errori *"Refused to load the image"*. Portare le favicon in `assets/dist/favicon/` e usare percorsi relativi risolve il problema.
