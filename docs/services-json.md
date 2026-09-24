---
title: Il file services.json
nav_order: 5
description: "Riferimento completo del catalogo dei servizi: campi, dataset, filtri, avvisi."
---

# Il file `services.json`
{: .no_toc }

Il catalogo di tutti i servizi del portale. È l'unico file che un redattore deve modificare per pubblicare un servizio, cambiarne i testi, aggiungere un avviso o collegare un dataset.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Indice</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Dove si trova e chi lo legge

Il file è `assets/data/services.json` ed è letto da tre parti del portale:

- la **pagina iniziale**, che mostra una scheda per ogni elemento, nello stesso ordine del file;
- la **testata di ogni servizio** (titolo, sottotitolo, avviso, descrizione, elenco dataset);
- il **router PHP**, che controlla il campo `allowEmbed` per decidere se il servizio può essere incorporato.

Dopo ogni modifica non serve riavviare nulla: il portale aggiunge automaticamente all'indirizzo del file la data di ultima modifica (`services.json?v=20260924105300`), così i browser scaricano subito la versione nuova.

{: .importante }
Il file deve essere **JSON valido**. Una virgola di troppo o una virgoletta mancante bastano a far comparire in home *"Impossibile caricare i servizi."* e a svuotare la testata di tutti i servizi. Prima di salvare, controllalo con un validatore (per esempio `php -r 'json_decode(file_get_contents("assets/data/services.json"), flags: JSON_THROW_ON_ERROR); echo "OK\n";'`).

## Esempio completo

```json
[
  {
    "id": "cerca-manutenzioni",
    "type": "service",
    "title": "Cerca manutenzioni",
    "desc": "Trova gli interventi di manutenzione programmati nella tua zona.",
    "link": "?service=cerca-manutenzioni&map=true&breadCrumb=true",
    "new": true,
    "allowEmbed": true,
    "tags": ["Manutenzioni", "Interventi", "Programmazione", "Verde Pubblico"],
    "alert": [
      { "typeAlert": "warning", "text": "I dati vengono aggiornati ogni mattina alle 7." }
    ],
    "descriptiion": [
      { "h4": "Cosa trovi" },
      { "p": "L'elenco degli interventi programmati dal Comune per i prossimi giorni." },
      { "p": "Puoi cercare per via oppure consultare la mappa." }
    ],
    "datasets": [
      {
        "nameDataset": "Piano delle manutenzioni",
        "alias": "plan",
        "url": "https://dati.toscana.it/dataset/piano-manutenzioni",
        "fetch": "api/opendata_relay.php?u=https%3A%2F%2Fdati.toscana.it%2Fdataset%2F...%2Fdownload%2Fpiano.json",
        "cache": true,
        "recordsPath": "giorni.*.interventi",
        "groupKeyField": "date",
        "groupCountField": "totale",
        "filters": [
          { "field": "status", "operator": "!=", "value": "Annullato" }
        ]
      }
    ]
  }
]
```

## Campi del servizio

| Campo | Tipo | Obbligatorio | Descrizione |
| :--- | :--- | :---: | :--- |
| `id` | testo | sì | Identificativo del servizio. **Deve coincidere con il nome del file** `views/services/services/<id>.php`. Usa solo minuscole, cifre, `-` e `_`. |
| `title` | testo | sì | Titolo della scheda in home e della pagina del servizio. |
| `desc` | testo | sì | Descrizione breve: testo della scheda e sottotitolo della pagina. |
| `link` | testo | sì | Indirizzo aperto da **Accedi al servizio**. Vedi [Il campo link](#il-campo-link). |
| `tags` | elenco di testi | consigliato | Parole chiave. In scheda si vedono le prime due, le altre con il pulsante **+N**. Sono usate dalla ricerca. |
| `new` | vero/falso | no | `true` mostra l'etichetta **New** sulla scheda. |
| `allowEmbed` | vero/falso | no | `true` consente di [incorporare il servizio](embed) in un altro sito. Predefinito `false`. |
| `alert` | elenco | no | Avviso mostrato sotto il titolo. Vedi [Avvisi](#avvisi). |
| `descriptiion` | elenco | no | Descrizione estesa, aperta dall'icona ⓘ accanto al titolo. Vedi [Descrizione estesa](#descrizione-estesa). |
| `datasets` | elenco | no | Dataset usati dal servizio. Vedi [Dataset](#dataset). |
| `type` | testo | no | `"service"` per i servizi. Oggi è solo informativo. |
| `icona` | testo | no | Nome di un'icona di Bootstrap Italia (es. `it-calendar`). Riservato: il modello attuale della scheda non la mostra. |
| `lastUpdate` | oggetto | no | `{ "alias": "...", "field": "..." }`: dataset e campo che contengono la data di aggiornamento. Riservato: non ancora usato dal codice. |

{: .attenzione }
Il campo della descrizione estesa si chiama proprio **`descriptiion`**, con due *i*. È il nome letto dal codice: scrivendo `description` la descrizione non compare.

### Il campo `link`

È un indirizzo relativo al portale, formato da parametri:

```text
?service=<id>&map=true&breadCrumb=true
```

| Parametro | Effetto |
| :--- | :--- |
| `service=<id>` | Obbligatorio. Apre il servizio con quell'`id`. |
| `breadCrumb=true` | Mostra il percorso *Home / NOME SERVIZIO* sopra il titolo. |
| `map=true` | Carica le librerie per le mappe: [Leaflet](https://leafletjs.com/) e [Turf](https://turfjs.org/). |
| `scan=true` | Carica la libreria per leggere i QR code con la fotocamera ([html5-qrcode](https://github.com/mebjas/html5-qrcode)). |

Metti `map=true` e `scan=true` solo se la pagina del servizio li usa davvero: sono librerie pesanti.

### Avvisi

```json
"alert": [
  { "typeAlert": "warning", "text": "Servizio in manutenzione fino alle 14." }
]
```

| Campo | Valori |
| :--- | :--- |
| `typeAlert` | `info` (predefinito), `success`, `warning`, `danger` |
| `text` | Testo semplice (l'HTML non viene interpretato) |

Viene mostrato **solo il primo** elemento dell'elenco. Per togliere l'avviso, elimina il campo o lascia l'elenco vuoto (`"alert": []`).

{: .nota }
Se il portale open data non risponde, l'avviso della redazione viene sostituito dall'avviso rosso di errore.

### Descrizione estesa

Un elenco di blocchi. Ogni blocco è un oggetto con **un tag** come chiave e il testo come valore:

```json
"descriptiion": [
  { "h4": "Come funziona" },
  { "p": "Scegli una via dall'elenco." },
  { "strong": "I dati sono indicativi." }
]
```

Tag ammessi: `p`, `h3`, `h4`, `b`, `strong`, `i`, `em`, `span`, `ul`, `li`, `br`. Qualunque altro tag viene trasformato in `p`. Il testo è sempre inserito come testo semplice: non si possono annidare tag né inserire link.

## Dataset

Ogni elemento di `datasets` descrive una fonte dati. I dataset vengono mostrati nella finestra **Elenco dei dataset usati** e sono letti dal JavaScript del servizio tramite il loro `alias` (vedi [Libreria OpenData](opendata-js)).

| Campo | Tipo | Obbligatorio | Descrizione |
| :--- | :--- | :---: | :--- |
| `alias` | testo | sì | Nome breve con cui il codice chiede il dato: `OpenData.getData("plan")`. Unico all'interno del servizio. |
| `nameDataset` | testo | sì | Nome mostrato ai cittadini nell'elenco dei dataset. |
| `url` | testo | sì | Pagina del dataset sul portale open data (link per i cittadini). |
| `fetch` | testo o elenco | sì | Indirizzo da cui scaricare i dati (JSON o GeoJSON). Se è un elenco, gli indirizzi vengono provati **in ordine** finché uno risponde. |
| `cache` | vero/falso | no | `true` (predefinito): il dato viene scaricato una sola volta per pagina e riusato. `false`: ogni richiesta lo riscarica. |
| `recordsPath` | testo | no | Per dati raggruppati. Vedi [Dati raggruppati](#dati-raggruppati). |
| `groupKeyField` | testo | no | Per dati raggruppati: nome del campo in cui copiare la chiave del gruppo. |
| `groupCountField` | testo | no | Per dati raggruppati: campo contatore da ricalcolare dopo i filtri. |
| `filters` | elenco | no | Filtri applicati automaticamente ai record. Vedi [Filtri](#filtri). |
| `sort`, `limit` | — | no | Riservati a ordinamento e limite dei risultati: **non ancora implementati**, oggi vengono ignorati. |

{: .nota }
Il **primo** dataset dell'elenco viene anche usato per verificare, all'apertura della pagina, che il portale open data sia raggiungibile.

### Il campo `fetch`

Tre forme possibili:

```json
"fetch": "https://dati.toscana.it/api/3/action/datastore_search?resource_id=..."
```

```json
"fetch": "api/opendata_relay.php?u=https%3A%2F%2Fdati.toscana.it%2Fdataset%2F...%2Fdownload%2Ffile.json"
```

```json
"fetch": [
  "https://dati.toscana.it/dataset/.../download/file.json",
  "api/opendata_relay.php?u=https%3A%2F%2Fdati.toscana.it%2Fdataset%2F...%2Fdownload%2Ffile.json"
]
```

- Un indirizzo **assoluto** viene chiamato direttamente dal browser: il dominio deve essere in `csp_connect_src_extra` e deve rispondere con l'header CORS.
- Un indirizzo **relativo** (`api/...`, `assets/...`) è sullo stesso dominio del portale: funziona sempre.
- Nel parametro `u=` del relay l'indirizzo va **codificato** (`:` → `%3A`, `/` → `%2F`). In JavaScript: `encodeURIComponent(url)`.

Il portale riprova ogni indirizzo fino a **3 volte** (attese crescenti: 0,7 s, poi 1,4 s) con un tempo massimo di **12 secondi** per tentativo. Se l'errore è un blocco CORS o CSP, che riprovando non si risolverebbe, passa subito all'indirizzo successivo.

### Formati di dati riconosciuti

| Formato | Esempio | Come viene trattato |
| :--- | :--- | :--- |
| Elenco JSON | `[ {...}, {...} ]` | Ogni elemento è un record. |
| GeoJSON | `{ "type": "FeatureCollection", "features": [...] }` | Ogni *feature* è un record. Il risultato resta un GeoJSON valido (con eventuali altre chiavi, come `crs`). |
| Dati raggruppati | `{ "giorni": { "2026-09-24": { "totale": 2, "interventi": [...] } } }` | Serve `recordsPath` (sotto). |
| Altro | un oggetto qualsiasi | Restituito così com'è, senza filtri. |

{: .suggerimento }
Per le risposte delle API CKAN (`{"success": true, "result": {"records": [...]}}`) i filtri non si applicano, perché i record non sono al primo livello: il codice del servizio riceve la risposta completa e legge `result.records`.

### Dati raggruppati

Alcuni dataset raggruppano i record sotto chiavi variabili, per esempio per data:

```json
{
  "generated_at": "2026-09-24T06:00:00",
  "giorni": {
    "2026-09-24": { "totale": 2, "interventi": [ { "via": "Via Roma" }, { "via": "Via Verdi" } ] },
    "2026-09-25": { "totale": 1, "interventi": [ { "via": "Piazza Centrale" } ] }
  }
}
```

Si descrivono con tre campi:

| Campo | Valore nell'esempio | Significato |
| :--- | :--- | :--- |
| `recordsPath` | `giorni.*.interventi` | Percorso fino all'elenco dei record. L'asterisco `*` sta per "qualunque chiave di gruppo" (qui, la data). Un solo `*`. |
| `groupKeyField` | `date` | Durante l'elaborazione, a ogni record viene aggiunto il campo `date` con la chiave del gruppo, così si può filtrare per data. Nel risultato finale il campo viene tolto. |
| `groupCountField` | `totale` | Dopo i filtri, `totale` viene ricalcolato con il numero di record rimasti nel gruppo. |

Il risultato mantiene la stessa struttura dell'originale (compresi `generated_at` e gli altri campi), con i soli record che hanno superato i filtri.

{: .attenzione }
Quando usi `recordsPath`, indica **sempre** anche `groupKeyField`: senza, il portale non sa ricollocare i record nei rispettivi gruppi e nel risultato tutti i gruppi risultano vuoti.

### Filtri

I filtri scartano i record che non soddisfano una condizione. Se ne indichi più d'uno, un record deve soddisfarli **tutti**.

**Filtro con valore fisso:**

```json
{ "field": "status", "operator": "=", "value": "Programmato" }
```

**Filtro con valore preso dall'indirizzo della pagina:**

```json
{ "field": "via", "operator": "contains", "source": "query", "parameter": "via" }
```

Con questo filtro, `?service=manutenzioni&via=roma` mostra solo i record la cui via contiene "roma". Se il parametro non è presente nell'indirizzo, il filtro viene **ignorato** (vengono mostrati tutti i record).

| Campo | Descrizione |
| :--- | :--- |
| `field` | Nome del campo. Per i campi annidati si usa il punto: nei GeoJSON `properties.status`. |
| `operator` | Uno degli operatori della tabella sotto. |
| `value` | Valore di confronto fisso. Per `in` e `notIn` è un elenco. |
| `source` | `"query"` per leggere il valore dall'indirizzo della pagina. |
| `parameter` | Nome del parametro dell'indirizzo (con `source: "query"`). |

**Operatori:**

| Operatore | Vero se il campo… | Maiuscole/minuscole |
| :--- | :--- | :--- |
| `=` | è uguale al valore (`"5"` e `5` sono considerati uguali) | **distingue** |
| `!=` | è diverso dal valore | distingue |
| `>` `>=` `<` `<=` | è maggiore / maggiore o uguale / minore / minore o uguale (confronto numerico) | — |
| `contains` | contiene il valore | non distingue |
| `startsWith` | inizia con il valore | non distingue |
| `endsWith` | finisce con il valore | non distingue |
| `in` | è uno dei valori dell'elenco (`"value": ["A", "B"]`) | distingue |
| `notIn` | non è nessuno dei valori dell'elenco | distingue |
| `isNull` | è vuoto o assente (non serve `value`) | — |
| `isNotNull` | è valorizzato (non serve `value`) | — |

{: .suggerimento }
Per i valori che arrivano dall'indirizzo (digitati dall'utente o stampati in un QR code) preferisci `contains` o `startsWith`, che non distinguono tra maiuscole e minuscole: con `=`, `?stato=buono` **non** trova i record con `"Buono"`.

I confronti `>`, `<` ecc. sono numerici: per confrontare date usa un formato numerico (per esempio `20260924`) oppure filtra nel codice del servizio.

Un operatore scritto male non genera errori: il filtro viene ignorato e nella console del browser compare un avviso `[OpenData:FilterEngine] operatore non riconosciuto`.
