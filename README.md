# Bank Practice

**Python, MySQL, and Tkinter practice project demonstrating SQL CRUD operations and account-balance updates.**

I built this banking application to practice connecting a Python interface to a relational database. The database functions create account records, retrieve account details, update balances, and delete matching records. The code links below make those SQL operations easy to review.

## Database code at a glance

| Operation | Direct code link | What it demonstrates |
| --- | --- | --- |
| CREATE | [Create an account](https://github.com/AlexV4981/Bank-Practice/blob/f05323ede62b544010582de0691fe8d082b46fbd/Bank%20Project/bank.py#L8-L27) | Parameterized INSERT for name, account number, PIN, and initial balance; explicit commit |
| READ | [Lookup by account number](https://github.com/AlexV4981/Bank-Practice/blob/f05323ede62b544010582de0691fe8d082b46fbd/Bank%20Project/bank.py#L85-L111) | Parameterized SELECT with an account-number WHERE condition and fetchone() |
| READ | [Lookup by name and PIN](https://github.com/AlexV4981/Bank-Practice/blob/f05323ede62b544010582de0691fe8d082b46fbd/Bank%20Project/bank.py#L140-L165) | Matching records with two bound conditions: name AND PIN |
| UPDATE | [Deposit funds](https://github.com/AlexV4981/Bank-Practice/blob/f05323ede62b544010582de0691fe8d082b46fbd/Bank%20Project/bank.py#L32-L52) | Arithmetic UPDATE that adds an amount to the stored balance, followed by a SELECT of the resulting balance |
| UPDATE | [Withdraw funds](https://github.com/AlexV4981/Bank-Practice/blob/f05323ede62b544010582de0691fe8d082b46fbd/Bank%20Project/bank.py#L53-L84) | Reading a balance, checking whether it covers a requested withdrawal, then subtracting the amount and committing |
| DELETE | [Delete a matching account](https://github.com/AlexV4981/Bank-Practice/blob/f05323ede62b544010582de0691fe8d082b46fbd/Bank%20Project/bank.py#L112-L139) | Parameterized DELETE with three matching conditions, followed by a rowcount check |

**Suggested review order:** account creation → deposit → withdrawal → deletion. A reviewer can inspect these functions without installing MySQL or running the GUI.

## Database structure

The application connects to a MySQL database named `bank` and uses a table named `bank_data`.

| Column | How the application uses it |
| --- | --- |
| `name` | Account holder name; used in lookup and deletion conditions |
| `account_number` | Account identifier; used to retrieve and update records |
| `pin` | Practice PIN field; used in lookup and deletion conditions |
| `balance` | Stored account balance, initialized to zero and modified by deposits and withdrawals |

The source expects SELECT * results in the order shown above. A CREATE TABLE script is not included, so this repository does not establish the table's actual types, keys, or constraints.

The [Account number generator](https://github.com/AlexV4981/Bank-Practice/blob/f05323ede62b544010582de0691fe8d082b46fbd/Bank%20Project/bank.py#L28-L31) produces an eight-digit string. The code does not check for collisions or demonstrate a database uniqueness constraint.

## SQL implementation highlights

### Parameterized statements

The database functions use MySQL Connector/Python's `%s` placeholders and pass the values separately to `cursor.execute()`. For example, [Create an account](https://github.com/AlexV4981/Bank-Practice/blob/f05323ede62b544010582de0691fe8d082b46fbd/Bank%20Project/bank.py#L8-L27) inserts a new record with:

```sql
INSERT INTO bank_data (name, account_number, pin, balance)
VALUES (%s, %s, %s, %s)
```

[Delete a matching account](https://github.com/AlexV4981/Bank-Practice/blob/f05323ede62b544010582de0691fe8d082b46fbd/Bank%20Project/bank.py#L112-L139) restricts deletion using multiple conditions:

```sql
DELETE FROM bank_data
WHERE name = %s AND account_number = %s AND pin = %s
```

The deletion function checks `cursor.rowcount` to distinguish a matching deletion from no matching account.

### Updating and checking balances

[Deposit funds](https://github.com/AlexV4981/Bank-Practice/blob/f05323ede62b544010582de0691fe8d082b46fbd/Bank%20Project/bank.py#L32-L52) updates the stored balance with arithmetic in SQL:

```sql
UPDATE bank_data
SET balance = balance + %s
WHERE account_number = %s
```

It then selects the balance for display in the terminal. [Withdraw funds](https://github.com/AlexV4981/Bank-Practice/blob/f05323ede62b544010582de0691fe8d082b46fbd/Bank%20Project/bank.py#L53-L84) first selects the current balance and checks it in Python before issuing the corresponding subtraction UPDATE.

These are account-balance operations in a learning application. The read/check/write withdrawal sequence does not include row locking or an atomic conditional update for concurrent requests.

### Persistence and connection handling

Mutation functions explicitly call `connection.commit()`. Functions open their own MySQL connections and close their cursors and connections on the normal execution path. The deletion function catches MySQL errors; the application does not yet provide consistent exception-safe cleanup or explicit rollback handling across operations.

## Project files

| File | Purpose |
| --- | --- |
| [bank.py](Bank%20Project/bank.py) | Database operations, balance logic, account lookup, and number generation |
| [bank_GUI.py](Bank%20Project/bank_GUI.py) | Tkinter windows and callbacks for the banking operations |

## Running and reviewing the project

The intended GUI entry point is:

```bash
cd "Bank Project"
python -m pip install mysql-connector-python
python bank_GUI.py
```

A local MySQL server, the `bank` database, a compatible `bank_data` table, and Python with Tkinter are required. Use disposable test records only. Database connection settings currently live in the source files; use a local development account and do not reuse the committed credential.

**The GUI requires fixes before it is a reliable end-to-end demo:**

- Its lookup callback calls `bank.getAccountInfo()`, which is not defined in `bank.py`.
- The account display callback expects a returned record, but `displayAccountInfoN()` currently prints the information without returning it.
- The window-close registration calls `closeBank()` immediately instead of registering the function as a callback.
- Several GUI success messages are not based on a returned database-operation result.
- Deletion asks for terminal confirmation, but does not use the response to guard the DELETE.

The SQL functions are the main review focus of this version. No automated test suite or schema setup script is included, and these instructions are not a claim of a verified working GUI.

## Scope and next improvements

This is a database learning project, not software for real financial accounts.

- Move connection settings out of source and use a database user with limited privileges.
- Add a reproducible schema and account-number uniqueness enforcement.
- Validate positive amounts and PIN format; use decimal-safe money handling.
- Hash PINs and remove PIN display and terminal output.
- Align GUI callbacks with explicit return values from database functions.
- Add transaction/error handling, concurrency protection, and tests for missing accounts, invalid input, and insufficient funds.

The current code demonstrates **MySQL SQL operations and Python integration**. Oracle/PLSQL, a transaction ledger, and production data-correction workflows are not implemented.
