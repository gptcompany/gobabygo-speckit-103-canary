# Implementation Plan: normalize_label

**Branch**: `001-normalize-label` | **Date**: 2026-09-02 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/001-normalize-label/spec.md`

## Summary

Introdurre una libreria Python minima che espone `normalize_label(text) -> str`: rimuove il
whitespace agli estremi e comprime ogni run di whitespace interno in un singolo `U+0020`.
L'approccio tecnico scelto è la forma canonica `" ".join(text.split())`, preceduta da una
validazione esplicita del tipo che solleva `TypeError` sugli input non-`str`.

Questa scelta soddisfa contemporaneamente FR-002..FR-006 e FR-008 senza espressioni regolari:
`str.split()` senza argomenti usa esattamente la definizione Unicode di whitespace richiesta da
FR-002 (verificata sul runtime: `" ".isspace()` è `True`, `"​".isspace()` è `False`),
scarta i token vuoti agli estremi e collassa i run interni in un solo passaggio lineare.
L'idempotenza (FR-008) discende direttamente: l'output non contiene né whitespace agli estremi né
run interni, quindi un secondo passaggio è l'identità.

## Technical Context

**Language/Version**: Python 3.10+ (runtime di sviluppo verificato: CPython 3.14.4)

**Primary Dependencies**: nessuna dipendenza di runtime; solo la standard library

**Storage**: N/A — funzione pura, nessuno stato persistente

**Testing**: pytest e pytest-cov (uniche dipendenze di sviluppo; presenti sulla workstation:
pytest 9.0.2, coverage 7.15.4)

**Target Platform**: Linux (workstation di sviluppo e runner GitHub Actions `ubuntu-latest`)

**Project Type**: libreria Python a progetto singolo

**Performance Goals**: complessità lineare O(n) sulla lunghezza dell'input; nessun target
throughput specifico, l'input atteso è una singola etichetta di lunghezza tipica < 1 KB

**Constraints**: nessuna dipendenza esterna di runtime; nessun I/O; nessuno stato globale

**Scale/Scope**: una funzione pubblica, un modulo, un file di test

## Constitution Check

*GATE: deve passare prima della Phase 0. Ricontrollato dopo la Phase 1.*

`.specify/memory/constitution.md` è ancora il template Spec Kit 1.0.3 non ratificato: contiene
esclusivamente placeholder (`[PRINCIPLE_1_NAME]`, `[GOVERNANCE_RULES]`, …) e nessun principio di
progetto. Non impone quindi alcun gate specifico verificabile. La ratifica della constitution è
fuori dallo scope di questa feature e non viene simulata.

In assenza di principi ratificati si applicano i gate generici del template, entrambi soddisfatti:

- **Semplicità / YAGNI**: un solo modulo, una sola funzione pubblica, zero dipendenze di runtime,
  nessuna astrazione intermedia. Nessuna deviazione da giustificare.
- **Test-First**: la Phase 2 impone evidenza RED osservabile prima di qualsiasi edit al codice di
  produzione (vedi "Strategia di test").

**Esito**: PASS (nessun gate di progetto applicabile; gate generici soddisfatti).

## Project Structure

### Documentation (this feature)

```text
specs/001-normalize-label/
├── spec.md              # Specifica della feature
├── plan.md              # Questo file
├── tasks.md             # Elenco task eseguibili
└── github-ledger.json   # Binding del work ledger derivato (GitHub Issues)
```

`research.md`, `data-model.md`, `contracts/` e `quickstart.md` non vengono prodotti: non esistono
incognite tecnologiche da ricercare, entità dati da modellare né contratti di interfaccia oltre
alla singola firma di funzione, già fissata in FR-001.

### Source Code (repository root)

```text
pyproject.toml           # Metadati progetto + configurazione pytest (pythonpath = ["src"])
src/
└── labels.py            # normalize_label(text)
tests/
└── unit/
    └── test_labels.py   # Test unitari degli acceptance scenario e degli invarianti
```

Layout a progetto singolo. `pyproject.toml` imposta `[tool.pytest.ini_options] pythonpath = ["src"]`
così che i test importino `labels` senza installazione del pacchetto e senza `conftest.py` di
supporto: una sola riga di configurazione al posto di un layer di indirezione.

## Invarianti critici

Sono i tre invarianti che la review e il budget di mutazione devono coprire:

- **INV-1 (bordi)**: l'output non inizia né termina con un carattere whitespace.
- **INV-2 (separatori)**: l'output non contiene mai due caratteri whitespace consecutivi e ogni
  separatore interno è esattamente `U+0020`.
- **INV-3 (contenuto)**: la sequenza dei token non-whitespace dell'output è identica, per contenuto
  e ordine, a quella dell'input.

INV-1 + INV-2 + INV-3 implicano l'idempotenza di FR-008.

## Strategia di test

- **TDD**: obbligatorio per il task che modifica il comportamento (T002). Evidenza RED osservabile
  richiesta prima di qualsiasi edit a `src/labels.py`, poi il cambiamento GREEN minimo.
- **Comando di test (GREEN)**: `python3 -m pytest tests/unit/test_labels.py -v`
- **Comando di coverage (gate SC-004)**:
  `python3 -m pytest tests/unit/test_labels.py --cov=labels --cov-report=term-missing --cov-fail-under=100`
  Fallisce con exit code diverso da 0 se la copertura di `src/labels.py` scende sotto il 100%.
- **Budget di mutazione**: 1 mutazione rappresentativa per invariante critico → 3 totali, il
  default. Ogni mutazione deve far fallire almeno un test:
  1. INV-1: sostituire `" ".join(text.split())` con `" ".join(text.split(" "))` (non rimuove i
     bordi in presenza di whitespace multiplo, e non tratta tab/newline come separatori).
  2. INV-2: sostituire il separatore `" "` con `"  "` (due spazi).
  3. INV-3: aggiungere un `sorted(...)` sui token (altera l'ordine preservando il multiinsieme).

  Per ciascuna delle tre mutazioni va catturato l'output pytest fallito (evidenza di sensibilità
  richiesta da SC-003) e la mutazione va poi revertita, lasciando l'albero di lavoro identico allo
  stato GREEN. Le mutazioni sono evidenza di qualità dei test, non un contatore da massimizzare:
  ogni espansione oltre le tre previste richiede una modalità di fallimento concreta e non coperta.
- **Coverage attesa**: 100% delle righe di `src/labels.py`, imposta dal gate `--cov-fail-under=100`
  (SC-004).
- **Contratto sul messaggio d'errore**: il `TypeError` di FR-007 deve contenere `normalize_label` e
  il nome del tipo ricevuto, asserito con `pytest.raises(TypeError, match=...)`.

## Milestone e ordinamento delle dipendenze

1. **M1 — Scaffolding (T001)**: `pyproject.toml` e struttura delle directory. Nessun comportamento
   applicativo. Sblocca l'esecuzione di pytest.
2. **M2 — Funzione e test (T002)**: dipende da M1. Ciclo TDD completo su `src/labels.py` e
   `tests/unit/test_labels.py`.

Le milestone sono strettamente sequenziali: T002 non può produrre evidenza RED valida finché pytest
non è eseguibile, quindi non esiste parallelismo sfruttabile.

## Controlli di rischio

| Rischio | Impatto | Controllo |
| --- | --- | --- |
| Semantica di whitespace fraintesa (NBSP incluso o escluso per errore) | Output errato e silenzioso su dati reali | FR-002 fissa la definizione; un test dedicato copre `U+00A0` e uno copre `U+200B` come non-whitespace |
| Implementazione via regex `\s+` divergente da `str.split()` | Comportamento diverso su whitespace Unicode | Il piano impone `str.split()`; la review verifica l'aderenza |
| Coercizione implicita di input non-`str` | Errori del chiamante mascherati | FR-007 impone `TypeError`; test dedicato su `None`, `int`, `list` |
| Test scritti dopo l'implementazione (TDD apparente) | Test che non possono fallire | Evidenza RED osservabile richiesta e verificata dal reviewer nello scope immutabile |
| `pythonpath` non configurato | Test non eseguibili in CI | T001 è prerequisito bloccante di T002; `pythonpath` è dimostrabile solo quando esiste un modulo da importare, quindi la sua verifica è esplicitamente assegnata a T002 (import di `labels`) |
| Copertura dichiarata ma non misurata | SC-004 chiuso senza evidenza | Gate `--cov-fail-under=100` nel comando di done criteria di T002 |

## Rollback

Ogni task è un commit isolato su un branch non-default. Il rollback è un `git revert` del commit
corrispondente: non esistono migrazioni, stato persistente, feature flag o consumatori a valle,
quindi non c'è alcun percorso di rollback parziale da progettare.

## Fuori scope del piano

- Ratifica della constitution del progetto.
- Pubblicazione del pacchetto su un indice (PyPI o interno).
- Workflow CI di test aggiuntivi oltre al caller del ledger Spec Kit già gestito dal runtime.
