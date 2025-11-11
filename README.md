# ✨ `dynamic-matrix-test`

This repository contains a simple GitHub Actions workflow demonstrating how to create a **dynamic job matrix** using JSON data generated within a preceding job and then **calling a reusable workflow** for each matrix configuration.

This pattern is ideal for scenarios where the necessary configurations (e.g., projects to build, environments to deploy to, or test permutations) are not known statically and must be determined at runtime, and where the execution logic is centralized in a separate reusable workflow.

## 📁 Repository Structure

The core logic is now split across two workflow files:

```
.
└── .github
    └── workflows
        ├── dynamic-matrix.yml (The Orchestrator)
        └── deployProject.yml (The Reusable Logic)
```

---

## ⚙️ Workflow Breakdown

### 1. `dynamicMatrix.yml` (The Orchestrator)

This workflow defines the sequencing and the dynamic matrix.

#### A. `generate_matrix` Job (The Producer)

This job determines and generates the job configurations.

* It uses a simple `run` step to define a JSON array of objects (`JSON_DATA`). Each object in this array represents a single parallel job to be executed later.
* It sets this JSON string as a job output named `matrix_json`.

**Key Concept:** The shell script simulates dynamic logic (e.g., checking changed files, querying a manifest file, etc.) and formats the result as a valid JSON array.

#### B. `dynamic_job` Job (The Consumer / Caller)

This job is configured to consume the dynamic matrix and call the reusable workflow.

* It specifies `needs: generate_matrix` to ensure it only runs after the configuration is available.
* **Dynamic Matrix:** It uses the `strategy: matrix:` block with the `fromJSON()` function to convert the JSON string output from the previous job into a consumable array of objects, launching a separate job for each configuration.
    ```yaml
    strategy:
      matrix:
        post: ${{ fromJSON(needs.generate_matrix.outputs.matrix_json) }}
    ```
* **Reusable Workflow Call:** It replaces traditional `steps:` with a single `uses:` key pointing to the local reusable workflow:
    ```yaml
    uses: ./.github/workflows/deployProject.yml
    ```
* **Input Mapping:** It maps the dynamic matrix variables (e.g., `${{ matrix.post.project }}`) to the reusable workflow's inputs (`project_name`, etc.). It successfully uses the quoted bracket notation for keys with hyphens: `test_type: "${{ matrix.post['test-type'] }}"`.

### 2. `deployProject.yml` (The Reusable Logic)

This workflow is the execution target, defining the actual steps to be run for each parallel job.

* It uses `on: workflow_call:` to accept inputs defined by the calling job.
* It defines a single job, `run_deployment`, which runs on `ubuntu-latest`.
* Its steps access the passed configurations using the standard reusable workflow syntax: `${{ inputs.project_name }}`.

---

## 🚀 How to Run and Observe

1.  **Trigger the Workflow:** Push to the `main` branch or manually trigger the workflow using the `workflow_dispatch` button in the Actions tab on GitHub.
2.  **View the Run:** Go to the Actions tab and select the **`Dynamic Matrix Generation Example`** workflow run.
3.  **Observe Parallelism:** The `dynamic_job` will be displayed as **four parallel jobs** (one for each configuration object defined in the generator job). Each of these parallel jobs is a call to the reusable workflow.

## Example Console Output (from one parallel reusable workflow run)

The output in the console for one of the running jobs (e.g., the `Deploy/Test data-api - prod` job) will look similar to this:

```
--- Reusable Workflow Execution ---
Received Inputs:
Project: client-portal
Environment: stage
Test Type: smoke
Status: Deployment/Test started successfully.
-----------------------------------
```