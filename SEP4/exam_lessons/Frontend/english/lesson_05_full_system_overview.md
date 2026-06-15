# Lesson 5: Full System Overview

This is not about knowing someone else's code perfectly. The point is to show that you understand **how the frontend fits into the whole system**.

## 1. AeroSense as a pipeline

The simplest model:

```text
IoT device
  -> MQTT broker
  -> MQTT-to-HTTP bridge
  -> Backend / MainServer
  -> Database
  -> ML service
  -> Frontend
  -> User reaction
```

One sentence:

> AeroSense collects sensor data from IoT devices, processes it through the backend and ML service, stores it, and presents it in the frontend so users can monitor room conditions and react to risks.

## 2. IoT

IoT is responsible for the physical world.

It:

- measures temperature, humidity, CO2, TVOC, eCO2, AQI / air quality,
- sends readings,
- can receive an alarm command,
- can trigger a buzzer.

You do not need to know the drivers. You need to know the role:

> The IoT layer collects environmental data from sensors and sends it into the system. It can also receive alarm commands from the backend, for example to trigger a buzzer in critical situations.

## 3. MQTT / bridge

MQTT is the communication layer between IoT and the system.

In `System/docker-compose.vps.yml`, there is:

```text
mqtt-broker
mqtt-http-bridge
```

The bridge has env values:

```text
MQTT_TOPIC: iot/readings
BACKEND_URL: http://main-server:8080/api/readings
```

So:

```text
IoT publishes reading
MQTT broker receives it
bridge forwards it to backend POST /api/readings
```

Good answer:

> MQTT was used because IoT devices often communicate through lightweight publish/subscribe messaging. The MQTT-to-HTTP bridge connected this IoT communication with the backend API by forwarding readings to POST /api/readings.

## 4. Backend

The backend is the coordination center.

It:

- receives sensor readings,
- stores data in the database,
- sends data to the ML service,
- receives prediction/classification,
- exposes an API for the frontend,
- publishes alarm commands through MQTT,
- handles auth, roles, users, rooms, and devices.

From the frontend perspective, the most important endpoints are in `System/FrontEnd/src/services/api.js`:

```text
/api/auth/login
/api/auth/register
/api/readings
/api/readings/devices
/api/readings/alerts
/api/readings/alarm-test
/api/readings/stream
/api/room
/api/users
/api/logs
```

Good sentence:

> The backend coordinates the system. It receives readings, stores them, calls the ML service for classification, exposes data to the frontend through REST and live streams, and can publish alarm commands back to IoT devices.

## 5. ML service

ML is not in the frontend. It is a separate service.

It:

- receives sensor data,
- classifies the state,
- returns something like category / confidence / risk level.

Example:

```text
Normal
Cooking Smoke
Unhealthy Air
Fire Risk
```

Good sentence:

> The ML service is responsible for classification. The frontend does not calculate the risk itself; it displays the classification and recommendations based on data received from the backend.

This is important because it shows the boundary of responsibility.

## 6. Database

The database stores:

- readings,
- predictions,
- users,
- devices,
- rooms,
- alert history,
- logs / events.

In `docker-compose.vps.yml`, the database is `mysql:8`, and the backend receives `DB_HOST`, `DB_NAME`, `DB_USER`, and `DB_PASSWORD`.

Why?

> The database makes it possible to show historical readings, alerts, device status, and management data in the frontend.

## 7. Frontend

The frontend is the presentation and interaction layer.

It receives from the backend:

- latest readings,
- historical readings,
- classifications,
- alerts,
- rooms,
- devices,
- users,
- logs.

It shows the user:

- dashboard,
- charts,
- alerts,
- recommendations,
- role-based views.

Most importantly:

> The frontend depends on the backend API. It does not communicate directly with IoT devices, the database, or the ML service.

## 8. Example flow: fire risk

You should be able to explain this.

```text
1. IoT sensor measures abnormal values.
2. IoT publishes the reading to MQTT topic iot/readings.
3. MQTT-to-HTTP bridge forwards it to backend POST /api/readings.
4. Backend stores the reading and sends it to ML.
5. ML returns classification and risk level.
6. Backend stores the prediction.
7. Frontend receives updated data through REST/SSE and shows alert/recommendation.
8. If critical, backend can publish alarm command back to IoT.
```

Nice answer in English:

> If the sensors detect abnormal room conditions, the IoT device sends the reading through MQTT. The bridge forwards it to the backend, which stores the reading and sends it to the ML service. The ML service returns a classification and risk level. The backend stores the result and exposes it to the frontend through REST endpoints and a live readings stream. The frontend then shows the alert, classification, and recommendation to the user. In critical cases, the backend can also send an alarm command back to the IoT device.

## 9. What NOT to say

Do not say:

> The frontend detects fire.

Because that is not true.

Better:

> The frontend displays fire-risk information calculated by the backend and ML pipeline.

Do not say:

> The frontend talks to sensors.

Better:

> The frontend talks to the backend API, which represents sensor/device data.

Do not say:

> Playwright tests the whole IoT/backend/ML system.

Better:

> Playwright tests frontend browser flows with mocked backend responses. Full live-system verification was still partly manual.

## Code check

Confirmed in the code:

- `System/docker-compose.vps.yml` defines the full stack.
- `mqtt-http-bridge` forwards `iot/readings` to `http://main-server:8080/api/readings`.
- `frontend` depends on the `main-server` healthcheck.
- `System/FrontEnd/src/services/api.js` shows that the frontend only talks to backend `/api/...`.
- `connectReadingsStream()` uses SSE through `EventSource`.
- `postAlarmTest()` sends `POST /api/readings/alarm-test?sensorId=...&level=...`, so the alarm command also goes through the backend, not directly to IoT.

## Remember

```text
IoT collects
MQTT transports
Bridge forwards
Backend coordinates
ML classifies
Database stores
Frontend presents
User reacts
```

This is exactly the level of "frontend developer understands the system". You do not need to pretend to be an embedded/backend/ML expert. It is enough to clearly show the dependencies.
