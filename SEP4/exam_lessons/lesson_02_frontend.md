# Lekcja 2: Frontend AeroSense

Cel tej lekcji: masz umieć wyjaśnić **co frontend robi**, **z czego się składa** i **dlaczego tak został zaprojektowany**.

**1. Rola frontendu**
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

**2. Główne widoki**
Z raportu wynika, że frontend miał kilka głównych części:

- **Dashboard / Home**  
  Pokazuje najnowsze odczyty: temperature, humidity, CO2, TVOC, eCO2, AQI.  
  To jest główny ekran do szybkiego sprawdzenia stanu pokoju.

- **Sensors / Samples**  
  Pokazuje dane konkretnych sensorów i historię odczytów.  
  Tu user może zobaczyć trend, a nie tylko ostatnią wartość.

- **Alerts**  
  Pokazuje sytuacje, gdzie classification nie była normalna.  
  Czyli np. cooking smoke, unhealthy air, possible fire risk.

- **Rooms**  
  Widok admina do tworzenia pokojów i przypisywania sensorów do pokojów.

- **Payload viewer**  
  Pokazuje raw JSON payload.  
  To jest bardziej techniczny widok, przydatny do debugowania: czy dane przyszły w dobrym formacie.

- **Users / Devices / Logs / Alarm**  
  Bardziej administracyjne widoki: użytkownicy, urządzenia, logi, test alarmu.

Na egzaminie nie musisz recytować wszystkich nazw. Ważniejsze jest pogrupowanie:

> The frontend has monitoring views, management views, technical/debug views, and alert/history views.

**3. Role-based access**
To jest bardzo ważny temat.

Frontend nie pokazuje wszystkim tego samego. Są role:

- **resident**  
  Widzi ograniczone informacje, głównie swoje przypisane pokoje/sensory.

- **building administrator**  
  Może widzieć więcej, np. pokoje, sensory, alerty.

- **system administrator**  
  Ma dostęp do bardziej technicznych rzeczy: users, devices, logs, alarm controls.

Po co to?

Bo realny system budynkowy nie powinien dawać wszystkim pełnej kontroli. Resident nie powinien zarządzać userami albo testować alarmów.

Dobra odpowiedź:

> We used role-based access because different users need different levels of control. Residents mainly need to monitor their own room conditions, while administrators need management views for rooms, devices, users, alerts, and logs.

Trade-off:

> It makes the frontend more complex, because navigation and default views depend on the current role, but it makes the application closer to a real-world system.

**4. React architecture**
Frontend był React appką.

Nie mów tylko “we used React because it is popular”. To brzmi słabo.

Lepsza wersja:

> React was suitable because the dashboard is built from reusable UI parts, such as cards, charts, navigation, status components, and forms. It also works well for state changes, for example when new readings arrive or when the user role changes.

Czyli React pasuje, bo frontend ma dużo dynamicznych elementów.

Z raportu struktura wygląda mniej więcej tak:

```text
App.jsx
  -> handles session/auth state
  -> shows LoginPage or Dashboard

Dashboard
  -> Sidebar
  -> Topbar
  -> role-based views
  -> reusable cards/charts/forms

api.js
  -> communication with backend
```

Najważniejsza separacja:

- komponenty UI pokazują dane,
- `api.js` gada z backendem,
- access-control logic decyduje, kto co widzi.

**5. Komunikacja z backendem**
Frontend komunikuje się z backendem przez API.

Typowe rzeczy:

- login,
- loading readings,
- fetching alerts,
- managing rooms,
- managing users,
- loading logs,
- sending alarm test commands.

Plus live updates przez **Server-Sent Events**.

Prosto:

> Normal requests use REST API calls, and live sensor updates use Server-Sent Events, so the dashboard can update when new readings arrive.

Ważna rzecz: frontend **nie wymyśla klasyfikacji sam**. On dostaje wynik z backendu, który wcześniej dostał/obsłużył wynik ML.

**6. Loading and error states**
To też jest dobre na egzamin.

Frontend musi obsłużyć sytuacje:

- backend offline,
- brak odczytów,
- błąd logowania,
- brak uprawnień,
- dane jeszcze się ładują.

Dlaczego to ważne?

Bo system zależy od wielu części: IoT, backend, ML, baza, stream. Coś może nie działać.

Dobra odpowiedź:

> We added loading and error states so the user does not see a broken or empty interface when data is delayed or unavailable. For example, if the backend is offline, the frontend shows an offline status screen instead of failing silently.

**7. Najlepsza odpowiedź egzaminacyjna**
Jak ktoś spyta:

> Can you explain the frontend architecture?

Możesz powiedzieć:

> The frontend is a React application structured around reusable components, role-based views, and an API service layer. App.jsx handles the user session and decides whether to show the login page or the dashboard. After login, the dashboard layout uses components like sidebar, topbar, cards, charts, room views, alerts, and status screens. Communication with the backend is separated into api.js, which handles authentication, readings, alerts, rooms, users, devices, logs, alarm commands, and live readings through Server-Sent Events. This structure keeps UI rendering separate from backend communication and makes the application easier to maintain.

To jest bardzo dobra odpowiedź. Nie za długa, nie za krótka.

**Do zapamiętania**
Frontend w AeroSense to:

```text
React dashboard
+ role-based access
+ API service layer
+ live readings stream
+ cards/charts/alerts/recommendations
+ loading/error handling
```

Na luzie: frontend jest “tłumaczem” między technicznymi danymi z systemu a człowiekiem, który musi szybko wiedzieć: **czy jest okej, czy trzeba reagować**.