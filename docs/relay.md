---
title: Relay open data
nav_order: 9
description: "api/opendata_relay.php: leggere dataset che il browser non può scaricare direttamente."
---

# Relay open data
{: .no_toc }

<details open markdown="block">
  <summary>Indice</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## A cosa serve

I dati dei servizi vengono scaricati **dal browser** del cittadino. Il browser però può leggere la risposta di un altro sito solo se questo lo autorizza con l'header CORS `Access-Control-Allow-Origin`.

Su `dati.toscana.it` le API CKAN rispondono con questo header, ma molti **file caricati nel FileStore** (i link `.../download/file.json`) no: il browser li scarica ma ne blocca la lettura. Non è un problema risolvibile nel browser: dipende dal server remoto.

Il file `api/opendata_relay.php` aggira il problema: il browser chiede il dato al relay, che si trova **sullo stesso dominio** del portale; il relay lo scarica dal portale open data lato server (dove CORS non esiste) e lo restituisce.

```text
browser ──(stesso dominio)──▶ api/opendata_relay.php ──(server-server)──▶ dati.toscana.it
```

## Configurazione

Le impostazioni sono costanti in cima al file `api/opendata_relay.php`.

### Host consentiti (obbligatorio)

```php
const ALLOWED_HOSTS = ['dati.toscana.it'];
```

Il relay inoltra richieste **solo** verso i domini di questo elenco. È la protezione contro l'uso improprio del relay per raggiungere altri siti o server della rete interna (*SSRF*): non rimuoverla.

{: .importante }
Nel repository l'elenco è `['']`: **il relay rifiuta tutte le richieste** con `403 host non consentito` finché non si aggiunge il dominio del portale open data.

### Proxy in uscita

Se il server esce su internet solo attraverso un proxy:

```php
const OUTBOUND_PROXY = 'http://proxy.intra:8080';
```

### Tempi

| Costante | Valore | Significato |
| :--- | :--- | :--- |
| `CONNECT_TIMEOUT` | 8 s | Tempo massimo per stabilire la connessione. |
| `TIMEOUT` | 15 s | Tempo massimo totale della richiesta. |
| `MAX_REDIRECTS` | 5 | Reindirizzamenti seguiti (i download CKAN spesso rimbalzano). |

## Uso in `services.json`

L'indirizzo originale va nel parametro `u`, **codificato**:

```json
"fetch": "api/opendata_relay.php?u=https%3A%2F%2Fdati.toscana.it%2Fdataset%2Fabc%2Fresource%2Fdef%2Fdownload%2Fpiano.json"
```

Per ottenere la forma codificata, nella console del browser:

```js
encodeURIComponent('https://dati.toscana.it/dataset/abc/resource/def/download/piano.json')
```

Si può anche indicare prima l'indirizzo diretto e poi il relay come riserva: se un giorno il portale open data abiliterà CORS, verrà usato direttamente senza cambiare nulla.

```json
"fetch": [
  "https://dati.toscana.it/dataset/abc/resource/def/download/piano.json",
  "api/opendata_relay.php?u=https%3A%2F%2Fdati.toscana.it%2Fdataset%2Fabc%2Fresource%2Fdef%2Fdownload%2Fpiano.json"
]
```

In questo caso aggiungi `https://dati.toscana.it` a `csp_connect_src_extra`, altrimenti il primo tentativo viene bloccato dalla CSP (senza danni, ma con un errore in console).

{: .nota }
Il commento in testa al file PHP indica il percorso `assets/api/opendata_relay.php`: nel repository il file è in `api/opendata_relay.php`, ed è questo il percorso da usare.

## Comportamento

- Accetta solo richieste `GET`.
- **Nessuna cache**: ogni richiesta interroga la fonte in tempo reale. Per ridurre le richieste si usa la cache del browser (`"cache": true` nel dataset).
- Verifica i certificati HTTPS della fonte.
- Restituisce il contenuto e il `Content-Type` della fonte (se assente, `application/json`).

In caso di errore risponde in JSON:

```json
{ "error": "host non consentito", "status": 403, "detail": "" }
```

| Codice | `error` | Causa |
| :--- | :--- | :--- |
| 400 | `parametro "u" mancante` | Manca `?u=`. |
| 403 | `host non consentito` | Dominio non in `ALLOWED_HOSTS`, schema diverso da http/https o indirizzo non valido. |
| 405 | `metodo non consentito` | Richiesta diversa da `GET`. |
| 502 | `opendata non raggiungibile` | Il server non riesce a contattare la fonte (rete, DNS, proxy, timeout). In `detail` il messaggio di curl. |
| 4xx/5xx | `errore dalla sorgente opendata` | La fonte ha risposto con un errore. |

## Verifica

```bash
curl -s "https://<dominio>/comune-in-chiaro/api/opendata_relay.php?u=https%3A%2F%2Fdati.toscana.it%2Fapi%2F3%2Faction%2Fsite_read"
```

Risposta attesa: `{"help": "...", "success": true, "result": true}`. Se ottieni `host non consentito`, controlla `ALLOWED_HOSTS`; se ottieni `opendata non raggiungibile`, controlla la connessione in uscita del server e l'eventuale proxy.
