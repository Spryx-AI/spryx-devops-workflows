# GitHub Workflow Documentation

This repository contains reusable GitHub workflows for Python projects. These workflows can be called from other workflows in your repositories.

## Available Workflows

### 1. Python CI Workflow

The `python-ci.yml` workflow handles linting and testing for Python projects.

**Features:**
- Code linting with Ruff
- Type checking with MyPy
- Testing with pytest through tox
- Matrix testing with multiple Python versions

**Usage Example:**

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    uses: spryx-devops-workflows/.github/workflows/python-ci.yml@main
    with:
      python-versions: '["3.9", "3.10", "3.11", "3.12"]'  # Optional, this is the default
```

### 2. Python Public Release Workflow

The `python-public-release.yml` workflow builds and publishes Python packages to PyPI.

**Features:**
- Optionally runs the CI workflow first
- Builds source distribution and wheel packages
- Publishes to PyPI
- Supports skipping already published versions

**Usage Example:**

```yaml
name: Release

on:
  release:
    types: [published]

jobs:
  publish:
    uses: spryx-devops-workflows/.github/workflows/python-public-release.yml@main
    with:
      python-version: "3.12"  # Optional, this is the default
      test-matrix: true       # Optional, run CI tests before publishing (default: true)
    secrets:
      pypi-token: ${{ secrets.PYPI_API_TOKEN }}
```

## Configuration Options

### Python CI Workflow

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `python-versions` | No | `["3.9", "3.10", "3.11", "3.12"]` | JSON array of Python versions to test against |

### Python Public Release Workflow

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `python-version` | No | `"3.12"` | Python version to use for building and publishing |
| `test-matrix` | No | `true` | Whether to run CI tests before publishing |
| `pypi-token` (secret) | Yes | - | PyPI API token for publishing |

## Project Requirements

To use these workflows effectively, your Python project should have:

1. A properly configured `pyproject.toml` or `setup.py` file
2. Configuration for ruff and mypy (if using the CI workflow)
3. Tests that can be run with tox