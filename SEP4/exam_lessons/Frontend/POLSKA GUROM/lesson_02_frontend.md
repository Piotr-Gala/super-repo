# Lekcja 2: Frontend AeroSense

Cel tej lekcji: masz umieć wyjaśnić **co frontend robi**, **z czego się składa** i **dlaczego tak został zaprojektowany**.

## 1. Rola frontendu

Frontend to część systemu, którą widzi użytkownik.

Nie robi pomiarów.  
Nie trenuje ML.  
Nie zapisuje bezpośrednio do bazy.

Frontend:

- pokazuje aktualne dane z sensorów,
- pokazuje klasyfikację ryzyka,
- pokazuje alerty i historię,
- daje rekomendacje,
- pozwala adminom zarządzać pokojami, sensorami, userami,
- obsługuje logowanie i role,
- reaguje na błędy backendu / brak danych.

Najważniejsze zdanie:

> The frontend is responsible for presenting processed sensor data in a clear and usable way, so residents and administrators can understand the room condition and react if needed.

## 2. Główne widoki

Z raportu i kodu wychodzą takie widoki:

- **Home / Dashboard**  
  Pokazuje najnowsze odczyty: temperature, humidity, CO2, TVOC, eCO2, AQI, classification i recommendations.

- **Sensors / Samples**  
  Pokazuje dane konkretnych sensorów i historię odczytów. Tu user może zobaczyć trend, a nie tylko ostatnią wartość.

- **Payload**  
  Pokazuje raw JSON payload. To jest techniczny/debug widok: czy dane mają dobry format.

- **Rooms**  
  Widok do tworzenia pokojów i przypisywania sensorów do pokojów.

- **Alerts**  
  Pokazuje readings, gdzie classification nie była normalna.

- **Alarm / Settings**  
  Pozwala wysłać test alarmu dla konkretnego sensora, np. `critical`, `warn`, `off`.

- **Users / Devices / Logs**  
  Widoki system-admina: zarządzanie użytkownikami, status urządzeń, logi systemowe.

Na egzaminie nie musisz recytować wszystkich nazw. Ważniejsze jest pogrupowanie:

> The frontend has monitoring views, management views, technical/debug views, and alert/history views.

## 3. Role-based access

Frontend nie pokazuje wszystkim tego samego. Są role:

- **resident**  
  Widzi ograniczone informacje, głównie swoje przypisane sensory/pokoje.

- **building-administrator**  
  Widzi więcej: wszystkie urządzenia, pokoje, alerty, alarm controls.

- **admin / system admin**  
  Ma pełniejsze widoki: users, devices, logs, role changes, sensor assignment.

Po co to?

Bo realny system budynkowy nie powinien dawać wszystkim pełnej kontroli. Resident nie powinien zarządzać userami albo logami.

Dobra odpowiedź:

> We used role-based access because different users need different levels of control. Residents mainly need to monitor their own room conditions, while administrators need management views for rooms, devices, users, alerts, and logs.

Trade-off:

> It makes the frontend more complex, because navigation and default views depend on the current role, but it makes the application closer to a real-world system.

## 4. React architecture

Frontend jest React + Vite.

Nie mów tylko “we used React because it is popular”. To brzmi słabo.

Lepsza wersja:

> React was suitable because the dashboard is built from reusable UI parts, such as cards, charts, navigation, status components, and forms. It also works well for state changes, for example when new readings arrive or when the user role changes.

Struktura z kodu wygląda tak:

```text
System/FrontEnd/src/App.jsx
  -> stores session in localStorage
  -> decides LoginPage vs Dashboard
  -> owns dashboard state
  -> loads readings/devices
  -> connects SSE stream
  -> chooses role-based views

System/FrontEnd/src/services/api.js
  -> communication with backend

System/FrontEnd/src/components/...
  -> reusable UI components
```

Najważniejsza separacja:

- komponenty UI pokazują dane,
- `api.js` gada z backendem,
- role/permissions decydują, kto co widzi.

## 5. Komunikacja z backendem

Frontend komunikuje się z backendem przez API.

W kodzie `api.js` są m.in.:

- `login()` -> `POST /api/auth/login`,
- `register()` -> `POST /api/auth/register`,
- `getReadings()` -> `GET /api/readings`,
- `getDevices()` -> `GET /api/readings/devices`,
- `getAlerts()` -> `GET /api/readings/alerts`,
- `postAlarmTest()` -> `POST /api/readings/alarm-test`,
- `connectReadingsStream()` -> `EventSource /api/readings/stream`,
- `getRooms()`, `createRoom()`, `assignSensorToRoom()`,
- `getUsers()`, `changeUserRole()`, `assignSensorToUser()`, `deleteUser()`,
- `getLogs()`.

Prosto:

> Normal requests use REST API calls, and live sensor updates use Server-Sent Events, so the dashboard can update when new readings arrive.

Ważna rzecz: frontend **nie wymyśla klasyfikacji sam**. On dostaje wynik z backendu, który wcześniej dostał/obsłużył wynik ML.

## 6. Loading and error states

Frontend musi obsłużyć sytuacje:

- backend offline,
- brak odczytów,
- błąd logowania,
- brak uprawnień,
- dane jeszcze się ładują.

W kodzie `App.jsx` pokazuje `StatusScreen` przy loadingu i przy błędzie backendu:

> Dashboard offline / Backend API not reachable.

Dobra odpowiedź:

> We added loading and error states so the user does not see a broken or empty interface when data is delayed or unavailable. For example, if the backend is offline, the frontend shows an offline status screen instead of failing silently.

## 7. Ważna korekta z kodu

W raporcie role są opisane prosto, ale kod ma mały historyczny bałagan:

- `App.jsx` ma właściwą sesję z backendu, token w `localStorage` i role z zalogowanego usera.
- `src/auth/accessControl.js` wygląda jak starsza/demo warstwa do testowania roli i uprawnień. Ma nawet tekst: `Prototype role selection only. No backend authentication or JWT is implemented.`
- To nie znaczy, że cała appka nie ma auth. `api.js` i `App.jsx` realnie obsługują token z backendu.

Jak ktoś spyta, możesz powiedzieć uczciwie:

> The final app uses backend login and stores the returned token locally. There was also a smaller access-control helper used for role mapping and tests, which reflects earlier prototype logic, but the main application session is based on the backend auth response.

## Najlepsza odpowiedź egzaminacyjna

> The frontend is a React and Vite application structured around reusable components, role-based views, and an API service layer. App.jsx handles the logged-in session, stores the token and user data locally, and decides whether to show the login page or the dashboard. The dashboard loads readings, devices and other data from api.js, connects to a Server-Sent Events stream for live readings, and renders views such as Home, Sensors, Samples, Payload, Rooms, Alerts, Alarm, Users, Devices and Logs depending on the user role. This keeps UI rendering separated from backend communication and makes the dashboard easier to maintain.

## Do zapamiętania

```text
React + Vite dashboard
+ App.jsx session/dashboard state
+ api.js backend contract
+ role-based views
+ SSE live readings
+ cards/charts/alerts/recommendations
+ loading/error handling
```
