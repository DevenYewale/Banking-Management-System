# Banking Management System

A Java desktop application for basic banking workflows, built with Swing and a MySQL database. The project brings customer registration, account creation, balance enquiry, deposits, withdrawals and account-to-account transfers into one desktop interface.

This is an educational project for Object Oriented Programming (Java), developed for the 2024-25 academic year. It demonstrates event-driven GUI programming, JDBC database access, SQL queries and input validation. It is a coursework demo, not software for handling real bank accounts or real customer data.

## Features

- **Login:** checks a username and password against the database before opening the main menu.
- **Customer registration:** saves a customer's name, address, branch and phone number, with required-field and digits-only phone checks.
- **Account creation:** links an account to an existing customer ID, offers Saving, Fix and Current account types, and checks that the opening balance is numeric and non-negative.
- **Balance enquiry:** looks up an account and displays the customer's name and current balance.
- **Deposits:** records a deposit and updates the account balance, with positive-amount validation.
- **Withdrawals:** records a withdrawal and updates the balance, with positive-amount and displayed-balance checks.
- **Fund transfers:** debits one account, credits another and writes a transfer record. The form checks required values, a positive whole-number amount and the displayed source balance.
- **Customer report:** displays customer details and branch information in a table.
- **User management:** lists application users and allows user creation and deletion.

These describe the workflows implemented in the source. They are not a claim that every workflow has passed end-to-end testing.

## Technology

- **Java:** application logic, event handlers and desktop entry point.
- **Swing:** windows, internal frames, forms, tables and message dialogs.
- **JDBC:** database connections, prepared statements and result sets.
- **MySQL:** customer, account, user and transaction records.
- **NetBeans GUI Builder:** accompanying `.form` files store the visual form definitions.

The archive contains Java source and GUI form files. It does not include a Maven or Gradle build, NetBeans project metadata, a JDBC driver JAR, a database export or an automated test suite.

## How It Works

### 1. Sign in and open the desktop

`bank.login` is the login entry point. A successful database lookup opens `mainmenu`, which hosts the banking screens inside a `JDesktopPane`. Each task opens in its own `JInternalFrame`.

### 2. Register customers and create accounts

The customer form loads branches from the database and generates a customer reference from the current maximum ID. The account form looks up a customer ID, then saves an account type and opening balance. Account references are also generated from existing records.

### 3. Look up balances and record transactions

The balance, deposit and withdrawal screens join customer and account records to show the account holder and balance. Deposit and withdrawal actions write a transaction row and update the account balance. The transfer screen updates both account balances and inserts a transfer record using an explicit JDBC transaction.

### 4. Review records

The customer report loads a customer-and-branch query into a Swing table. The user-management screen reads from the application-user table and supports adding and deleting login records. It does not implement a separate role or permission model.

## Banking Workflow in Detail

### Customer registration

The customer screen collects first name, last name, street, city, branch and phone number. Its branch selector is populated from the `branch` table rather than a fixed list. A small inner `Branch` class keeps a branch ID with its display name, so the form can show a readable branch and save its database reference.

Before saving, the handler checks that the required fields are filled and that the phone contains digits. It writes a new `customer` row through a prepared statement, clears the fields and generates the next customer reference. This is registration, not a complete customer-editing or deletion module.

### Account creation and enquiry

In the account form, entering a customer ID and pressing Enter runs a lookup and displays the customer's first name. The user then selects an account type and supplies an opening balance. The saved account holds the customer reference, account reference, type and balance.

The balance-enquiry screen takes an account number, joins the corresponding customer and account records, and shows the holder's first name, last name and balance. An unsuccessful lookup shows an invalid-account message and clears the displayed name and balance.

### Deposits and withdrawals

Both forms begin with an account lookup. They show the linked customer, balance and a date formatted as `yyyy/MM/dd`. A deposit creates a row containing the account ID, customer ID, date, displayed starting balance and deposit amount, then adds the amount to the account balance. A withdrawal records the equivalent withdrawal details and subtracts the amount.

The withdrawal form rejects an amount above the balance currently shown in the form. These checks help demonstrate basic validation, but they do not replace database-level financial controls or the transaction-handling fixes described below.

### Transfers

The transfer screen uses a source account, destination account, source balance and amount. It rejects the same account on both sides and requires a positive whole-number amount no greater than the displayed source balance. After validation, it starts a JDBC transaction, subtracts from the source, adds to the destination and inserts a `transfer` row before committing. Its SQL-error handler attempts a rollback.

This implementation demonstrates a multi-statement transaction. It is not a guarantee that every invalid destination or concurrent balance change is handled correctly.

### Reports and application users

The customer report loads a table with a database ID, customer name, street, city, branch and phone. It has no handler for saving edits back to the database. The user-management table lists application-user IDs and usernames. Adding a user saves the username and password; deleting a user requires a selected table row and confirmation. The screen also contains a name field, but the insert query does not save that field.

## Code Organisation and Database Model

The application is organised by screen rather than by a separate service or repository layer. Each screen owns its Swing components, event handlers and database queries. The main menu creates screen objects and adds them to the desktop container.

The implementation uses inheritance from Swing's `JFrame` and `JInternalFrame` classes. Form events connect button clicks and key presses to Java methods. JDBC `Connection`, `PreparedStatement` and `ResultSet` objects connect those methods to MySQL. Parameterised statements bind form input for lookups, inserts, updates and deletions; the ID-generation routines also use plain statements for fixed queries.

The seven tables have distinct purposes:

- `branch` supplies branch IDs and names for registration and reporting.
- `customer` stores customer references, names, contact details and branch references.
- `account` stores account references, customer references, account types and balances.
- `deposit` stores deposit records with the date and starting balance.
- `withdraw` stores withdrawal records with the date and starting balance.
- `transfer` stores source account, destination account and amount.
- `user` stores application login records.

The source links customer and account records by `cust_id` and uses account references for banking actions. This describes the queries and supplied report's schema screenshots; it does not claim that foreign-key constraints or other database protections have been verified.

## Project Structure

All classes are in the `bank` package. Each Java file has a matching `.form` file.

- `bank/login.java` - login window and application entry point.
- `bank/mainmenu.java` - desktop container and menu navigation.
- `bank/customer.java` - customer registration and branch selection.
- `bank/account.java` - customer lookup and account creation.
- `bank/balance.java` - account balance enquiry.
- `bank/deposit.java` - deposit form and database updates.
- `bank/withdraw.java` - withdrawal form and database updates.
- `bank/transfer.java` - account-to-account transfer form.
- `bank/cusreport.java` - customer details table.
- `bank/user1.java` - application-user listing, creation and deletion.

## Local Setup

### Requirements

- A Java Development Kit with `javac` and `java`.
- A local MySQL server.
- A MySQL Connector/J JAR compatible with the source's legacy `com.mysql.jdbc.Driver` class name.
- Optionally, NetBeans for editing the forms visually.

No specific JDK, Connector/J or MySQL version has been verified for a clean installation of this source archive.

### Database preparation

The source expects a database named `sanbank` on `localhost:3306`, with these tables: `account`, `branch`, `customer`, `deposit`, `transfer`, `user` and `withdraw`.

A SQL creation script and seed data are **not included**. Restore the original database or create a compatible schema before trying the application. Existing branch records and an application-user record are needed for the relevant screens and login. The Java queries show the column names each screen expects.

Database table-name casing also needs care: some balance updates use `Account`, while other queries use `account`. This can matter on case-sensitive MySQL installations.

### Database credentials

Set `BANK_DB_USER` and `BANK_DB_PASSWORD` in the environment that launches Java. Use credentials for an isolated local demonstration database. Do not commit passwords or real customer data.

For example, in PowerShell:

```powershell
$env:BANK_DB_USER = "your_local_database_user"
$env:BANK_DB_PASSWORD = "your_local_database_password"
```

These are placeholders, not supplied login credentials. Database credentials are separate from the username and password entered in the application's login screen.

### Compile and launch

From the repository root, compile the package and launch the login window with Connector/J on the runtime classpath. Replace the JAR path with the file installed locally.

```text
javac -d out bank/*.java
```

Windows:

```text
java -cp "out;path/to/mysql-connector.jar" bank.login
```

macOS / Linux:

```text
java -cp "out:path/to/mysql-connector.jar" bank.login
```

These are manual setup instructions, not a verified fresh-install procedure. The database schema and driver must be available before database-backed workflows can work.

## Scope and Limitations

- **Educational use only.** The application stores and compares application passwords as plain text. It has no password hashing or role-based access control.
- **Transaction handling needs review.** Deposit and withdrawal handlers disable auto-commit on a connection and then replace that connection before writing. Their multiple writes should not be assumed atomic.
- **Validation is not a complete financial control.** Withdrawal and transfer checks rely on a balance displayed in the form. Concurrent changes, affected-row checks, invalid destination accounts and ID-generation races need further review.
- **Money handling is limited.** Amount handling mixes floating-point parsing, integers and database integer balances. Transfers accept whole numbers; the project does not provide a consistent decimal-money model.
- **Database access runs in GUI event handlers.** Slow queries can block the interface. Some exceptions are swallowed or logged without a clear user-facing recovery path.
- **The report describes a broader design.** Loan handling, an overdraft-enabled checking-account class and a transaction inheritance hierarchy are not implemented in this source archive and are not listed as working features here.
- **Verification is limited to source inspection.** No automated tests, compilation result or live-database test results are included.
