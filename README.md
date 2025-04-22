# GitHub Workflow Documentation

This repository contains reusable GitHub workflows for Python projects. These workflows can be called from other workflows in your repositories.

## Available Workflows

### 1. Python CI Workflow

The `python-ci.yml` workflow handles testing for Python projects using tox.

**Features:**
- Uses Poetry for dependency management
- Single job with sequential quality checks
- Uses project's tox configuration for all steps
- Simplified configuration with sensible defaults
- Unified caching for faster builds
- Coverage reporting to Codecov

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
      python-version: "3.12"    # Optional, Python version for running tox (default: "3.12")
      run-lint: true            # Optional, run the linting step (default: true)
      run-typecheck: true       # Optional, run the type checking step (default: true)
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
| `run-lint` | No | `true` | Whether to run the linting step |
| `run-typecheck` | No | `true` | Whether to run the type checking step |

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

## tox.ini Example Configuration

Here's a sample `tox.ini` configuration that works well with the CI workflow:

```ini
[tox]
envlist = py38, py39, py310, py311, py312, lint, typecheck
isolated_build = True
skip_missing_interpreters = True

[gh-actions]
python =
    3.8: py38
    3.9: py39
    3.10: py310
    3.11: py311
    3.12: py312, lint, typecheck

[testenv]
deps =
    pytest
    pytest-cov
commands =
    pytest {posargs:tests} --cov=your_package_name --cov-report=xml

[testenv:lint]
deps =
    ruff
skip_install = True
commands =
    ruff check {posargs:.}
    ruff format --check {posargs:.}

[testenv:typecheck]
deps =
    mypy
    types-requests
    # Add other type stubs as needed
commands =
    mypy {posargs:your_package_name}

[pytest]
testpaths = tests
python_files = test_*.py
python_functions = test_*
```

### Explanation

1. **Basic tox configuration**:
   - `envlist`: Defines all environments to run, including Python versions and quality checks
   - `isolated_build`: Creates an isolated build environment
   - `skip_missing_interpreters`: Skips Python versions not available on the system

2. **GitHub Actions integration**:
   - `[gh-actions]` section maps GitHub's Python versions to tox environments
   - When running Python 3.12 in GitHub Actions, it will also run lint and typecheck

3. **Test environments**:
   - `[testenv]` defines settings for all test environments
   - Uses pytest and pytest-cov for running tests and collecting coverage
   - Generates an XML coverage report that Codecov can read

4. **Lint environment**:
   - Uses ruff for both linting and formatting checks
   - `skip_install = True` means it doesn't need to install the package

5. **Type check environment**:
   - Uses mypy for static type checking
   - Includes common type stubs like `types-requests`

### Usage with Poetry

If your project uses Poetry, you should have a `pyproject.toml` file. Here's a basic example that works with the tox configuration above:

```toml
[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"

[tool.poetry]
name = "your-package-name"
version = "0.1.0"
description = "Your package description"
authors = ["Your Name <your.email@example.com>"]
readme = "README.md"
packages = [{include = "your_package_name"}]

[tool.poetry.dependencies]
python = "^3.8"
# Add your dependencies here

[tool.poetry.group.dev.dependencies]
pytest = "^7.3.1"
pytest-cov = "^4.1.0"
ruff = "^0.1.3"
mypy = "^1.5.1"
tox = "^4.11.3"

[tool.ruff]
line-length = 88
target-version = "py38"
select = ["E", "F", "I", "UP"]

[tool.mypy]
python_version = "3.8"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
```

## Implementation Notes

- Uses Poetry for dependency management and virtual environments
- More efficient design with a single job for all quality checks
- Requires tox-gh-actions plugin which maps GitHub's Python version to tox environments
- Coverage reports are automatically uploaded to Codecov
- The public release workflow will proceed even if tests fail, but only if `test-matrix` is false
- Full git history is fetched for proper versioning with tools like setuptools-scm