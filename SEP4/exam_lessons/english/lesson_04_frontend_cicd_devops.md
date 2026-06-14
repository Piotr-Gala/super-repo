# Lesson 4: Frontend CI/CD and DevOps

The point here is that you should be able to explain: **how the frontend was automatically checked, built, and deployed**.

## 1. Why CI/CD?

CI/CD is not "fancy GitHub Actions". It is a way to make every change checked in a repeatable way.

For the frontend, the goal was:

> Every frontend change should be tested, built, packaged, and deployed in a reproducible way, without manual steps from the developer.

In simple terms: you push code, the pipeline checks whether you broke the app, builds the production version, and can deploy it.

## 2. CI vs CD

**CI - Continuous Integration**  
Checks code after push/PR:

```text
install dependencies
run tests
run production build
build Docker image
```

So the question is: "does the frontend still work?"

**CD - Continuous Deployment/Delivery**  
Deploys a working version:

```text
push to Dev/main
pipeline passes
SSH to VPS
docker compose rebuild/restart services
```

So the question is: "does the working version reach the server?"

## 3. What the frontend pipeline really does

In the code: `.github/workflows/ci.yml`.

The workflow runs on:

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

So for the frontend, CI really does:

- dependency installation,
- unit/component/API tests through Vitest,
- production build through Vite,
- Chromium installation,
- Playwright black-box tests.

Good answer:

> The frontend CI pipeline runs in GitHub Actions. It installs dependencies with npm ci, runs Vitest tests, validates the production build, installs Playwright Chromium, and runs black-box browser tests.

## 4. Docker

The frontend has `System/FrontEnd/Dockerfile`.

The Dockerfile uses a multi-stage build:

```text
node:24-alpine
  -> npm ci
  -> npm run build

nginx:alpine
  -> serves /app/dist from /usr/share/nginx/html
  -> exposes port 80
```

Better answer than "because Docker is good":

> Docker made the frontend deployment more reproducible. The React app is built in a Node container and then served from Nginx, so the deployed frontend does not depend on a developer machine.

## 5. How the frontend fits into the whole deployment

In `System/docker-compose.vps.yml`, there are services:

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

So the frontend starts as an Nginx container and depends on a healthy backend.

Good answer:

> The frontend was deployed as part of the full Docker Compose setup. It was built from the FrontEnd Dockerfile and served by Nginx. In the VPS compose file it depends on the main server being healthy, while the backend depends on the database, ML server, MQTT broker, and bridge.

## 6. Deploy job

In the workflow, deploy has:

```text
needs: [mainserver, mlserver, frontend, docker]
if: push to Dev or main
```

So deploy does not run on PRs, and it does not run if the frontend or other required jobs fail.

Deployment through SSH does roughly this on the VPS:

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

Good answer:

> Deployment only runs on push to Dev or main after the required jobs pass. It connects to the VPS over SSH, resets the server repository to the pushed branch, then rebuilds and restarts the Docker Compose services.

## 7. What tests have to do with CI/CD

Local tests are useful, but CI gives confidence that every push goes through the same checks.

Good answer:

> The tests were integrated into CI so that regressions could be caught before deployment. For frontend, this included unit/component/API tests, Playwright checks, and production build validation.

Trade-off:

> This increases pipeline time, but it reduces the risk of deploying a broken frontend.

## 8. Limitations

Do not pretend the pipeline was perfect.

Most important limitations:

- no automatic smoke test after deployment against the live VPS URL,
- Playwright tests the frontend with a mocked API, not the full real system,
- deployment uses `git reset --hard` on the VPS, so it assumes the VPS has no local changes,
- the pipeline builds images in CI, but the Docker job has `push: false`, so images are rebuilt again on the VPS.

Good answer:

> The main limitation was that the pipeline did not include an automatic smoke test against the live VPS after deployment. So even if tests and build passed, final verification of the deployed application still had to be done manually.

## Best exam answer

> The frontend CI/CD setup was implemented with GitHub Actions as part of the shared workflow. The frontend job uses Node 24, installs dependencies with npm ci, runs Vitest unit/component/API tests, validates the Vite production build, installs Playwright Chromium, and runs black-box browser tests. The Dockerfile builds the React app in a Node stage and serves the production files with Nginx. Deployment runs only on push to Dev or main after mainserver, mlserver, frontend and docker jobs pass. It connects to the VPS over SSH, resets the server repository to the pushed branch, and rebuilds/restarts the Docker Compose services. The main limitation was no automatic post-deployment smoke test against the live VPS.

## Remember

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
