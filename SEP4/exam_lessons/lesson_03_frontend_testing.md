# Lekcja 3: Frontend Testing

To jest ważne, bo w raporcie ta część jest podpisana przez Ciebie, więc egzaminator może naturalnie pójść w pytania typu: “what exactly did you test and why?”.

**1. Po co testowaliście frontend?**
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

**2. Mieliście 3 poziomy testów**
Z raportu:

```text
Unit tests
Component tests
Black-box / browser tests
```

Plus osobno: API service tests.

Czyli praktycznie:

```text
Vitest
React Testing Library
fetch mocks / API tests
Playwright
```

**3. Unit tests**
Unit testy sprawdzają małą logikę bez UI.

U was: **access-control logic**.

Czyli np.:

- unknown role fallbackuje do resident,
- aliasy ról mapują się poprawnie,
- resident i admin mają inne permissions,
- default dashboard view jest wybrany z dozwolonych widoków.

Po co?

Bo role-based access to potencjalnie ryzykowna część. Jak to się zepsuje, user może zobaczyć widoki, których nie powinien.

Odpowiedź:

> Unit tests were used for pure access-control logic, because this logic does not need a browser or React rendering. We tested role mapping, permission differences, fallback behavior, and default view selection.

**4. Component tests**
Component tests sprawdzają pojedyncze komponenty UI.

U was były testowane m.in.:

- `LoginPage`,
- `LoginRoleSelect`,
- `UserMenu`,
- `RestrictedNotice`,
- `AdminControls`.

Czyli:

- czy formularz logowania ma username/password/submit,
- czy role są widoczne i wybieralne,
- czy logout odpala callback,
- czy restricted notice pokazuje komunikat,
- czy alarm buttons są disabled kiedy trzeba,
- czy kliknięcie alarmu wywołuje dobry handler: `critical`, `warn`, `off`.

Po co?

Bo te komponenty mają zachowanie, nie tylko wygląd.

Dobra odpowiedź:

> Component tests verified isolated UI behavior. For example, the login form renders the expected controls, the user menu calls logout, restricted views show an access-denied message, and alarm controls call the correct handler with the correct alarm mode.

**5. API service tests**
To bardzo dobry temat na egzamin, bo brzmi konkretnie.

Frontend gada z backendem przez `api.js`.  
Najczęstsze bugi są właśnie tam:

- zły URL,
- zła metoda HTTP,
- brak `Authorization` header,
- złe JSON body,
- złe error message,
- źle zbudowany stream URL.

Więc testowaliście requesty mockując `fetch`.

Dobra odpowiedź:

> API service tests mocked fetch and verified that the frontend sends the correct URLs, HTTP methods, authorization headers, request bodies, and handles errors correctly. This was important because many frontend bugs in our project would happen at the backend/frontend boundary.

To zdanie jest złoto. Zapamiętaj.

**6. Playwright tests**
Playwright to testy w prawdziwej przeglądarce.

To nie sprawdza jednego komponentu, tylko user flow:

- login,
- dashboard loads,
- role-based access,
- backend offline screen,
- responsive layout,
- desktop + mobile viewport.

Po co Playwright, skoro są unit/component tests?

Bo unit test nie powie Ci, czy cała aplikacja działa razem w przeglądarce.

Dobra odpowiedź:

> Playwright tests checked the application from the user’s point of view in a real browser. This gives confidence that routing, rendering, mocked backend responses, role-based navigation, and responsive layout work together, not only as isolated functions.

**7. Responsive testing**
W raporcie jest ważny konkret:

- desktop Chromium,
- Pixel 5 mobile viewport,
- test sprawdza, czy dashboard/navigation są widoczne,
- brak horizontal overflow.

Czyli jeśli pytają o mobile:

> We tested responsive behavior with Playwright using desktop and Pixel 5 mobile viewports. The goal was to verify that the main dashboard and navigation remain usable and that the page does not create horizontal overflow.

**8. Wyniki**
Z raportu:

```text
Vitest: 7 test files, 36 tests passed
Playwright: 6 browser tests across desktop and mobile Chromium
npm run build passed
Latest local verification: 2026-05-26
```

Nie musisz tego recytować perfekcyjnie, ale warto znać liczby.

**9. Limitations**
Nie udawaj, że testy były idealne. Egzaminator lubi uczciwość.

Limitacje:

- nie każdy visual case był automatyzowany,
- chart readability głównie manualnie,
- live backend data nie było w pełni testowane end-to-end,
- abnormal alarm states częściowo mockowane/manualne,
- brak pełnego e2e z realnym backendem/IoT/ML.

Dobra odpowiedź:

> The main limitation is that not every visual and live-data scenario was automated. Some chart readability, live backend data, and abnormal alarm states were checked manually or with mocked data instead of a full end-to-end setup with real IoT, backend, and ML services.

**Najlepsza odpowiedź egzaminacyjna**
Pytanie:

> How did you test the frontend?

Odpowiedź:

> We tested the frontend on several levels. Unit tests with Vitest covered access-control logic, such as role mapping, permissions, fallback roles, and default views. Component tests with React Testing Library checked isolated UI behavior, for example login, role selection, user menu logout, restricted notices, and alarm controls. API service tests mocked fetch to verify correct endpoints, HTTP methods, authorization headers, request bodies, and error handling. Finally, Playwright black-box tests checked important user flows in a real browser, including login, dashboard display, backend-offline behavior, role-based access, and responsive layout on desktop and mobile.

To jest Twoja bazowa odpowiedź. Jak ją powiesz spokojnie, brzmisz jak ktoś, kto realnie wie, po co te testy były.