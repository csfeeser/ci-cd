# Warmup Activity: Fixing a NASA APOD Matrix Workflow

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
          - name: "last_year_apod"
            query: "date=2024-12-03"

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Call NASA APOD API
        run: |
          # HARD-CODED API KEY -- THIS IS SUPER BAD PRACTICE ON PURPOSE
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
```

5. Commit the file.

6. Push a change to `main` (or edit and re-commit the workflow) so that the workflow runs at least once.

7. Open the **Actions** tab and inspect the logs for the **NASA APOD Matrix Warmup** run.
   You should see:

   * Three matrix jobs (one per request: today, yesterday, last_year).
   * The API call step running in each job.
   * No artifact created yet, because nothing is being uploaded.

---

## Part 2: Objective

Your job is to fix the workflow so GitHub Actions can:

* Call the NASA API using a **repository secret** for the API key (no hard-coding in YAML).
* Successfully download the HD APOD image for each matrix job.
* Save the downloaded images as **artifacts** that you can download from the Actions run (one artifact per matrix job is fine).

Then:

* Verify the log output in the Actions run and confirm:

  * All matrix jobs succeeded.
  * There are three artifacts (one per matrix job), each containing an APOD image.

---

## Hints

<details>
<summary>How do I make a secret again?</summary>

Peep [this lab from yesterday!](https://live.alta3.com/content/github-actions/labs/content/github-actions/github_actions_securing_secrets.html)

Short version:

1. Go to **Settings → Secrets and variables → Actions**.
2. Click **New repository secret**.
3. Name it something like `NASA_API_KEY`.
4. Paste the real NASA API key as the value and save.

Then, in the workflow, you can read it with:

```bash
API_KEY="${{ secrets.NASA_API_KEY }}"
```

</details>

<details>
<summary>Why don’t I see any artifacts?</summary>

Look at the bottom of the workflow:

* Is there any step using `actions/upload-artifact`?
* Is there any `path:` pointing to the `images/` directory?

Right now, all you do is *download* images into a folder.
Nothing is uploading them.

You’ll need to add a new step that uses `actions/upload-artifact@v4` and points at the `images/` folder.

Remember: because you’re using a matrix, each job should use a **unique artifact name**, or you’ll get a 409 conflict.

</details>

<details>
<summary>How do I know the matrix is working correctly?</summary>

Each matrix job uses a different `name` and `query` value.

Check that:

1. The logs show three separate jobs (today, yesterday, last_year).
2. Each job calls a different APOD URL based on `matrix.request.query`.
3. Each job saves an image with a different filename (using `matrix.request.name`).
4. You see three separate artifacts, one per job.

If all three jobs finish successfully and the artifacts contain APOD images, your matrix is behaving as expected.

</details>

---

## Solution (Click to Reveal)

<details>
<summary>Click to show the corrected workflow and fixes</summary>

### 1. Replace the hard-coded API key with a repository secret

**Steps:**

1. Go to **Settings → Secrets and variables → Actions → New repository secret**.
2. Create a secret named `NASA_API_KEY`.
3. Paste the real NASA API key as the value and save it.

Then in your workflow, replace the hard-coded line:

```bash
API_KEY="CHAD-WILL-GIVE-THIS-TO-YOU-IN-THE-ZOOM-CHAT"
```

with:

```bash
API_KEY="${{ secrets.NASA_API_KEY }}"
```

This lets the workflow read the key securely from GitHub’s secret vault.

---

### 2. Add a step to upload the downloaded images as artifacts

Right now, images are saved into `images/`, but nothing uploads them.

Add a new step after the API call step, and make sure each matrix job uses a **unique artifact name**:

```yaml
      - name: Upload APOD images
        uses: actions/upload-artifact@v4
        with:
          name: apod-images-${{ matrix.request.name }}
          path: images/
```

This tells GitHub Actions to package everything in `images/` as an artifact, with one artifact per matrix job like:

* `apod-images-today_apod`
* `apod-images-yesterday_apod`
* `apod-images-last_year_apod`

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
          - name: "last_year_apod"
            query: "date=2024-12-03"

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
          name: apod-images-${{ matrix.request.name }}
          path: images/
```

---

### After making these changes:

1. Push a new commit to `main`.
2. Open the **Actions** tab.
3. Open the latest **NASA APOD Matrix Warmup** run.
4. Confirm in the logs that:

   * Each matrix job calls the APOD API with a different date.
   * The **Call NASA APOD API** step succeeds in all jobs.
   * The **Upload APOD images** step succeeds in all jobs.
   * You see three artifacts:

     * `apod-images-today_apod`
     * `apod-images-yesterday_apod`
     * `apod-images-last_year_apod`

Each artifact should contain one APOD image file from that job.

</details>
