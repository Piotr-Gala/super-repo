# Lekcja 1: Co to jest AeroSense?

AeroSense to system do monitorowania ryzyka pożaru i jakości powietrza w pomieszczeniach.

Nie myśl o tym jako “frontend appka”. Myśl o tym jako o całym pipeline danych:

```text
Physical room
  -> IoT device reads sensors
  -> backend receives readings
  -> ML classifies risk
  -> backend stores result and may trigger alarm
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

**Jak przepływa dane?**

Przykład:

1. Sensor mierzy np. `temperature`, `humidity`, `CO2`, `TVOC`.
2. Dane trafiają do backendu.
3. Backend wysyła dane do ML service.
4. ML zwraca klasyfikację, np. `Normal`, `Cooking Smoke`, `Fire Risk`.
5. Backend zapisuje reading + prediction.
6. Frontend pobiera albo dostaje live update.
7. User widzi kartę, alert, rekomendację albo historię.

Czyli jeśli egzaminator spyta:

> How does the frontend fit into the system?

Nie odpowiadasz tylko:

> It displays data.

Bo to za biedne.

Lepsza odpowiedź:

> The frontend is the user-facing part of AeroSense. It receives processed sensor readings from the backend, including classifications from the ML service, and presents them through dashboards, charts, alerts, recommendations, and role-based management views. It does not classify the data itself; it depends on the backend and ML pipeline, but it makes the result understandable and actionable for users.

To jest dobra, spokojna odpowiedź.

Na tym etapie masz zapamiętać 3 rzeczy:

- **IoT collects data**
- **Backend processes/stores/coordinates**
- **Frontend visualizes and helps users react**