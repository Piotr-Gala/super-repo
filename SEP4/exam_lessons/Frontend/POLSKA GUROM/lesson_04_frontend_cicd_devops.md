# Lekcja 4: Frontend CI/CD i DevOps

Tu chodzi o to, żebyś umiał wyjaśnić: **jak frontend był automatycznie sprawdzany, budowany i wdrażany**.

## 1. Po co CI/CD?

CI/CD to nie jest “fancy GitHub Actions”. To sposób, żeby każda zmiana była powtarzalnie sprawdzona.

Dla frontendu cel był taki:

> Every frontend change should be tested, built, packaged, and deployed in a reproducible way, without manual steps from the developer.

Po ludzku: pushujesz kod, pipeline sprawdza czy nie zepsułeś appki, buduje wersję produkcyjną i może ją wdrożyć.

## 2. CI vs CD

**CI - Continuous Integration**
Sprawdza kod po pushu/PR:

```text
install dependencies
run tests
run production build
build Docker image
```

Czyli pytanie: “czy frontend nadal działa?”

**CD - Continuous Deployment/Delivery**
Wdraża działającą wersję:

```text
push to Dev/main
pipeline passes
SSH to VPS
docker compose rebuild/restart services
```

Czyli pytanie: “czy działająca wersja trafia na serwer?”

## 3. Co naprawdę robi frontend pipeline

W kodzie: `.github/workflows/ci.yml`.

Workflow odpala się na:

```text
push: Dev, main, ML-team
pull_request: Dev, main, ML-team
workflow_dispatch
```

Frontend job:

```text
setup node 24
npm ci
npm run test:run
npm run build
npx playwright install --with-deps chromium
npm run test:e2e
```

Czyli dla frontendu CI realnie robi:

- instalację zależności,
- unit/component/API tests przez Vitest,
- production build przez Vite,
- instalację Chromium,
- Playwright black-box tests.

Dobra odpowiedź:

> The frontend CI pipeline runs in GitHub Actions. It installs dependencies with npm ci, runs Vitest tests, validates the production build, installs Playwright Chromium, and runs black-box browser tests.

## 4. Docker

Frontend ma `System/FrontEnd/Dockerfile`.

Dockerfile robi multi-stage build:

```text
node:24-alpine
  -> npm ci
  -> npm run build

nginx:alpine
  -> serves /app/dist from /usr/share/nginx/html
  -> exposes port 80
```

Lepsza odpowiedź niż “because Docker is good”:

> Docker made the frontend deployment more reproducible. The React app is built in a Node container and then served from Nginx, so the deployed frontend does not depend on a developer machine.

## 5. Jak frontend pasuje do całego deploymentu

W `System/docker-compose.vps.yml` są usługi:

```text
mqtt-broker
mqtt-http-bridge
main-server
frontend
mysql
ml-server
```

Frontend:

```text
build: ./FrontEnd
image: kamtjatka/frontend:latest
container_name: kamtjatka-frontend
depends_on: main-server healthy
ports: FRONTEND_PORT -> 80
```

Czyli frontend startuje jako kontener Nginx i zależy od zdrowego backendu.

Dobra odpowiedź:

> The frontend was deployed as part of the full Docker Compose setup. It was built from the FrontEnd Dockerfile and served by Nginx. In the VPS compose file it depends on the main server being healthy, while the backend depends on the database, ML server, MQTT broker, and bridge.

## 6. Deploy job

W workflow deploy ma:

```text
needs: [mainserver, mlserver, frontend, docker]
if: push to Dev or main
```

Czyli deploy nie odpala się na PR i nie odpala się, jeśli frontend albo inne wymagane joby nie przejdą.

Deploy przez SSH robi na VPS mniej więcej:

```text
cd ~/SEP4
git fetch origin
git checkout branch
git reset --hard origin/branch
cd System
docker compose -f docker-compose.vps.yml stop/rm selected services
docker compose -f docker-compose.vps.yml up -d --build --remove-orphans
docker image prune -f
```

Dobra odpowiedź:

> Deployment only runs on push to Dev or main after the required jobs pass. It connects to the VPS over SSH, resets the server repository to the pushed branch, then rebuilds and restarts the Docker Compose services.

## 7. Co testy mają wspólnego z CI/CD

Testy lokalnie są fajne, ale CI daje pewność, że każdy push przechodzi ten sam zestaw kontroli.

Dobra odpowiedź:

> The tests were integrated into CI so that regressions could be caught before deployment. For frontend, this included unit/component/API tests, Playwright checks, and production build validation.

Trade-off:

> This increases pipeline time, but it reduces the risk of deploying a broken frontend.

## 8. Ograniczenia

Nie udawaj, że pipeline był idealny.

Najważniejsze ograniczenia:

- brak automatycznego smoke testu po deployu na live VPS URL,
- Playwright testuje frontend z mockowanym API, nie pełny realny system,
- deploy robi `git reset --hard` na VPS, więc zakłada, że VPS nie ma lokalnych zmian,
- pipeline buduje obrazy w CI, ale docker job ma `push: false`, więc na VPS obrazy są budowane ponownie.

Dobra odpowiedź:

> The main limitation was that the pipeline did not include an automatic smoke test against the live VPS after deployment. So even if tests and build passed, final verification of the deployed application still had to be done manually.

## Najlepsza odpowiedź egzaminacyjna

> The frontend CI/CD setup was implemented with GitHub Actions as part of the shared workflow. The frontend job uses Node 24, installs dependencies with npm ci, runs Vitest unit/component/API tests, validates the Vite production build, installs Playwright Chromium, and runs black-box browser tests. The Dockerfile builds the React app in a Node stage and serves the production files with Nginx. Deployment runs only on push to Dev or main after mainserver, mlserver, frontend and docker jobs pass. It connects to the VPS over SSH, resets the server repository to the pushed branch, and rebuilds/restarts the Docker Compose services. The main limitation was no automatic post-deployment smoke test against the live VPS.

## Do zapamiętania

```text
GitHub Actions frontend job
  -> npm ci
  -> Vitest tests
  -> Vite build
  -> Playwright Chromium
  -> Playwright tests

Dockerfile
  -> Node build
  -> Nginx serve

Deploy
  -> only push to Dev/main
  -> after required jobs pass
  -> SSH to VPS
  -> docker compose up -d --build
```
