# Lesson 1: What is AeroSense?

AeroSense is a system for monitoring fire risk and indoor air quality.

Do not think about it only as a "frontend app". Think about it as a full data pipeline:

```text
Physical room
  -> IoT device reads sensors
  -> MQTT broker receives readings
  -> MQTT-to-HTTP bridge forwards readings
  -> backend receives readings
  -> backend stores data and calls ML
  -> ML classifies risk
  -> backend exposes data/API/stream
  -> frontend shows status to users
```

So the frontend is the **last layer of the system**, but for the user it is the most visible one, because this is where they see:

- current room conditions,
- historical measurements,
- alerts,
- recommendations,
- device status,
- rooms and assigned sensors,
- sometimes technical data such as the raw JSON payload.

The most important sentence to remember:

> The frontend turns raw sensor and prediction data into understandable information for residents and administrators.

In simple terms: backend, IoT, and ML produce data, and the frontend turns that data into something a person can quickly understand.

## How does the data flow?

Example:

1. A sensor measures values such as `temperature`, `humidity`, `CO2`, and `TVOC`.
2. IoT publishes the reading through MQTT.
3. The MQTT-to-HTTP bridge sends it to the backend through `POST /api/readings`.
4. The backend stores the reading and sends the data to the ML service.
5. ML returns a classification, for example `Normal`, `Cooking Smoke`, or `Fire Risk`.
6. The backend stores the reading and the prediction.
7. The frontend fetches the data through REST or receives a live update through SSE.
8. The user sees a card, alert, recommendation, or history view.

So if the examiner asks:

> How does the frontend fit into the system?

Do not answer only:

> It displays data.

That is too weak.

A better answer:

> The frontend is the user-facing part of AeroSense. It receives processed sensor readings from the backend, including classifications from the ML service, and presents them through dashboards, charts, alerts, recommendations, and role-based management views. It does not classify the data itself; it depends on the backend and ML pipeline, but it makes the result understandable and actionable for users.

## Code check

This is confirmed in the code:

- `System/FrontEnd/src/services/api.js` shows that the frontend talks to the backend through `/api/...`, for example `/api/readings`, `/api/readings/devices`, `/api/readings/alerts`, `/api/room`, `/api/users`, `/api/logs`, and `/api/auth/login`.
- `connectReadingsStream()` uses `EventSource` on `/api/readings/stream`, so live updates use Server-Sent Events.
- `System/docker-compose.vps.yml` shows the full stack: `mqtt-broker`, `mqtt-http-bridge`, `main-server`, `frontend`, `mysql`, and `ml-server`.
- The frontend container depends on a healthy `main-server`, so the frontend is not a separate system. It is the UI layer on top of the backend.

## Remember

- **IoT collects data**
- **MQTT transports readings**
- **Backend processes, stores, and coordinates**
- **ML classifies risk**
- **Frontend visualizes data and helps users react**

Most importantly: the frontend **does not detect fire by itself**. The frontend shows the result of the backend and ML pipeline.
