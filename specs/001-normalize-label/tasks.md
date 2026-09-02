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

- [ ] T001 Add Python test scaffolding: create pyproject.toml at repository root declaring project name gobabygo-speckit-103-canary, requires-python >=3.10, and [tool.pytest.ini_options] with pythonpath = ["src"] and testpaths = ["tests"]; create empty directories src/ and tests/unit/ tracked via .gitkeep files. No application behaviour in this task.

**Done criteria T001**: `python3 -m pytest tests -q` termina con exit code 5 (`no tests ran`) invece di
un errore di configurazione, dimostrando che pytest legge la configurazione e trova `testpaths`.

**Checkpoint**: pytest eseguibile; l'implementazione di US1 può iniziare.

---

## Phase 2: User Story 1 - Normalizzare un'etichetta testuale (Priority: P1) 🎯 MVP

**Goal**: esporre `normalize_label(text)` che trimma gli estremi e comprime ogni run di whitespace
interno in un singolo `U+0020`, con test che coprono gli acceptance scenario e i tre invarianti
critici.

**Independent Test**: `python3 -m pytest tests/unit/test_labels.py -v` passa e copre gli
Acceptance Scenarios 1-6 di spec.md.

- [ ] T002 [US1] Implement normalize_label in src/labels.py using the TDD cycle: first write failing unit tests in tests/unit/test_labels.py covering Acceptance Scenarios 1-6 plus invariants INV-1 (no leading or trailing whitespace), INV-2 (no consecutive whitespace and separators are exactly U+0020) and INV-3 (non-whitespace token sequence preserved), capture the RED pytest output, then add the minimal implementation returning " ".join(text.split()) guarded by an isinstance(text, str) check raising TypeError, and capture the GREEN pytest output.

**Done criteria T002**:

- Evidenza RED osservabile (output pytest con almeno un fallimento) precedente a qualunque
  contenuto funzionale in `src/labels.py`.
- Evidenza GREEN: `python3 -m pytest tests/unit/test_labels.py -v` con 0 fallimenti.
- FR-001..FR-009 di spec.md soddisfatti; SC-001..SC-004 verificabili dall'output dei test.
- Nessun file modificato al di fuori di `src/labels.py` e `tests/unit/test_labels.py`.

**Checkpoint**: User Story 1 completa e testabile in modo indipendente.

---

## Dependencies

- T002 dipende da T001 (pytest deve essere eseguibile per produrre evidenza RED valida).
- Nessun task marcato `[P]`: le due fasi sono strettamente sequenziali.

## Test command

```bash
python3 -m pytest tests/unit/test_labels.py -v
```

## Mutation budget

Default: 1 mutazione rappresentativa per invariante critico (3 totali), come elencate in plan.md
alla sezione "Strategia di test". Ogni espansione richiede una modalità di fallimento concreta e non
coperta.
