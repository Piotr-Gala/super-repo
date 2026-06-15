# Lesson 9: ML Testing

Goal of this lesson: you should be able to explain **what was tested in ML**, **why those tests matter**, and **what the limitations are**.

## 1. What is special about ML testing?

With normal software, the main question is often:

> Does the code run?

With ML, that is not enough.

The model can run successfully and still make bad predictions.

So ML testing has two concerns:

- service/API behavior,
- model behavior on representative inputs.

Good answer:

> For the ML service, we tested both the FastAPI contract and the model behavior. The API tests check that the service accepts and rejects the right payloads, while the model tests check that the deployed Random Forest gives expected classifications for representative Normal, Cooking, and Fire readings.

## 2. Test tools

The tests are in:

```text
System/MlServer/tests/test_main.py
```

They use:

```text
pytest
pytest-cov
fastapi.testclient.TestClient
```

`TestClient` runs the FastAPI application in-process, so the tests do not need a running Docker container.

Good answer:

> The ML tests use FastAPI's TestClient, which allows us to call `/health` and `/predict` like HTTP endpoints without starting the server separately. This keeps the tests fast and suitable for CI.

## 3. Health test

The simplest test:

```text
GET /health -> 200 {"status": "ok"}
```

Why is this important?

Because Docker Compose uses `/health` to decide whether the ML service is healthy.

Good answer:

> `/health` looks simple, but it matters operationally. If this endpoint breaks, Docker Compose may keep the service unhealthy and the MainServer may not start.

## 4. Response shape test

The tests verify that `POST /predict` returns exactly:

```text
predictedCategory
confidenceScore
riskLevel
```

And they check:

- category is one of `Normal`, `Cooking`, `Fire`,
- risk is one of `Low`, `Medium`, `High`,
- confidence is between `0.0` and `1.0`.

Good answer:

> The response shape test protects the contract with the MainServer. If someone changes field names or adds/removes fields accidentally, the test fails before the backend integration breaks.

## 5. Scenario-based model tests

The model tests use three representative profiles.

Normal:

```text
temperature: 22.0
humidity: 40.0
tvoc: 100.0
eco2: 400.0
expected: Normal / Low
```

Cooking:

```text
temperature: 28.0
humidity: 60.0
tvoc: 800.0
eco2: 1200.0
expected: Cooking / Medium
```

Fire:

```text
temperature: 38.0
humidity: 15.0
tvoc: 1000.0
eco2: 8000.0
expected: Fire / High
```

Good answer:

> These tests do not prove the model is perfect. They act as regression tests for known representative points, so a model swap or feature-order bug cannot silently change the expected behavior for basic Normal, Cooking, and Fire examples.

## 6. Feature isolation test

There is a test for this case:

```text
temperature, humidity, tvoc, eco2 are clean
co2Level is very high
aqi is maximum
expected result stays Normal
```

Why?

Because the deployed model does not use `co2Level` or `aqi`.

Good answer:

> The feature isolation test confirms that the production feature vector matches training. CO2 and AQI are present in the API payload, but they should not affect the model output because the trained features are only temperature, humidity, TVOC, and eCO2.

## 7. Validation tests

The tests also check invalid payloads:

- legacy flat payload is rejected,
- `aqi` below minimum is rejected,
- `aqi` above maximum is rejected,
- missing required sensor field is rejected.

Expected response:

```text
HTTP 422
```

Good answer:

> Validation tests are important because accepting the wrong payload shape could produce a prediction from incomplete or misinterpreted input. Rejecting invalid payloads with 422 makes integration mistakes visible.

## 8. Coverage and CI

The CI command is:

```text
pytest --cov --cov-report=xml --cov-report=term-missing --cov-fail-under=70
```

So the ML service must keep at least 70% line coverage.

Ruff also runs before tests:

```text
ruff check main.py tests/
```

Good answer:

> The ML CI job enforces linting and a 70% coverage floor. This prevents the inference service from being changed without maintaining basic automated coverage.

## 9. Limitations

Be honest here.

Limitations:

- tests use fixed representative points, not a broad statistical sample,
- tests do not prove real-world model reliability,
- no full automated contract test starts both MainServer and ML service together,
- backend tests mock ML responses, while ML tests use TestClient in isolation,
- drift monitoring is offline and not part of CI/CD.

Good answer:

> The tests are good regression checks, but they are not a complete model validation framework. A model could pass these fixed examples and still perform poorly on real sensor data. Stronger testing would include held-out dataset evaluation in CI, cross-validation evidence, and an automated contract test between MainServer and ML.

## Best exam answer

> The ML tests cover both API behavior and model behavior. With FastAPI TestClient, we test `/health`, the `/predict` response shape, schema validation, and invalid payload rejection. The model is checked with representative Normal, Cooking, and Fire profiles, plus a feature-isolation test confirming that only temperature, humidity, TVOC, and eCO2 affect predictions. CI runs ruff and pytest with coverage, enforcing a 70% floor. The main limitation is that fixed scenario tests do not prove real-world model reliability, so future work should include broader dataset evaluation and automated integration tests with the MainServer.

## Remember

```text
Test API contract
Test known model scenarios
Test validation failures
Test feature vector assumptions
Enforce ruff + pytest coverage
Be honest about real-world limits
```
