# GitHub Workflow Documentation

This repository contains reusable GitHub workflows for Python projects. These workflows can be called from other workflows in your repositories.

## Available Workflows

### 1. Python CI Workflow

The `python-ci.yml` workflow handles testing for Python projects using tox.

**Features:**
- Uses Poetry for dependency management
- Separate jobs for linting, type checking, and testing
- Shared tox cache between jobs for efficiency
- Uses project's tox configuration for all steps
- Automatically detects and runs the right tox environments
- Parallel execution of quality checks
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
      python-version: "3.12"                               # Optional, Python version for running tox
      skip-lint: false                                     # Optional, skip the linting step
      tox-lint-env: "lint"                                 # Optional, tox environment for linting
      skip-typecheck: false                                # Optional, skip the type checking step
      tox-typecheck-env: "typecheck"                       # Optional, tox environment for type checking
      upload-coverage: true                                # Optional, upload to Codecov
```

### 2. Python Public Release Workflow

The `python-public-release.yml` workflow builds and publishes Python packages to PyPI.

**Features:**
- Uses uv for faster package installation
- Optionally runs the CI workflow first
- Builds source distribution and wheel packages
- Validates package description with twine
- Option to publish to TestPyPI before PyPI
- Supports skipping already published versions
- Improved dependency caching for faster builds
- Customizable build arguments

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
      python-version: "3.12"         # Optional, Python version to use (default: "3.12")
      test-matrix: true              # Optional, run CI tests before publishing (default: true)
      build-args: "setuptools-scm"   # Optional, additional build dependencies (default: "")
      publish-to-testpypi: true      # Optional, publish to TestPyPI first (default: false)
      check-description: true        # Optional, validate package description (default: true)
    secrets:
      pypi-token: ${{ secrets.PYPI_API_TOKEN }}
      testpypi-token: ${{ secrets.TEST_PYPI_API_TOKEN }}  # Required if publish-to-testpypi is true
```

## Configuration Options

### Python CI Workflow

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `python-version` | No | `"3.12"` | Python version to use for running tox |
| `skip-lint` | No | `false` | Skip linting step entirely |
| `tox-lint-env` | No | `"lint"` | Tox environment name for linting |
| `skip-typecheck` | No | `false` | Skip type checking step entirely |
| `tox-typecheck-env` | No | `"typecheck"` | Tox environment name for type checking |
| `upload-coverage` | No | `false` | Upload coverage reports to Codecov |

### Python Public Release Workflow

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `python-version` | No | `"3.12"` | Python version to use for building and publishing |
| `test-matrix` | No | `true` | Whether to run CI tests before publishing |
| `build-args` | No | `""` | Additional build dependencies to install |
| `publish-to-testpypi` | No | `false` | Whether to publish to TestPyPI first |
| `check-description` | No | `true` | Validate package description with twine |
| `pypi-token` (secret) | Yes | - | PyPI API token for publishing |
| `testpypi-token` (secret) | No | - | TestPyPI API token (required if publishing to TestPyPI) |

## Project Requirements

To use these workflows effectively, your Python project should have:

1. A properly configured `pyproject.toml` or `setup.py` file
2. A tox.ini file with appropriate environments:
   - A lint environment (default: "lint") for code style checks with ruff/flake8
   - A typecheck environment (default: "typecheck") for static type checking with mypy
   - Test environments configured for the Python versions you want to test
3. For coverage reporting: Configure coverage in your tox.ini and pytest settings

## Implementation Notes

- Uses Poetry for dependency management and virtual environments
- CI workflow splits tasks into parallel jobs for faster execution
- Requires tox-gh-actions plugin which maps GitHub's Python version to tox environments
- Shares tox cache between jobs to avoid rebuilding environments
- Tests run only if linting and type checking pass (unless skipped)
- Recommend adding this configuration to your tox.ini:
  ```ini
  [gh-actions]
  python =
      3.8: py38
      3.9: py39
      3.10: py310
      3.11: py311
      3.12: py312
  ```
- When using codecov integration, make sure your project generates coverage reports
- The public release workflow will proceed even if tests fail, but only if `test-matrix` is false
- Full git history is fetched for proper versioning with tools like setuptools-scm