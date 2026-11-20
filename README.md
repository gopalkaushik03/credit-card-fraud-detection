# Credit Card Fraud Detection

An end-to-end machine learning demonstration for classifying credit-card transactions as legitimate or fraudulent. It combines a scikit-learn training pipeline, a Flask prediction API, and a responsive browser interface.

## Overview

The application accepts five human-readable transaction attributes, sends them to a local REST API, and returns:

- A binary prediction: legitimate or fraudulent
- A fraud probability percentage
- A clear, browser-friendly result

The backend uses median imputation for numeric features and a balanced Random Forest classifier. The frontend provides a dashboard-style form and live API connection status.

## Features

- Random Forest classification with 120 estimators
- Median imputation within an sklearn pipeline
- Stratified 80/20 train-test split with a fixed random seed
- Class-balanced training for an imbalanced synthetic dataset
- Flask REST API with CORS enabled
- Responsive HTML, CSS, and JavaScript interface
- Joblib model serialization and reload support
- API input validation and JSON error responses

## Model and Dataset

The bundled `data/creditcard.csv` file contains 15,000 rows and six columns:

| Column | Type | Description |
|---|---|---|
| `TransactionAmount` | float | Amount of the current transaction |
| `TransactionFrequency` | integer | Number of recent transactions |
| `AverageTransactionAmount` | float | Average of recent transaction amounts |
| `InternationalTransaction` | binary | `0` for domestic, `1` for international |
| `TransactionHour` | integer | Hour of the transaction, from `0` to `23` |
| `Class` | binary | Target: `0` legitimate, `1` fraudulent |

The dataset contains 14,667 legitimate transactions and 333 fraudulent transactions (approximately 2.22% fraud).

### Evaluation

The held-out test set contains 3,000 transactions. The included model reports:

| Metric | Legitimate (`0`) | Fraudulent (`1`) |
|---|---:|---:|
| Precision | 0.98 | 0.06 |
| Recall | 0.01 | 0.01 |
| F1-score | 0.99 | 0.02 |
| Support | 2,933 | 67 |

- **Overall accuracy:** 97.27%
- **Random split:** 80% training / 20% testing
- **Stratification:** enabled
- **Random seed:** `42`

Accuracy is not a reliable measure for this imbalanced dataset. The low fraud precision and recall demonstrate that class-balanced training and aggregate accuracy alone do not make this model suitable for production fraud detection.

## Architecture

```text
Browser UI (frontend/)
       |
       | HTTP/JSON
       v
Flask API (app.py, port 5001)
       |
       v
sklearn Pipeline (fraud_model.pkl)
       |
       v
Random Forest predictions
```

The UI is static and communicates with the API at `http://127.0.0.1:5001`.

## Technology Stack

- Python
- pandas
- scikit-learn
- Flask
- Flask-CORS
- Joblib
- HTML, CSS, and JavaScript

## Project Structure

```text
Credit-Card-Fraud-Detection-main/
├── app.py                 # Flask REST API
├── train.py               # Training pipeline and model export
├── test.py                # Held-out evaluation script
├── fraud_model.pkl        # Serialized trained pipeline
├── requirements.txt       # Python dependencies
├── README.md
├── data/
│   └── creditcard.csv      # Synthetic demonstration dataset
└── frontend/
    ├── index.html          # Dashboard interface
    ├── script.js           # API requests and result rendering
    └── style.css           # Responsive application styling
```

## Getting Started

### Prerequisites

- Python 3.9 or newer
- `pip`
- A modern web browser

Python 3.10+ is recommended for broad package compatibility.

### 1. Clone the repository

```powershell
git clone <repository-url>
cd Credit-Card-Fraud-Detection-main
```

### 2. Create a virtual environment

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

For Command Prompt:

```bat
.venv\Scripts\activate.bat
```

For Linux or macOS:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```powershell
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### 4. Train the model

Run the training script from the project root:

```powershell
python train.py
```

This reads `data/creditcard.csv`, trains the pipeline, prints the test report, and writes `fraud_model.pkl`.

### 5. Evaluate the model

```powershell
python test.py
```

The script loads the serialized model, evaluates it on the reproducible held-out split, and asserts that the prediction count matches the test-set size.

### 6. Start the API

```powershell
python app.py
```

The API runs at `http://127.0.0.1:5001`.

### 7. Start the frontend

Open a second terminal, activate the environment if necessary, and serve the static files:

```powershell
cd frontend
python -m http.server 5500
```

Visit `http://localhost:5500`.

Keep both the API server and frontend server running.

## API Reference

### `GET /`

Checks whether the API is running.

Example:

```powershell
Invoke-RestMethod -Uri "http://127.0.0.1:5001/"
```

Response:

```json
{
  "message": "Credit Card Fraud Detection API is running"
}
```

### `POST /predict`

Returns a fraud prediction for a transaction.

Required JSON fields:

| Field | Type | Constraints |
|---|---|---|
| `TransactionAmount` | number | Non-negative |
| `TransactionFrequency` | integer | Non-negative |
| `AverageTransactionAmount` | number | Non-negative |
| `InternationalTransaction` | integer | `0` or `1` |
| `TransactionHour` | integer | `0` through `23` |

PowerShell example:

```powershell
$body = @{
  TransactionAmount = 2500
  TransactionFrequency = 3
  AverageTransactionAmount = 1200
  InternationalTransaction = 0
  TransactionHour = 14
} | ConvertTo-Json

Invoke-RestMethod `
  -Uri "http://127.0.0.1:5001/predict" `
  -Method Post `
  -ContentType "application/json" `
  -Body $body
```

cURL example:

```bash
curl -X POST http://127.0.0.1:5001/predict \
  -H "Content-Type: application/json" \
  -d '{
    "TransactionAmount": 2500,
    "TransactionFrequency": 3,
    "AverageTransactionAmount": 1200,
    "InternationalTransaction": 0,
    "TransactionHour": 14
  }'
```

Example success response:

```json
{
  "prediction": 0,
  "result": "Legitimate Transaction",
  "fraud_probability": 12.5
}
```

- `prediction = 0`: legitimate
- `prediction = 1`: fraudulent
- `fraud_probability`: model-estimated probability of class `1`, rounded to two decimal places

Example missing-field response:

```json
{
  "error": "Missing field: TransactionHour"
}
```

HTTP status codes:

- `200` – Successful prediction
- `400` – Missing request data or required field
- `500` – Server or model inference error

## Model Persistence

The model is serialized in `fraud_model.pkl` with Joblib. If you encounter scikit-learn version warnings or deserialization errors after changing environments, retrain the pipeline:

```powershell
python train.py
```

Then restart `app.py` so it loads the rebuilt artifact.

## Limitations and Responsible Use

- The dataset is synthetic, not a real bank transaction corpus.
- The project is optimized for demonstration, not deployment.
- Fraud recall is very low on the held-out sample.
- The API does not authenticate requests, enforce rate limits, or log audit events.
- Debug mode is enabled in `app.py`; do not expose it directly to the internet.
- Prediction probabilities should not be interpreted as calibrated financial risk scores.
- The backend has limited schema validation and no database or fraud-review workflow.
- Do not use this model for real approvals, account restrictions, or payment decisions.

For a production system, retrain and validate on representative real-world data, improve minority-class recall, calibrate thresholds, add monitoring and explainability, and implement authentication and deployment controls.

## Troubleshooting

### Port 5001 is already in use

Stop the existing process or change the port in `app.py` and `frontend/script.js` to a matching value.

### The frontend cannot connect

Confirm that `python app.py` is running and that `http://127.0.0.1:5001/` is reachable. The frontend must be served over HTTP or opened from a local web server to avoid browser restrictions.

### The model does not load

Delete or replace the existing artifact by retraining:

```powershell
python train.py
```

If warnings involve scikit-learn versions, recreate the virtual environment and install the dependencies again.

### Missing packages

Run:

```powershell
python -m pip install -r requirements.txt
```

## Contributing

Contributions are welcome. Suggested improvements include:

- Pinning dependency versions for reproducible environments
- Adding automated tests
- Returning clearer schema-validation errors
- Improving fraud recall and threshold selection
- Adding Docker support
- Separating configuration from application code
- Adding model cards and deployment monitoring

Before submitting a change, run:

```powershell
python train.py
python test.py
```

## License

This project is distributed under the MIT License. See [`LICENSE`](./LICENSE) for the full text.
