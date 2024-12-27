# gh-actions-quirks
Unexpected Github Actions behaviors

## Unexpected Job Skip Behavior in GitHub Actions Dependencies

### Issue Description:
When dealing with a workflow containing jobs A -> B -> C where:
- B runs even if A is skipped
- C depends only on B
- C unexpectedly skips if A is skipped, despite B running successfully

Example Workflow Structure:
```yaml
name: Problematic Workflow
jobs:
  A:
    runs-on: ubuntu-latest
    if: github.event_name == 'push'
    steps:
      - run: echo "Job A"

  B:
    runs-on: ubuntu-latest
    needs: [A]
    if: always()  # B will run even if A is skipped
    steps:
      - run: echo "Job B"

  C:
    runs-on: ubuntu-latest
    needs: [B]  # C will unexpectedly skip if A was skipped, even though B ran successfully
    steps:
      - run: echo "Job C"
```

Observed Behavior:
1. Job A is skipped
2. Job B executes successfully
3. Job C skips unexpectedly due to A's skip status, despite only depending on B

Solution:
Add explicit condition to prevent skip propagation:
```yaml
jobs:
  # ... other jobs
  C:
    runs-on: ubuntu-latest
    needs: [B]
    if: always() && !cancelled() && !failure()  # This prevents skip propagation from A
    steps:
      - run: echo "Job C"
```

Conclusion:
GitHub Actions propagates skip status through the dependency chain even for indirect dependencies, requiring explicit handling with `always()` to prevent unintended skips.
