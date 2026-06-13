# Lekcja 3: Frontend Testing

To jest ważne, bo w raporcie ta część jest podpisana przez Ciebie, więc egzaminator może naturalnie pójść w pytania typu: “what exactly did you test and why?”.

## 1. Po co testowaliście frontend?

Nie testujesz Reacta. React już działa.  
Testujesz, czy **wasza aplikacja** zachowuje się poprawnie.

Główne ryzyka frontendu w AeroSense:

- user nie może się zalogować,
- role pokazują złe widoki,
- frontend wysyła request do złego endpointu,
- brakuje tokena w requestach,
- alarm button wyśle złą komendę,
- dashboard rozwala się, gdy backend nie działa,
- layout nie działa na mobile.

Dobra odpowiedź:

> The goal of frontend testing was to verify the user-facing behavior and the API contract. We did not test React itself, but our own access-control logic, UI components, API calls, error handling, and important browser flows.

## 2. Narzędzia z kodu

W `System/FrontEnd/package.json` są:

```text
npm run test:run       -> vitest run
npm run test:e2e       -> playwright test
npm run build          -> vite build
```

Zależności potwierdzają:

```text
Vitest
React Testing Library
@testing-library/user-event
Playwright
Vite
ESLint
```

Czyli raport nie wymyśla narzędzi, one faktycznie są w projekcie.

## 3. Unit tests

Unit testy sprawdzają małą logikę bez UI.

W kodzie: `System/FrontEnd/src/auth/accessControl.test.js`.

Testy sprawdzają m.in.:

- unknown role fallbackuje do `resident`,
- alias `building-admin` mapuje się na `building-administrator`,
- alias `system-admin` mapuje się na `admin`,
- resident ma ograniczone permissions,
- admin może widzieć więcej,
- default view to `Home`,
- demo session tworzy poprawną rolę.

Dobra odpowiedź:

> Unit tests were used for pure access-control logic, because this logic does not need a browser or React rendering. We tested role mapping, permission differences, fallback behavior, and default view selection.

Uczciwy detal:

> This helper also reflects some prototype role-selection logic, while the main App.jsx uses backend login and the returned user role for the real session.

## 4. Component tests

Component tests sprawdzają pojedyncze komponenty UI.

W kodzie są testy m.in. dla:

- `LoginPage`,
- `LoginRoleSelect`,
- `UserMenu`,
- `RestrictedNotice`,
- `AdminControls`.

Najbardziej konkretne: `AdminControls.test.jsx` sprawdza, że:

- alarm buttons są disabled, gdy wybrane są `all` devices,
- buttons są disabled, gdy request jest busy,
- kliknięcia wywołują handler z `critical`, `warn`, `off`,
- error message z alarm testu jest renderowany jako danger.

Dobra odpowiedź:

> Component tests verified isolated UI behavior. For example, the login form renders the expected controls, the user menu calls logout, restricted views show an access-denied message, and alarm controls call the correct handler with the correct alarm mode.

## 5. API service tests

To bardzo dobry temat na egzamin, bo brzmi konkretnie.

W kodzie: `System/FrontEnd/src/services/api.test.js`.

Testy mockują `fetch` i sprawdzają:

- `GET /api/readings?sensorId=101&limit=25&hours=2`,
- `Authorization: Bearer test-token`,
- pomijanie `sensorId`, gdy wybrane jest `all`,
- błędy typu `Readings request failed (500)`,
- `/api/readings/devices`,
- `/api/readings/alerts`,
- `POST /api/readings/alarm-test`,
- `EventSource /api/readings/stream?sensorId=101&token=...`,
- `/api/room` i przypisywanie sensorów do pokoi,
- `/api/users`, role changes, assign sensor to user, delete user,
- `/api/logs`,
- login/register i fallback error messages.

Dobra odpowiedź:

> API service tests mocked fetch and verified that the frontend sends the correct URLs, HTTP methods, authorization headers, request bodies, stream URLs, and handles errors correctly. This was important because many frontend bugs in our project would happen at the backend/frontend boundary.

To zdanie jest złoto. Zapamiętaj.

## 6. Playwright tests

Playwright to testy w prawdziwej przeglądarce.

W kodzie: `System/FrontEnd/e2e/frontend-blackbox.spec.js`.

Są 3 scenariusze:

1. Po sign in dashboard renderuje dane.
2. Gdy readings API zwraca błąd, pokazuje się `Dashboard offline`.
3. Dashboard zostaje usable na responsive viewport i nie robi horizontal overflow.

W `playwright.config.js` te 3 scenariusze lecą na dwóch projektach:

```text
chromium-desktop: 1440x900
chromium-mobile: Pixel 5
```

Dlatego w raporcie wychodzi 6 browser tests.

Dobra odpowiedź:

> Playwright tests checked the application from the user’s point of view in a real browser. This gives confidence that routing, rendering, mocked backend responses, role-based navigation, error handling, and responsive layout work together, not only as isolated functions.

## 7. Responsive testing

W kodzie responsive test sprawdza:

```js
document.documentElement.scrollWidth <= window.innerWidth + 1
```

Czyli konkretnie: nie ma poziomego overflow.

Dobra odpowiedź:

> We tested responsive behavior with Playwright using desktop and Pixel 5 mobile viewports. The goal was to verify that the main dashboard and navigation remain usable and that the page does not create horizontal overflow.

## 8. Wyniki

Z raportu:

```text
Vitest: 7 test files, 36 tests passed
Playwright: 6 browser tests across desktop and mobile Chromium
npm run build passed
Latest local verification: 2026-05-26
```

Z kodu wynika, skąd to się bierze:

- Vitest zbiera unit/component/API tests,
- Playwright ma 3 scenariusze x 2 projekty,
- build to `vite build`.

## 9. Limitations

Nie udawaj, że testy były idealne. Egzaminator lubi uczciwość.

Limitacje:

- Playwright mockuje backend, więc to nie jest pełny end-to-end z realnym backendem, IoT i ML,
- live SSE jest raczej sprawdzane jako URL/flow, nie jako prawdziwy live stream z produkcji,
- chart readability i część visual checks były manualne,
- abnormal alarm states były częściowo mockowane/manualne,
- testy nie pokrywają każdego możliwego widoku dashboardu.

Dobra odpowiedź:

> The main limitation is that not every visual and live-data scenario was automated. Some chart readability, live backend data, and abnormal alarm states were checked manually or with mocked data instead of a full end-to-end setup with real IoT, backend, and ML services.

## Najlepsza odpowiedź egzaminacyjna

> We tested the frontend on several levels. Unit tests with Vitest covered access-control logic, such as role mapping, permissions, fallback roles, and default views. Component tests with React Testing Library checked isolated UI behavior, for example login, role selection, user menu logout, restricted notices, and alarm controls. API service tests mocked fetch to verify correct endpoints, HTTP methods, authorization headers, request bodies, stream URLs, and error handling. Finally, Playwright black-box tests checked important user flows in a real browser, including login, dashboard display, backend-offline behavior, and responsive layout on desktop and Pixel 5 mobile viewport.

To jest Twoja bazowa odpowiedź. Jak ją powiesz spokojnie, brzmisz jak ktoś, kto realnie wie, po co te testy były.
