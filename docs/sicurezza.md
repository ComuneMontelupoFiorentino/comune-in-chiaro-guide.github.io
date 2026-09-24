---
title: Sicurezza
nav_order: 11
description: "Le misure di sicurezza di Comune in Chiaro e cosa comportano per chi sviluppa servizi."
---

# Sicurezza
{: .no_toc }

Il portale è costruito secondo il principio **"negato se non espressamente consentito"**. Questa pagina riassume le misure presenti e cosa significano per chi scrive o configura un servizio.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Indice</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Riepilogo

| Misura | Dove | Cosa impedisce |
| :--- | :--- | :--- |
| Configurazione obbligatoria (*fail-closed*) | `index.php` | Funzionamento con impostazioni mancanti o indovinate. |
| Solo `GET` e `HEAD` | `index.php`, relay | Invio di dati al portale. |
| Parametri ripuliti | `index.php` | *Path traversal* e *injection* tramite l'indirizzo. |
| Whitelist basata sui file | `index.php` | Inclusione di file arbitrari: si apre solo ciò che esiste nelle cartelle delle viste. |
| Content Security Policy | `index.php` | Script iniettati, caricamento da domini non previsti, *clickjacking*. |
| Embed doppiamente autorizzato | `index.php` + `services.json` | Incorporamento del portale in siti non autorizzati. |
| Cartelle riservate | `.htaccess` in `config/` e `views/` | Lettura diretta di configurazione e template. |
| Allowlist degli host | relay | Uso del relay per raggiungere altri server (*SSRF*). |
| Escape dei dati | JavaScript | Inserimento di codice attraverso i dati aperti o `services.json`. |
| Log degli eventi sospetti | `error_log` di PHP | — (permette di accorgersene) |

## Header HTTP

Ogni risposta contiene:

```text
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(self), camera=(), fullscreen=(self)
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Resource-Policy: same-origin
X-Frame-Options: DENY            (SAMEORIGIN in modalità embed autorizzata)
Content-Security-Policy: ...
```

### Content Security Policy

```text
default-src 'self';
img-src     'self' <csp_img_src_extra> data: https://tile.openstreetmap.org https://*.tile.openstreetmap.org;
font-src    'self' data:;
connect-src 'self' <csp_connect_src_extra>;
script-src  'self' 'nonce-<casuale>';
style-src   'self' 'unsafe-inline';
object-src  'none';
base-uri    'self';
frame-ancestors 'none'           (oppure <embed_parent_origin>);
form-action 'self';
```

Cosa comporta per chi sviluppa:

| Vuoi… | Serve… |
| :--- | :--- |
| eseguire JavaScript | un file `.js` sul portale. **Niente** `<script>` in linea né attributi `onclick`, `onchange`, ecc. |
| scaricare dati con `fetch()` da un altro dominio | aggiungerlo a `csp_connect_src_extra`, oppure passare dal [relay](relay). |
| mostrare immagini da un altro dominio | aggiungerlo a `csp_img_src_extra`. |
| usare mappe diverse da OpenStreetMap | aggiungere il dominio delle tessere a `csp_img_src_extra`. |
| caricare una libreria da una CDN | copiarla in `assets/dist/` (le CDN non sono ammesse). |
| usare stili in linea | nulla: `style="..."` è ammesso. |

{: .nota }
Il `nonce` generato a ogni richiesta è già presente nella policy, ma nessun template lo usa ancora. Se in futuro servisse uno script in linea, andrebbe scritto come `<script nonce="<?= $nonce ?>">`.

### Permessi del browser

`Permissions-Policy` consente la **geolocalizzazione** e lo **schermo intero** solo al portale stesso e **disattiva la fotocamera**.

{: .attenzione }
Con `camera=()` la lettura dei QR code con la fotocamera (`scan=true`, libreria html5-qrcode) viene bloccata dal browser. Per usarla, la policy in `index.php` va cambiata in `camera=(self)`.

## Log di sicurezza

Gli eventi anomali vengono scritti nel log degli errori di PHP con il prefisso `[SECURITY]`:

| Messaggio | Quando |
| :--- | :--- |
| `metodo non ammesso: POST` | Richiesta con metodo diverso da GET/HEAD. |
| `service non in whitelist [...]` | `service` non corrisponde a nessun file. |
| `sheet non in whitelist [...]` | `sheet` non corrisponde a nessun file. |
| `obj_id presente senza sheet valido` | Richiesta di scheda incompleta. |
| `sheet valido senza obj_id` | Richiesta di scheda incompleta. |
| `richiesta embed non autorizzata [...]` | `embed=true` per un servizio senza `allowEmbed`. |
| `richiesta non valida (fallback 404)` | Nessuna pagina corrispondente. |

Gli errori di configurazione hanno invece il prefisso `[ERROR]` (config mancante, chiave mancante, `services.json` non valido o non leggibile).

## Buone pratiche

- Tieni `config.php` e le personalizzazioni fuori dal repository pubblico, se contengono informazioni non destinate alla pubblicazione.
- Non allentare la CSP con `'unsafe-inline'` negli script: sposta il codice in file esterni.
- Non rimuovere l'allowlist `ALLOWED_HOSTS` del relay e non aggiungervi domini interni.
- Inserisci i dati nel DOM con `textContent` o con una funzione di escape.
- Servi il portale solo in **HTTPS**.
