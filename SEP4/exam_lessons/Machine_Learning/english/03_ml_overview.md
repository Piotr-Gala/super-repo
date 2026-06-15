# Lesson 6: AeroSense Machine Learning Overview

Goal of this lesson: you should be able to explain **what the ML part does**, **why it exists**, and **where its responsibility ends**.

## 1. Role of ML in AeroSense

The ML part is responsible for classifying room conditions from sensor readings.

It does not read sensors directly.  
It does not store data in the database.  
It does not show alerts in the frontend.  
It does not publish MQTT alarm messages by itself.

The ML service:

- receives a sensor payload from the MainServer,
- uses a trained model to classify the reading,
- returns `predictedCategory`,
- returns `confidenceScore`,
- returns `riskLevel`.

The most important sentence:

> The ML service turns sensor measurements into a room-condition classification, so the backend and frontend can react to Normal, Cooking, or Fire situations.

## 2. Where ML fits in the system

The full flow is:

```text
IoT device
  -> MQTT broker
  -> MQTT-to-HTTP bridge
  -> MainServer
  -> ML service
  -> MainServer stores prediction
  -> Frontend displays result
  -> MainServer may publish alarm command
```

So ML is in the middle of the pipeline, not at the edge.

Good answer:

> The ML service is a separate Python FastAPI service. The MainServer calls it through `POST /predict` whenever a reading arrives. The ML service returns a classification, confidence score, and risk level. The backend then stores this result, exposes it to the frontend, and can publish an alarm command if the result is Fire.

## 3. Classes

The final model predicts three classes:

- **Normal** - ordinary room conditions,
- **Cooking** - abnormal air readings caused by cooking,
- **Fire** - dangerous smoke/fire-like condition.

Why three classes?

Because the project is not only "fire vs no fire". Cooking is important because it can look abnormal but should not automatically be treated as a fire.

Good answer:

> We used three-class classification because cooking can produce smoke, humidity, and gas readings that look abnormal. If we used only fire/not-fire, the system would be more likely to create false fire alarms during normal cooking.

## 4. Input features

The deployed model uses exactly four features:

```text
temperature
humidity
tvoc
eco2
```

This is confirmed in `System/MlServer/main.py`:

```text
FEATURE_ORDER = ("temperature", "humidity", "tvoc", "eco2")
```

Important correction:

The API payload also contains `co2Level` and `aqi`, but the model does **not** use them for prediction.

Good answer:

> The model only consumes temperature, humidity, TVOC, and eCO2 because these were the shared features available in both training data sources and in the deployed system. CO2 and AQI are still part of the payload schema, but they are not part of the trained feature vector.

## 5. Output contract

The ML response has exactly three fields:

```json
{
  "predictedCategory": "Normal",
  "confidenceScore": 0.84,
  "riskLevel": "Low"
}
```

In code:

- `predictedCategory` is one of `Normal`, `Cooking`, `Fire`,
- `confidenceScore` is a probability-like value between `0.0` and `1.0`,
- `riskLevel` is one of `Low`, `Medium`, `High`.

Mapping from class to output:

```text
0 -> Normal  -> Low
1 -> Cooking -> Medium
2 -> Fire    -> High
```

## 6. Boundary of responsibility

Do not say:

> The ML service sends alarms to the IoT device.

Better:

> The ML service only returns the prediction. The MainServer decides what to do with it, stores it, sends it to the frontend, and may publish MQTT alarm commands.

Do not say:

> The frontend predicts fire.

Better:

> The frontend displays classifications received from the backend, which got them from the ML service.

## Code check

Confirmed in the code:

- `System/MlServer/main.py` defines the FastAPI app, `/health`, `/predict`, model loading, feature order, and output mapping.
- `System/MainServer/Services/MlClient.cs` sends readings to `POST /predict`.
- `System/docker-compose.vps.yml` runs `ml-server` as its own container and configures `ML_SERVER_URL=http://ml-server:8000`.
- `System/MlServer/model/model_random_forest.joblib` and `System/MlServer/model/scaler.joblib` are the deployed artifacts.

## Best exam answer

> The ML part of AeroSense is a separate Python FastAPI inference service. It receives sensor readings from the MainServer through `POST /predict`, uses a trained Random Forest model with four features - temperature, humidity, TVOC, and eCO2 - and returns `predictedCategory`, `confidenceScore`, and `riskLevel`. The three output categories are Normal, Cooking, and Fire. Keeping Cooking as a separate class helps reduce false fire alarms from normal cooking activity. The ML service does not store data or trigger alarms directly; the MainServer coordinates storage, frontend updates, and MQTT alarm commands.

## Remember

```text
ML does classification
MainServer does coordination
Frontend does presentation
IoT does sensing and alarm output
```
