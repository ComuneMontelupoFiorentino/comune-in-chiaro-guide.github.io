---
title: Risoluzione dei problemi
nav_order: 12
description: "Problemi frequenti, come diagnosticarli, e punti aperti nel codice."
---

# Risoluzione dei problemi
{: .no_toc }

<details open markdown="block">
  <summary>Indice</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Dove guardare

1. **Console del browser** (F12 → *Console*): messaggi di `OpenData`, errori CSP (*"Refused to…"*), errori JavaScript.
2. **Scheda Rete** (F12 → *Network*): quali richieste falliscono e con che codice.
3. **Log degli errori di PHP** del server: messaggi `[ERROR]` e `[SECURITY]` (vedi [Sicurezza](sicurezza#log-di-sicurezza)).

## Problemi frequenti

### Ogni pagina mostra "Configurazione mancante / non valida / incompleta"

`config/config.php` manca, non restituisce un array, oppure manca una chiave obbligatoria (o è vuota). Il log PHP indica quale: `config/config.php: chiave mancante o non valida [csp_img_src_extra]`. Vedi [Configurazione](configurazione#parametri).

### La pagina appare senza grafica

I CSS non vengono caricati. Controlla che `base_url` corrisponda al percorso reale, con `/` iniziale e finale. Se usi il server integrato di PHP, serve il router descritto in [Prova in locale](installazione#prova-in-locale).

### In home compare "Impossibile caricare i servizi."

`services.json` non è raggiungibile o non è JSON valido. Aprilo direttamente nel browser (`/comune-in-chiaro/assets/data/services.json`) e controllalo con un validatore.

### In home compare "Nessun servizio disponibile."

`services.json` è valido ma è un elenco vuoto `[]`.

### "Accedi al servizio" porta alla pagina di errore

Il router accetta solo servizi per cui esiste `views/services/services/<id>.php`. Controlla che:

- il file esista e abbia esattamente il nome dell'`id` (minuscolo, estensione `.php`);
- il `link` in `services.json` usi lo stesso `id` (`?service=<id>`).

Nel log PHP comparirà `[SECURITY] service non in whitelist [...]`.

### Il servizio si apre ma titolo e sottotitolo restano vuoti

ServiceHead non trova la voce in `services.json`: l'`id` non coincide con il valore di `?service=`, oppure `services.json` non è stato caricato (vedi sopra).

### Compare "Servizio opendata non disponibile al momento"

Nessuno degli indirizzi `fetch` del **primo** dataset ha risposto. In console trovi il motivo accanto a `FALLITO`:

| In console | Causa probabile | Soluzione |
| :--- | :--- | :--- |
| `Refused to connect to '…' because it violates … connect-src` | Il dominio non è in `csp_connect_src_extra`. | Aggiungilo, oppure usa il relay. |
| `… has been blocked by CORS policy: No 'Access-Control-Allow-Origin'` | La fonte non consente la lettura dal browser. | Usa il [relay](relay). |
| `HTTP 403 (…opendata_relay.php…)` | Host non in `ALLOWED_HOSTS` del relay. | Aggiungi `dati.toscana.it` in `ALLOWED_HOSTS`. |
| `HTTP 502 (…opendata_relay.php…)` | Il server non raggiunge la fonte. | Verifica connessione in uscita e `OUTBOUND_PROXY`. |
| `HTTP 404 (…)` | Indirizzo del dataset cambiato o errato. | Aggiorna `fetch` in `services.json`. |
| `The user aborted a request` | Tempo scaduto (12 s per tentativo). | Fonte lenta: aumenta `timeout` con `OpenData.configure`. |

### I dati arrivano ma l'elenco è vuoto

- Un filtro esclude tutto: ricorda che l'operatore `=` **distingue maiuscole e minuscole**; per i valori digitati o dall'indirizzo usa `contains`.
- Dati raggruppati con `recordsPath` ma senza `groupKeyField`: i gruppi risultano vuoti.
- La risposta non è un elenco né un GeoJSON (per esempio una risposta API CKAN): i filtri non si applicano e il dato va letto nel codice (`dati.result.records`).

### Lo script del servizio dà `OpenData is not defined`

Il codice non è dentro `document.addEventListener('DOMContentLoaded', …)`. Vedi [Creare un servizio → Scrivi il JavaScript](creare-servizio).

### Un pulsante con `onclick` non funziona

La CSP blocca i gestori di eventi in linea. In console: *"Refused to execute inline event handler"*. Sposta il codice in un file JS e usa `addEventListener`.

### L'iframe incorporato resta vuoto o dice "rifiutato"

Controlla, nell'ordine: `"allowEmbed": true` nel servizio, `embed=true` nell'indirizzo dell'`iframe`, `embed_parent_origin` uguale all'origine del sito ospitante (schema e dominio esatti, senza percorso). Vedi [Incorporare un servizio](embed).

### In console: "Refused to load the image … favicon"

Le favicon puntano al dominio del Comune di Montelupo Fiorentino, non incluso in `csp_img_src_extra`. Vedi [Personalizzare per un altro ente](configurazione#personalizzare-per-un-altro-ente).

### Logo non visibile

Manca `assets/dist/logo.png`: il file non è incluso nel repository.

## Punti aperti nel codice

Durante la stesura di questa guida sono emersi alcuni aspetti del codice (versione del 24 settembre 2026) che chi installa o sviluppa il portale deve conoscere. Sono elencati qui in attesa di essere corretti.

| # | Punto | Effetto | Correzione possibile |
| :---: | :--- | :--- | :--- |
| 1 | `footer.php` carica `assets/dist/js/services/services/embed.js`, ma il file è in `assets/dist/js/embed.js`. | In modalità embed la pagina non viene ridotta ai soli elementi `embed-visible`. | Spostare il file o correggere il percorso. |
| 2 | `footer.php` carica `assets/dist/js/services/sheets/copy-obj-id.js`, che non è nel repository; `segnalazioni_obj.php` contiene uno `<script>` in linea e un `onclick`, bloccati dalla CSP; il campo del codice ha il valore fisso `ABC123456`. | Nelle schede il pulsante di copia del codice oggetto non funziona. | Aggiungere `copy-obj-id.js` con la funzione di copia e la valorizzazione del codice, togliendo lo script in linea. |
| 3 | Nelle pagine generali (`home`, `about`) la variabile `$folder` non è definita. | Un *PHP Warning: Undefined variable $folder* nel log a ogni visita. | Inizializzare `$folder = ''` in `index.php`. |
| 4 | Nel relay `ALLOWED_HOSTS = ['']`; il commento indica il percorso `assets/api/`. | Il relay rifiuta tutte le richieste finché non viene configurato. | Documentato in [Relay](relay); eventualmente preimpostare `dati.toscana.it`. |
| 5 | `Permissions-Policy` contiene `camera=()`. | La scansione dei QR con `scan=true` non può accedere alla fotocamera. | `camera=(self)` se la funzione serve. |
| 6 | La home mostra tutte le voci di `services.json`, anche quelle delle schede. | Le schede compaiono in elenco come servizi. | Filtrare per `type` in `renderers.js`. |
| 7 | Favicon, link dell'ente, contatti e modulo di segnalazione sono scritti nei template. | Il riuso da parte di altri enti richiede modifiche ai template. | Portare questi valori in `config.php`. |
| 8 | `logo.png` e la cartella `favicon/` non sono nel repository. | Logo e icone assenti in una nuova installazione. | Aggiungere file di esempio. |
| 9 | La cartella `data/` non è protetta. | `data/stats.jsonl` è pubblicamente scaricabile. | Aggiungere un `.htaccess` o spostarla fuori dalla cartella pubblica. |
| 10 | `sort` e `limit` dei dataset sono segnaposto; `icona` e `lastUpdate` non sono usati. | Questi campi di `services.json` non hanno effetto. | Previsti nelle prossime iterazioni del codice. |
| 11 | In `segnalazioni.php` l'attributo è scritto `target= “_blank”` (virgolette tipografiche). | Il link si apre in una finestra con nome `“_blank”` invece che in una nuova scheda ogni volta. | Scrivere `target="_blank" rel="noopener"`. |
| 12 | `header.php` include `bootstrap-italia.min.css.map` come foglio di stile. | Una richiesta inutile a ogni pagina. | Rimuovere la riga. |
| 13 | Il `services.json` di esempio contiene due volte la chiave `"type"`. | Nessuno (vale l'ultima), ma può confondere. | Rimuovere il duplicato. |
