# MLOPS — Vehicle Insurance Domain

This repository contains an end-to-end MLOps example for the vehicle insurance domain. It demonstrates how to structure a Python machine learning project, build a reproducible model pipeline (without DVC), manage secrets using dotenv, package the project, use FastAPI to serve the model, and deploy artifacts to AWS S3 with appropriate IAM usage. It also shows how to push and retrieve data from MongoDB Atlas and general best practices for writing maintainable Python code (type hints, explicit return types, modular structure).

This README documents the repository goals, layout, how to set up the environment, how to run the project locally, and notes on deployment and design decisions.

Table of contents
- Project goals
- Key learnings and features
- Repository layout
- Requirements
- Environment variables (.env)
- Installation
- Usage
  - Data operations (MongoDB Atlas)
  - Training pipeline
  - Packaging the project
  - Serving the model with FastAPI
  - Uploading and retrieving artifacts from AWS S3
- Testing
- Design notes & coding practices

Project goals
- Provide a clear, production-minded structure for ML code.
- Demonstrate simple reproducible pipelines without DVC.
- Show secure configuration with dotenv for secrets and environment variables.
- Demonstrate packaging a Python project so it can be reused or installed.
- Provide a simple FastAPI model serving example.
- Demonstrate storing and retrieving artifacts and data using MongoDB Atlas and AWS S3 with proper IAM usage.

Key learnings and features (what this repo shows)
- Directory and module structuring of an ML project for clarity and maintainability.
- Use of dotenv (.env) to keep secrets/configuration out of source control.
- Building an ML pipeline (data loading, preprocessing, training, evaluation, model serialization) without DVC.
- Uploading and retrieving model and dataset artifacts to/from AWS S3 using IAM credentials.
- Connecting to MongoDB Atlas to push and pull data.
- Converting the project into an installable Python package (pip install -e .) to enable reuse and clearer imports.
- Serving the trained model via FastAPI with typed endpoints and dependency injection patterns.
- Emphasis on Python typing (type hints and explicit return types) and modular code for readability and testing.

Repository layout (example)
- src/ or app/ — core package source code (models, data, pipeline, utils, api)
- notebooks/ — exploratory analysis or notes (if present)
- requirements.txt / pyproject.toml / setup.cfg — dependency and packaging definitions
- .env.example — example environment variables (DO NOT commit secrets)
- README.md — this file

Requirements
- Python 3.9+ (adjust as required by the repo)
- pip
- virtualenv (recommended)
- MongoDB Atlas account and connection string
- AWS account, S3 bucket, and IAM credentials with minimal permissions to the target bucket
- (Optional) uvicorn for FastAPI serving

Environment variables
Create a .env file in the project root (use .env.example as a template). Example keys typically used in this project:

MONGODB_URI="mongodb+srv://<user>:<password>@cluster0.mongodb.net/mydatabase?retryWrites=true&w=majority"
MONGO_DB_NAME="vehicle_insurance_db"

AWS_ACCESS_KEY_ID="AKIA..."
AWS_SECRET_ACCESS_KEY="..."
AWS_REGION="us-east-1"
S3_BUCKET="your-s3-bucket-name"

MODEL_ARTIFACT_PATH="models/latest_model.pkl"
LOCAL_MODEL_DIR="./artifacts/models"

FASTAPI_HOST="127.0.0.1"
FASTAPI_PORT="8000"

OTHER_CONFIG="value"

Important: never commit the real .env to version control.

Installation (local development)
1. Clone the repository:
   git clone https://github.com/senku0-0/MLOPS-Vehicle-insurance-domain.git
   cd MLOPS-Vehicle-insurance-domain

2. Create & activate virtual environment:
```python
   python -m venv .venv
```
   source .venv/bin/activate   # Linux / macOS
   .venv\Scripts\activate      # Windows

4. Install dependencies:
 ```python
 pip install -r requirements.txt
 ```

Packaging the project
This repository is organized as a Python package to allow:
- Clean imports (from myproject.module import ...)
- Easy installation into environments
- Better reuse across services and tests

To install the package in editable mode:
```python
pip install -e.
```

(Ensure setup.py or pyproject.toml is configured; the package root is in src/ or app/ accordingly.)

Usage

1) Data operations with MongoDB Atlas
- The repo contains utilities to connect to MongoDB Atlas using the MONGODB_URI and MONGO_DB_NAME environment variables.
- Typical usage:
  - Push raw or preprocessed data to MongoDB for centralized storage.
  - Retrieve datasets for training or inference.
- The connection code uses a small wrapper that enforces typed inputs and safe error handling.

2) Training pipeline (no DVC)
- The pipeline is implemented as a set of modular steps:
  - load_data -> preprocess -> split -> train -> evaluate -> serialize
- Each step is implemented as a function with explicit input/output types and small surface area for unit testing.
- To run training:
```
   python -m demo.py
```
- Model artifacts are written to LOCAL_MODEL_DIR and optionally uploaded to S3.

3) Uploading model artifacts to AWS S3 (IAM & S3)
- The AWS credentials defined in the .env file are used by boto3 to programmatically upload artifacts.
- The code is written to use minimal IAM privileges: give the IAM user/role only the required s3:PutObject / s3:GetObject / s3:ListBucket permissions on the designated bucket.
- Example upload flow:
  - After training, call artifact_uploader.upload_model(local_path, s3_key)
  - The function returns the S3 URI of the uploaded artifact.

4) Serving with FastAPI
- FastAPI app provides typed endpoints for:
  - health checks
  - predictions (POST with typed input schema)
  - model metadata
- To start the server locally:
```
   uvicorn app.main:app --reload --host ${FASTAPI_HOST} --port ${FASTAPI_PORT}
```
- The API uses the installed package modules to load the model and perform preprocessing so that runtime behavior mirrors training logic.


Design notes & coding practices
- Type hints and explicit return types:
  - Functions and methods include type annotations to make the code self-documenting and to improve static analysis and testing.
- Small, well-named modules:
  - Each module encapsulates a small set of responsibilities (data, model, evaluation, persistence, api).
- Config & secrets:
  - Configuration lives in a single module that reads from environment variables; this module provides typed config objects for the rest of the codebase.
- Reproducibility:
  - Training code accepts seeds, saves preprocessing artifacts, and timestamps models to help with reproducibility.
- Logging & errors:
  - Standardized logging is used across modules and exceptions are wrapped into domain-specific errors where appropriate.

Why package the project?
- Packaging enables:
  - Cleaner imports (avoid relative import hell)
  - Easy installation into environments and CI
  - Explicit dependency surface and versioning
  - Reuse of core model code in other services (e.g., batch inference workers, other APIs)


If you want, I can:
- Generate a .env.example file with recommended variables.
- Create a sample IAM policy for S3 use.
- Produce an example FastAPI request/response schema for the prediction endpoint.
- Draft a simple CI workflow for tests and packaging.
