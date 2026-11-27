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

- Local branch:
- Local commit SHA:
- Local API test count:
- Actions run source (`main`, `PULL REQUEST` or `UNAVAILABLE`):
- Actions run URL:
- `tests` job status:
- `docker-build` job status:
- Container smoke result or first failed CI step:

For the two job statuses, use `PASS`, `FAIL`, `PENDING`, `SKIPPED`, `CANCELLED` or `UNAVAILABLE`. Use `SKIPPED` when Actions shows that a job was skipped. Use `UNAVAILABLE` for missing CI evidence.

### Question 3 · Explanation

Answer:

## Milestone 4 · Verify a staging release

### Checkpoint 4 · Evidence

- Mode (`RENDER` or `LOCAL`):
- Target URL:
- Tested commit SHA:
- Expected model version from Checkpoint 1:
- Observed model version from `/ready` (`UNAVAILABLE` if it returned none):
- Smoke check result (`PASS` or `FAIL`):
- First error, if any:
- Earlier Render attempt, if switching to local (URL if available and reason for switching):

### Question 4 · Explanation

Answer:

## Milestone 5 · Compare input distributions

### Checkpoint 5 · Evidence

Fill each row from its own `summary.json`. Use the file from the CURRENT report for the first row and the file from the SHIFTED report for the second. In both files, `current_mean` is the mean brightness of the batch you checked. Use the exact `score` (p-value) and `threshold` (cutoff). Enter `DRIFT` when `drift_detected` is true and `NO DRIFT` when it is false.

| Batch | Reference mean (`reference_mean`) | Batch mean (`current_mean`) | Method | Score | Threshold | Result (`DRIFT` or `NO DRIFT`) |
| --- | --- | --- | --- | --- | --- | --- |
| `CURRENT` | | | | | | |
| `SHIFTED` | | | | | | |

- Final full test count:

### Question 5 · Explanation

Answer:
