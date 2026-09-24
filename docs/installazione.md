---
title: Requisiti e installazione
nav_order: 3
description: "Come installare Comune in Chiaro su un server web con PHP."
---

# Requisiti e installazione
{: .no_toc }

<details open markdown="block">
  <summary>Indice</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Requisiti

Comune in Chiaro non ha database né dipendenze da installare con Composer o npm: tutte le librerie (Bootstrap Italia, Leaflet, Turf, html5-qrcode) sono già incluse nella cartella `assets/`.

| Componente | Requisito | Note |
| :--- | :--- | :--- |
| **PHP** | 8.0 o superiore | Il codice usa `declare(strict_types=1)` e il tipo `mixed`, disponibili da PHP 8.0. |
| **Estensione PHP `curl`** | Necessaria solo per il [relay](relay) | Serve se qualche dataset viene letto tramite `api/opendata_relay.php`. |
| **Server web** | Apache 2.4 (consigliato) oppure Nginx | Con Apache i file `.htaccess` inclusi proteggono le cartelle riservate. Con Nginx vanno riportate le stesse regole a mano (vedi sotto). |
| **HTTPS** | Fortemente consigliato | Necessario per la copia negli appunti del codice oggetto, per la fotocamera (scansione QR) e per l'incorporamento cross-origin. |
| **Connessione in uscita** | Verso `dati.toscana.it` | Solo per il relay: è il server a scaricare i dati. Se l'ente esce su internet tramite proxy, va configurato nel relay. |
| **Browser degli utenti** | Qualsiasi browser moderno | Serve JavaScript attivo: i contenuti dei servizi vengono caricati nel browser. |

## Struttura dei file

```text
comune-in-chiaro/
├── index.php                  ← unico punto di ingresso (router)
├── config/
│   ├── config.php             ← configurazione dell'installazione
│   └── .htaccess              ← nega l'accesso diretto
├── api/
│   └── opendata_relay.php     ← relay verso il portale open data (facoltativo)
├── assets/
│   ├── data/services.json     ← catalogo dei servizi
│   └── dist/                  ← CSS, JS, font, icone, librerie
├── views/
│   ├── layout/                ← header.php, footer.php
│   ├── main/                  ← pagine generali (home, about)
│   ├── partials/              ← blocchi comuni dei servizi
│   ├── error/                 ← pagine di errore
│   ├── services/              ← DA CREARE: pagine dei servizi e delle schede
│   └── .htaccess              ← nega l'accesso diretto
└── data/stats.jsonl
```

Una spiegazione dettagliata di ogni cartella è nella pagina [Architettura](architettura).

## Installazione passo per passo

### 1. Scarica il codice

Nella cartella pubblica del server web (per esempio `/var/www/html`):

```bash
cd /var/www/html
git clone https://github.com/ComuneMontelupoFiorentino/comune-in-chiaro.git
```

Il portale sarà raggiungibile all'indirizzo `https://<tuo-dominio>/comune-in-chiaro/`.

{: .suggerimento }
Puoi usare qualunque nome di cartella, oppure installarlo nella radice del sito. L'importante è che il valore di `base_url` nella configurazione corrisponda al percorso reale (vedi passo 3).

### 2. Crea le cartelle dei servizi

Il repository non contiene le pagine dei singoli servizi, che vanno aggiunte da ogni installazione. Crea le cartelle attese dal router:

```bash
cd comune-in-chiaro
mkdir -p views/services/services views/services/sheets
mkdir -p assets/dist/js/services/services assets/dist/js/services/sheets
```

{: .importante }
Finché `views/services/services/` è vuota, la home mostra le schede dei servizi presenti in `services.json`, ma cliccando **Accedi al servizio** si ottiene la pagina di errore 404: il router accetta solo servizi per cui esiste il file PHP corrispondente. Vedi [Creare un servizio](creare-servizio).

### 3. Configura l'installazione

Modifica `config/config.php` con i dati dell'ente. Tutti i parametri sono descritti in [Configurazione](configurazione). Il minimo indispensabile:

```php
'base_url' => '/comune-in-chiaro/',   // percorso con / iniziale e finale
'ente'     => 'Comune di Esempio',
```

### 4. Aggiungi logo e favicon

Il layout si aspetta:

- il logo in `assets/dist/logo.png` (mostrato in testata e nel piè di pagina);
- le favicon nella cartella `assets/dist/favicon/` (`favicon.ico`, `apple-touch-icon.png`, `android-chrome-192x192.png`, ecc.).

Questi file non sono nel repository. Vedi anche [Personalizzare per un altro ente](configurazione#personalizzare-per-un-altro-ente).

### 5. Imposta i permessi

L'utente del server web deve poter **leggere** tutti i file. Non serve alcun permesso di scrittura.

```bash
sudo chown -R root:www-data /var/www/html/comune-in-chiaro
sudo find /var/www/html/comune-in-chiaro -type d -exec chmod 750 {} \;
sudo find /var/www/html/comune-in-chiaro -type f -exec chmod 640 {} \;
```

### 6. Proteggi le cartelle riservate

**Con Apache** le cartelle `config/` e `views/` contengono già un `.htaccess` che ne nega l'accesso diretto. Verifica che il virtual host consenta la lettura degli `.htaccess`:

```apache
<Directory /var/www/html/comune-in-chiaro>
    AllowOverride All
</Directory>
```

**Con Nginx** i file `.htaccess` vengono ignorati: aggiungi al blocco `server` regole equivalenti.

```nginx
location ^~ /comune-in-chiaro/config/ { deny all; }
location ^~ /comune-in-chiaro/views/  { deny all; }
location ^~ /comune-in-chiaro/data/   { deny all; }

location /comune-in-chiaro/ {
    index index.php;
    try_files $uri $uri/ /comune-in-chiaro/index.php?$args;
}

location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/run/php/php8.2-fpm.sock;
}
```

{: .attenzione }
La cartella `data/` non ha un `.htaccess`, quindi il file `data/stats.jsonl` è scaricabile da chiunque. Se non ti serve pubblico, aggiungi anche lì un `.htaccess` identico a quello di `config/`.

### 7. Verifica

Apri il portale nel browser e controlla:

| Indirizzo | Risultato atteso |
| :--- | :--- |
| `/comune-in-chiaro/` | Pagina "Scopri i servizi" con le schede di `services.json` |
| `/comune-in-chiaro/?main=about` | Pagina "Cos'è" |
| `/comune-in-chiaro/?service=nonesiste` | Pagina di errore (codice HTTP 404) |
| `/comune-in-chiaro/config/config.php` | Accesso negato (403) |
| `/comune-in-chiaro/views/main/home.php` | Accesso negato (403) |

Da riga di comando:

```bash
curl -sI https://<tuo-dominio>/comune-in-chiaro/ | grep -i "content-security-policy\|x-frame-options"
curl -s -o /dev/null -w "%{http_code}\n" https://<tuo-dominio>/comune-in-chiaro/config/config.php   # atteso 403
```

## Prova in locale

Per provare il portale sul proprio computer basta PHP. Il server integrato di PHP non legge gli `.htaccess` e manda tutte le richieste a `index.php`, quindi serve un piccolo *router* che lasci passare i file statici. Crea nella cartella del progetto il file `_router.php`:

```php
<?php
$path = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);
if ($path !== '/' && is_file(__DIR__ . $path)) {
    return false;               // file statico: lo serve PHP direttamente
}
require __DIR__ . '/index.php';
```

Poi imposta `'base_url' => '/'` in `config/config.php` e avvia:

```bash
php -S 127.0.0.1:8080 _router.php
```

Il portale è su <http://127.0.0.1:8080/>.

{: .nota }
Non caricare `_router.php` sul server di produzione: serve solo per le prove in locale.

## Aggiornare

```bash
cd /var/www/html/comune-in-chiaro
git pull
```

Prima di aggiornare, salva una copia dei file che hai modificato o aggiunto: `config/config.php`, `assets/data/services.json`, le cartelle `views/services/` e `assets/dist/js/services/`, logo e favicon.

{: .suggerimento }
Per evitare conflitti a ogni aggiornamento, tieni la configurazione reale fuori dal controllo di versione (per esempio aggiungendo `config/config.php` al file `.gitignore` e conservandone un modello come `config/config.example.php`).
