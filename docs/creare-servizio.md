---
title: Creare un servizio
nav_order: 6
description: "Passo per passo: dalla voce in services.json alla pagina PHP e al JavaScript che mostra i dati."
---

# Creare un servizio
{: .no_toc }

Un esempio completo e funzionante: il servizio **Censimento alberi**, che mostra un elenco letto da un dataset.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Indice</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## I file di un servizio

Un servizio con `id` = `alberi` è composto da:

| File | Obbligatorio | Contenuto |
| :--- | :---: | :--- |
| `assets/data/services.json` | sì | La voce del servizio: testi, tag, dataset. |
| `views/services/services/alberi.php` | sì | Il markup HTML del contenuto (senza testata e piè di pagina). |
| `assets/dist/js/services/services/services-alberi.js` | quasi sempre | Il codice che legge i dati e li mostra. |
| `assets/dist/css/loading/skeleton-load-alberi.css` | no | Stili dello *skeleton* mostrato durante il caricamento. |

Il nome del file PHP **è** l'autorizzazione: il router accetta `?service=alberi` solo se esiste `views/services/services/alberi.php`. Ogni altro valore porta alla pagina 404.

## 1. Aggiungi la voce in `services.json`

```json
{
  "id": "alberi",
  "type": "service",
  "title": "Censimento alberi",
  "desc": "Consulta gli alberi censiti sul territorio comunale.",
  "link": "?service=alberi&breadCrumb=true",
  "new": true,
  "tags": ["Verde pubblico", "Alberi", "Ambiente"],
  "alert": [ { "typeAlert": "info", "text": "Dati aggiornati settimanalmente." } ],
  "descriptiion": [ { "p": "Questo servizio mostra gli alberi censiti." } ],
  "datasets": [
    {
      "nameDataset": "Censimento del verde",
      "alias": "alberi",
      "url": "https://dati.toscana.it/dataset/censimento-verde",
      "fetch": "api/opendata_relay.php?u=https%3A%2F%2Fdati.toscana.it%2Fdataset%2F...%2Fdownload%2Falberi.json",
      "cache": true,
      "filters": [
        { "field": "stato", "operator": "contains", "source": "query", "parameter": "stato" }
      ]
    }
  ]
}
```

Tutti i campi sono spiegati in [Il file services.json](services-json).

## 2. Crea la pagina PHP

`views/services/services/alberi.php`:

```html
<section class="py-4 embed-visible">
  <div class="container">
    <div class="row justify-content-center">
      <div class="col-12 col-lg-10">
        <h2 class="h4">Elenco alberi</h2>
        <ul id="lista-alberi" class="list-group" aria-live="polite"></ul>
      </div>
    </div>
  </div>
</section>
```

Il router inserisce automaticamente, prima e dopo questo contenuto:

1. `layout/header.php` – testata del portale e menu;
2. `partials/header_service.php` – percorso, titolo, sottotitolo, avviso (riempiti da `services.json`);
3. **la tua pagina**;
4. `partials/notice_dataset.php` – riquadro "Note sui dati" ed elenco dataset;
5. `partials/segnalazioni.php` – riquadro "Problemi?";
6. `layout/footer.php` – piè di pagina e script.

Puoi usare tutte le classi e i componenti di [Bootstrap Italia](https://italia.github.io/bootstrap-italia/docs/). La classe `embed-visible` indica cosa mostrare quando il servizio è [incorporato](embed).

{: .importante }
**Niente JavaScript nella pagina.** La Content Security Policy blocca i tag `<script>` in linea e gli attributi come `onclick="..."`. Tutto il codice va nel file JS del passo successivo, collegando gli eventi con `addEventListener`. Gli attributi `style="..."` invece sono ammessi.

Nella pagina sono disponibili le variabili PHP del router, per esempio `$service` (id del servizio), `$obj_id` (nelle schede) e le costanti `BASE_URL`, `ENTE`, `NAME_SERVICE`. Se stampi un valore nella pagina, usa sempre `htmlspecialchars()`.

## 3. Scrivi il JavaScript

`assets/dist/js/services/services/services-alberi.js` (il nome deve essere esattamente `services-<id>.js`):

```js
document.addEventListener('DOMContentLoaded', function () {

  function esc(s) {
    var d = document.createElement('div');
    d.textContent = s == null ? '' : s;
    return d.innerHTML;
  }

  var ul = document.getElementById('lista-alberi');

  OpenData.loadService('alberi')                   // registra i dataset del servizio
    .then(function () {
      return OpenData.getData('alberi');           // scarica e filtra il dataset "alberi"
    })
    .then(function (alberi) {
      if (!alberi.length) {
        ul.innerHTML = '<li class="list-group-item">Nessun albero trovato.</li>';
        return;
      }
      ul.innerHTML = alberi.map(function (a) {
        return '<li class="list-group-item"><strong>' + esc(a.specie) + '</strong> — '
             + esc(a.via) + ' <span class="badge bg-secondary">' + esc(a.stato) + '</span></li>';
      }).join('');
    })
    .catch(function () {
      // l'avviso "Servizio opendata non disponibile" è già mostrato da OpenData
      ul.innerHTML = '';
    });
});
```

{: .importante }
Metti **sempre** il codice dentro `DOMContentLoaded`. Il file del servizio viene caricato prima delle librerie del portale (che sono `defer`): fuori da quell'evento `OpenData` non esiste ancora.

Il file viene incluso automaticamente dal piè di pagina se esiste. Tutte le funzioni di `OpenData` sono descritte in [Libreria OpenData](opendata-js).

{: .suggerimento }
Inserisci i dati nella pagina sempre tramite una funzione di *escape* (come `esc()` sopra) o con `textContent`: i dati aperti arrivano da una fonte esterna e non vanno mai inseriti come HTML.

## 4. Verifica

Apri `?service=alberi&breadCrumb=true`:

![Il servizio di esempio]({{ '/assets/img/servizio.png' | relative_url }})

Grazie al filtro con `source: "query"`, lo stesso servizio aperto con `&stato=buono` mostra solo gli alberi in buono stato:

![Il servizio filtrato dall'indirizzo]({{ '/assets/img/servizio-filtro.png' | relative_url }})

Nella console del browser (F12) i messaggi di `OpenData` confermano il caricamento:

```text
[OpenData] verifica raggiungibilita CKAN, dataset in prova: alberi
[OpenData] "alberi" OK in 19ms
[OpenData] CKAN RAGGIUNGIBILE 1/1 dataset OK (19ms)
```

## Skeleton di caricamento (facoltativo)

Mentre i dati arrivano puoi mostrare delle sagome grigie animate usando la classe `skeleton`, già definita negli stili del portale:

```html
<ul id="lista-alberi" class="list-group">
  <li class="list-group-item"><span class="skeleton" style="display:inline-block;width:60%;height:1rem"></span></li>
  <li class="list-group-item"><span class="skeleton" style="display:inline-block;width:45%;height:1rem"></span></li>
</ul>
```

Il JavaScript le sostituisce con i dati veri. Se ti servono stili dedicati, crea `assets/dist/css/loading/skeleton-load-alberi.css`: viene incluso automaticamente nelle pagine del servizio.

## Mappe

Con `map=true` nel `link` del servizio, la pagina carica [Leaflet](https://leafletjs.com/reference.html) (oggetto `L`) e [Turf](https://turfjs.org/) (oggetto `turf`). Le mappe di base di **OpenStreetMap** sono già ammesse dalla CSP.

```html
<div id="mappa" class="embed-visible" style="height: 420px;"></div>
```

```js
document.addEventListener('DOMContentLoaded', function () {
  var map = L.map('mappa').setView([43.73, 11.02], 14);
  L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; OpenStreetMap'
  }).addTo(map);

  OpenData.loadService('alberi')
    .then(function () { return OpenData.getData('alberi-geo'); })   // un GeoJSON
    .then(function (geojson) { L.geoJSON(geojson).addTo(map); });
});
```

{: .nota }
Le immagini dei marcatori di Leaflet sono in `assets/dist/leaflet/images/`. Se le icone non compaiono, indica il percorso con `L.Icon.Default.imagePath = BASE_URL + 'assets/dist/leaflet/images/';`.

## Schede di un oggetto (QR code)

Una **scheda** è una pagina dedicata a un singolo oggetto, identificato dal parametro `obj_id`. È pensata per essere aperta da un QR code:

```text
https://<dominio>/comune-in-chiaro/?sheet=albero&obj_id=a001
```

| File | Contenuto |
| :--- | :--- |
| `views/services/sheets/albero.php` | Markup della scheda. |
| `assets/dist/js/services/sheets/sheets-albero.js` | Codice della scheda. |
| Voce in `services.json` con `"id": "albero"` | Testata e dataset della scheda. |

Regole del router:

- `sheet` senza `obj_id`, oppure `obj_id` senza un `sheet` valido → errore **400** "Richiesta non valida";
- `obj_id` viene ripulito come gli altri parametri: **solo minuscole**, cifre, `-` e `_` (`A-001` diventa `a-001`). Usa codici già in questo formato;
- al posto di "Problemi?" viene incluso `partials/segnalazioni_obj.php`, che prima di aprire il modulo di segnalazione mostra il codice dell'oggetto da copiare.

Per mostrare solo l'oggetto richiesto, basta un filtro sul dataset:

```json
{ "field": "id", "operator": "=", "source": "query", "parameter": "obj_id" }
```

{: .attenzione }
Anche le voci delle schede in `services.json` vengono mostrate come schede nella pagina iniziale, perché la home non distingue per `type`. Vedi [Punti aperti](problemi#punti-aperti-nel-codice).

## Checklist

- [ ] `id` in `services.json` uguale al nome del file PHP (minuscolo, senza spazi)
- [ ] `services.json` ancora valido dopo la modifica
- [ ] file JS chiamato `services-<id>.js` (o `sheets-<id>.js`) e codice dentro `DOMContentLoaded`
- [ ] nessun `<script>` in linea né `onclick` nella pagina PHP
- [ ] dominio dei dati in `csp_connect_src_extra`, **oppure** lettura tramite relay con host in `ALLOWED_HOSTS`
- [ ] dati inseriti nella pagina con escape
- [ ] provato su smartphone e con la sola tastiera
