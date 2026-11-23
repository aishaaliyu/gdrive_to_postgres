# Google Drive to PostgreSQL

A Python ETL project that downloads data from Google Drive, cleans it, and loads it into PostgreSQL. It supports CSV files and Google Sheets, with cleaning rules for customer, account, and transaction data.

**Tools:** Python 3.12, pandas, PostgreSQL, SQLAlchemy, and Google Drive API.

## What it does

1. Downloads files from a Google Drive folder.
2. Cleans column names, removes empty rows and exact duplicates, and formats dates and amounts for the supported datasets.
3. Loads the cleaned data into PostgreSQL's `staging` schema, with the source filename and a UTC timestamp.

## Setup

You need Python 3.12 and a running PostgreSQL database. Your database user must be able to create schemas and tables, insert rows, and delete rows.

### 1. Install the project

Open a terminal in the project folder and create a virtual environment:

```bash
python -m venv venv
```

Activate the virtual environment:

```bash
# macOS / Linux
source venv/bin/activate
```

Install the dependencies:

```bash
python -m pip install -r requirements.txt
```

### 2. Set up Google Drive access

1. Enable the **Google Drive API** in your Google Cloud project.
2. Create a service account and download its JSON key.
3. Save the key as `credentials/service-account.json` inside the project folder.
4. Share your Google Drive folder with the service account's email address (`client_email` in the JSON key), giving it **Viewer** access.
5. Copy the folder ID from its Google Drive URL.

### 3. Add your configuration

Create a `.env` file in the project root and replace the example values:

```dotenv
DRIVE_FOLDER_ID=your_folder_id
SERVICE_ACCOUNT_FILE=credentials/service-account.json

DB_HOST=localhost
DB_PORT=5432
DB_NAME=your_database_name
DB_USER=your_database_user
DB_PASSWORD="your_database_password"
```

The database must already exist. The pipeline creates the `staging` schema and missing tables automatically.

Keep `.env` and your service-account key private. Both are excluded by the project's `.gitignore` settings.

## Prepare your files

Place files directly inside the shared Drive folder. Use these filenames to apply the built-in cleaning rules:

| File | Required columns |
| --- | --- |
| `customers.csv` | `full_name`, `email_address`, `join_date` |
| `accounts.csv` | `account_status`, `opening_balance`, `open_date` |
| `transactions.csv` | `transaction_date`, `amount`, `description`, `channel`, `transaction_type` |

Headers are normalized automatically: for example, `Full Name` becomes `full_name`.

For Google Sheets, use the same names without `.csv`, such as `customers`. Only the first worksheet is exported. Other CSV filenames receive general cleaning only.

Transaction types are case-sensitive: `Deposit`, `Refund`, `Purchase`, `Withdrawal`, `Transfer`, and `Fee`. Supply outgoing amounts as positive numbers; the pipeline creates their negative `signed_amount` values.

## Run the pipeline

From the project root, run:

```bash
python src/main.py
```

The example datasets load into `staging.customers`, `staging.accounts`, and `staging.transactions`. Downloaded files are saved in `data/raw/`. Check the terminal logs for row counts and processing errors.

## Exit the virtual environment

When you are finished, run:

```bash
deactivate
```

## Notes

- Nested folders are not processed. Use unique filenames for your source files.
- Reruns delete previously loaded rows for the same filename before inserting replacements. If insertion fails, those previous rows may already be removed.
