# ✨ `dynamic-matrix-test`

This repository contains a simple GitHub Actions workflow demonstrating how to create a **dynamic job matrix** using JSON data generated within a preceding job. This pattern is ideal for scenarios where the necessary configurations (e.g., projects to build, environments to deploy to, or test permutations) are not known statically and must be determined at runtime.

## 📁 Repository Structure

The core logic is contained within a single workflow file:

```
.
└── .github
    └── workflows
        └── dynamic-matrix.yml
```

## ⚙️ Workflow Breakdown (`dynamic-matrix.yml`)

The workflow consists of two main jobs designed to run sequentially:

### 1. `generate_matrix` Job (The Producer)

This job is responsible for determining and generating the job configurations.

  * It uses a simple `run` step to define a JSON array of objects (`JSON_DATA`). Each object in this array represents a single parallel job to be executed later.
  * It sets this JSON string as a job output named `matrix_json`.

**Key Concept:** The shell script simulates dynamic logic (e.g., checking changed files, querying a manifest file, etc.) and formats the result as a valid JSON array.

### 2. `dynamic_job` Job (The Consumer)

This job runs the actual work in parallel based on the generated configurations.

  * It specifies `needs: generate_matrix` to ensure it only runs after the configuration is available.
  * **Crucially**, it uses the `strategy: matrix:` block with the `include` key:

  ```yaml
  include: ${{ fromJSON(needs.generate_matrix.outputs.matrix_json) }}
  ```

  * The `fromJSON()` function converts the JSON string output from the previous job into a consumable array of objects for the matrix, launching a separate job for each configuration defined.
  * Steps within this job access the variables using `${{ matrix.variable_name }}`.

## 🚀 How to Run and Observe

1.  **Trigger the Workflow:** Push to the `main` branch or manually trigger the workflow using the `workflow_dispatch` button in the Actions tab on GitHub.
2.  **View the Run:** Go to the Actions tab and select the `Dynamic Matrix Generation Example` workflow run.
3.  **Observe Parallelism:** You will see the `dynamic_job` displayed as four parallel jobs (one for each configuration object defined in the generator job), all running simultaneously.

## Example Console Output (from one parallel `dynamic_job`)

The output in the console for one of the jobs (e.g., the `data-api`/`prod` job) will look similar to this:

```
Execute Test/Deploy Step
--- Configuration Summary ---
Project: data-api
Environment: prod
Test Type: integration
---------------------------
```
