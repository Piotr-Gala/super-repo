# Lesson 2: AeroSense Frontend

Goal of this lesson: you should be able to explain **what the frontend does**, **what it consists of**, and **why it was designed this way**.

## 1. Role of the frontend

The frontend is the part of the system that the user sees.

It does not take measurements.  
It does not train ML.  
It does not write directly to the database.

The frontend:

- shows current sensor data,
- shows risk classification,
- shows alerts and history,
- gives recommendations,
- allows administrators to manage rooms, sensors, and users,
- handles login and roles,
- reacts to backend errors or missing data.

The most important sentence:

> The frontend is responsible for presenting processed sensor data in a clear and usable way, so residents and administrators can understand the room condition and react if needed.

## 2. Main views

Based on the report and code, the frontend has these views:

- **Home / Dashboard**  
  Shows the latest readings: temperature, humidity, CO2, TVOC, eCO2, AQI, classification, and recommendations.

- **Sensors / Samples**  
  Shows data for specific sensors and historical readings. Here the user can see a trend, not only the latest value.

- **Payload**  
  Shows the raw JSON payload. This is a technical/debug view used to check whether the data has the correct format.

- **Rooms**  
  View for creating rooms and assigning sensors to rooms.

- **Alerts**  
  Shows readings where the classification was not normal.

- **Alarm / Settings**  
  Allows sending a test alarm for a specific sensor, for example `critical`, `warn`, or `off`.

- **Users / Devices / Logs**  
  System-admin views: user management, device status, and system logs.

In the exam, you do not need to recite every name. The grouping matters more:

> The frontend has monitoring views, management views, technical/debug views, and alert/history views.

## 3. Role-based access

The frontend does not show the same thing to every user. There are roles:

- **resident**  
  Sees limited information, mainly their assigned sensors/rooms.

- **building-administrator**  
  Sees more: all devices, rooms, alerts, and alarm controls.

- **admin / system admin**  
  Has more complete views: users, devices, logs, role changes, and sensor assignment.

Why?

Because a real building system should not give everyone full control. A resident should not manage users or logs.

Good answer:

> We used role-based access because different users need different levels of control. Residents mainly need to monitor their own room conditions, while administrators need management views for rooms, devices, users, alerts, and logs.

Trade-off:

> It makes the frontend more complex, because navigation and default views depend on the current role, but it makes the application closer to a real-world system.

## 4. React architecture

The frontend uses React + Vite.

Do not say only "we used React because it is popular". That sounds weak.

Better version:

> React was suitable because the dashboard is built from reusable UI parts, such as cards, charts, navigation, status components, and forms. It also works well for state changes, for example when new readings arrive or when the user role changes.

The structure from the code looks like this:

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

The most important separation:

- UI components display data,
- `api.js` talks to the backend,
- roles/permissions decide who sees what.

## 5. Communication with the backend

The frontend communicates with the backend through the API.

In `api.js`, there are functions such as:

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

Simply:

> Normal requests use REST API calls, and live sensor updates use Server-Sent Events, so the dashboard can update when new readings arrive.

Important: the frontend **does not invent the classification by itself**. It receives the result from the backend, which previously received or handled the ML result.

## 6. Loading and error states

The frontend must handle situations such as:

- backend offline,
- no readings,
- login error,
- missing permissions,
- data still loading.

In the code, `App.jsx` shows `StatusScreen` during loading and backend errors:

> Dashboard offline / Backend API not reachable.

Good answer:

> We added loading and error states so the user does not see a broken or empty interface when data is delayed or unavailable. For example, if the backend is offline, the frontend shows an offline status screen instead of failing silently.

## 7. Important correction from the code

The report describes roles simply, but the code has a small historical inconsistency:

- `App.jsx` has the proper session from the backend, a token in `localStorage`, and the role from the logged-in user.
- `src/auth/accessControl.js` looks like an older/demo layer for testing roles and permissions. It even contains the text: `Prototype role selection only. No backend authentication or JWT is implemented.`
- This does not mean that the whole app has no auth. `api.js` and `App.jsx` really handle the token from the backend.

If someone asks, you can answer honestly:

> The final app uses backend login and stores the returned token locally. There was also a smaller access-control helper used for role mapping and tests, which reflects earlier prototype logic, but the main application session is based on the backend auth response.

## Best exam answer

> The frontend is a React and Vite application structured around reusable components, role-based views, and an API service layer. App.jsx handles the logged-in session, stores the token and user data locally, and decides whether to show the login page or the dashboard. The dashboard loads readings, devices and other data from api.js, connects to a Server-Sent Events stream for live readings, and renders views such as Home, Sensors, Samples, Payload, Rooms, Alerts, Alarm, Users, Devices and Logs depending on the user role. This keeps UI rendering separated from backend communication and makes the dashboard easier to maintain.

## Remember

```text
React + Vite dashboard
+ App.jsx session/dashboard state
+ api.js backend contract
+ role-based views
+ SSE live readings
+ cards/charts/alerts/recommendations
+ loading/error handling
```
