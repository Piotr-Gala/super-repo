# Lesson 3: Frontend Testing

This is important because this part of the report is signed by you, so the examiner may naturally ask questions like: "what exactly did you test and why?".

## 1. Why did you test the frontend?

You do not test React. React already works.  
You test whether **your application** behaves correctly.

Main frontend risks in AeroSense:

- the user cannot log in,
- roles show the wrong views,
- the frontend sends a request to the wrong endpoint,
- the token is missing from requests,
- the alarm button sends the wrong command,
- the dashboard breaks when the backend is unavailable,
- the layout does not work on mobile.

Good answer:

> The goal of frontend testing was to verify the user-facing behavior and the API contract. We did not test React itself, but our own access-control logic, UI components, API calls, error handling, and important browser flows.

## 2. Tools from the code

In `System/FrontEnd/package.json`, there are:

```text
npm run test:run       -> vitest run
npm run test:e2e       -> playwright test
npm run build          -> vite build
```

The dependencies confirm:

```text
Vitest
React Testing Library
@testing-library/user-event
Playwright
Vite
ESLint
```

So the report does not invent the tools. They really exist in the project.

## 3. Unit tests

Unit tests check small logic without UI.

In the code: `System/FrontEnd/src/auth/accessControl.test.js`.

The tests check, among other things, that:

- an unknown role falls back to `resident`,
- alias `building-admin` maps to `building-administrator`,
- alias `system-admin` maps to `admin`,
- resident has limited permissions,
- admin can see more,
- the default view is `Home`,
- demo session creates the correct role.

Good answer:

> Unit tests were used for pure access-control logic, because this logic does not need a browser or React rendering. We tested role mapping, permission differences, fallback behavior, and default view selection.

Honest detail:

> This helper also reflects some prototype role-selection logic, while the main App.jsx uses backend login and the returned user role for the real session.

## 4. Component tests

Component tests check individual UI components.

In the code, there are tests for components such as:

- `LoginPage`,
- `LoginRoleSelect`,
- `UserMenu`,
- `RestrictedNotice`,
- `AdminControls`.

Most concrete example: `AdminControls.test.jsx` checks that:

- alarm buttons are disabled when `all` devices are selected,
- buttons are disabled when a request is busy,
- clicks call the handler with `critical`, `warn`, or `off`,
- the alarm test error message is rendered as danger.

Good answer:

> Component tests verified isolated UI behavior. For example, the login form renders the expected controls, the user menu calls logout, restricted views show an access-denied message, and alarm controls call the correct handler with the correct alarm mode.

## 5. API service tests

This is a very good exam topic because it sounds concrete.

In the code: `System/FrontEnd/src/services/api.test.js`.

The tests mock `fetch` and check:

- `GET /api/readings?sensorId=101&limit=25&hours=2`,
- `Authorization: Bearer test-token`,
- skipping `sensorId` when `all` is selected,
- errors like `Readings request failed (500)`,
- `/api/readings/devices`,
- `/api/readings/alerts`,
- `POST /api/readings/alarm-test`,
- `EventSource /api/readings/stream?sensorId=101&token=...`,
- `/api/room` and assigning sensors to rooms,
- `/api/users`, role changes, assigning sensors to users, deleting users,
- `/api/logs`,
- login/register and fallback error messages.

Good answer:

> API service tests mocked fetch and verified that the frontend sends the correct URLs, HTTP methods, authorization headers, request bodies, stream URLs, and handles errors correctly. This was important because many frontend bugs in our project would happen at the backend/frontend boundary.

This sentence is gold. Remember it.

## 6. Playwright tests

Playwright tests run in a real browser.

In the code: `System/FrontEnd/e2e/frontend-blackbox.spec.js`.

There are 3 scenarios:

1. After sign in, the dashboard renders data.
2. When the readings API returns an error, `Dashboard offline` is shown.
3. The dashboard remains usable on a responsive viewport and does not create horizontal overflow.

In `playwright.config.js`, these 3 scenarios run on two projects:

```text
chromium-desktop: 1440x900
chromium-mobile: Pixel 5
```

That is why the report shows 6 browser tests.

Good answer:

> Playwright tests checked the application from the user's point of view in a real browser. This gives confidence that routing, rendering, mocked backend responses, role-based navigation, error handling, and responsive layout work together, not only as isolated functions.

## 7. Responsive testing

In the code, the responsive test checks:

```js
document.documentElement.scrollWidth <= window.innerWidth + 1
```

So specifically: there is no horizontal overflow.

Good answer:

> We tested responsive behavior with Playwright using desktop and Pixel 5 mobile viewports. The goal was to verify that the main dashboard and navigation remain usable and that the page does not create horizontal overflow.

## 8. Results

From the report:

```text
Vitest: 7 test files, 36 tests passed
Playwright: 6 browser tests across desktop and mobile Chromium
npm run build passed
Latest local verification: 2026-05-26
```

From the code, this comes from:

- Vitest collecting unit/component/API tests,
- Playwright having 3 scenarios x 2 projects,
- the build being `vite build`.

## 9. Limitations

Do not pretend the tests were perfect. Examiners like honesty.

Limitations:

- Playwright mocks the backend, so it is not a full end-to-end test with the real backend, IoT, and ML,
- live SSE is mostly checked as a URL/flow, not as a real production live stream,
- chart readability and some visual checks were manual,
- abnormal alarm states were partly mocked/manual,
- the tests do not cover every possible dashboard view.

Good answer:

> The main limitation is that not every visual and live-data scenario was automated. Some chart readability, live backend data, and abnormal alarm states were checked manually or with mocked data instead of a full end-to-end setup with real IoT, backend, and ML services.

## Best exam answer

> We tested the frontend on several levels. Unit tests with Vitest covered access-control logic, such as role mapping, permissions, fallback roles, and default views. Component tests with React Testing Library checked isolated UI behavior, for example login, role selection, user menu logout, restricted notices, and alarm controls. API service tests mocked fetch to verify correct endpoints, HTTP methods, authorization headers, request bodies, stream URLs, and error handling. Finally, Playwright black-box tests checked important user flows in a real browser, including login, dashboard display, backend-offline behavior, and responsive layout on desktop and Pixel 5 mobile viewport.

This is your base answer. If you say it calmly, you sound like someone who actually understands why these tests existed.
