# Guida a Comune in Chiaro

Sorgenti della guida pubblicata su
**https://comunemontelupofiorentino.github.io/comune-in-chiaro-guide.github.io/**

La guida documenta [Comune in Chiaro](https://github.com/ComuneMontelupoFiorentino/comune-in-chiaro),
il portale del Comune di Montelupo Fiorentino per consultare gli open data comunali attraverso interfacce semplificate.

## Struttura

| File | Contenuto |
| --- | --- |
| `index.md` | Pagina iniziale |
| `docs/*.md` | Capitoli della guida (l'ordine nel menu è dato da `nav_order`) |
| `assets/img/` | Screenshot |
| `_config.yml` | Configurazione Jekyll e tema [Just the Docs](https://just-the-docs.com/) |

## Modificare la guida

1. Modifica i file `.md` (anche direttamente dall'interfaccia di GitHub: ogni pagina ha il link "Modifica questa pagina su GitHub").
2. Fai commit su `main`: GitHub Pages ripubblica il sito in un paio di minuti.

Anteprima in locale (facoltativa, serve Ruby):

```bash
bundle install
bundle exec jekyll serve
# poi apri http://127.0.0.1:4000/comune-in-chiaro-guide.github.io/
```

## Licenza

Contenuti rilasciati con licenza [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.it).
