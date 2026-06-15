# Lekcja 1: Co to jest AeroSense?

AeroSense to system do monitorowania ryzyka pożaru i jakości powietrza w pomieszczeniach.

Nie myśl o tym jako “frontend appka”. Myśl o tym jako o całym pipeline danych:

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

Czyli frontend jest **ostatnią warstwą systemu**, ale dla użytkownika najważniejszą, bo to tam widzi:

- aktualne warunki w pokoju,
- historię pomiarów,
- alerty,
- rekomendacje,
- status urządzeń,
- pokoje i przypisane sensory,
- czasem dane techniczne typu raw JSON payload.

Najważniejsze zdanie, które warto mieć w głowie:

> The frontend turns raw sensor and prediction data into understandable information for residents and administrators.

Po ludzku: backend/IoT/ML produkują dane, a frontend robi z nich coś, co człowiek może szybko zrozumieć.

## Jak przepływa dane?

Przykład:

1. Sensor mierzy np. `temperature`, `humidity`, `CO2`, `TVOC`.
2. IoT publikuje reading przez MQTT.
3. MQTT-to-HTTP bridge wysyła to do backendu przez `POST /api/readings`.
4. Backend zapisuje reading i wysyła dane do ML service.
5. ML zwraca klasyfikację, np. `Normal`, `Cooking Smoke`, `Fire Risk`.
6. Backend zapisuje reading + prediction.
7. Frontend pobiera dane przez REST albo dostaje live update przez SSE.
8. User widzi kartę, alert, rekomendację albo historię.

Czyli jeśli egzaminator spyta:

> How does the frontend fit into the system?

Nie odpowiadasz tylko:

> It displays data.

Bo to za biedne.

Lepsza odpowiedź:

> The frontend is the user-facing part of AeroSense. It receives processed sensor readings from the backend, including classifications from the ML service, and presents them through dashboards, charts, alerts, recommendations, and role-based management views. It does not classify the data itself; it depends on the backend and ML pipeline, but it makes the result understandable and actionable for users.

## Code check

To jest potwierdzone w kodzie:

- `System/FrontEnd/src/services/api.js` pokazuje, że frontend gada z backendem przez `/api/...`, np. `/api/readings`, `/api/readings/devices`, `/api/readings/alerts`, `/api/room`, `/api/users`, `/api/logs`, `/api/auth/login`.
- `connectReadingsStream()` używa `EventSource` na `/api/readings/stream`, czyli live updates idą przez Server-Sent Events.
- `System/docker-compose.vps.yml` pokazuje pełny stack: `mqtt-broker`, `mqtt-http-bridge`, `main-server`, `frontend`, `mysql`, `ml-server`.
- Frontend kontener zależy od zdrowego `main-server`, więc frontend nie jest osobnym systemem, tylko UI nad backendem.

## Do zapamiętania

- **IoT collects data**
- **MQTT transports readings**
- **Backend processes/stores/coordinates**
- **ML classifies risk**
- **Frontend visualizes and helps users react**

Najważniejsze: frontend **nie wykrywa pożaru sam**. Frontend pokazuje wynik pipeline'u backend + ML.
