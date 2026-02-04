# Test Plan: COBOL Account System

This test plan documents test cases to validate the current COBOL account application business logic. Leave `Actual Result` and `Status` blank when executing tests; capture outcomes during validation with stakeholders.

| Test Case ID | Test Case Description | Pre-conditions | Test Steps | Expected Result | Actual Result | Status (Pass/Fail) | Comments |
|---|---|---|---|---|---|---|---|
| TC-01 | View initial balance | Program started; no operations executed (fresh start) | 1) Select menu option `1` (View Balance) | Displays current balance equal to initial value 1000.00 |  | Not executed | Verify displayed value formatting (cents) |
| TC-02 | Credit a valid amount | Program started; current balance 1000.00 | 1) Select menu option `2` (Credit)
2) Enter `250.50` when prompted | Balance increases by 250.50; new balance = 1250.50 and is displayed; subsequent `View Balance` shows 1250.50 |  | Not executed | Confirms write-back to storage within same run |
| TC-03 | Debit a valid amount with sufficient funds | Program started; ensure balance >= debit amount (e.g., 1250.50) | 1) Select menu option `3` (Debit)
2) Enter `100.00` | Balance decreases by 100.00; new balance displayed and persisted (e.g., 1150.50) |  | Not executed | Validates subtraction and write-back |
| TC-04 | Debit amount when insufficient funds | Program started; balance lower than requested debit | 1) Select `3` (Debit)
2) Enter a value greater than current balance (e.g., balance=100, debit=1000) | Program displays: "Insufficient funds for this debit."; balance remains unchanged |  | Not executed | Verify no write occurs on insufficient funds |
| TC-05 | Sequence: multiple operations (credit then debit) | Program started; initial balance known | 1) Credit 200.00
2) Debit 50.00
3) View balance | Final balance equals initial +200 -50; operations applied in order and persisted between calls within the same run |  | Not executed | Confirms state shared across calls during runtime |
| TC-06 | Menu invalid selection handling | Program started | 1) Enter an invalid menu choice (e.g., `9` or non-numeric) | Program displays: "Invalid choice, please select 1-4." and returns to menu loop |  | Not executed | COBOL currently displays an invalid choice message in `WHEN OTHER` branch |
| TC-07 | Non-numeric amount input handling | Program started; choose credit or debit | 1) Select `2` or `3`
2) Enter a non-numeric value (e.g., `abc`) when prompted for amount | Expected (desired): input rejected and user re-prompted or error message shown; Current implementation: behavior may vary—capture Actual Result. |  | Not executed | Node.js port should validate and reject non-numeric input explicitly |
| TC-08 | Negative amount input handling | Program started; choose credit or debit | 1) Select `2` or `3`
2) Enter a negative value (e.g., `-50.00`) | Expected (desired): reject negative amounts; Current implementation: numeric PIC may not accept sign—capture Actual Result. |  | Not executed | Important: prevent negative credits/debits in migrated app |
| TC-09 | Amount precision and rounding | Program started | 1) Enter amount with two decimals (e.g., `10.75`) and with more than two decimals (e.g., `10.756`) | Values with two decimals should be handled exactly; values with >2 decimals should be rounded/truncated consistently—capture Actual Result. |  | Not executed | PIC 9(6)V99 supports two decimals; migration must define rounding rules |
| TC-10 | Large amount handling / upper limit | Program started | 1) Attempt to credit an amount near or above field capacity (e.g., `1000000.00`) | Expected: application rejects amounts exceeding 999999.99 or returns an error; Current behavior: capture Actual Result. |  | Not executed | PIC 9(6)V99 limits integer part to 6 digits (max 999999.99) |
| TC-11 | Operation-code/interop contract | When invoking `Operations` program externally (or via refactor) | 1) Call `Operations` with exact expected operation code strings and with trimmed/different-case variants | Expected: current programs require exact 6-character operation codes (e.g., `'TOTAL '`, `'DEBIT '`); mismatched codes may be ignored. Capture Actual Result. |  | Not executed | Important for designing interop in Node.js—normalize and validate operation codes |
| TC-12 | Persistence across restarts | Start program, perform writes, then restart program | 1) Credit 100.00
2) Exit program
3) Restart program and view balance | Expected (current): balance resets to initial 1000.00 because storage is in-memory; Node.js migration should define persistent behavior |  | Not executed | Confirms in-memory-only persistence in COBOL sample |
| TC-13 | Exit behavior | Program started | 1) Select `4` to exit | Program displays exit message and terminates (`STOP RUN`) |  | Not executed | Verify graceful shutdown and no background state persisted |


> Notes for stakeholders:

- `Actual Result` and `Status` should be filled during execution. Use `Comments` to capture environment, terminal input quirks, or differences between expected and observed behavior.
- This plan intentionally distinguishes between "Expected (desired)" behavior for a production system and the current COBOL sample behavior when the latter is unspecified (input validation, persistence). Use these distinctions to decide acceptance criteria for the Node.js migration.
