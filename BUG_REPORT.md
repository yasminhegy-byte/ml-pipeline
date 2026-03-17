# GitHub Actions ML Pipeline - Bug Report & Solutions

## Overview
This document details all syntax and logical errors found in the original GitHub Actions workflow YAML file and provides the corrected solutions.

---

## Bug #1: Invalid YAML Structure for `on:` Trigger

### ❌ Original Code (INCORRECT)
```yaml
on:
  push:
  branches: main
  pull_request:
```

### Problem
- **Location**: Lines 3-6
- **Issue**: The `branches:` key is at the wrong indentation level. It should be nested under `push:`, not at the same level as workflow triggers.
- **Syntax Error**: This violates YAML structure. The parser would interpret this as a top-level key rather than part of the push configuration.

### ✅ Solution
```yaml
on:
  push:
    branches: [main]
  pull_request:
```

### Explanation
- `branches:` must be indented under the `push:` key to properly configure which branches trigger the push event.
- Use the array syntax `[main]` for proper YAML formatting.

---

## Bug #2: Missing `run:` Command for Linter Check Step

### ❌ Original Code (INCORRECT)
```yaml
- name: Linter Check
```

### Problem
- **Location**: Line 20
- **Issue**: The step has a `name:` but no `run:` command specified. GitHub Actions requires either a `run:` command or a `uses:` action reference.
- **Consequence**: The job would fail with a validation error before execution.

### ✅ Solution
```yaml
- name: Linter Check
  run: pylint src/ --disable=all --enable=E
```

### Explanation
- Added `run:` command to execute a linter check using `pylint`.
- This validates Python code for critical errors (E level only).
- Alternative: Could use `uses: actions/setup-python@v5` if using a separate linting action.

---

## Bug #3: Incorrect Branch Filter Syntax for Push

### ❌ Original Code (INCORRECT)
```yaml
on:
  push:
    branches: main
```

### Problem
- **Location**: Branch specification
- **Issue**: `branches: main` is written as a string instead of an array. YAML requires array format for multiple items or proper syntax for single items.
- **Current Requirement**: The required behavior is to run on ALL branches EXCEPT main, but the original code only runs on main.

### ✅ Solution
```yaml
on:
  push:
    branches-ignore:
      - main
```

### Explanation
- Changed from `branches:` to `branches-ignore:` to invert the logic.
- Now the pipeline runs on every push EXCEPT to the main branch.
- This prevents duplicate CI runs when PRs are merged to main.
- Proper array syntax: `- main` represents a list item.

---

## Bug #4: PR Trigger Incomplete

### ❌ Original Code (VAGUE)
```yaml
pull_request:
```

### Problem
- **Location**: Line 21
- **Issue**: `pull_request:` has no configuration. It's unclear if PRs should trigger on all branches or specific ones.
- **Best Practice**: Should explicitly specify branches to avoid ambiguity and unintended triggers.

### ✅ Solution
```yaml
pull_request:
  branches: [main]
```

### Explanation
- Explicitly specifies that pull requests targeting the `main` branch trigger the workflow.
- Prevents unnecessary runs on internal branch PRs.
- Aligns with the common GitFlow pattern: develop → pull request → merge to main.

---

## Bug #5: Missing Checkout Step

### ❌ Original Code (MISSING)
```yaml
# No checkout action present
```

### Problem
- **Location**: Start of workflow steps
- **Issue**: The workflow needs to check out the repository code before installing dependencies or running tests. Without this step, `requirements.txt` cannot be accessed.
- **Consequence**: The `pip install -r requirements.txt` step would fail because the file doesn't exist in the runner environment.

### ✅ Solution
```yaml
- name: Checkout Code
  uses: actions/checkout@v4
```

### Explanation
- Added the standard `actions/checkout@v4` action as the first step.
- This clones your repository into the GitHub Actions runner environment.
- Essential for accessing project files (requirements.txt, source code, etc.).

---

## Bug #6: Missing README Upload Step

### ❌ Original Code (MISSING)
```yaml
# No artifact upload step
```

### Problem
- **Location**: End of workflow steps
- **Issue**: The requirement specifies uploading the README.md as a GitHub artifact named `project-doc`, but this step was missing entirely.

### ✅ Solution
```yaml
- name: Upload README as Artifact
  uses: actions/upload-artifact@v3
  with:
    name: project-doc
    path: README.md
```

### Explanation
- Uses the `actions/upload-artifact@v3` action to upload files.
- Specifies artifact name as `project-doc` for easy identification.
- Uploads the README.md file for project documentation.
- Artifacts are available in the "Artifacts" section of the workflow run.

---

## Summary of Changes

| Bug # | Issue | Type | Severity |
|-------|-------|------|----------|
| 1 | YAML indentation for `branches:` | Syntax Error | 🔴 Critical |
| 2 | Missing `run:` command for Linter | Configuration Error | 🔴 Critical |
| 3 | Wrong branch trigger logic | Logic Error | 🟠 High |
| 4 | Incomplete PR trigger config | Best Practice | 🟡 Medium |
| 5 | Missing checkout action | Runtime Error | 🔴 Critical |
| 6 | Missing artifact upload step | Missing Feature | 🟡 Medium |

---

## Final Corrected YAML File

```yaml
name: ML Model CI

on:
  push:
    branches-ignore:
      - main
  pull_request:
    branches: [main]

jobs:
  validate-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.10'

      - name: Install Dependencies
        run: pip install -r requirements.txt

      - name: Linter Check
        run: pylint src/ --disable=all --enable=E

      - name: Model Dry Test
        run: |
          python -c "import torch; print('Model environment ready!')"

      - name: Upload README as Artifact
        uses: actions/upload-artifact@v3
        with:
          name: project-doc
          path: README.md
```

---

## Testing the Workflow

1. Push code to a non-main branch → workflow triggers ✅
2. Push code to main branch → workflow does NOT trigger ✅
3. Create pull request to main → workflow triggers ✅
4. Workflow completes successfully → README.md appears in Artifacts ✅

---

## References

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Workflow Syntax for GitHub Actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Upload Artifacts Action](https://github.com/actions/upload-artifact)
- [Checkout Action](https://github.com/actions/checkout)
