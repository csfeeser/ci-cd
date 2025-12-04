# Warmup Activity: Fixing a Broken NASA APOD Matrix Workflow

<img src="https://blog.mahdyar.me/wp-content/uploads/2021/02/0cm6yx27tez21.jpg" width="400" />

## Part 1: Create the Required Files in GitHub

1. Create a new repository named `nasa-apod-warmup`.

2. Inside the repo, select **Add file → Create new file**.

3. In the root directory create a workflow at the following path:
   `.github/workflows/nasa-apod-matrix.yml`

4. Paste in the *broken* workflow below:

```yaml
name: NASA APOD Matrix Warmup

on:
  push:
    branches:
      - main

jobs:
  fetch-apod:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        request:
          - name: "today_apod"
            query: "date=2025-12-03"
          - name: "yesterday_apod"
            query: "date=2025-12-02"
          - name: "random_apod"
            query: "count=1"

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Call NASA APOD API
        run: |
          # HARD-CODED API KEY-- THIS IS SUPER BAD PRACTICE ON PURPOSE
          API_KEY="CHAD-WILL-GIVE-THIS-TO-YOU-IN-THE-ZOOM-CHAT"

          # Build the APOD URL using the matrix query value
          URL="https://api.nasa.gov/planetary/apod?${{ matrix.request.query }}&api_key=$API_KEY"

          echo "Calling: $URL"
          curl "$URL" -o apod.json

          echo "API response:"
          cat apod.json

          # Extract the HD image URL from the JSON
          HDURL=$(jq -r '.hdurl' apod.json)
          echo "HDURL is: $HDURL"

          # Download the image into the images/ folder
          mkdir -p images
          curl "$HDURL" -o "images/${{ matrix.request.name }}.jpg"

      - name: Upload APOD images
        uses: actions/upload-artifact@v4
        with:
          name: apod-images
          path: image/    # <- INTENTIONAL BUG: wrong path on purpose
```

5. Commit the file.

6. Push a change to `main` (or edit and re-commit the workflow) so that the workflow runs at least once.

7. Open the **Actions** tab and inspect the logs for the **NASA APOD Matrix Warmup** run.
   You should see:

   * Three matrix jobs (one per request: today, yesterday, random).
   * A failure related to the API key and/or artifact upload.

---

## Part 2: Objective

Your job is to fix the workflow so GitHub Actions can:

  * call the NASA API key **AS A REPOSITORY SECRET** (no hard-coding in YAML).
  * Successfully download the HD APOD image for each matrix job.
  * Correctly upload all images as a single artifact.

* Verify the log output in the Actions run and confirm:

  * All matrix jobs succeeded.
  * The artifact contains multiple APOD images.

## Hints

<details>
<summary>How do I make a secret again?</summary>

Peep [this lab from yesterday!](https://live.alta3.com/content/github-actions/labs/content/github-actions/github_actions_securing_secrets.html)

</details>

<details>
<summary>The artifact upload step fails. What’s going on?</summary>

Look carefully at:

* The directory where the images are actually downloaded.
* The `path:` used in the `upload-artifact` step.

</details>

<details>
<summary>How do I know the matrix is working correctly?</summary>

Each matrix job uses a different `name` and `query` value.

Check that:

1. The logs show three separate jobs (today, yesterday, random).
2. Each job calls a different APOD URL based on `matrix.request.query`.
3. Each job saves an image with a different filename (using `matrix.request.name`).

If all three jobs finish successfully and the artifact contains multiple images, your matrix is behaving as expected.

</details>

## Solution (Click to Reveal)

<details>
<summary>Click to show the corrected workflow and fixes</summary>

### 1. The API key is hard-coded in the workflow

1. Go to **Settings → Secrets and variables → Actions → New repository secret**.
2. Create a secret named `NASA_API_KEY`.
3. Paste the real NASA API key as the value and save it.

Then in your workflow, replace the hard-coded line with:

```bash
API_KEY="${{ secrets.NASA_API_KEY }}"
```

This lets the workflow read the key securely from GitHub’s secret vault.

---

### 2. The artifact upload step points at the wrong directory

**Problem in the original workflow:**

```yaml
- name: Upload APOD images
  uses: actions/upload-artifact@v4
  with:
    name: apod-images
    path: image/    # wrong path – folder does not exist
```

But the images are saved to `images/` (with an **s**):

```bash
mkdir -p images
curl "$HDURL" -o "images/${{ matrix.request.name }}.jpg"
```

**Fix:**
Update the artifact `path:` to match the real folder:

```yaml
path: images/
```

---

### Corrected Workflow

```yaml
name: NASA APOD Matrix Warmup

on:
  push:
    branches:
      - main

jobs:
  fetch-apod:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        request:
          - name: "today_apod"
            query: "date=2025-12-03"
          - name: "yesterday_apod"
            query: "date=2025-12-02"
          - name: "random_apod"
            query: "count=1"

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Call NASA APOD API
        run: |
          # Read the API key from a repository secret
          API_KEY="${{ secrets.NASA_API_KEY }}"

          # Build the APOD URL using the matrix query value
          URL="https://api.nasa.gov/planetary/apod?${{ matrix.request.query }}&api_key=$API_KEY"

          echo "Calling: $URL"
          curl "$URL" -o apod.json

          echo "API response:"
          cat apod.json

          # Extract the HD image URL from the JSON
          HDURL=$(jq -r '.hdurl' apod.json)
          echo "HDURL is: $HDURL"

          # Download the image into the images/ folder
          mkdir -p images
          curl "$HDURL" -o "images/${{ matrix.request.name }}.jpg"

      - name: Upload APOD images
        uses: actions/upload-artifact@v4
        with:
          name: apod-images
          path: images/          # FIX: correct folder name
```

---

### After making these changes:

1. Push a new commit to `main`.
2. Open the **Actions** tab.
3. Open the latest **NASA APOD Matrix Warmup** run.
4. Confirm in the logs that:

   * Each matrix job calls the APOD API with a different query.
   * The **Call NASA APOD API** step succeeds in all jobs.
   * The **Upload APOD images** step succeeds.
   * The artifact **apod-images** contains multiple `.jpg` files.

</details>
