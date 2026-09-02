# Feature Specification: normalize_label

**Feature Branch**: `001-normalize-label`

**Created**: 2026-09-02

**Status**: Ready for planning

**Input**: User description: "Una funzione Python normalize_label(text) che trimma gli estremi e comprime ogni run di whitespace interno in un singolo spazio, con test."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Normalizzare un'etichetta testuale (Priority: P1)

Chi integra la libreria riceve etichette testuali da sorgenti eterogenee (form, CSV, scraping,
copia-incolla). Le stesse etichette logiche arrivano con spaziatura incoerente: spazi iniziali o
finali, tabulazioni, a capo, doppi spazi tra le parole. L'integratore chiama `normalize_label(text)`
e ottiene una forma canonica in cui i token non-whitespace sono preservati nell'ordine originale e
separati da esattamente uno spazio, senza whitespace agli estremi.

**Why this priority**: È l'unica funzionalità della feature; senza di essa non esiste alcun valore
consegnabile. Rappresenta di per sé l'MVP completo.

**Independent Test**: Completamente verificabile invocando `normalize_label` su un insieme di
stringhe con spaziatura irregolare e confrontando l'output con la forma canonica attesa; non
richiede altre componenti, servizi o stato persistente.

**Acceptance Scenarios**:

1. **Given** la stringa `"  hello   world  "`, **When** si invoca `normalize_label`, **Then** il
   risultato è esattamente `"hello world"`.
2. **Given** una stringa con whitespace misto `"a\t\tb\n c"`, **When** si invoca `normalize_label`,
   **Then** il risultato è esattamente `"a b c"`.
3. **Given** una stringa già canonica `"hello world"`, **When** si invoca `normalize_label`,
   **Then** il risultato è identico all'input (idempotenza sul primo passaggio).
4. **Given** il risultato `r` di una prima invocazione, **When** si invoca `normalize_label(r)`,
   **Then** il risultato è ancora `r` (idempotenza).
5. **Given** la stringa vuota `""` oppure una stringa di soli whitespace `"   \t\n "`, **When** si
   invoca `normalize_label`, **Then** il risultato è la stringa vuota `""`.
6. **Given** un valore non stringa (es. `None`, `42`, `["a"]`), **When** si invoca
   `normalize_label`, **Then** viene sollevato `TypeError`.

---

### Edge Cases

- **Stringa vuota**: restituisce `""`, non solleva eccezioni.
- **Solo whitespace**: qualsiasi combinazione di whitespace restituisce `""`.
- **Token singolo con whitespace agli estremi**: `"  solo  "` → `"solo"`.
- **Whitespace non-ASCII**: i caratteri considerati whitespace da Unicode secondo la definizione
  di riferimento (vedi FR-002), incluso `U+00A0` NO-BREAK SPACE, sono trattati come separatori e
  quindi compressi in un singolo `U+0020`; non vengono preservati letteralmente.
- **Whitespace interno già singolo**: la funzione non altera separatori già canonici.
- **Input non stringa**: `TypeError` con messaggio esplicito; nessuna coercizione implicita
  (nessun `str()` automatico), perché coercire nasconderebbe errori del chiamante.
- **Sottoclassi di `str`**: accettate; il valore restituito è un `str` normale.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Il sistema MUST esporre una funzione pubblica `normalize_label(text)` che accetta un
  argomento posizionale e restituisce un `str`.
- **FR-002**: Il sistema MUST considerare come whitespace l'insieme di caratteri riconosciuti come
  tali dalla definizione Unicode adottata dal runtime Python (la stessa usata da `str.split()`
  senza argomenti). Questo insieme include `U+0020`, `\t`, `\n`, `\r`, `\v`, `\f` e `U+00A0`.
- **FR-003**: Il sistema MUST rimuovere ogni whitespace iniziale e finale dal valore restituito.
- **FR-004**: Il sistema MUST sostituire ogni sequenza di uno o più caratteri whitespace interni con
  esattamente un carattere `U+0020` SPACE.
- **FR-005**: Il sistema MUST preservare i token non-whitespace, invariati e nel loro ordine
  originale.
- **FR-006**: Il sistema MUST restituire `""` quando l'input è vuoto o composto solo da whitespace.
- **FR-007**: Il sistema MUST sollevare `TypeError` quando l'argomento non è un'istanza di `str`,
  senza tentare alcuna conversione.
- **FR-008**: Il sistema MUST essere idempotente: `normalize_label(normalize_label(x)) ==
  normalize_label(x)` per ogni `x` valido.
- **FR-009**: La funzione MUST essere pura: nessuno stato globale, nessun I/O, nessuna mutazione
  dell'input.

### Key Entities

Nessuna entità dati persistente. La feature espone una singola funzione pura su valori `str`.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Tutti gli Acceptance Scenarios 1-6 sono coperti da almeno un test automatico e la
  suite passa con 0 fallimenti.
- **SC-002**: La proprietà di idempotenza (FR-008) è verificata su un insieme di almeno 8 input
  eterogenei che includono whitespace agli estremi, whitespace interno multiplo, whitespace misto
  tab/newline, `U+00A0`, stringa vuota e stringa di soli whitespace.
- **SC-003**: Ognuno dei tre invarianti critici (nessun whitespace agli estremi; nessuna coppia di
  spazi consecutivi e separatori esclusivamente `U+0020`; sequenza di token preservata) è coperto da
  almeno un test che fallisce se l'invariante viene violato.
- **SC-004**: La copertura delle righe del modulo che implementa `normalize_label` è del 100%,
  misurabile perché il modulo contiene solo la funzione e la sua validazione di tipo.

## Assumptions

- Il progetto è una libreria Python pura; non esistono vincoli di prestazioni oltre a un tempo di
  esecuzione lineare rispetto alla lunghezza dell'input.
- Il runtime di riferimento è Python 3.10 o superiore, disponibile sulla workstation e sui runner
  GitHub Actions.
- La normalizzazione riguarda esclusivamente il whitespace: non sono richieste normalizzazioni
  Unicode (NFC/NFKC), case folding, rimozione di punteggiatura o traslitterazione.
- L'esposizione via CLI, l'internazionalizzazione e la gestione di input `bytes` sono fuori scope.

## Out of Scope

- Normalizzazione Unicode NFC/NFKD/NFKC e case folding.
- Rimozione o sostituzione di caratteri di controllo non-whitespace (es. `U+200B` ZERO WIDTH SPACE,
  che non è whitespace secondo FR-002 e viene quindi preservato come parte del token).
- Interfaccia a riga di comando, API HTTP, persistenza.
- Supporto per input `bytes` o oggetti file-like.
