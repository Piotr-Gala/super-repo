# Lesson 8: ML Service and Integration

Goal of this lesson: you should be able to explain **how the trained model became a running service** and **how it integrates with the backend**.

## 1. Why a separate ML service?

The backend is C#/.NET.  
The ML tooling is Python.

The project kept ML in a separate Python service because libraries such as scikit-learn, NumPy, and joblib fit naturally in Python.

Good answer:

> We separated ML into its own FastAPI service so the ML team could use the normal Python ML stack while the backend stayed in C#/.NET. The boundary between them is a simple HTTP API, which allowed model changes without rewriting the backend.

Trade-off:

> A separate service makes integration and deployment more complex because the backend now depends on ML availability, but it keeps responsibilities clean and technology choices practical.

## 2. FastAPI application

The deployed service is:

```text
System/MlServer/main.py
```

It exposes:

```text
GET  /health
POST /predict
```

`/health` is used by Docker Compose to check that the service is alive.

`/predict` is the real inference endpoint.

## 3. Startup loading

At startup, the service loads:

```text
System/MlServer/model/model_random_forest.joblib
System/MlServer/model/scaler.joblib
```

If either file is missing, `_load_artifacts()` raises `FileNotFoundError`.

Good answer:

> The model and scaler are loaded at startup, not on every request. This avoids repeated file loading and makes missing artifacts fail early. If the model files are missing, the container will not become healthy, so the backend will not start against a broken ML service.

## 4. Request schema

The request body follows the nested sensor payload format:

```json
{
  "sensorId": 102,
  "timestamp": "2026-04-08T12:15:00Z",
  "sensors": {
    "temperature": 22.0,
    "humidity": 40.0,
    "co2Level": 400.0,
    "tvoc": 100.0,
    "eco2": 400.0,
    "aqi": 1
  },
  "classification": "Normal"
}
```

Pydantic validates the payload before the model is called.

Important:

`aqi` must be between `1` and `5`, even though it is not used by the model.

## 5. Prediction logic

The prediction flow in `main.py` is:

```text
receive Reading
extract temperature, humidity, tvoc, eco2
build numpy array in FEATURE_ORDER
apply saved scaler
call model.predict_proba()
take class with highest probability
map class id to category and risk level
return JSON response
append drift log best-effort
```

The feature order matters:

```text
("temperature", "humidity", "tvoc", "eco2")
```

Good answer:

> The feature order is load-bearing because the scaler and model were trained on that exact order. If the order changes, the model still runs but interprets values incorrectly, which is dangerous because it may return confident but wrong predictions.

## 6. Backend integration

The backend calls ML through `MlClient`.

Flow:

```text
MainServer receives POST /api/readings
MainServer stores or prepares reading
MainServer sends payload to ML POST /predict
ML returns predictedCategory, confidenceScore, riskLevel
MainServer stores Prediction linked to SensorReading
MainServer broadcasts data to frontend
MainServer may publish MQTT alarm
```

Good answer:

> The MainServer is the coordinator. The ML service only classifies. The backend decides how to persist the prediction, how to expose it to the frontend, and whether an alarm command should be sent to IoT.

## 7. Failure behavior

From the report and backend tests:

- if ML is unreachable, the backend returns HTTP `502`,
- it does not silently create a fake prediction,
- this keeps prediction data consistent.

Good answer:

> If ML is unavailable, it is better to fail visibly than to store a reading with a missing or fake prediction. Returning 502 makes the integration problem clear and avoids misleading downstream data.

Trade-off:

> This means the reading pipeline depends on the ML service being available, but it avoids silently degrading the most important classification behavior.

## 8. Drift logging

`main.py` appends raw prediction inputs to:

```text
System/MlServer/drift_data/incoming.csv
```

This is controlled by:

```text
DRIFT_LOG_ENABLED
DRIFT_LOG_PATH
```

It is best-effort:

> Drift logging failure must not break prediction.

The actual drift monitor is:

```text
System/MlServer/ImprovedModels/RFModel/drift_monitor.py
```

It is an offline/research-ops tool, not part of the runtime Docker image.

Good answer:

> Drift logging was added because ML models can get worse without crashing. The running service logs raw feature values and predictions, while the separate drift monitor can compare current data against a baseline. It is not part of live prediction and it is not scheduled in CI/CD.

## 9. Docker Compose integration

In `System/docker-compose.vps.yml`, ML runs as:

```text
ml-server
container_name: kamtjatka-ml
port: 8000
healthcheck: GET /health
```

The MainServer uses:

```text
ML_SERVER_URL: http://ml-server:8000
```

The backend has:

```text
depends_on:
  ml-server:
    condition: service_healthy
```

Good answer:

> Docker Compose lets the backend call ML by service name on the internal network. The health check prevents the backend from starting before the ML container has loaded the model and responds to `/health`.

## Best exam answer

> The trained model is deployed as a separate FastAPI service in `System/MlServer/main.py`. At startup it loads `model_random_forest.joblib` and `scaler.joblib`. The MainServer sends each reading to `POST /predict`, where Pydantic validates the nested payload, the service extracts temperature, humidity, TVOC, and eCO2 in the trained feature order, scales them, runs Random Forest prediction probabilities, and returns `predictedCategory`, `confidenceScore`, and `riskLevel`. In Docker Compose the ML service has a `/health` check, and the MainServer depends on it being healthy. This makes the ML boundary explicit and keeps model inference separate from backend coordination.

## Remember

```text
FastAPI wraps model
joblib loads artifacts
Pydantic validates payload
Feature order must match training
MainServer calls POST /predict
Docker health check guards startup
```
