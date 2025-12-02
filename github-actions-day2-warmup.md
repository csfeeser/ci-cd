# Warmup Activity: Fixing a Broken GitHub Actions Workflow

<img src="https://cdn.prod.website-files.com/6541750d4db1a741ed66738c/66ba7e999d67c8c01f01b1b1_Perhaps%20the%20ultimate%20Orchestration%20Tool.webp" width="400" />

## Part 1: Create the Required Files in GitHub

1. Create a new repository named `star-wars-ascii-action`.
2. Inside the repo, select **Add file → Create new file**.
3. In the root directory (meaning, don't put it in any folders) create a file named `r2d2.sh`.
4. Paste in this script:

```bash
#!/usr/bin/env bash

# Simple R2-D2 ASCII
cat << "EOF"
⠀⠀⠀⠀⠀⠀⣠⣴⣾⣿⣿⣿⣿⣷⣦⣄⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⢠⣾⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣷⡄⠀⠀⠀⠀
⠀⠀⠀⢀⣿⣿⣿⣿⡿⠛⢿⡿⠛⢻⣿⣿⣿⣿⡀⠀⠀⠀
⠀⠀⠀⢸⣿⣿⣿⣿⡇⠀⢸⣷⣶⣾⣿⣿⣿⣿⡇⠀⠀⠀
⠀⠀⠀⠈⠉⠉⠉⠉⠁⠀⠈⠉⠉⠉⠉⠉⠉⠉⠁⠀⠀⠀
⠀⢀⣤⣀⣾⣿⣿⣿⠟⠛⠛⠛⠛⠻⣿⣿⣿⣷⣀⣤⡀⠀
⠀⢸⣿⣿⣿⣿⣿⣿⣤⣤⣤⣤⣤⣤⣿⣿⣿⣿⣿⣿⡇⠀
⠀⢸⣿⣿⣿⣿⣿⣿⣿⣿⡿⢿⣿⣿⣿⣿⣿⣿⣿⣿⡇⠀
⠀⢸⣿⣿⣿⣿⣿⣿⣿⣿⠀⠀⣿⣿⣿⣿⣿⣿⣿⣿⡇⠀
⠀⢸⣿⡟⢿⣿⣿⣿⣿⣿⠀⠀⣿⣿⣿⣿⣿⡿⢻⣿⡇⠀
⠀⢸⣿⡇⠈⠙⠛⢛⣿⣿⣤⣤⣿⣿⡛⠛⠋⠁⢸⣿⡇⠀
⣤⣼⣿⣧⣤⡀⠀⠙⠛⠛⠛⠛⠛⠛⠋⠀⢀⣤⣼⣿⣧⣤
⠛⠛⠛⠛⠛⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀  ⠈⠛⠛⠛⠛⠛
EOF
```

5. Commit the file.

6. Now create our workflow at the following path: `.github/workflows/r2d2.txt`

7. Paste in the *broken* workflow below:

```yaml
name: Star Wars ASCII Warmup

on:
  push:
    branches:
      - main

jobs:
  print-r2:
    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Run R2 script
        run: bash r2-d-2.sh
```

## Part 2: Objective

Your job is to:

* Find all errors in the workflow
* Fix the workflow so GitHub Actions can run it without error.
* Make the workflow successfully run the `r2d2.sh` script on an ubuntu VM.
* Verify the ASCII output in the Actions logs

## Hints

<details>
<summary>Why won't the workflow start?</summary>

HINT: GitHub only reads workflow files that end in **.yml** or **.yaml**.

</details>

<details>
<summary>Why does GitHub say “runs-on” is required?</summary>

Every job needs to specify what virtual machine it should run on. We want this to run on the latest Ubuntu VM. Something's missing!

</details>

<details>
<summary>Why does GitHub say the script can’t be found?</summary>

The workflow is trying to run `bash r2-d-2.sh`... but wait, was that the actual file name?

</details>


## Solution (Click to Reveal)

<details>
<summary>Click to show the corrected workflow and fixes</summary>

1. The workflow file extension is wrong! Even if the file is in the correct directory (`.github/workflows`) it won't be read unless it has a YAML extension. Edit the file in your browser. Click the file name and change it to `.github/workflows/r2d2.yml` 


2. Here are all the corrections necessary, indicated by comments.
   
```yaml
name: Star Wars ASCII Warmup

on:
  push:
    branches:
      - main

jobs:
  print-r2:
    runs-on: ubuntu-latest   # FIX: required field

    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Run R2 script
        run: bash r2d2.sh       # FIX: script name mismatch
```

</details>
