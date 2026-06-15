# Lekcja 5: Full System Overview

Tu nie chodzi o to, żebyś umiał cudzy kod. Chodzi o to, żebyś umiał pokazać, że rozumiesz **jak frontend pasuje do całego systemu**.

## 1. AeroSense jako pipeline

Najprostszy model:

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

Jedno zdanie:

> AeroSense collects sensor data from IoT devices, processes it through the backend and ML service, stores it, and presents it in the frontend so users can monitor room conditions and react to risks.

## 2. IoT

IoT odpowiada za fizyczny świat.

Robi:

- mierzy temperature, humidity, CO2, TVOC, eCO2, AQI / air quality,
- wysyła readings,
- może dostać alarm command,
- może uruchomić buzzer.

Nie musisz znać driverów. Masz znać rolę:

> The IoT layer collects environmental data from sensors and sends it into the system. It can also receive alarm commands from the backend, for example to trigger a buzzer in critical situations.

## 3. MQTT / bridge

MQTT to komunikacja między IoT a systemem.

W `System/docker-compose.vps.yml` jest:

```text
mqtt-broker
mqtt-http-bridge
```

Bridge ma env:

```text
MQTT_TOPIC: iot/readings
BACKEND_URL: http://main-server:8080/api/readings
```

Czyli:

```text
IoT publishes reading
MQTT broker receives it
bridge forwards it to backend POST /api/readings
```

Dobra odpowiedź:

> MQTT was used because IoT devices often communicate through lightweight publish/subscribe messaging. The MQTT-to-HTTP bridge connected this IoT communication with the backend API by forwarding readings to POST /api/readings.

## 4. Backend

Backend jest centrum koordynacji.

Robi:

- przyjmuje sensor readings,
- zapisuje dane do bazy,
- wysyła dane do ML service,
- odbiera prediction/classification,
- udostępnia API dla frontendu,
- publikuje alarm command przez MQTT,
- obsługuje auth, roles, users, rooms, devices.

Z perspektywy frontendu najważniejsze endpointy są w `System/FrontEnd/src/services/api.js`:

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

Dobre zdanie:

> The backend coordinates the system. It receives readings, stores them, calls the ML service for classification, exposes data to the frontend through REST and live streams, and can publish alarm commands back to IoT devices.

## 5. ML service

ML nie jest w frontendzie. To osobny serwis.

Robi:

- dostaje sensor data,
- klasyfikuje stan,
- zwraca np. category / confidence / risk level.

Przykład:

```text
Normal
Cooking Smoke
Unhealthy Air
Fire Risk
```

Dobre zdanie:

> The ML service is responsible for classification. The frontend does not calculate the risk itself; it displays the classification and recommendations based on data received from the backend.

To jest ważne, bo pokazuje granicę odpowiedzialności.

## 6. Database

Baza przechowuje:

- readings,
- predictions,
- users,
- devices,
- rooms,
- alert history,
- logs / events.

W `docker-compose.vps.yml` baza to `mysql:8`, a backend dostaje `DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`.

Po co?

> The database makes it possible to show historical readings, alerts, device status, and management data in the frontend.

## 7. Frontend

Frontend jest warstwą prezentacji i interakcji.

Dostaje z backendu:

- latest readings,
- historical readings,
- classifications,
- alerts,
- rooms,
- devices,
- users,
- logs.

Pokazuje użytkownikowi:

- dashboard,
- charts,
- alerts,
- recommendations,
- role-based views.

Najważniejsze:

> The frontend depends on the backend API. It does not communicate directly with IoT devices, the database, or the ML service.

## 8. Przykładowy flow: fire risk

To musisz umieć opowiedzieć.

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

Ładna odpowiedź po angielsku:

> If the sensors detect abnormal room conditions, the IoT device sends the reading through MQTT. The bridge forwards it to the backend, which stores the reading and sends it to the ML service. The ML service returns a classification and risk level. The backend stores the result and exposes it to the frontend through REST endpoints and a live readings stream. The frontend then shows the alert, classification, and recommendation to the user. In critical cases, the backend can also send an alarm command back to the IoT device.

## 9. Czego NIE mówić

Nie mów:

> The frontend detects fire.

Bo to nieprawda.

Lepsze:

> The frontend displays fire-risk information calculated by the backend and ML pipeline.

Nie mów:

> The frontend talks to sensors.

Lepsze:

> The frontend talks to the backend API, which represents sensor/device data.

Nie mów:

> Playwright tests the whole IoT/backend/ML system.

Lepsze:

> Playwright tests frontend browser flows with mocked backend responses. Full live-system verification was still partly manual.

## Code check

Potwierdzone w kodzie:

- `System/docker-compose.vps.yml` definiuje pełny stack.
- `mqtt-http-bridge` forwarduje `iot/readings` do `http://main-server:8080/api/readings`.
- `frontend` zależy od `main-server` healthcheck.
- `System/FrontEnd/src/services/api.js` pokazuje, że frontend gada tylko z backendowym `/api/...`.
- `connectReadingsStream()` używa SSE przez `EventSource`.
- `postAlarmTest()` wysyła `POST /api/readings/alarm-test?sensorId=...&level=...`, więc alarm command też idzie przez backend, nie bezpośrednio do IoT.

## Do zapamiętania

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

To jest dokładnie poziom “frontendowiec rozumie system”. Nie musisz udawać embedded/backend/ML eksperta. Wystarczy, że umiesz jasno pokazać zależności.
