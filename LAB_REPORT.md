# AI53 Phase 2 lab report

At each milestone, copy the observed results under **Checkpoint**, then answer the matching **Question** from the Lab Guide.

Teacher note: If you use Render in this workshop, you do not need to do part 4.3 Local staging. Using Render is recommended.

## Milestone 1 · Inspect the model handoff

### Checkpoint 1 · Evidence

- GitHub repository URL: https://github.com/GitGudShu/AI53_MLOPS_P2
- Expected model version: `phase1-reference-v1`
- Three class labels: `angular_leaf_spot`, `bean_rust`, `healthy`
- First pushed commit SHA: `4068c873a3c3be802a124d0f98ec52fee1300d3b`

### Question 1 · Explanation

Answer: The model version identifies the trained weights inside `bean-cnn.onnx` from Phase 1, shaped by the training data, settings, and selected run. The API reads it from the ONNX metadata and returns it with responses so we can check which model answered.

The Git SHA identifies one exact snapshot of the repo, covering the API code, Dockerfile, tests, dependency lock file, and committed model file. By itself and in theory, it shouldn't say anything about how the model was trained.

## Milestone 2 · Serve predictions with FastAPI

### Checkpoint 2 · Evidence

- Final API test count after TODO 1: 7 passed (after running `uv run --locked pytest -q tests/test_api.py`)
- Browser prediction JSON for `data/current/leaf-01.jpg` (`UNAVAILABLE` and the error shown if the page gives no prediction):

```json
{
  "label": "angular_leaf_spot",
  "confidence": 0.5016346573829651,
  "model_version": "phase1-reference-v1"
}
```

### Question 2 · Explanation

Answer:`/health` only checks that the service process is running and answering HTTP requests. It skips the model and returns `{"status": "ok"}` with a 200 status regardless of the model's state. `/ready` checks that the ONNX model loaded at startup with valid version and label metadata.

Including the model version in every prediction response lets you trace each result back to the exact model that made it. This is important for debugging, auditing, and comparing results across different model versions.

## Milestone 3 · Run tests and build in GitHub Actions

### Checkpoint 3 · Evidence

- Local branch: `main`
- Local commit SHA: `a316026f326102ecb44f9fe7f83b1d067c8edcd3`
- Local API test count: 7 passed
- Actions run source (`main`, `PULL REQUEST` or `UNAVAILABLE`): `main`
- Actions run URL: https://github.com/GitGudShu/AI53_MLOPS_P2/actions/runs/46016240995
- `tests` job status: `PASS`
- `docker-build` job status: `PASS`
- Container smoke result or first failed CI step: PASS.

The earlier pull request run on `ci-checks` (commit `621c0ab`, https://github.com/GitGudShu/AI53_MLOPS_P2/actions/runs/46016031100) also passed both jobs before the merge.

For the two job statuses, use `PASS`, `FAIL`, `PENDING`, `SKIPPED`, `CANCELLED` or `UNAVAILABLE`. Use `SKIPPED` when Actions shows that a job was skipped. Use `UNAVAILABLE` for missing CI evidence.

### Question 3 · Explanation

Answer: `needs: tests` enforces that the `docker-build` job wait until `tests` passes.

Checking the `main` run after merge confirms the exact code on `main` that we will deploy later passed tests and built cleanly, matching the local `git rev-parse HEAD` recorded at Checkpoint 3.

## Milestone 4 · Verify a staging release

### Checkpoint 4 · Evidence

- Mode (`RENDER` or `LOCAL`): `RENDER`  
- Target URL: https://ai53-leaf-chu.onrender.com
- Tested commit SHA: `a316026f326102ecb44f9fe7f83b1d067c8edcd3` (Live deploy on Render, same as Checkpoint 3)
- Expected model version from Checkpoint 1: `phase1-reference-v1`
- Observed model version from `/ready` (`UNAVAILABLE` if it returned none): `phase1-reference-v1`
- Smoke check result (`PASS` or `FAIL`): `PASS`
- First error, if any: None
- Earlier Render attempt, if switching to local (URL if available and reason for switching): Not applicable (Render succeeded)

```text
PASS GET /ready (model_version=phase1-reference-v1)
PASS GET /health
PASS POST /predict (label=angular_leaf_spot, model_version=phase1-reference-v1)
{
  "label": "angular_leaf_spot",
  "confidence": 0.5016346573829651,
  "model_version": "phase1-reference-v1"
}
```

### Question 4 · Explanation

Answer: No, I would not approve it. `/health` returning 200 only shows the server is running, not that it serves the right model. 

First I would check what was actually deployed. Does the commit SHA of the Live deploy on Render match the SHA that passed CI (Checkpoint 3)? If the SHA is right, I would check the model file in that commit (`model/bean-cnn.onnx` and its `model_version` metadata) against `model/README.md`, and the service logs for a different model being loaded. Until the versions match, the deployment stays unapproved.

## Milestone 5 · Compare input distributions

### Checkpoint 5 · Evidence

Fill each row from its own `summary.json`. Use the file from the CURRENT report for the first row and the file from the SHIFTED report for the second. In both files, `current_mean` is the mean brightness of the batch you checked. Use the exact `score` (p-value) and `threshold` (cutoff). Enter `DRIFT` when `drift_detected` is true and `NO DRIFT` when it is false.

| Batch | Reference mean (`reference_mean`) | Batch mean (`current_mean`) | Method | Score | Threshold | Result (`DRIFT` or `NO DRIFT`) |
| --- | --- | --- | --- | --- | --- | --- |
| `CURRENT` | 0.4724440528104574 | 0.47574189751633994 | K-S p_value | 0.8079631540901643 | 0.05 | `NO DRIFT` |
| `SHIFTED` | 0.4724440528104574 | 0.16466240836601304 | K-S p_value | 1.6911233892144742e-17 | 0.05 | `DRIFT` |

- Final full test count: 9 passed (after running `uv run --locked --group monitoring pytest -q`)

### Question 5 · Explanation

Answer: For CURRENT, we should go with MONITOR because Evidently shows a p-value of 0.81 so no drift, and nearly identical average brightness (0.476 vs 0.472), meaning we just keep logging incoming images and re-run the check on our normal schedule.

For SHIFTED, we should go with REPAIR first, or RETRAIN if the change is permanent. Evidently flags drift with a p-value around 1.7e-17, average brightness drops from 0.472 to 0.165, and accuracy falls from 57% (17/30) to 33% (10/30) because the model predicts `bean_rust` for every photo with inflated confidence (0.87).

If accuracy stays low after a repair and we have another model version, we should test both models side by side on the recent labeled images. If the other version already achieves better accuracy and balanced results across classes, we switch to it since deploying an existing model is faster, whereas we retrain if both models struggle because neither has learned the new conditions.

Before promoting anything from staging, we will confirm `/ready` returns 200 with the expected `model_version`, run the recent labeled images through `/predict` to verify accuracy and confidence levels, check latency and error rates, and re-run Evidently to make sure the inputs look normal.