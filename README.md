# 🚗 Vehicle Insurance Domain – Comprehensive MLOps & Deployment Showcase

> **Project Objective:** Develop and deploy a scalable, maintainable machine learning system for vehicle insurance risk prediction. The system ingests real-world data from MongoDB Atlas, validates and transforms features, trains and evaluates models, stores artifacts in AWS S3, and serves predictions through a Flask web interface. A fully automated CI/CD pipeline and cloud infrastructure integration showcase professional-grade MLOps practices.

This repository represents a **complete software engineering lifecycle** for a machine learning product—ideal for technical reviewers and hiring managers to assess architecture, quality, and deployment expertise.

[![Python](https://img.shields.io/badge/python-3.10-blue)](https://www.python.org/)
[![Docker](https://img.shields.io/badge/docker-enabled-blue)](https://www.docker.com/)
[![AWS](https://img.shields.io/badge/aws-integrated-orange)](https://aws.amazon.com/)
[![MongoDB](https://img.shields.io/badge/mongodb-atlas-green)](https://www.mongodb.com/atlas)

---

## 📁 Repository Overview
The project follows a modular architecture with clear separation of concerns. Below is a high‑level tree highlighting key folders and files:

```
Vehicle-Insurance-Domain--MLOps-project/
├── app.py                    # Flask app exposing training/prediction endpoints
├── src/                      # Python package containing all business logic
│   ├── components/           # Reusable pipeline components (ingestion, validation, …)
│   ├── configuration/        # Helpers for MongoDB & AWS connections
│   ├── constants/            # Global constants and environment variables
│   ├── data_access/          # Data layer specific to MongoDB
│   ├── entity/               # Dataclasses for configs/artifacts (DTOs)
│   ├── exception/            # Custom exception hierarchy for robustness
│   ├── logger/               # Logging configuration & utilities
│   ├── pipline/              # Orchestrators for training & prediction
│   └── utils/                # Miscellaneous helpers (schema, file I/O)
├── config/                   # YAML definitions (dataset schema, parameters)
├── notebook/                 # Jupyter notebooks for EDA and MongoDB demo
├── artifact/                 # Output from pipeline runs (timestamped)
├── static/ templates/        # Web assets for the Flask UI
├── Dockerfile               # Containerization instructions
├── .github/workflows/       # CI/CD workflow definitions (AWS deployment)
├── requirements.txt         # Python dependencies
├── setup.py, pyproject.toml # Packaging configuration for editable install
└── project_flow.txt         # Step‑by‑step development notes (useful read)
```

> 💡 *Tip:* The `src` package is installed locally via `pip install -e .`, enabling imports like `from src.components import DataIngestion` throughout the codebase.

### 🧱 Core Package (`src/`) Detailed Breakdown
The `src` directory houses all the business logic and is organized by responsibility. Each folder contains the code needed for a specific stage of the pipeline or utility.

- **`components/`** – Individual pipeline steps implemented as classes (`DataIngestion`, `DataValidation`, `DataTransformation`, `ModelTrainer`, `ModelEvaluation`, `ModelPusher`). Each component defines a `initiate_<step>` method and returns an artifact object.
- **`configuration/`** – Configuration helpers, such as `mongo_db_connection.py` and `aws_connection.py`. These encapsulate connection logic to external services and read from environment variables.
- **`constants/`** – Central location for constants like database names, bucket keys, thresholds, and file path templates. Updating values here propagates across components.
- **`data_access/`** – Data layer that knows how to fetch raw data (MongoDB) and convert it to pandas DataFrame. `proj1_data.py` encapsulates queries and transformation logic.
- **`entity/`** – Data Transfer Objects (DTOs) using `@dataclass`. Contains config classes (e.g., `DataIngestionConfig`) and artifact classes (e.g., `ModelTrainerArtifact`). An additional `s3_estimator.py` holds AWS S3 helper logic.
- **`exception/`** – Defines custom exception types (`InsuranceException`) and utilities for wrapping and logging errors consistently.
- **`logger/`** – Logging setup, providing a module‑level logger used by all other packages. Helps maintain uniform log formatting.
- **`pipline/`** – High‑level orchestrators: `training_pipeline.py` sequences the six components; `prediction_pipeline.py` handles inference path.
- **`utils/`** – Miscellaneous helpers including schema validation, file I/O helpers, and common utilities used by multiple components.

Having this clean separation improves maintainability, enables unit testing of individual parts, and lets you swap or extend functionality (e.g., adding a new data source) without touching unrelated code.


---

## 🛠️ Development Setup
Follow these steps to replicate the environment on your machine:

1. **Environment creation**
   ```bash
   conda create -n vehicle python=3.10 -y
   conda activate vehicle
   pip install -r requirements.txt
   pip install -e .    # makes `src` available as a package
   ```

2. **Configuration**
   - Populate `config/schema.yaml` with your dataset's feature definitions and accepted ranges.
   - Add secrets via environment variables (`MONGODB_URL`, `AWS_ACCESS_KEY_ID`, etc.).
   - Update `src/constants/__init__.py` if you change bucket names or thresholds.

3. **Verify local package**
   ```bash
   python -c "import src; print(src)")
   ```
   Ensuring the project is importable avoids circular import issues later.

4. **Run demonstrations**
   - Execute `demo.py` to test logging, exception handling, and simple component runs.
   - Open notebooks in `notebook/` to explore the data and interact with MongoDB.

---

## 🗄️ MongoDB Atlas Integration
Data is stored and retrieved from a MongoDB Atlas cluster:

| Step | Description |
|------|-------------|
| 1    | Sign up for MongoDB Atlas; create a project and an M0 (free) cluster. |
| 2    | Add a database user and whitelist `0.0.0.0/0` for development. |
| 3    | Obtain the connection string, replace `<password>`, and set `MONGODB_URL` env variable. |
| 4    | Use `notebook/mongoDB_demo.ipynb` to upload sample data and inspect collections. |

The connection helper lives in `src/configuration/mongo_db_connection.py` and is used by `src/data_access/proj1_data.py` to pull documents and frame them as `pandas.DataFrame` objects.

---

## 🧵 Logging & Error Handling
Robust logging and exception handling ensure visibility when pipelines run:

- **Logger**: `src/logger/__init__.py` uses Python's `logging` module to create a reusable logger with timestamped, levelled messages.
- **Custom Exception**: `src/exception/insurance_exception.py` defines `InsuranceException` that wraps underlying errors, preserving stack traces and context.
- Extensive unit tests in `tests/` (if present) exercise both facilities.

> ⚠️ All `except` blocks re‑raise `InsuranceException` to unify error handling across components.

---

## 🚰 Data Ingestion Component
Transforms raw MongoDB documents into a usable dataset:

1. **Constants**: Declare field names, DB names, and file paths in `src/constants/__init__.py`.
2. **Configuration entity**: `DataIngestionConfig` captures parameters such as database name and export path.
3. **Artifact entity**: `DataIngestionArtifact` records output locations and status.
4. **Component**: `src/components/data_ingestion.py` contains `DataIngestion` class with an `initiate_data_ingestion()` method which:
   - Connects to MongoDB via the configuration helper.
   - Reads raw documents.
   - Converts them to a DataFrame and writes them to feature store CSV.
5. **Pipeline integration**: The ingestion step is invoked by `src/pipline/training_pipeline.py`, maintaining dependency order.

This modular design allows you to swap the data source (e.g., S3, PostgreSQL) by implementing a new component with the same interface.

---

## ✅ Data Validation
Ensures data quality before any downstream processing:

- Schema defined in `config/schema.yaml` includes type expectations, allowed categories, and numerical ranges.
- `src/utils/main_utils.py` houses helpers (`validate_schema`, `report_missing_values`, etc.).
- `src/components/data_validation.py` compares ingested data against the schema and generates a human‑readable report (`artifact/report.yaml`).
- Validation artifacts are propagated through `training_pipeline` for logging and decision‑making (e.g., abort if critical errors). 

Maintaining a separate validation component improves reproducibility and provides a clear audit trail when data drifts occur.

---

## 🔄 Data Transformation
Prepares features for model consumption:

- `entity/estimator.py` defines classes like `StandardScalerEstimator` or `InsuranceModelInput` that store fitted transformers.
- `src/components/data_transformation.py` applies transformations such as scaling, encoding, and saves the transformed dataset and transformer object.
- Outputs are recorded in `DataTransformationArtifact` which includes paths to transformed files and the transformer pickle.

Transformer objects are versioned and later used by the prediction pipeline to ensure consistency between training and serving.

---

## 🏋️ Model Training
Trains a machine learning model using scikit‑learn (or your preferred library):

- Parameters (algorithm choice, hyperparameters, validation split) are stored in `config/model.yaml`.
- `src/components/model_trainer.py` contains `ModelTrainer` class with methods like `train_model()` and `evaluate_model()`.
- Trained models are serialized (`.pkl`) and their metadata stored in `ModelTrainerArtifact`.

Evaluation metrics (accuracy, ROC AUC, etc.) are logged and later compared during the model evaluation stage.

---

## 📏 Model Evaluation & Model Pusher
This stage decides whether a newly trained model is worthy of promotion:

1. **AWS prerequisites**
   - Create an IAM user with `AdministratorAccess`.
   - Set `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` in your environment.
   - Add the credentials and other constants (`MODEL_BUCKET_NAME`, etc.) to `src/constants/__init__.py`.
   - Create an S3 bucket (`my-model-mlopsproj`) and adjust public access settings.

2. **Evaluation**
   - `src/components/model_evaluation.py` compares new and baseline models using a chosen metric and a threshold defined by `MODEL_EVALUATION_CHANGED_THRESHOLD_SCORE`.
   - The result (`ModelEvaluationArtifact`) indicates whether the new model is promoted.

3. **Pusher**
   - If approved, `src/components/model_pusher.py` uses `entity/s3_estimator.py` to upload the model to S3 under the key defined in `MODEL_PUSHER_S3_KEY`.
   - Artifacts include the S3 URI and local archive path.

This separation allows you to implement alternative registries later (e.g., MLflow, DVC) by conforming to the same artifact contracts.

---

## 🔮 Prediction Pipeline & Web Application
Provides real‑time predictions via REST:

- `src/pipline/prediction_pipeline.py` reads the transformer and model, applies them to incoming JSON requests, and returns predictions.
- `app.py` (Flask) defines endpoints:
  - `POST /predict` accepts feature payloads and returns probabilities/labels.
  - `GET /training` triggers the full training pipeline when data has changed (optional; used in demos).
- Basic HTML interface under `templates/vehicledata.html` allows users to input values via browser.

The pipeline ensures the same preprocessing used during training is applied in production, preventing training/serving skew.

---

## 🛠️ CI/CD with GitHub Actions
Automation keeps the system deployable with every commit:

| Feature | Description |
|---------|-------------|
| Dockerfile | Builds a container housing the Flask app and pipeline. |
| `.github/workflows/aws.yaml` | Workflow to build, test, push image to ECR, and SSH‑deploy to EC2. |
| Self‑hosted runner | An EC2 Ubuntu instance running GitHub Actions jobs, giving full control over environment.
| GitHub Secrets | Stores AWS creds, `ECR_REPO` URI, and optionally `MONGODB_URL` for training in CI. |

### Deployment Steps
1. Commit and push code → workflow triggered.
2. Image built and pushed to ECR `vehicleproj` repo.
3. EC2 runner pulls the latest image and restarts the service.
4. Security group rule opens port 5000 for external access.

> ⚙️ The self‑hosted runner setup commands are recorded in `project_flow.txt` for reproducibility.

---

## 🚀 Running the System
1. Ensure the required environment variables are set locally or via GitHub Secrets.
2. Execute the training pipeline manually:
   ```bash
   python -c "from src.pipline.training_pipeline import TrainingPipeline; TrainingPipeline().run_pipeline()"
   ```
3. Start the web server:
   ```bash
   python app.py
   ```
4. Access the UI at `http://localhost:5000` or `http://<EC2_IP>:5000` in production.

Predictions can also be made programmatically using `curl` or Postman.

---

## 📄 Documentation & Artifacts
- `project_flow.txt` contains chronological development notes—you can follow it to understand design decisions and build the project yourself.
- Notebooks in `notebook/` provide exploratory data analysis and MongoDB interaction examples.
- `artifact/` stores every pipeline run's outputs, including data, models, and validation reports; it serves as an audit trail.
- Logs in `logs/` capture runtime information useful for debugging and monitoring.

---

## 🏁 Why This Project Impresses
- **Comprehensive MLOps pipeline** from ingest to deployment.
- **Modular, testable components** align with software engineering best practices.
- **Cloud integration** with AWS and MongoDB demonstrates real‑world skills.
- **CI/CD & automation** using Docker and GitHub Actions ensures production readiness.
- **Documentation & reproducibility** make it easy for reviewers to verify and extend the work.

Feel free to clone the repository and explore—this project is designed to be read, run, and expanded upon by hiring managers and collaborators alike. Happy exploring! 👏
