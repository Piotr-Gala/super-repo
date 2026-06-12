# Lekcja 4: Frontend CI/CD i DevOps

Tu chodzi o to, żebyś umiał wyjaśnić: **jak frontend był automatycznie sprawdzany, budowany i wdrażany**.

**1. Po co CI/CD?**
CI/CD to nie jest “fancy GitHub Actions”. To sposób, żeby każda zmiana była powtarzalnie sprawdzona.

Dla frontendu cel był taki:

> Every frontend change should be tested, built, packaged, and deployed in a reproducible way, without manual steps from the developer.

Po ludzku: pushujesz kod, pipeline sprawdza czy nie zepsułeś appki, buduje wersję produkcyjną i może ją wdrożyć.

**2. CI vs CD**
To musisz rozróżniać.

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
docker compose pulls/restarts frontend
```

Czyli pytanie: “czy działająca wersja trafia na serwer?”

**3. Co robił frontend pipeline**
Z raportu wynika, że frontend był częścią wspólnego GitHub Actions workflow dla całego systemu.

Dla frontendu sensownie mówisz:

- pipeline odpalał się na push / pull request do `Dev` albo `main`,
- uruchamiał testy frontendu,
- sprawdzał production build,
- budował Docker image,
- deploy był robiony na VPS przez SSH,
- deployment odpalał się po przejściu wymaganych jobów.

Dobra odpowiedź:

> The frontend CI pipeline ran automatically on pushes and pull requests to Dev or main. It installed dependencies, ran frontend tests, validated the production build, and built the Docker image. For deployment, the pipeline used SSH to update the VPS and restart the Docker Compose services after the required jobs passed.

**4. Dlaczego Docker?**
Docker pozwala uruchamiać frontend w takim samym środowisku niezależnie od maszyny.

Nie mów tylko “because Docker is good”.

Lepsza odpowiedź:

> Docker made the frontend deployment more reproducible, because the application could be packaged with its runtime configuration and run as a container together with the backend, database, ML server, MQTT broker, and bridge.

Czyli frontend nie żyje samotnie. Jest jednym kontenerem w większym systemie.

**5. Jak frontend pasuje do całego deploymentu**
Cały system miał kilka usług:

```text
frontend
main-server
ml-server
mysql database
mqtt-broker
mqtt-http-bridge
```

Frontend komunikuje się z backendem. Backend gada z ML, bazą i MQTT.

Na egzaminie możesz powiedzieć:

> The frontend was deployed as part of the full Docker Compose setup. It depended on the backend API being available, while the backend depended on the database, ML server, MQTT broker, and bridge.

**6. Co testy mają wspólnego z CI/CD**
To jest ważne połączenie z Lekcją 3.

Testy lokalnie są fajne, ale CI daje pewność, że każdy push przechodzi ten sam zestaw kontroli.

Dobra odpowiedź:

> The tests were integrated into CI so that regressions could be caught before deployment. For frontend, this included unit/component tests, API service tests, Playwright checks, and production build validation.

Mały trade-off:

> This increases pipeline time, but it reduces the risk of deploying a broken frontend.

**7. Ograniczenia**
Nie udawaj, że pipeline był idealny. W raporcie jest konkretna słabość:

> The pipeline did not run a post-deployment smoke test against the live VPS URL.

Czyli po deployu nadal trzeba było ręcznie sprawdzić, czy live app działa.

Dobra odpowiedź:

> The main limitation was that the pipeline did not include an automatic smoke test against the live VPS after deployment. So even if tests and build passed, final verification of the deployed application still had to be done manually.

To brzmi dojrzałe, bo pokazuje, że rozumiesz różnicę między “build passed” a “production works”.

**8. Najlepsza odpowiedź egzaminacyjna**
Pytanie:

> Can you explain the frontend CI/CD setup?

Odpowiedź:

> The frontend CI/CD setup was implemented with GitHub Actions as part of the shared project workflow. On pushes and pull requests to Dev or main, the pipeline installed dependencies, ran frontend tests, validated the production build, and built the Docker image. For deployment, successful changes could be deployed to the VPS through SSH and Docker Compose, together with the other services such as the backend, ML server, database, MQTT broker, and bridge. This made frontend delivery more reproducible and reduced manual deployment work. The main limitation was that we did not have an automated post-deployment smoke test against the live VPS, so final live verification was still manual.

**Do zapamiętania**
Krótka mapa:

```text
GitHub Actions
  -> install
  -> test
  -> build
  -> Docker image
  -> SSH deploy to VPS
  -> Docker Compose restart
```

Najważniejsza myśl:

> CI/CD was used to make frontend verification and deployment repeatable, not dependent on one developer’s machine.

To jest bardzo dobre zdanie na egzamin.