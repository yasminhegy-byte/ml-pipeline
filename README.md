# ML Model CI/CD Pipeline

This repository contains a machine learning model with automated CI/CD validation using GitHub Actions.

## Project Structure

```
├── .github/
│   └── workflows/
│       └── ml-pipeline.yml          # GitHub Actions workflow for ML model CI
├── requirements.txt                  # Python dependencies
└── README.md                         # This file
```

## Features

- **Automated Testing**: Runs on every push to non-main branches and pull requests to main
- **Linter Checks**: Validates Python code style and errors
- **Model Validation**: Ensures model environment dependencies are available
- **Artifact Generation**: Uploads project documentation for each run

## Requirements

- Python 3.10+
- PyTorch
- Dependencies listed in `requirements.txt`

## Setup

1. Clone the repository
2. Install dependencies: `pip install -r requirements.txt`
3. The workflow will automatically run on push or pull requests

## Pipeline Trigger Rules

- **Push**: Runs on all branches EXCEPT `main`
- **Pull Request**: Runs on pull requests targeting `main` branch
