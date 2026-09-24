---
title: Usare il portale
nav_order: 2
description: "Guida per i cittadini: trovare un servizio, leggere i dati, segnalare un problema."
---

# Usare il portale
{: .no_toc }

Questa sezione è pensata per chi **consulta** il portale: cittadini, ma anche operatori dello sportello che devono spiegarlo.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Indice</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Due modi per arrivare a un'informazione

Comune in Chiaro si può usare in due modi:

1. **Inquadrando un QR code** affisso sul territorio (su un albero, un cantiere, un impianto…): si apre direttamente la scheda di quell'oggetto.
2. **Cercando dal sito**: si parte dalla pagina iniziale, si sceglie o si cerca un servizio.

## La pagina iniziale

La pagina iniziale ("**Scopri i servizi**") mostra una scheda per ogni servizio disponibile.

![Pagina iniziale con l'elenco dei servizi]({{ '/assets/img/home.png' | relative_url }})

Ogni scheda riporta:

- il **titolo** del servizio e, se è stato pubblicato da poco, l'etichetta **New**;
- una **breve descrizione**;
- il link **ACCEDI AL SERVIZIO**;
- alcune **parole chiave** (tag), precedute da `#`. Se i tag sono più di due, il pulsante **+N** mostra quelli rimanenti.

Durante il caricamento compaiono delle sagome grigie animate: è normale, indicano che l'elenco sta arrivando.

Su smartphone le schede si dispongono una sotto l'altra e il menu si apre con il pulsante ☰ in alto a sinistra.

<img src="{{ '/assets/img/home-mobile.png' | relative_url }}" alt="Pagina iniziale su smartphone" width="300">

## Cercare un servizio

1. Nella pagina iniziale premi l'icona della **lente** in alto a destra.
2. Scrivi una o più parole nel campo **Parola chiave**. L'elenco si filtra già mentre scrivi.
3. Premi **Cerca** o il tasto **Invio** per chiudere il pannello e vedere i risultati.

![Pannello di ricerca]({{ '/assets/img/ricerca-modale.png' | relative_url }})

I risultati riportano il numero di servizi trovati ("Trovati N risultati") e le parole cercate sono **evidenziate** in giallo.

![Risultati della ricerca con evidenziazione]({{ '/assets/img/ricerca-risultati.png' | relative_url }})

Come funziona la ricerca:

- cerca nel **titolo**, nella **descrizione** e nei **tag** del servizio;
- non distingue tra maiuscole e minuscole, né tra lettere accentate e non (`città` = `citta`);
- **tollera piccoli errori di battitura**: `manutenzoni` trova comunque "Manutenzioni";
- se scrivi più parole, vengono mostrati solo i servizi che le contengono **tutte**;
- le parole di una sola lettera vengono ignorate;
- per tornare all'elenco completo basta svuotare il campo.

## La pagina di un servizio

![Pagina di un servizio]({{ '/assets/img/servizio.png' | relative_url }})

Dall'alto verso il basso trovi:

| Elemento | A cosa serve |
| :--- | :--- |
| **Percorso** (es. *Home / ALBERI*) | Permette di tornare alla pagina iniziale. Non è presente in tutti i servizi. |
| **Titolo** e icona **ⓘ** | L'icona apre una finestra con la descrizione estesa del servizio. |
| **Sottotitolo** | Descrive in una riga cosa mostra il servizio. |
| **Avviso** (riquadro colorato) | Comunicazioni della redazione: per esempio la frequenza di aggiornamento o un disservizio temporaneo. |
| **Contenuto del servizio** | Elenchi, mappe, calendari… cambia da servizio a servizio. |
| **Note sui dati** | Link al portale open data e all'elenco dei dataset usati. |
| **Problemi?** | Link per segnalare un disservizio al Comune. |

### Descrizione del servizio

Premendo l'icona **ⓘ** accanto al titolo si apre la descrizione estesa.

![Finestra di descrizione del servizio]({{ '/assets/img/modale-descrizione.png' | relative_url }})

### Da dove vengono i dati

Nel riquadro **Note sui dati** il link **Elenco dei dataset usati** mostra i dataset da cui il servizio legge le informazioni. Ogni voce porta alla pagina del dataset sul portale open data, dove è possibile scaricarlo in formato aperto.

![Elenco dei dataset usati]({{ '/assets/img/modale-dataset.png' | relative_url }})

Tutti i dati sono pubblicati con licenza **CC-BY 4.0**: si possono riutilizzare liberamente citando la fonte.

### Se i dati non si caricano

Se il portale open data non risponde, in cima alla pagina compare l'avviso rosso:

> Servizio opendata non disponibile al momento. Riprova più tardi.

Il portale ritenta automaticamente alcune volte prima di mostrarlo. Se il problema persiste, riprova dopo qualche minuto.

## Le schede degli oggetti (QR code)

Alcuni servizi hanno anche delle **schede**: pagine dedicate a un singolo oggetto (per esempio un albero) identificato da un **codice**. Sono le pagine che si aprono inquadrando i QR code sul territorio.

Nelle schede il pulsante **Segnala un disservizio** apre prima una finestra con il **codice dell'oggetto** e un pulsante per copiarlo: incollalo nel titolo o nel testo della segnalazione, così l'ufficio saprà subito a quale oggetto ti riferisci. Premi poi **Continua** per aprire il modulo di segnalazione del Comune.

## Segnalare un problema

In fondo a ogni servizio, nel riquadro **Problemi?**, il link **Segnala un disservizio** porta al modulo di segnalazione sul sito istituzionale del Comune.

## Pagina di errore

Se l'indirizzo non corrisponde a nessun servizio (per esempio un link vecchio o scritto male) compare la pagina *"Qualcosa non va…"* con il link **Torna alla home**.

![Pagina di errore]({{ '/assets/img/errore-404.png' | relative_url }})

## Accessibilità

Il portale è stato sviluppato secondo le linee guida di design per i servizi della Pubblica Amministrazione ed è utilizzabile con **lettori di schermo** e **da tastiera** (`Tab` per spostarsi, `Invio` per attivare, `Esc` per chiudere le finestre). La [dichiarazione di accessibilità](https://form.agid.gov.it/view/eb222cc0-b937-11ef-92a6-495355065ba9) è raggiungibile dal piè di pagina.
