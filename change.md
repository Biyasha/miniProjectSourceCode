# changes.md — Summary of modifications to `trans.c`

---

## 1. Original Program Overview

`trans.c` manages 100 fixed-size records in a binary file (`credit.dat`). Each record holds an account number, first/last name, and balance. Records are accessed via `fseek` using the offset `(n-1) * sizeof(struct clientData)`.

---

## 2. New Menu Options

- **Option 5 – List all accounts**: Added `listAccounts()` to display all non-empty records.
- **Option 6 – Search and display account**: Added `displayAccount()` for random-access reads without modification.
- **"End program" moved to option 9** to accommodate new entries.

---

## 3. File Initialization

- Replaced the original `fopen("credit.dat", "rb+")` (which exited on failure) with `openDataFile()`:
  1. Tries `rb+`; if that fails, creates the file using `wb+`.
  2. On creation, calls `initializeFile()` to write 100 blank records (`acctNum = 0`, empty names, `balance = 0.0`).
- This ensures all 100 record slots are valid before any reads or writes.

---

## 4. Fixed `feof()` Bug in `textFile`

**Before:** `while (!feof(readPtr)) { fread(...); ... }` — causes an extra loop iteration.

**After:** `while (fread(...) == 1) { ... }` — loops only on successful reads.

---

## 5. Input Validation

- **`clearInputLine()`** — flushes bad input from the buffer after a `scanf` failure.
- **`getAccountNumber(prompt)`** — loops until a valid account number (1–100) is entered.
- **`getDouble(prompt)`** — loops until a valid numeric value is entered.

---

## 6. `scanf` Format Fix

Changed `%d` to `%u` wherever `unsigned int` values are read, avoiding undefined behavior.

---

## 7. Error Handling

- **`fread` return checks**: Code now verifies `fread` returns 1; prints an error and returns otherwise.
- **`fflush(fPtr)` after writes**: Added in `newRecord`, `updateRecord`, `deleteRecord`, and `initializeFile` to ensure data is saved to disk immediately.

---

## 8. New Functions

| Function | Purpose |
|---|---|
| `listAccounts()` | Rewinds file, prints all records where `acctNum != 0` |
| `displayAccount()` | Seeks to a record, reads it, prints it (or "no information" if empty) |
| `debit_transaction()` | ATM-style withdrawal — validates balance won't go negative before writing |
| `write_sorted_text_file()` | Reads all records, bubble-sorts by last/first name, writes to `accounts_sorted.txt` |

---

## 9. Minor Cleanups

- `main` now returns `0` on success and `EXIT_FAILURE` if the file can't be opened.
- Added `(void)argc;` to suppress unused parameter warnings.

---

## 10. Demo Talking Points

- **Random access**: "Account `n` maps to byte offset `(n-1) * sizeof(record)` via `fseek`."
- **Empty slot**: "`acctNum == 0` marks an unused record."
- **Why not `feof`**: "`feof` goes true only after reading past EOF — `fread(...) == 1` is the correct loop condition."
- **Input safety**: "Bad input leaves characters in the buffer; `clearInputLine()` discards them so the next prompt works correctly."

---

## 11. Rubric Alignment

| Requirement | Implementation |
|---|---|
| Code style | Renamed to `snake_case` throughout |
| Functional decomposition | Input helpers (`get_account_number`, `get_double`) separate from feature functions |
| Memory usage | Fixed arrays; sorted output uses at most 100 records |
| Processing efficiency | `fread(...) == 1` loop; O(1) record access via `fseek` |
| Negative balance limit | Enforced in `update_record` and `debit_transaction` |
| ATM debit transaction | `debit_transaction()` |
| Sorting | `write_sorted_text_file()` using bubble sort |
