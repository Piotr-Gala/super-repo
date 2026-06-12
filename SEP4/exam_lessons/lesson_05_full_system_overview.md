# Lekcja 5: Full System Overview

Tu nie chodzi o to, żebyś umiał cudzy kod. Chodzi o to, żebyś umiał pokazać, że rozumiesz **jak frontend pasuje do całego systemu**.

**1. AeroSense jako pipeline**
Najprostszy model:

```text
IoT device
  -> MQTT / bridge
  -> Backend
  -> Database
  -> ML service
  -> Frontend
  -> User reaction
```

Jedno zdanie:

> AeroSense collects sensor data from IoT devices, processes it through the backend and ML service, stores it, and presents it in the frontend so users can monitor room conditions and react to risks.

**2. IoT**
IoT odpowiada za fizyczny świat.

Robi:

- mierzy temperature, humidity, CO2, TVOC, eCO2, AQI / air quality,
- wysyła readings,
- może dostać alarm command,
- może uruchomić buzzer.

Nie musisz znać driverów. Masz znać rolę:

> The IoT layer collects environmental data from sensors and sends it into the system. It can also receive alarm commands from the backend, for example to trigger a buzzer in critical situations.

**3. MQTT / bridge**
MQTT to komunikacja między IoT a systemem.

W projekcie był też bridge, który przekładał dane z MQTT na HTTP request do backendu.

Prosto:

> MQTT was used because IoT devices often communicate through lightweight publish/subscribe messaging. The MQTT-to-HTTP bridge connected this IoT communication with the backend API.

Czyli:

```text
IoT publishes reading
MQTT broker receives it
bridge forwards it to backend
```

**4. Backend**
Backend jest centrum koordynacji.

Robi:

- przyjmuje sensor readings,
- zapisuje dane do bazy,
- wysyła dane do ML service,
- odbiera prediction/classification,
- udostępnia API dla frontendu,
- publikuje alarm command przez MQTT,
- obsługuje auth, roles, users, rooms, devices.

Dobre zdanie:

> The backend coordinates the system. It receives readings, stores them, calls the ML service for classification, exposes data to the frontend through REST and live streams, and can publish alarm commands back to IoT devices.

**5. ML service**
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

**6. Database**
Baza przechowuje:

- readings,
- predictions,
- users,
- devices,
- rooms,
- alert history,
- logs / events.

Po co?

> The database makes it possible to show historical readings, alerts, device status, and management data in the frontend.

**7. Frontend**
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

**8. Przykładowy flow: fire risk**
To musisz umieć opowiedzieć.

```text
1. IoT sensor measures abnormal values.
2. IoT publishes the reading through MQTT.
3. Bridge forwards it to the backend.
4. Backend stores the reading and sends it to ML.
5. ML returns classification and risk level.
6. Backend stores the prediction.
7. Frontend receives updated data and shows alert/recommendation.
8. If critical, backend can publish alarm command to IoT.
```

Ładna odpowiedź po angielsku:

> If the sensors detect abnormal room conditions, the IoT device sends the reading through MQTT. The bridge forwards it to the backend, which stores the reading and sends it to the ML service. The ML service returns a classification and risk level. The backend stores the result and exposes it to the frontend. The frontend then shows the alert, classification, and recommendation to the user. In critical cases, the backend can also send an alarm command back to the IoT device.

**9. Czego NIE mówić**
Nie mów:

> The frontend detects fire.

Bo to nieprawda.

Lepsze:

> The frontend displays fire-risk information calculated by the backend and ML pipeline.

Nie mów:

> The frontend talks to sensors.

Lepsze:

> The frontend talks to the backend API, which represents sensor/device data.

**Do zapamiętania**
Najkrótsza mapa:

```text
IoT collects
MQTT transports
Backend coordinates
ML classifies
Database stores
Frontend presents
User reacts
```

To jest dokładnie poziom “frontendowiec rozumie system”. Nie musisz udawać embedded/backend/ML eksperta. Wystarczy, że umiesz jasno pokazać zależności.