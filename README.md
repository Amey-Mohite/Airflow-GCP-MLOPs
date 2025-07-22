## ✅ **README: Continuous Training Pipelines using Airflow and GCP**

### 📌 Overview

This project implements a **Continuous Training (CT)** pipeline for ML models using **Apache Airflow** (via **Cloud Composer**), **Vertex AI**, **BigQuery**, **GCS**, and **Cloud Logging**. The pipeline ensures that models are retrained and evaluated automatically using production data, storing metrics, and deploying models only when performance improves.

---

### 🧰 Tools & Technologies Used

| Component        | Tool / Service                    |
| ---------------- | --------------------------------- |
| Orchestration    | Apache Airflow (Cloud Composer)   |
| ML Training      | Vertex AI Workbench               |
| Artifact Storage | Google Cloud Storage (GCS)        |
| Metric Storage   | BigQuery                          |
| Monitoring       | Cloud Logging + Log-based Metrics |
| CI/CD            | Cloud Build + GitHub              |

---

### ⚙️ Implementation Steps

#### 🧱 1. Setup Infrastructure

* Enable **Cloud Composer** and create an Airflow environment.
* Enable **Vertex AI APIs**, create a **Vertex AI Workbench instance**.

#### 🛠️ 2. Local Development

* Train the initial model using **Jupyter Notebook** in Vertex AI.
* Upload the training dataset (`CSV`) to a GCS bucket.
* Create a **BigQuery table** to store model performance metrics:

```sql
CREATE TABLE `<gcp-project>.ml_ops.bank_campaign_model_metrics` (
  algo_name STRING,
  training_time TIMESTAMP,
  model_metrics STRING
);
```

#### 📦 3. Prepare Artifact Storage

* Store trained models as `.joblib` files in a GCS bucket:

```python
model_artifact = bucket.blob('ml-artifacts/'+artifact_name)
model_artifact.upload_from_filename(artifact_name)
```

#### 📊 4. Push Metrics to BigQuery

```python
def write_metrics_to_bigquery(algo_name, training_time, model_metrics):
    ...
    client.insert_rows_json(table, [row])
```

---

### 🪝 5. Build Airflow DAGs

* DAG includes:

  * **Data Validation**
  * **Model Evaluation**
  * **Model Saving (only if improved)**

```python
validate_csv_task >> evaluation_task
```

* Tasks use functions from a separate training module (e.g., `bank_campaign_model_training.py`).
* Metrics logged to Cloud Logging for alerting.

---

### 🚨 6. Setup Alerting on Model Failures

* Use **Log Explorer** → Filter by `Bank_Campaign_Model_Training`
* Create a **log-based metric** on `training_status`
* Setup alert policies (e.g., if status == 0 for last 6 hours)

---

### 🔄 7. CI/CD with Cloud Build

#### Files Required:

* `bank_campaign_model_training.py`
* `requirements.txt`
* `cloudbuild.yaml`
* `Dockerfile`
* `test_training.py`

#### Cloud Build Steps:

```yaml
- Build & Push Docker image
- Run unit tests
- Clone GitHub repo
- Upload code to Airflow DAGs bucket
```

#### Trigger Flow:

```bash
gcloud builds submit --region us-central1
```

Set up **Cloud Build Trigger** linked to your GitHub repo to automate the whole process.

---

### 🔁 End-to-End Flow

1. Upload CSV to GCS.
2. DAG validates & evaluates model.
3. If metrics improve:
   * Save model to GCS.
   * Store metrics to BigQuery.
   * Log status in Cloud Logging.
4. If thresholds aren't met:
   * Training is skipped.
   * Alert is sent via email/SMS/Slack.

---

## 📚 Folder Structure

```
.
├── dags/
│   └── dag_bank_campaign_continuous_training.py
├── training/
│   ├── bank_campaign_model_training.py
│   └── test_training.py
├── cloudbuild.yaml
├── requirements.txt
└── Dockerfile
```

---

## ✅ Outcome

* Efficient ML retraining pipeline.
* Only deploys models when performance improves.
* Full observability with logs and alerts.
* Automated with CI/CD and Cloud Build.
