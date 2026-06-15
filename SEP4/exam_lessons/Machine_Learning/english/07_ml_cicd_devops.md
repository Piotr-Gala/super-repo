# Lesson 10: ML CI/CD and DevOps

Goal of this lesson: you should be able to explain **how the ML service was automatically checked, built, and deployed**.

## 1. Why DevOps matters for ML

For ML, CI/CD is not only:

```text
does the server start?
```

It is also:

```text
does the model still return sensible outputs for known inputs?
does the API contract still match the backend?
does the Docker image include the right model artifacts?
```

Good answer:

> ML DevOps matters because a model can be technically deployable but behaviorally wrong. The pipeline therefore checks linting, tests, coverage, and Docker buildability, so a changed model or service cannot silently break the MainServer contract.

## 2. CI pipeline

The workflow is:

```text
.github/workflows/ci.yml
```

It runs on:

```text
push: Dev, main, ML-team
pull_request: Dev, main, ML-team
workflow_dispatch
```

The ML job:

```text
setup Python 3.12
pip install -r requirements.txt -r requirements-dev.txt
ruff check main.py tests/
pytest --cov --cov-report=xml --cov-report=term-missing --cov-fail-under=70
upload coverage artifact
write coverage summary
```

Good answer:

> The ML CI job runs on GitHub Actions with Python 3.12. It installs runtime and dev dependencies, runs ruff linting, runs pytest with coverage, enforces a 70% coverage threshold, and uploads the coverage XML as an artifact.

## 3. Docker build

The shared Docker job builds the ML image:

```text
context: System/MlServer
tag: kamtjatka/ml-server:ci
push: false
```

This means CI verifies that the image builds, but it does not push it to a registry.

Good answer:

> The Docker build step validates that the ML service can be packaged with its dependencies and model artifacts. The image is not pushed in CI, so deployment rebuilds it on the VPS from the checked-out branch.

## 4. ML Dockerfile

The ML Dockerfile:

```text
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY main.py .
COPY model/ ./model/
USER appuser
EXPOSE 8000
CMD uvicorn main:app --host 0.0.0.0 --port 8000
```

Important points:

- production image contains only runtime dependencies,
- notebooks are not copied,
- `ImprovedModels` is not copied,
- model artifacts are copied into the image,
- service runs as non-root `appuser`.

Good answer:

> The Docker image packages the inference service, not the research environment. It copies `main.py` and the deployed model folder, installs runtime dependencies, and runs Uvicorn on port 8000. This keeps the production image smaller and separate from notebooks and offline experiments.

## 5. Deployment

The deploy job runs only on:

```text
push to Dev
push to main
```

It does not run on pull requests.

The deploy job depends on:

```text
mainserver
mlserver
frontend
docker
```

So if ML tests fail, deployment is blocked.

Deployment flow:

```text
SSH to VPS
cd ~/SEP4
git fetch origin
git checkout branch
git reset --hard origin/branch
cd System
docker compose -f docker-compose.vps.yml up -d --build --remove-orphans
docker image prune -f
```

Good answer:

> Deployment is SSH-based. After the required jobs pass, the VPS resets its repository to the pushed branch and rebuilds the Docker Compose stack. This is simple and reproducible for the project scale, but it assumes the VPS has no important local changes.

## 6. Runtime deployment setup

In `System/docker-compose.vps.yml`, ML runs as:

```text
ml-server:
  build: ./MlServer
  image: kamtjatka/ml-server:latest
  container_name: kamtjatka-ml
  ports:
    - "${ML_SERVER_PORT:-8000}:8000"
  healthcheck:
    GET /health
```

The MainServer uses:

```text
ML_SERVER_URL: http://ml-server:8000
```

And waits for:

```text
ml-server:
  condition: service_healthy
```

Good answer:

> Docker Compose makes the ML service available to the backend through the internal service name `ml-server`. The health check protects startup because the backend waits until ML is healthy before serving the integrated prediction flow.

## 7. Model artifacts in Git

The deployed artifacts are committed directly:

```text
System/MlServer/model/model_random_forest.joblib
System/MlServer/model/scaler.joblib
```

Why?

Because for this project they are small enough, and it keeps deployment self-contained.

Trade-off:

> This avoids needing MLflow, DVC, or a model registry, but it is not the best long-term production setup for model versioning.

Good answer:

> We committed the model and scaler directly because it was pragmatic for a semester project. It made deployment deterministic: the model on the branch is the model deployed. For a larger system, a model registry or artifact store would be better.

## 8. Tools not used

The report says these were considered but not fully set up:

- MLflow,
- DVC,
- automated retraining,
- live scheduled drift monitoring,
- ML-specific load testing.

Why not?

Because the model iteration cycle was short, and retraining requires human decisions about data quality and labeling.

Good answer:

> We did not automate retraining because model promotion should not be a blind CI step. New data must be checked, labels must be trusted, and evaluation must be reviewed before replacing the deployed model.

## 9. What worked well

What worked:

- ruff caught code quality issues,
- pytest protected the inference contract,
- coverage floor forced tests to grow with the service,
- Docker build caught dependency/image issues,
- CI caught a feature-order/model swap type of problem before merge.

Good answer:

> The ML pipeline gave fast feedback. It did not only check that Python code runs; it also checked that known predictions and the API contract still behave as expected.

## 10. Limitations

Main limitations:

- no automatic retraining pipeline,
- no model registry,
- no automated promotion/approval process,
- no scheduled drift monitor in production,
- no full automated MainServer + ML contract test,
- Docker image is rebuilt on the VPS instead of pulled from a registry,
- deployment uses `git reset --hard` on the VPS.

Good answer:

> The pipeline is good for a prototype inference service, but it is not a full MLOps platform. It verifies service behavior and Docker buildability, but it does not manage model lifecycle, retraining, approval, or live drift alerts automatically.

## Best exam answer

> The ML CI/CD setup is part of the shared GitHub Actions workflow. The ML job runs on Python 3.12, installs runtime and dev dependencies, runs ruff, and executes pytest with coverage using a 70% fail-under threshold. A separate Docker job builds the ML image from `System/MlServer` to verify that the service, dependencies, and model artifacts can be packaged. Deployment only runs on push to Dev or main after the required jobs pass. On the VPS, Docker Compose runs the ML service as `ml-server`, exposes `/health`, and the MainServer waits for it to become healthy before using `http://ml-server:8000`. The main limitation is that retraining, model approval, and drift monitoring are not automated.

## Remember

```text
GitHub Actions
  -> ruff
  -> pytest + 70% coverage
  -> Docker build
  -> deploy only on Dev/main

Docker Compose
  -> ml-server container
  -> /health check
  -> MainServer waits for healthy ML

MLOps limitation
  -> no automatic retraining or model registry
```
