# CI/CD — Continuous Integration and Continuous Delivery

> Automatically build, test, and deploy every code change so that software is always in a releasable state.

---

## The Problem

Without automation, releasing software involves:

- Manual steps that differ between developers ("works on my machine")
- Long-lived feature branches that diverge and are painful to merge
- Integration bugs discovered days or weeks after a change was written
- Manual deployment steps that are error-prone under pressure

CI/CD compresses the feedback loop: from a change being written to knowing it's correct and deployed.

---

## Continuous Integration (CI)

**CI** means every push triggers an automated build and test run. The rule: the main branch must always be green (all tests pass).

```mermaid
flowchart LR
    Push([git push]) --> CI[CI server triggered]
    CI --> Lint[Lint + type check]
    Lint --> UnitTests[Unit tests]
    UnitTests --> IntTests[Integration tests]
    IntTests --> Build[Build artifact / Docker image]
    Build --> Pass{All green?}
    Pass -- Yes --> Notify([✓ Ready to deploy])
    Pass -- No --> Alert([✗ Author notified\nMerge blocked])
```

---

## Continuous Delivery vs. Continuous Deployment

| | Continuous Delivery | Continuous Deployment |
|-|--------------------|-----------------------|
| **After CI passes** | Artifact ready to deploy | Automatically deployed to production |
| **Human in the loop** | Yes — manual release approval | No — fully automated |
| **Good for** | Regulated industries, careful rollout | High-trust codebases, fast iteration |

---

## GitHub Actions Pipeline

```yaml
# .github/workflows/ci.yml
name: CI / CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Lint
        run: |
          ruff check .
          mypy src/

      - name: Unit tests
        run: pytest tests/unit/ -v --tb=short

      - name: Integration tests
        run: pytest tests/integration/ -v
        env:
          DATABASE_URL: postgresql://postgres:test@localhost/test_db

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    outputs:
      image: ${{ steps.meta.outputs.tags }}

    steps:
      - uses: actions/checkout@v4

      - name: Docker meta
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=sha,prefix=sha-

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment: staging

    steps:
      - name: Deploy to staging
        run: |
          kubectl set image deployment/api \
            api=${{ needs.build.outputs.image }} \
            --namespace staging

      - name: Run smoke tests
        run: pytest tests/smoke/ --base-url https://staging.myapp.com

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://myapp.com

    steps:
      - name: Deploy to production
        run: |
          kubectl set image deployment/api \
            api=${{ needs.build.outputs.image }} \
            --namespace production
```

---

## Trunk-Based Development

A branching strategy that keeps the main branch always deployable.

```
AVOID (Long-lived feature branches):

main ──────────────────────────────────────── main
       \                                     /
        feature/big-change ── 2 weeks ──────
        (massive merge conflict, integrated tests fail late)

USE (Trunk-based development):

main ──●──────●──────●──────●──────●──── main
      day1   day2   day3   day4   day5
       ↑ small, focused, always tested ↑
```

**Rules:**
- Every commit to `main` goes through CI
- Feature branches last at most 1–2 days
- Use feature flags to ship incomplete features safely

```python
# Feature flag: deploy the code, but hide the feature
from myapp.flags import feature_enabled

@app.get("/api/new-checkout")
def new_checkout():
    if not feature_enabled("new_checkout_flow", user_id=request.user_id):
        abort(404)
    return new_checkout_handler()
```

---

## Blue/Green Deployment

Run two identical production environments. Switch traffic instantly between them.

```mermaid
flowchart LR
    LB[Load Balancer]
    Blue[Blue\nv1.0 — live]
    Green[Green\nv2.0 — new]

    LB -- 100% --> Blue
    LB -. 0% .-> Green

    Note[Deploy v2 to Green\nRun tests\nSwitch LB to Green\nBlue kept on standby for rollback]
```

```yaml
# Kubernetes: two deployments, one service selector switched
# Switch with zero downtime:
kubectl patch service api \
  -p '{"spec": {"selector": {"version": "green"}}}'

# Rollback in seconds:
kubectl patch service api \
  -p '{"spec": {"selector": {"version": "blue"}}}'
```

---

## Canary Deployment

Gradually roll new version to a subset of users. Catch issues before full rollout.

```yaml
# Kubernetes with Argo Rollouts
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: api
spec:
  strategy:
    canary:
      steps:
        - setWeight: 5         # 5% of traffic to new version
        - pause: {duration: 5m}
        - setWeight: 25
        - pause: {duration: 10m}
        - setWeight: 50
        - pause: {duration: 10m}
        - setWeight: 100       # full rollout
      analysis:
        templates:
          - templateName: error-rate
        args:
          - name: service-name
            value: api
```

---

## Pipeline Security

```yaml
# Store secrets in GitHub Actions Secrets — never in code
- name: Deploy
  env:
    KUBE_TOKEN: ${{ secrets.KUBE_TOKEN }}    # ← injected at runtime
    DATABASE_URL: ${{ secrets.DATABASE_URL }}
  run: ./deploy.sh

# Enforce minimum required reviews before merging to main
# Branch protection rules (GitHub Settings → Branches):
# - Require status checks to pass
# - Require 1+ approvals
# - Dismiss stale reviews on new commits
# - Restrict who can push directly
```

---

## Key Takeaways

- CI means every push is automatically built and tested — the main branch is always green.
- Continuous Delivery makes every passing build deployable; Continuous Deployment goes one further and deploys automatically.
- Trunk-based development + small commits keep merges painless and feedback fast.
- Blue/green gives instant rollback; canary gives gradual risk reduction.
- Never put secrets in code or CI logs — use secret stores injected at runtime.
