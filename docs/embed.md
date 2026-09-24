---
title: Incorporare un servizio
nav_order: 8
description: "Come inserire un servizio di Comune in Chiaro in un altro sito tramite iframe."
---

# Incorporare un servizio
{: .no_toc }

Un servizio può essere inserito in un'altra pagina web, per esempio una notizia del sito istituzionale, tramite un `iframe`.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Indice</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Come funziona

Per impostazione predefinita **nessuna pagina** del portale può essere incorporata: ogni risposta contiene `frame-ancestors 'none'` e `X-Frame-Options: DENY`.

La protezione viene allentata solo quando si verificano **tutte** queste condizioni:

1. il servizio esiste davvero (c'è il suo file PHP);
2. in `services.json` il servizio ha `"allowEmbed": true`;
3. l'indirizzo contiene `embed=true`.

In quel caso la risposta consente l'incorporamento **solo** al sito indicato in `embed_parent_origin` di `config.php`:

```text
Content-Security-Policy: ... frame-ancestors https://www.comune.esempio.it; ...
X-Frame-Options: SAMEORIGIN
```

Una richiesta con `embed=true` per un servizio che non lo consente viene registrata nel log (`[SECURITY] richiesta embed non autorizzata`) e resta non incorporabile.

## Passi

### 1. Abilita il servizio

```json
{
  "id": "cerca-manutenzioni",
  "allowEmbed": true,
  ...
}
```

### 2. Indica il sito che lo ospiterà

In `config/config.php`:

```php
'embed_parent_origin' => 'https://www.comune.esempio.it',
```

Si può indicare più di un sito separandoli con uno spazio.

### 3. Scegli cosa mostrare

In modalità embed il portale nasconde **tutto** tranne gli elementi con la classe `embed-visible` (e ciò che contengono). Aggiungila ai blocchi della pagina del servizio che devono comparire nell'`iframe`:

```html
<section class="py-4 embed-visible">
  ... contenuto del servizio ...
</section>
```

Testata, menu, piè di pagina e riquadri comuni vengono nascosti.

{: .attenzione }
Questa funzione è svolta da `embed.js`, che il piè di pagina carica da `assets/dist/js/services/services/embed.js`. Nel repository il file si trova invece in `assets/dist/js/embed.js`: finché non viene spostato (o corretto il percorso in `views/layout/footer.php`), l'`iframe` mostra la pagina completa. Vedi [Punti aperti](problemi#punti-aperti-nel-codice).

### 4. Inserisci l'iframe

Nella pagina del sito ospitante:

```html
<iframe
  src="https://www.comune.esempio.it/comune-in-chiaro/?service=cerca-manutenzioni&map=true&embed=true"
  title="Cerca manutenzioni – Comune in Chiaro"
  width="100%" height="700"
  style="border:0"
  loading="lazy">
</iframe>
```

- Includi nell'indirizzo gli stessi parametri del `link` del servizio (per esempio `map=true`), più `embed=true`.
- L'attributo `title` è necessario per l'accessibilità: i lettori di schermo lo annunciano.
- L'altezza non si adatta da sola al contenuto: scegline una adeguata.

## Pulsante "Copia codice di incorporamento" (facoltativo)

`embed.js` gestisce anche un pulsante che mostra il codice dell'`iframe` pronto da copiare. Se la pagina del servizio contiene questi elementi, vengono collegati automaticamente:

| id | Elemento |
| :--- | :--- |
| `embedURL` | Elemento che apre la finestra (attivabile anche con `Invio` e `Spazio`). |
| `CopyPasteEmbed` | La finestra (modale Bootstrap Italia). |
| `iframeEmbed` | Campo di testo che contiene il codice dell'`iframe`. |
| `btnCopy` | Pulsante che copia il contenuto di `iframeEmbed`. |

```html
<div id="embedURL" class="btn btn-outline-primary btn-sm" role="button" tabindex="0"
     data-bs-toggle="modal" data-bs-target="#CopyPasteEmbed">Incorpora</div>

<div class="modal fade" id="CopyPasteEmbed" tabindex="-1" role="dialog" aria-labelledby="titoloEmbed">
  <div class="modal-dialog" role="document">
    <div class="modal-content">
      <div class="modal-header"><h2 class="modal-title h5" id="titoloEmbed">Incorpora il servizio</h2></div>
      <div class="modal-body">
        <div class="input-group">
          <input type="text" class="form-control" id="iframeEmbed" readonly
                 value='&lt;iframe src="https://www.comune.esempio.it/comune-in-chiaro/?service=cerca-manutenzioni&amp;embed=true" title="Cerca manutenzioni" width="100%" height="700" style="border:0"&gt;&lt;/iframe&gt;'>
          <button id="btnCopy" class="btn btn-primary" type="button" aria-label="Copia">
            <svg class="icon icon-white"><use href="assets/dist/svg/sprites.svg#it-copy"></use></svg>
          </button>
        </div>
      </div>
    </div>
  </div>
</div>
```

La copia usa gli appunti del sistema quando la pagina è in HTTPS; negli altri casi (e su alcuni tablet) usa un metodo alternativo. Il pulsante diventa verde se la copia riesce, rosso se fallisce.

## Provare in locale

Per provare l'incorporamento dal proprio computer senza toccare `config.php`, imposta la variabile d'ambiente `EMBED_PARENT_ORIGIN` (vedi [Configurazione](configurazione#sovrascrivere-lorigine-dellembed-in-locale)).

Per verificare gli header:

```bash
curl -sI "https://<dominio>/comune-in-chiaro/?service=cerca-manutenzioni&embed=true" | grep -i "frame"
```

{: .nota }
`X-Frame-Options` non consente di indicare un'origine esterna: resta `SAMEORIGIN` solo per i browser molto vecchi che non leggono la CSP. Su quei browser l'`iframe` da un altro dominio non viene mostrato, ma il portale resta protetto.
