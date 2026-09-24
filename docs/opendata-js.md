---
title: Libreria OpenData
nav_order: 7
description: "Riferimento dell'API JavaScript OpenData per leggere i dataset dal codice dei servizi."
---

# Libreria `OpenData`
{: .no_toc }

L'oggetto globale che il codice dei servizi usa per ottenere i dati. Si chiede un dataset per **alias**, senza conoscerne l'indirizzo: tentativi, cache, normalizzazione del formato e filtri li gestisce la libreria.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Indice</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Dove si trova

`OpenData` è definito in `assets/dist/js/loader/service-renderer.js`, caricato in tutte le pagine di servizi e schede. Lo stesso file contiene **ServiceHead**, che riempie da `services.json` la testata del servizio (titolo, sottotitolo, avviso, descrizione, elenco dataset).

{: .importante }
Usa `OpenData` solo dentro `document.addEventListener('DOMContentLoaded', ...)`: il file è caricato con `defer` e non esiste ancora quando viene eseguito il JavaScript del servizio.

## Uso tipico

```js
document.addEventListener('DOMContentLoaded', async function () {
  await OpenData.loadService('cerca-manutenzioni');   // legge services.json e registra i dataset
  const plan = await OpenData.getData('plan');        // dati già normalizzati e filtrati
  console.log(OpenData.getInfo('plan'));              // { elapsedMs, recordCount, source, timestamp }
});
```

## Funzioni

### `OpenData.loadService(serviceId)`

Scarica `services.json` (una sola volta per pagina), trova il servizio e **registra tutti i suoi dataset**. Restituisce una *Promise* con la voce del servizio (l'oggetto JSON), oppure `null` se non esiste.

Il servizio viene cercato per `id` e, in subordine, per il parametro `service=` contenuto nel suo `link`.

ServiceHead chiama già `loadService` per il servizio corrente: chiamarlo di nuovo dal proprio codice non genera altre richieste, perché `services.json` resta in memoria.

### `OpenData.getData(alias, opzioni?)`

Restituisce una *Promise* con i dati del dataset. Il dato passa per queste fasi:

```text
fetch (con tentativi) → normalizzazione → filtri → [ordinamento] → [limite] → ricostruzione del formato originale
```

- Un **elenco JSON** torna come elenco, un **GeoJSON** come `FeatureCollection`, un dato **raggruppato** con la sua struttura originale.
- Con `cache: true` (predefinito) la *Promise* viene memorizzata: più chiamate con lo stesso alias, anche contemporanee, fanno **una sola richiesta**. Se la richiesta fallisce non viene memorizzata.
- In caso di errore la *Promise* viene rifiutata e compare l'avviso rosso *"Servizio opendata non disponibile al momento. Riprova più tardi."*

| Opzione | Tipo | Effetto |
| :--- | :--- | :--- |
| `silent` | vero/falso | `true`: in caso di errore non mostra l'avviso rosso. |
| `config` | oggetto | Sovrascrive i parametri di rete solo per questa chiamata (es. `{ timeout: 30000 }`). |
| `init` | oggetto | Opzioni passate a `fetch()` (es. intestazioni). |

```js
const alberi = await OpenData.getData('alberi', { silent: true, config: { retries: 1 } });
```

Errori possibili:

| Messaggio | Causa |
| :--- | :--- |
| `[OpenData] alias sconosciuto: …` | Alias non presente nei dataset del servizio, oppure `loadService` non ancora eseguito. |
| `[OpenData] nessun endpoint "fetch" per: …` | Nel dataset manca il campo `fetch`. |
| `HTTP 404 (…)`, `HTTP 500 (…)` | La fonte ha risposto con un errore. |
| `Failed to fetch` / `NetworkError` | Rete assente, oppure blocco CORS o CSP (vedi [Risoluzione dei problemi](problemi)). |

### `OpenData.refresh(alias, opzioni?)`

Come `getData`, ma ignora la cache e riscarica il dato. Utile per un pulsante "Aggiorna" o per un aggiornamento periodico:

```js
setInterval(function () {
  OpenData.refresh('plan').then(disegna);
}, 5 * 60 * 1000);   // ogni 5 minuti
```

### `OpenData.getInfo(alias)`

Statistiche dell'ultima lettura:

```js
{ elapsedMs: 182, recordCount: 37, source: "network", timestamp: 1790000000000 }
```

`source` vale `"network"` o `"cache"`. `recordCount` è il numero di record dopo i filtri (`null` se il dato non è un elenco).

### `OpenData.probe(opzioni?)`

Verifica che i dataset siano raggiungibili, li mette in cache e restituisce `{ ok, results, failed }`. ServiceHead la esegue all'apertura della pagina sul **primo** dataset del servizio, e lo *spinner* di caricamento resta visibile finché non termina.

| Opzione | Predefinito | Effetto |
| :--- | :--- | :--- |
| `aliases` | tutti i dataset registrati | Elenco degli alias da provare. |
| `requireAll` | `true` | `true`: basta un errore per considerare la prova fallita. `false`: fallisce solo se falliscono tutti. |

### `OpenData.configure(opzioni)`

Cambia i parametri globali. I valori predefiniti:

| Parametro | Predefinito | Significato |
| :--- | :--- | :--- |
| `retries` | `3` | Tentativi per ogni indirizzo. |
| `retryDelay` | `700` | Attesa (ms) prima del secondo tentativo. |
| `backoff` | `2` | Fattore di crescita dell'attesa (700 ms, 1400 ms, …). |
| `timeout` | `12000` | Tempo massimo (ms) per ogni tentativo. |
| `cache` | `true` | Cache predefinita per i dataset che non specificano `cache`. |
| `errorText` | *"Servizio opendata non disponibile…"* | Testo dell'avviso di errore. |
| `servicesUrl` | preso da `#service-head` | Indirizzo di `services.json`. |

```js
OpenData.configure({ timeout: 20000, errorText: 'Dati momentaneamente non disponibili.' });
```

### `OpenData.onError(funzione)`

Registra una funzione chiamata a ogni errore, oltre all'avviso standard. Riceve l'errore e un contesto (`{ phase: 'getData', alias }` oppure `{ phase: 'probe', failed }`).

```js
OpenData.onError(function (err, ctx) {
  document.getElementById('lista-alberi').innerHTML = '';
});
```

### Altre funzioni

| Funzione | Descrizione |
| :--- | :--- |
| `OpenData.register([dataset, …])` | Registra a mano definizioni di dataset (stessi campi di `services.json`). Utile per dataset non dichiarati nel catalogo. |
| `OpenData.get(alias)` | Definizione registrata di un dataset. |
| `OpenData.aliases()` | Elenco degli alias registrati. |
| `OpenData.loadServices()` | L'intero contenuto di `services.json`. |
| `OpenData.showAlert(tipo, testo)` / `OpenData.hideAlert()` | Mostra o nasconde l'avviso sotto il titolo (`tipo`: `info`, `success`, `warning`, `danger`). |

## Messaggi in console

La libreria scrive nella console del browser cosa sta facendo. Sono il primo posto da guardare se un servizio non mostra dati:

| Messaggio | Significato |
| :--- | :--- |
| `"<alias>" OK in N ms` | Dato scaricato correttamente. |
| `CKAN RAGGIUNGIBILE 1/1 dataset OK` | Prova di raggiungibilità superata. |
| `tentativo 1/3 fallito su … riprovo tra 700ms` | Errore temporaneo, nuovo tentativo in corso. |
| `"…" sembra bloccato da CSP/CORS …` | Il browser ha bloccato la richiesta: nessun nuovo tentativo, si passa all'indirizzo successivo. |
| `"<alias>" FALLITO motivo: …` | Tutti i tentativi e tutti gli indirizzi sono falliti. |
| `CKAN NON RAGGIUNGIBILE` | Prova di raggiungibilità fallita. |
| `operatore non riconosciuto` | Un filtro in `services.json` ha un operatore scritto male. |
