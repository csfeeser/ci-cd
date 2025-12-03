# Warmup Activity: Fixing a Broken Error-Handling Workflow

<img src="https://img.ifunny.co/images/350fbba1526b08f0176f86dd945a9362ff8385f680b0cb32c29209958912ebc7_1.jpg" width="400" />

## Part 1: Create the Required Files in GitHub

1. Create a new repository named `failure-recovery-warmup`.
2. Inside the repo, select **Add file → Create new file**.
3. In the root directory create a file named `fail-or-pass.sh`.
4. Paste in this script:

```bash
#!/usr/bin/env bash

echo "Running a step that is supposed to fail on purpose. WOMP WOMP."
exit 1
```

5. Commit the file.

6. Now create your workflow at the following path: `.github/workflows/failure-warmup.yml`

7. Paste in the *broken* workflow below:

```yaml
name: Failure Handling Warmup

on:
  push:
    branches:
      - main

jobs:
  failure-demo:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Intentionally fail
        id: failing-step
        run: bash fail-or-pass.sh

      - name: Recovery step
        if: success() && steps.failing-step.outcome == 'failure'
        run: echo "Hop on the struggle bus, because we're trying to recover from the failure!"

      - name: Summary step
        run: echo "Workflow finished (success or failure)."
```

## Part 2: Objective

Your job is to:

* Find all errors in the workflow.
* Fix the workflow so GitHub Actions can:

  * Run the failing step.
  * Run a recovery step when the failing step fails.
  * Always print a final summary message, no matter if the workflow errors or not.
    
* Verify the log output in the Actions run.

## Hints

<details>
<summary>The recovery step never runs. Why?</summary>

Look at the condition on the recovery step.
Ask yourself two questions:

1. Which built-in function should you use when you want something to run only after a failure?
2. Does the current condition make sense for a step that is supposed to run when the previous step fails?

</details>

<details>
<summary>Why do later steps get skipped after the failure?</summary>

When a step exits with code `1`, GitHub treats that as a failure and stops the job by default.

In the failures lab you saw a way to let a step fail without stopping the whole job. That same idea is needed here so the recovery and summary steps can still run.

</details>

<details>
<summary>Why does the summary step never print?</summary>

You already know a special function that makes a step *always* run, error or no (wink).

</details>

## Solution (Click to Reveal)

<details>
<summary>Click to show the corrected workflow and fixes</summary>

Here are the three main problems and how to fix them.

1. The failing step stops the whole job.
   By default, a non-zero exit code stops the job and skips later steps.
   To let the job continue and give the recovery logic a chance to run, add `continue-on-error: true` to the failing step.

2. The recovery condition uses `success()` instead of `failure()`.
   A recovery step should run when something failed, not when everything succeeded.
   Change the condition to use `failure()` so it only runs if a previous step failed.

3. Since the summary step should always run, it needs an `if:` condition with `always()`.

Corrected workflow with comments explaining the fixes:

```yaml
name: Failure Handling Warmup

on:
  push:
    branches:
      - main

jobs:
  failure-demo:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Intentionally fail
        id: failing-step
        continue-on-error: true          # FIX 1: allow the job to continue after this failure
        run: bash fail-or-pass.sh

      - name: Recovery step
        if: failure() && steps.failing-step.outcome == 'failure'   # FIX 2: use failure(), not success()
        run: echo "Hop on the struggle bus, because we're trying to recover from the failure!"

      - name: Summary step
        if: always()                         # FIX 3: use always() so this runs no matter what
        run: echo "Workflow finished (success or failure)."
```

After making these changes:

1. Push a new commit to `main`.
2. Open the **Actions** tab.
3. Open the latest **Failure Handling Warmup** run.
4. Confirm in the logs that:

   * The **Intentionally fail** step fails.
   * The **Recovery step** runs after the failure.
   * The **Summary step** always runs at the end.

</details>
