# GitHub Workflow Documentation

This repository contains reusable GitHub workflows for Python projects. These workflows can be called from other workflows in your repositories.

## Available Workflows

### 1. Python CI Workflow

The `python-ci.yml` workflow handles linting and testing for Python projects.

**Features:**
- Code linting with Ruff (checking and formatting)
- Type checking with MyPy
- Testing with pytest through tox
- Matrix testing with multiple Python versions
- Conditional linting with skip option
- Improved caching for faster builds
- Optional coverage reporting to Codecov

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
      python-versions: '["3.9", "3.10", "3.11", "3.12"]'  # Optional, test matrix
      fail-fast: false                                     # Optional, continue tests if one fails
      skip-lint: false                                     # Optional, skip the linting step
      tox-env: "py"                                        # Optional, tox environment to run
      upload-coverage: true                                # Optional, upload to Codecov
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
| `fail-fast` | No | `false` | Whether to stop all matrix tests if one fails |
| `skip-lint` | No | `false` | Skip linting step entirely |
| `tox-env` | No | `"py"` | Tox environment name to run |
| `upload-coverage` | No | `false` | Upload coverage reports to Codecov |

### Python Public Release Workflow

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `python-version` | No | `"3.12"` | Python version to use for building and publishing |
| `test-matrix` | No | `true` | Whether to run CI tests before publishing |
| `pypi-token` (secret) | Yes | - | PyPI API token for publishing |

## Project Requirements

To use these workflows effectively, your Python project should have:

1. A properly configured `pyproject.toml` or `setup.py` file
2. For linting: Ruff and mypy configurations
3. For testing: Tox configuration with appropriate environments  
4. For coverage reporting: Configure coverage in your tox.ini and pytest settings

## Implementation Notes

- The linting job uses Python 3.12 regardless of the test matrix
- Tests will run even if linting is skipped or fails (`always()` condition)
- Caching is applied to both pip and tox environments for faster builds
- When using codecov integration, make sure your project generates coverage reports