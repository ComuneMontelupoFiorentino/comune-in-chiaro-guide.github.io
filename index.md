---
title: Home
layout: home
nav_order: 1
description: "Guida a Comune in Chiaro: uso, installazione, configurazione e sviluppo di nuovi servizi."
permalink: /
---

# Comune in Chiaro
{: .fs-9 }

Il portale che trasforma gli **open data comunali** in servizi semplici da consultare per i cittadini.
{: .fs-6 .fw-300 }

[Inizia dalla guida per i cittadini](docs/uso){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[Installa il portale](docs/installazione){: .btn .fs-5 .mb-4 .mb-md-0 }

---

![Home page di Comune in Chiaro]({{ '/assets/img/home.png' | relative_url }})

## Cos'è Comune in Chiaro

**Comune in Chiaro** è un'applicazione web open source sviluppata dal servizio *Supporto alla Transizione Digitale* del **Comune di Montelupo Fiorentino**.

Prende i dataset pubblicati dal Comune sul portale open data regionale ([dati.toscana.it](https://dati.toscana.it/organization/comune-di-montelupo-fiorentino)) e li presenta come **servizi**: pagine pensate per rispondere a una domanda concreta del cittadino (per esempio "quali manutenzioni sono in programma nella mia via?") invece di file da scaricare e interpretare.

### Caratteristiche principali

| | |
| :--- | :--- |
| **Dati sempre aggiornati** | I dati sono letti in tempo reale dal portale open data (CKAN): nessuna copia locale da tenere allineata. |
| **Servizi configurabili** | Ogni servizio è descritto in un unico file JSON (`services.json`): titolo, descrizione, tag, dataset, filtri, avvisi. |
| **Design Italia** | Interfaccia realizzata con [Bootstrap Italia](https://italia.github.io/bootstrap-italia/), coerente con le linee guida di design della PA e attenta all'accessibilità (WCAG). |
| **Ricerca intelligente** | Ricerca tra i servizi per titolo, descrizione e tag, tollerante a errori di battitura e accenti. |
| **QR code sul territorio** | Le *schede* di singoli oggetti (un albero, un impianto…) possono essere raggiunte inquadrando un QR code. |
| **Incorporabile** | Un servizio può essere inserito in un'altra pagina web (per esempio il sito istituzionale) tramite `iframe`. |
| **Sicuro per impostazione predefinita** | Whitelist delle pagine, Content Security Policy restrittiva, input ripuliti, relay protetto da SSRF. |
| **Riusabile** | Codice rilasciato con licenza [AGPL-3.0](https://interoperable-europe.ec.europa.eu/licence/gnu-affero-general-public-license-v30): qualunque ente può installarlo e adattarlo. |

## Come è organizzata questa guida

| Se sei… | Leggi |
| :--- | :--- |
| **Cittadino** o operatore che deve spiegare il portale | [Usare il portale](docs/uso) |
| **Tecnico** che deve installarlo su un server | [Requisiti e installazione](docs/installazione) → [Configurazione](docs/configurazione) |
| **Redattore** che deve pubblicare o modificare un servizio | [Il file services.json](docs/services-json) |
| **Sviluppatore** che deve creare la pagina di un nuovo servizio | [Creare un servizio](docs/creare-servizio) → [Libreria OpenData](docs/opendata-js) |
| **Chi vuole capire come funziona dentro** | [Architettura](docs/architettura) → [Sicurezza](docs/sicurezza) |
| Qualcosa non funziona | [Risoluzione dei problemi](docs/problemi) |

## Licenze

- Codice dell'applicazione: [AGPL-3.0](https://github.com/ComuneMontelupoFiorentino/comune-in-chiaro/blob/main/LICENCE)
- Contenuti e dati: [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.it)
- Questa guida: [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.it)

{: .nota }
Gli screenshot di questa guida sono stati realizzati su un'installazione di prova con dati dimostrativi: nomi dei servizi e contenuti possono differire da quelli pubblicati dal Comune.
