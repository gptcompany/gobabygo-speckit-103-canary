---
description: "Task list for feature 001-normalize-label"
---

# Tasks: normalize_label

**Input**: Design documents from `/specs/001-normalize-label/`

**Prerequisites**: [plan.md](./plan.md) (required), [spec.md](./spec.md) (required for user stories)

**Tests**: I test sono richiesti esplicitamente dalla feature description e dai criteri SC-001..SC-004.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: eseguibile in parallelo (file diversi, nessuna dipendenza)
- **[Story]**: user story di appartenenza (US1)

## Path Conventions

Progetto singolo: `src/` e `tests/` alla radice del repository, come da plan.md.

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: rendere pytest eseguibile in modo che il ciclo TDD di US1 possa produrre evidenza RED.

- [ ] T001 Add Python test scaffolding: create pyproject.toml at repository root declaring project name gobabygo-speckit-103-canary, requires-python >=3.10, an optional dev dependency group listing pytest and pytest-cov, and a [tool.pytest.ini_options] table with pythonpath = ["src"] and testpaths = ["tests"]; create directories src/ and tests/unit/ each tracked via a .gitkeep file. No application behaviour in this task.

**Done criteria T001** (ognuno verificabile dall'output di un comando):

- `python3 -m pytest --collect-only` invocato **senza argomenti di percorso** termina con exit code
  5 (`no tests ran`) e l'intestazione pytest mostra `configfile: pyproject.toml` e
  `testpaths: tests`. Senza argomenti espliciti l'esito dipende davvero dalla configurazione, quindi
  il comando stabilisce ciò che dichiara.
- `src/.gitkeep` e `tests/unit/.gitkeep` esistono e sono tracciati da Git.
- Nessun file oltre a `pyproject.toml`, `src/.gitkeep` e `tests/unit/.gitkeep` è creato o modificato.

`pythonpath` non è dimostrabile in T001, perché non esiste ancora un modulo da importare: la sua
verifica è assegnata a T002 (l'import di `labels` da parte dei test).

**Checkpoint**: pytest eseguibile; l'implementazione di US1 può iniziare.

---

## Phase 2: User Story 1 - Normalizzare un'etichetta testuale (Priority: P1) 🎯 MVP

**Goal**: esporre `normalize_label(text)` che trimma gli estremi e comprime ogni run di whitespace
interno in un singolo `U+0020`, con test che coprono gli acceptance scenario e i tre invarianti
critici.

**Independent Test**: `python3 -m pytest tests/unit/test_labels.py -v` passa e copre gli
Acceptance Scenarios 1-6 di spec.md.

- [ ] T002 [US1] Implement normalize_label in src/labels.py using the TDD cycle: first write failing unit tests in tests/unit/test_labels.py covering Acceptance Scenarios 1-6, the three invariants INV-1 (no leading or trailing whitespace), INV-2 (no consecutive whitespace and separators are exactly U+0020) and INV-3 (non-whitespace token sequence preserved), at least 8 parametrized idempotence cases, and a pytest.raises(TypeError, match=...) assertion requiring the message to contain normalize_label and the received type name; capture the RED pytest output; then add the minimal implementation returning " ".join(text.split()) guarded by an isinstance(text, str) check raising TypeError; capture the GREEN output, the coverage output and the three mutation failures.

**Done criteria T002** (ognuno verificabile dall'output di un comando):

1. **RED**: output di `python3 -m pytest tests/unit/test_labels.py -v` con almeno un fallimento,
   catturato **prima** che `src/labels.py` contenga qualsiasi logica di normalizzazione.
2. **GREEN**: `python3 -m pytest tests/unit/test_labels.py -v` con 0 fallimenti.
3. **Idempotenza osservabile (SC-002)**: l'output `-v` elenca almeno 8 test id distinti per i casi
   di idempotenza parametrizzati.
4. **Coverage (SC-004)**:
   `python3 -m pytest tests/unit/test_labels.py --cov=labels --cov-report=term-missing --cov-fail-under=100`
   esce con codice 0.
5. **Sensibilità agli invarianti (SC-003)**: per ciascuna delle tre mutazioni elencate in plan.md,
   output pytest fallito catturato, poi mutazione revertita; a fine task `git diff` non contiene
   alcuna mutazione residua.
6. **Contratto d'errore (FR-007)**: il test con `pytest.raises(TypeError, match=...)` passa per
   `None`, `int` e `list`.
7. Nessun file modificato al di fuori di `src/labels.py` e `tests/unit/test_labels.py`.

**Checkpoint**: User Story 1 completa e testabile in modo indipendente.

---

## Dependencies

- T002 dipende da T001 (pytest deve essere eseguibile per produrre evidenza RED valida).
- Nessun task marcato `[P]`: le due fasi sono strettamente sequenziali.

## Test commands

```bash
python3 -m pytest tests/unit/test_labels.py -v
python3 -m pytest tests/unit/test_labels.py --cov=labels --cov-report=term-missing --cov-fail-under=100
```

## Mutation budget

Default: 1 mutazione rappresentativa per invariante critico (3 totali), come elencate in plan.md
alla sezione "Strategia di test". Ogni espansione richiede una modalità di fallimento concreta e non
coperta.
