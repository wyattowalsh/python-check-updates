---
# python check updates

like npm-check-updates but for python

![Python Version](https://img.shields.io/badge/python-3.13-blue)
![License](https://img.shields.io/badge/license-MIT%20License%20(MIT)-blue)
![Stars](https://img.shields.io/github/stars/wyattowalsh/python-check-updates?style=social)
![Forks](https://img.shields.io/github/forks/wyattowalsh/python-check-updates?style=social)
![Issues](https://img.shields.io/github/issues/wyattowalsh/python-check-updates)
![Contributors](https://img.shields.io/github/contributors/wyattowalsh/python-check-updates)
![Last Commit](https://img.shields.io/github/last-commit/wyattowalsh/python-check-updates)
![Commit Activity](https://img.shields.io/github/commit-activity/m/wyattowalsh/python-check-updates)

<p align="center">
  <img src="https://user-images.githubusercontent.com/your-username/your-project-banner.png" alt="Project Banner" />
</p>

## Table of Contents

- [Technologies Used](#technologies-used)
- [Project Demo](#project-demo)
- [GitHub Actions](#github-actions)
- [Screenshots](#screenshots)
- [Star History](#star-history)
- [Installation](#installation)
- [Usage](#usage)
- [Development](#development)
- [Documentation](#documentation)
- [Contributing](#contributing)
- [Issue Templates](#issue-templates)
- [License](#license)
- [Feedback](#feedback)

## Technologies Used

![Pyenv](https://img.shields.io/badge/version%20manager-pyenv-blue)

## Project Demo

![Demo GIF](https://user-images.githubusercontent.com/your-username/your-project-demo.gif)

## GitHub Actions

![Build Status](https://github.com/wyattowalsh/python-check-updates/actions/workflows/format-lint-test.yml/badge.svg)

## Screenshots

<p align="center">
  <img src="https://user-images.githubusercontent.com/your-username/screenshot1.png" alt="Screenshot 1" width="400"/>
  <img src="https://user-images.githubusercontent.com/your-username/screenshot2.png" alt="Screenshot 2" width="400"/>
</p>

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=wyattowalsh/python-check-updates&type=Date)](https://star-history.com/#wyattowalsh/python-check-updates&Date)

## Installation

### Prerequisites

- [Python 3.13](https://www.python.org/)
- [Poetry](https://python-poetry.org/docs/#installation)
- [Pyenv](https://github.com/pyenv/pyenv#installation)

```bash
# Clone the repository
git clone https://github.com/wyattowalsh/python-check-updates.git

# Navigate into the project directory
cd python-check-updates

# Install dependencies using Poetry
poetry install
```

## Usage

```bash
poetry run python -m python-check-updates
```

## Development

For details on contributing to development and setting up the development environment, please refer to [DEVELOPMENT.md](DEVELOPMENT.md).

### Issue Templates

To help facilitate contributions and track issues effectively, please refer to our [Issue Templates](https://github.com/wyattowalsh/python-check-updates/issues/new/choose) to submit:

- Bug reports
- Feature requests
- Documentation issues

## Documentation

Documentation can be found at [Documentation Link](https://github.com/wyattowalsh/python-check-updates/wiki).

## Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## Feedback

If you have any feedback or suggestions, feel free to open an issue or reach out directly at [email@example.com](mailto:email@example.com).

## License

Distributed under the MIT License (MIT) License. See `LICENSE` for more information.

---

<p align="center">💻 Made with ❤️ and Python by Wyatt Walsh</p>

# Project Structure



# File Contents

## tests/__init__.py

```py

```

## tests/test_config.py

```py
import os
from pathlib import Path
import pytest
import yaml
from typing import Dict, Any

from python-check-updates.config import (
    Config, 
    AppSettings,
    LoggingSettings,
    FrozenModel,
    config
)
from python-check-updates.logging import LogLevel

# Fixtures

@pytest.fixture
def temp_config_file(tmp_path) -> Path:
    """Create a temporary valid config file."""
    config_data = {
        "app_name": "test_app",
        "version": "0.1.0",
        "debug": False,
        "logging": {
            "app_name": "test_app",
            "level": "INFO",
            "log_dir": str(tmp_path / "logs"),
            "format_string": "<green>{time}</green> | {message}",
            "console": {"enabled": True},
            "file": {"enabled": True},
            "json": {"enabled": True},
            "batch": {"initial_size": 100},
            "progress": {
                "theme": "NEON",
                "themes": {
                    "neon": {
                        "bar_color": "cyan",
                        "complete_style": {"color": "green", "bold": True},
                        "progress_style": {"color": "white"},
                        "spinner_style": {"color": "magenta"},
                        "description_style": {"color": "yellow"}
                    }
                }
            },
            "parallel": {"max_workers": 2}
        }
    }
    config_path = tmp_path / "config.yaml"
    with open(config_path, "w") as f:
        yaml.dump(config_data, f)
    return config_path

@pytest.fixture
def config_instance(temp_config_file: Path) -> Config:
    """Get a fresh config instance with temporary file."""
    Config._instance = None
    instance = Config()
    instance._config_path = temp_config_file
    instance.load_config()
    return instance

# Test Cases

def test_singleton_pattern():
    """Test Config class implements singleton pattern correctly."""
    Config._instance = None
    first = Config()
    second = Config()
    assert first is second
    assert id(first) == id(second)

def test_config_initialization(config_instance: Config):
    """Test basic configuration initialization."""
    assert config_instance._settings is not None
    assert isinstance(config_instance._settings, AppSettings)
    assert config_instance._settings.app_name == "test_app"
    assert config_instance._settings.version == "0.1.0"
    assert config_instance._settings.debug is False

def test_yaml_loading(config_instance: Config, temp_config_file: Path):
    """Test YAML configuration loading."""
    config_instance.load_config(temp_config_file)
    assert config_instance._settings is not None
    assert config_instance._settings.logging.level == LogLevel.INFO
    assert Path(config_instance._settings.logging.log_dir).name == "logs"

def test_env_var_override(config_instance: Config):
    """Test environment variable overrides."""
    os.environ["APP_DEBUG"] = "true"
    os.environ["APP_LOGGING__LEVEL"] = "DEBUG"
    
    config_instance.reload()
    
    assert config_instance.settings.debug is True
    assert config_instance.settings.logging.level == LogLevel.DEBUG
    
    # Cleanup
    del os.environ["APP_DEBUG"]
    del os.environ["APP_LOGGING__LEVEL"]

def test_settings_cache(config_instance: Config):
    """Test settings caching behavior."""
    # First access caches the value
    value1 = config_instance.get_setting("logging.level")
    value2 = config_instance.get_setting("logging.level")
    assert value1 == value2
    
    # Cache should be cleared on reload
    config_instance.reload()
    assert not hasattr(config_instance.get_setting, "cache_info") or \
           config_instance.get_setting.cache_info().hits == 0

@pytest.mark.parametrize("invalid_path", [
    "nonexistent.path",
    "logging.invalid",
    "",
    None
])
def test_invalid_setting_paths(config_instance: Config, invalid_path):
    """Test handling of invalid setting paths."""
    assert config_instance.get_setting(invalid_path, default="default") == "default"

def test_config_reload(config_instance: Config, temp_config_file: Path):
    """Test configuration reloading."""
    original_level = config_instance.settings.logging.level
    
    # Modify config file
    with open(temp_config_file) as f:
        config_data = yaml.safe_load(f)
    config_data["logging"]["level"] = "DEBUG"
    with open(temp_config_file, "w") as f:
        yaml.dump(config_data, f)
    
    config_instance.reload()
    assert config_instance.settings.logging.level == LogLevel.DEBUG
    assert config_instance.settings.logging.level != original_level

def test_missing_config_file():
    """Test handling of missing config file."""
    config = Config()
    config._config_path = Path("nonexistent.yaml")
    
    with pytest.raises(FileNotFoundError):
        config.load_config()

def test_invalid_yaml_format(tmp_path):
    """Test handling of invalid YAML format."""
    config_path = tmp_path / "invalid.yaml"
    with open(config_path, "w") as f:
        f.write("invalid: yaml: content: {")
    
    config = Config()
    config._config_path = config_path
    
    with pytest.raises(yaml.YAMLError):
        config.load_config()

def test_frozen_model_immutability():
    """Test FrozenModel immutability."""
    class TestModel(FrozenModel):
        value: str = "test"
    
    model = TestModel()
    with pytest.raises(Exception):  # Pydantic raises TypeError or ValidationError
        model.value = "changed"

def test_nested_settings_access(config_instance: Config):
    """Test accessing nested settings."""
    # Valid nested access
    assert config_instance.get_setting("logging.console.enabled") is True
    
    # Deep nesting
    assert config_instance.get_setting("logging.progress.themes.neon.bar_color") == "cyan"
    
    # Invalid nesting with default
    assert config_instance.get_setting("logging.invalid.nested.path", "default") == "default"

@pytest.mark.parametrize("setting_path,expected_type", [
    ("app_name", str),
    ("version", str),
    ("debug", bool),
    ("logging.level", LogLevel),
    ("logging.parallel.max_workers", int)
])
def test_setting_types(config_instance: Config, setting_path: str, expected_type: type):
    """Test setting value types are correct."""
    value = config_instance.get_setting(setting_path)
    assert isinstance(value, expected_type)

def test_multiple_reloads(config_instance: Config):
    """Test multiple consecutive reloads."""
    for _ in range(5):
        config_instance.reload()
        assert config_instance._settings is not None
        assert isinstance(config_instance._settings, AppSettings)

def test_concurrent_access(config_instance: Config):
    """Test concurrent access to settings."""
    import threading
    import queue
    
    results = queue.Queue()
    def worker():
        try:
            value = config_instance.get_setting("logging.level")
            results.put(("success", value))
        except Exception as e:
            results.put(("error", str(e)))
    
    threads = [threading.Thread(target=worker) for _ in range(10)]
    for t in threads:
        t.start()
    for t in threads:
        t.join()
    
    while not results.empty():
        status, value = results.get()
        assert status == "success"
        assert value == LogLevel.INFO

def test_large_config(tmp_path):
    """Test handling of large configuration files."""
    large_config = {
        "app_name": "test_app",
        "version": "0.1.0",
        "logging": {
            "level": "INFO",
            "large_data": ["item" + str(i) for i in range(10000)]
        }
    }
    
    config_path = tmp_path / "large_config.yaml"
    with open(config_path, "w") as f:
        yaml.dump(large_config, f)
    
    config = Config()
    config._config_path = config_path
    config.load_config()
    
    assert len(config.get_setting("logging.large_data")) == 10000

def test_config_performance(config_instance: Config, benchmark):
    """Test configuration access performance."""
    def access_settings():
        return config_instance.get_setting("logging.level")
    
    result = benchmark(access_settings)
    assert result == LogLevel.INFO

```

## tests/test_logging.py

```py
# test_logging.py

import importlib
from unittest import mock

import pytest
from loguru import logger

from python-check-updates.logging import LoggingConfig

import asyncio
import json
import os
from pathlib import Path
from unittest.mock import AsyncMock, MagicMock, Mock, patch

import pytest
import yaml
from rich.progress import Progress

from python-check-updates.logging import (LogLevel, LogStats, LoggingConfig,
                                                 ProgressBarStyle, ProgressTheme)

"""Test suite for logging configuration and functionality.

This module tests all aspects of the logging system:
- Configuration loading and validation
- Log output formats and sinks
- Progress bar functionality
- Performance tracking
- Batch logging
- Thread safety
- Resource cleanup
"""

# Fixtures
@pytest.fixture
def mock_config_path(tmp_path):
    """Create a temporary config file for testing.
    
    Creates a minimal valid configuration file with all required
    logging settings for testing purposes.
    
    Args:
        tmp_path: Pytest fixture providing temporary directory
    
    Returns:
        Path: Path to temporary config file
    """
    config = {
        "logging": {
            "app_name": "test_app",
            "level": "INFO",
            "log_dir": str(tmp_path),
            "console": {"enabled": True},
            "file": {"enabled": True},
            "json": {"enabled": True},
            "batch": {"initial_size": 100},
            "progress": {
                "theme": "NEON",
                "themes": {
                    "neon": ProgressTheme.NEON,
                    "minimal": ProgressTheme.MINIMAL
                }
            },
            "parallel": {"max_workers": 2}
        }
    }
    config_path = tmp_path / "config.yaml"
    with open(config_path, "w") as f:
        yaml.dump(config, f)
    return config_path

@pytest.fixture
def logging_config(mock_config_path, tmp_path):
    """Create a LoggingConfig instance for testing."""
    with patch("rich.console.Console"), patch("rich.progress.Progress"), patch("loguru.logger"):
        config = LoggingConfig.from_yaml(mock_config_path)
        config.log_dir = tmp_path
        return config

# Basic Configuration Tests
def test_logging_config_initialization(logging_config):
    """Test basic initialization of LoggingConfig."""
    assert logging_config.app_name == "test_app"
    assert logging_config.log_level == LogLevel.INFO
    assert isinstance(logging_config.progress, Progress)

def test_logging_config_from_yaml(mock_config_path):
    """Test configuration loading from YAML."""
    config = LoggingConfig.from_yaml(mock_config_path)
    assert config.app_name == "test_app"
    assert config.log_level == LogLevel.INFO

# Logging Functionality Tests
@pytest.mark.asyncio
async def test_async_logging(logging_config):
    """Test async logging capabilities."""
    with patch("loguru.logger") as mock_logger:
        mock_logger.complete = AsyncMock()
        await logging_config.alog("INFO", "test message")
        mock_logger.complete.assert_awaited_once()
        mock_logger.log.assert_called_once_with("INFO", "test message")

def test_batch_logging(logging_config):
    """Test batch logging functionality."""
    with patch("loguru.logger") as mock_logger:
        # Test batch accumulation
        for i in range(50):
            logging_config.batch_log("INFO", f"msg {i}")
        assert len(logging_config._log_batch) == 50
        mock_logger.log.assert_not_called()

        # Test batch flush
        for i in range(51):
            logging_config.batch_log("INFO", f"msg {i}")
        assert len(logging_config._log_batch) == 0
        assert mock_logger.log.call_count > 0

# Progress Bar Tests
def test_progress_bar_features(logging_config):
    """Test progress bar creation and updates."""
    with patch("rich.progress.Progress") as MockProgress:
        # Test creation
        task_id = logging_config.create_progress_bar(100, "test")
        MockProgress.return_value.add_task.assert_called_once()

        # Test update
        logging_config.update_progress(task_id, 5)
        MockProgress.return_value.update.assert_called_once()

        # Test themed progress
        progress = logging_config.create_themed_progress(ProgressTheme.NEON)
        assert isinstance(progress, Progress)

# Performance and Resource Tests
def test_performance_tracking(logging_config):
    """Test performance tracking functionality."""
    with patch("time.perf_counter") as mock_time, \
         patch("psutil.Process") as mock_process, \
         patch("loguru.logger") as mock_logger:
        
        mock_time.side_effect = [0, 1]
        mock_process.return_value.memory_info.return_value.rss = 1024 * 1024
        
        with logging_config.performance_tracking("test_op"):
            pass
        
        mock_logger.info.assert_called_once()

# Error Handling and Edge Cases
@pytest.mark.parametrize("invalid_config", [
    {},
    {"logging": {}},
    {"logging": {"invalid": "config"}},
])
def test_invalid_config_handling(tmp_path, invalid_config):
    """Test handling of invalid configurations."""
    config_path = tmp_path / "invalid_config.yaml"
    with open(config_path, "w") as f:
        yaml.dump(invalid_config, f)
    
    with pytest.raises((KeyError, ValueError)):
        LoggingConfig.from_yaml(config_path)

def test_log_interpolation_edge_cases(logging_config):
    """Test log message interpolation with edge cases."""
    # Valid interpolation
    assert logging_config.interpolate("Hello {name}", name="World") == "Hello World"
    
    # Missing parameter
    result = logging_config.interpolate("Hello {name}")
    assert "Failed to interpolate" in result

    # Empty message
    assert logging_config.interpolate("") == ""

# Statistics and Metrics Tests
def test_logging_statistics(logging_config):
    """Test logging statistics collection and accuracy.
    
    Verifies that:
    - Message counts are accurate
    - Error counts are tracked correctly
    - Performance metrics are calculated
    
    Approach:
    1. Generate known number of logs
    2. Include specific error logs
    3. Verify all metrics match expectations
    """
    logging_config.stats.update("INFO")
    logging_config.stats.update("ERROR", is_error=True)
    
    stats = logging_config.get_stats()
    assert stats["total_messages"] == 2
    assert stats["errors_count"] == 1
    assert "messages_per_second" in stats

# Thread Safety Tests
def test_thread_safety(logging_config):
    """Test thread-safe operations."""
    import threading
    
    def log_messages():
        for _ in range(100):
            logging_config.batch_log("INFO", "test")
    
    threads = [threading.Thread(target=log_messages) for _ in range(5)]
    for t in threads:
        t.start()
    for t in threads:
        t.join()
    
    logging_config.flush_logs()
    assert logging_config.stats.total_messages > 0

# Cleanup and Resource Management
def test_resource_cleanup(logging_config):
    """Test proper resource cleanup."""
    with patch("loguru.logger") as mock_logger:
        logging_config.flush_logs()
        del logging_config
        mock_logger.remove.assert_called()

# Integration Tests
def test_full_logging_workflow(logging_config, tmp_path):
    """Test complete logging workflow."""
    log_file = tmp_path / "test.log"
    json_file = tmp_path / "test.json"
    
    # Test different logging methods
    with patch("loguru.logger") as mock_logger:
        # Regular logging
        logging_config.batch_log("INFO", "test message")
        
        # Async logging
        asyncio.run(logging_config.alog("DEBUG", "async message"))
        
        # Progress tracking
        with logging_config.progress_context(10, "test") as task_id:
            for _ in range(10):
                logging_config.update_progress(task_id)
        
        # Performance tracking
        with logging_config.performance_tracking("test"):
            pass
        
        # Check final stats
        stats = logging_config.get_stats()
        assert stats["total_messages"] > 0

def test_logger_configuration():
    with mock.patch('rich.logging.RichHandler') as MockRichHandler, \
         mock.patch('rich.console.Console') as MockConsole, \
         mock.patch('loguru.logger.add') as MockLoggerAdd, \
         mock.patch('loguru.logger.remove') as MockLoggerRemove:

        # Re-import the logger module to trigger the configuration
        importlib.reload(
            importlib.import_module('python-check-updates.logging'))

        # Check if logger.remove was called
        MockLoggerRemove.assert_called_once()

        # Check if logger.add was called three times (console, file, structured file)
        assert MockLoggerAdd.call_count == 3

        # Check the arguments for each call to logger.add
        console_call_args = MockLoggerAdd.call_args_list[0]
        file_call_args = MockLoggerAdd.call_args_list[1]
        structured_file_call_args = MockLoggerAdd.call_args_list[2]

        # Verify console logger configuration
        assert console_call_args[1]['sink'] == MockRichHandler.return_value
        assert console_call_args[1]['level'] == "INFO"
        assert console_call_args[1]['format'] == LoggingConfig.DEFAULT_FORMAT

        # Verify file logger configuration
        assert file_call_args[1]['sink'] == "logs/python-check-updates.log"
        assert file_call_args[1]['level'] == "DEBUG"
        assert file_call_args[1]['format'] == LoggingConfig.DEFAULT_FORMAT

        # Verify structured file logger configuration
        assert structured_file_call_args[1]['sink'] == "logs/python-check-updates.json"
        assert structured_file_call_args[1]['level'] == "DEBUG"
        assert structured_file_call_args[1]['format'] == LoggingConfig.DEFAULT_FORMAT

```

## config.yaml

```yaml
# Application Configuration
# ------------------------
# This YAML file contains all application configuration settings.
# Settings can be overridden via environment variables using the APP_ prefix.

# Core application settings
app_name: "python-check-updates"  # Application identifier
version: "0.1.0"                           # Semantic version
debug: false                               # Enable debug mode

# Logging configuration
logging:
  # Basic logging settings
  app_name: "python-check-updates" # Logger identifier
  level: "INFO"                            # Minimum log level
  log_dir: "logs"                          # Log file directory
  
  # Log message format
  format_string: "<green>{time:YYYY-MM-DD at HH:mm:ss.SSS}</green> | <level>{level: <8}</level> | <cyan>{module}</cyan>:<cyan>{function}</cyan>:<cyan>{line}</cyan> | PID: <cyan>{process}</cyan> | TID: <cyan>{thread}</cyan> | <level>{message}</level>"
  
  # Console output configuration
  console:
    enabled: true                          # Enable console logging
    show_time: true                        # Show timestamp
    show_path: true                        # Show file path
    rich_tracebacks: true                  # Use Rich for tracebacks
    traceback_extra_lines: 3               # Context lines in tracebacks
    traceback_theme: "monokai"             # Traceback color theme
  
  file:
    enabled: true
    rotation_size: "100 MB"
    compression: "zip"
    retention_days: 7
    buffer_size: 8192  # Added buffering
  
  json:
    enabled: true
    rotation_size: "100 MB"
    compression: "zip"
    retention_days: 7
  
  batch:
    initial_size: 1000  # Increased for better throughput
    max_size: 10000
    min_size: 100
    check_interval: 30  # Reduced for more responsive adaptation
    high_load_threshold: 75
    low_load_threshold: 25
  
  progress:
    theme: "NEON"     # NEON, MINIMAL, DEFAULT, RAINBOW
    refresh_rate: 10  # Limit screen updates
    transient: true   # Clean up completed bars
    force_terminal: false
    themes:
      neon:
        bar_color: "bright_cyan"
        complete_style:
          color: "bright_green"
          bold: true
        progress_style:
          color: "bright_white"
        spinner_style:
          color: "bright_magenta"
        description_style:
          color: "bright_yellow"
      minimal:
        bar_color: "white"
        complete_style:
          color: "grey70"
        progress_style:
          color: "grey50"
        spinner_style:
          color: "grey85"
        description_style:
          color: "grey74"

  parallel:
    max_workers: 4

# Add other configuration sections as needed
# database:
#   url: "postgresql:

#   pool_size: 5

# api:
#   host: "0.0.0.0"
#   port: 8000

# cache:
#   backend: "redis"
#   url: "redis:

```

## DEVELOPMENT.md

```md
# Development Guide

## Setting Up Your Development Environment

1. **Clone the repository**:

   ```bash
   git clone https:
   cd python-check-updates
   ```
2. **Install dependencies**:

   ```bash
   poetry install
   ```
3. **Activate the development environment**:

   ```bash
   poetry shell
   ```
4. **Run the test suite**:

   ```bash
   poetry run pytest
   ```
5. **Format and lint code**:

   ```bash
   poetry run black .
   poetry run flake8
   ```

## Issue Tracking and Contributions

Please refer to our [Issue Templates](https:

- [Bug Reports](https:
- [Feature Requests](https:

```

## LICENSE

```
MIT License

Copyright (c) 2024 Wyatt Walsh

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

```

## Makefile

```
#  python check updates Makefile

.ONESHELL:

# https:
SHELL = /bin/bash

help:           ## Show this help.
	fgrep -h "##" $(MAKEFILE_LIST) | fgrep -v fgrep | sed -e 's/\\$$


notebook:  ## Launch jupyter lab from project root
	echo "Starting Jupyter Lab..."
	poetry shell && poetry install --with notebook
	poetry run jupyter lab

format: ## Run formatters on the python check updates package
	echo "Running formatters..."
	poetry shell && poetry install
	poetry run isort python check updates/ tests/
	poetry run autoflake --recursive python check updates/ tests/
	poetry run yapf -i --recursive python check updates/ tests/

lint: ## Run linters on the python check updates package
	echo "Running linters..."
	poetry shell && poetry install
	poetry run ruff check python check updates/ tests/
	poetry run mypy python check updates/ tests/
	poetry run pylint python check updates/ tests/

test: ## Run tests on the python check updates package
	echo "Running tests..."
	poetry shell && poetry install
	poetry run pytest tests/
```

## poetry.lock

```lock
# This file is automatically @generated by Poetry 1.8.5 and should not be changed by hand.

[[package]]
name = "annotated-types"
version = "0.7.0"
description = "Reusable constraint types to use with typing.Annotated"
optional = false
python-versions = ">=3.8"
files = [
    {file = "annotated_types-0.7.0-py3-none-any.whl", hash = "sha256:1f02e8b43a8fbbc3f3e0d4f0f4bfc8131bcb4eebe8849b8e5c773f3a1c582a53"},
    {file = "annotated_types-0.7.0.tar.gz", hash = "sha256:aff07c09a53a08bc8cfccb9c85b05f1aa9a2a6f23728d790723543408344ce89"},
]

[[package]]
name = "anyio"
version = "4.7.0"
description = "High level compatibility layer for multiple asynchronous event loop implementations"
optional = false
python-versions = ">=3.9"
files = [
    {file = "anyio-4.7.0-py3-none-any.whl", hash = "sha256:ea60c3723ab42ba6fff7e8ccb0488c898ec538ff4df1f1d5e642c3601d07e352"},
    {file = "anyio-4.7.0.tar.gz", hash = "sha256:2f834749c602966b7d456a7567cafcb309f96482b5081d14ac93ccd457f9dd48"},
]

[package.dependencies]
idna = ">=2.8"
sniffio = ">=1.1"

[package.extras]
doc = ["Sphinx (>=7.4,<8.0)", "packaging", "sphinx-autodoc-typehints (>=1.2.0)", "sphinx_rtd_theme"]
test = ["anyio[trio]", "coverage[toml] (>=7)", "exceptiongroup (>=1.2.0)", "hypothesis (>=4.0)", "psutil (>=5.9)", "pytest (>=7.0)", "pytest-mock (>=3.6.1)", "trustme", "truststore (>=0.9.1)", "uvloop (>=0.21)"]
trio = ["trio (>=0.26.1)"]

[[package]]
name = "appnope"
version = "0.1.4"
description = "Disable App Nap on macOS >= 10.9"
optional = false
python-versions = ">=3.6"
files = [
    {file = "appnope-0.1.4-py2.py3-none-any.whl", hash = "sha256:502575ee11cd7a28c0205f379b525beefebab9d161b7c964670864014ed7213c"},
    {file = "appnope-0.1.4.tar.gz", hash = "sha256:1de3860566df9caf38f01f86f65e0e13e379af54f9e4bee1e66b48f2efffd1ee"},
]

[[package]]
name = "argon2-cffi"
version = "23.1.0"
description = "Argon2 for Python"
optional = false
python-versions = ">=3.7"
files = [
    {file = "argon2_cffi-23.1.0-py3-none-any.whl", hash = "sha256:c670642b78ba29641818ab2e68bd4e6a78ba53b7eff7b4c3815ae16abf91c7ea"},
    {file = "argon2_cffi-23.1.0.tar.gz", hash = "sha256:879c3e79a2729ce768ebb7d36d4609e3a78a4ca2ec3a9f12286ca057e3d0db08"},
]

[package.dependencies]
argon2-cffi-bindings = "*"

[package.extras]
dev = ["argon2-cffi[tests,typing]", "tox (>4)"]
docs = ["furo", "myst-parser", "sphinx", "sphinx-copybutton", "sphinx-notfound-page"]
tests = ["hypothesis", "pytest"]
typing = ["mypy"]

[[package]]
name = "argon2-cffi-bindings"
version = "21.2.0"
description = "Low-level CFFI bindings for Argon2"
optional = false
python-versions = ">=3.6"
files = [
    {file = "argon2-cffi-bindings-21.2.0.tar.gz", hash = "sha256:bb89ceffa6c791807d1305ceb77dbfacc5aa499891d2c55661c6459651fc39e3"},
    {file = "argon2_cffi_bindings-21.2.0-cp36-abi3-macosx_10_9_x86_64.whl", hash = "sha256:ccb949252cb2ab3a08c02024acb77cfb179492d5701c7cbdbfd776124d4d2367"},
    {file = "argon2_cffi_bindings-21.2.0-cp36-abi3-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:9524464572e12979364b7d600abf96181d3541da11e23ddf565a32e70bd4dc0d"},
    {file = "argon2_cffi_bindings-21.2.0-cp36-abi3-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:b746dba803a79238e925d9046a63aa26bf86ab2a2fe74ce6b009a1c3f5c8f2ae"},
    {file = "argon2_cffi_bindings-21.2.0-cp36-abi3-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:58ed19212051f49a523abb1dbe954337dc82d947fb6e5a0da60f7c8471a8476c"},
    {file = "argon2_cffi_bindings-21.2.0-cp36-abi3-musllinux_1_1_aarch64.whl", hash = "sha256:bd46088725ef7f58b5a1ef7ca06647ebaf0eb4baff7d1d0d177c6cc8744abd86"},
    {file = "argon2_cffi_bindings-21.2.0-cp36-abi3-musllinux_1_1_i686.whl", hash = "sha256:8cd69c07dd875537a824deec19f978e0f2078fdda07fd5c42ac29668dda5f40f"},
    {file = "argon2_cffi_bindings-21.2.0-cp36-abi3-musllinux_1_1_x86_64.whl", hash = "sha256:f1152ac548bd5b8bcecfb0b0371f082037e47128653df2e8ba6e914d384f3c3e"},
    {file = "argon2_cffi_bindings-21.2.0-cp36-abi3-win32.whl", hash = "sha256:603ca0aba86b1349b147cab91ae970c63118a0f30444d4bc80355937c950c082"},
    {file = "argon2_cffi_bindings-21.2.0-cp36-abi3-win_amd64.whl", hash = "sha256:b2ef1c30440dbbcba7a5dc3e319408b59676e2e039e2ae11a8775ecf482b192f"},
    {file = "argon2_cffi_bindings-21.2.0-cp38-abi3-macosx_10_9_universal2.whl", hash = "sha256:e415e3f62c8d124ee16018e491a009937f8cf7ebf5eb430ffc5de21b900dad93"},
    {file = "argon2_cffi_bindings-21.2.0-pp37-pypy37_pp73-macosx_10_9_x86_64.whl", hash = "sha256:3e385d1c39c520c08b53d63300c3ecc28622f076f4c2b0e6d7e796e9f6502194"},
    {file = "argon2_cffi_bindings-21.2.0-pp37-pypy37_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:2c3e3cc67fdb7d82c4718f19b4e7a87123caf8a93fde7e23cf66ac0337d3cb3f"},
    {file = "argon2_cffi_bindings-21.2.0-pp37-pypy37_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:6a22ad9800121b71099d0fb0a65323810a15f2e292f2ba450810a7316e128ee5"},
    {file = "argon2_cffi_bindings-21.2.0-pp37-pypy37_pp73-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:f9f8b450ed0547e3d473fdc8612083fd08dd2120d6ac8f73828df9b7d45bb351"},
    {file = "argon2_cffi_bindings-21.2.0-pp37-pypy37_pp73-win_amd64.whl", hash = "sha256:93f9bf70084f97245ba10ee36575f0c3f1e7d7724d67d8e5b08e61787c320ed7"},
    {file = "argon2_cffi_bindings-21.2.0-pp38-pypy38_pp73-macosx_10_9_x86_64.whl", hash = "sha256:3b9ef65804859d335dc6b31582cad2c5166f0c3e7975f324d9ffaa34ee7e6583"},
    {file = "argon2_cffi_bindings-21.2.0-pp38-pypy38_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:d4966ef5848d820776f5f562a7d45fdd70c2f330c961d0d745b784034bd9f48d"},
    {file = "argon2_cffi_bindings-21.2.0-pp38-pypy38_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:20ef543a89dee4db46a1a6e206cd015360e5a75822f76df533845c3cbaf72670"},
    {file = "argon2_cffi_bindings-21.2.0-pp38-pypy38_pp73-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:ed2937d286e2ad0cc79a7087d3c272832865f779430e0cc2b4f3718d3159b0cb"},
    {file = "argon2_cffi_bindings-21.2.0-pp38-pypy38_pp73-win_amd64.whl", hash = "sha256:5e00316dabdaea0b2dd82d141cc66889ced0cdcbfa599e8b471cf22c620c329a"},
]

[package.dependencies]
cffi = ">=1.0.1"

[package.extras]
dev = ["cogapp", "pre-commit", "pytest", "wheel"]
tests = ["pytest"]

[[package]]
name = "arrow"
version = "1.3.0"
description = "Better dates & times for Python"
optional = false
python-versions = ">=3.8"
files = [
    {file = "arrow-1.3.0-py3-none-any.whl", hash = "sha256:c728b120ebc00eb84e01882a6f5e7927a53960aa990ce7dd2b10f39005a67f80"},
    {file = "arrow-1.3.0.tar.gz", hash = "sha256:d4540617648cb5f895730f1ad8c82a65f2dad0166f57b75f3ca54759c4d67a85"},
]

[package.dependencies]
python-dateutil = ">=2.7.0"
types-python-dateutil = ">=2.8.10"

[package.extras]
doc = ["doc8", "sphinx (>=7.0.0)", "sphinx-autobuild", "sphinx-autodoc-typehints", "sphinx_rtd_theme (>=1.3.0)"]
test = ["dateparser (==1.*)", "pre-commit", "pytest", "pytest-cov", "pytest-mock", "pytz (==2021.1)", "simplejson (==3.*)"]

[[package]]
name = "astroid"
version = "3.3.8"
description = "An abstract syntax tree for Python with inference support."
optional = false
python-versions = ">=3.9.0"
files = [
    {file = "astroid-3.3.8-py3-none-any.whl", hash = "sha256:187ccc0c248bfbba564826c26f070494f7bc964fd286b6d9fff4420e55de828c"},
    {file = "astroid-3.3.8.tar.gz", hash = "sha256:a88c7994f914a4ea8572fac479459f4955eeccc877be3f2d959a33273b0cf40b"},
]

[[package]]
name = "asttokens"
version = "3.0.0"
description = "Annotate AST trees with source code positions"
optional = false
python-versions = ">=3.8"
files = [
    {file = "asttokens-3.0.0-py3-none-any.whl", hash = "sha256:e3078351a059199dd5138cb1c706e6430c05eff2ff136af5eb4790f9d28932e2"},
    {file = "asttokens-3.0.0.tar.gz", hash = "sha256:0dcd8baa8d62b0c1d118b399b2ddba3c4aff271d0d7a9e0d4c1681c79035bbc7"},
]

[package.extras]
astroid = ["astroid (>=2,<4)"]
test = ["astroid (>=2,<4)", "pytest", "pytest-cov", "pytest-xdist"]

[[package]]
name = "async-lru"
version = "2.0.4"
description = "Simple LRU cache for asyncio"
optional = false
python-versions = ">=3.8"
files = [
    {file = "async-lru-2.0.4.tar.gz", hash = "sha256:b8a59a5df60805ff63220b2a0c5b5393da5521b113cd5465a44eb037d81a5627"},
    {file = "async_lru-2.0.4-py3-none-any.whl", hash = "sha256:ff02944ce3c288c5be660c42dbcca0742b32c3b279d6dceda655190240b99224"},
]

[[package]]
name = "attrs"
version = "24.3.0"
description = "Classes Without Boilerplate"
optional = false
python-versions = ">=3.8"
files = [
    {file = "attrs-24.3.0-py3-none-any.whl", hash = "sha256:ac96cd038792094f438ad1f6ff80837353805ac950cd2aa0e0625ef19850c308"},
    {file = "attrs-24.3.0.tar.gz", hash = "sha256:8f5c07333d543103541ba7be0e2ce16eeee8130cb0b3f9238ab904ce1e85baff"},
]

[package.extras]
benchmark = ["cloudpickle", "hypothesis", "mypy (>=1.11.1)", "pympler", "pytest (>=4.3.0)", "pytest-codspeed", "pytest-mypy-plugins", "pytest-xdist[psutil]"]
cov = ["cloudpickle", "coverage[toml] (>=5.3)", "hypothesis", "mypy (>=1.11.1)", "pympler", "pytest (>=4.3.0)", "pytest-mypy-plugins", "pytest-xdist[psutil]"]
dev = ["cloudpickle", "hypothesis", "mypy (>=1.11.1)", "pre-commit-uv", "pympler", "pytest (>=4.3.0)", "pytest-mypy-plugins", "pytest-xdist[psutil]"]
docs = ["cogapp", "furo", "myst-parser", "sphinx", "sphinx-notfound-page", "sphinxcontrib-towncrier", "towncrier (<24.7)"]
tests = ["cloudpickle", "hypothesis", "mypy (>=1.11.1)", "pympler", "pytest (>=4.3.0)", "pytest-mypy-plugins", "pytest-xdist[psutil]"]
tests-mypy = ["mypy (>=1.11.1)", "pytest-mypy-plugins"]

[[package]]
name = "autoflake"
version = "2.3.1"
description = "Removes unused imports and unused variables"
optional = false
python-versions = ">=3.8"
files = [
    {file = "autoflake-2.3.1-py3-none-any.whl", hash = "sha256:3ae7495db9084b7b32818b4140e6dc4fc280b712fb414f5b8fe57b0a8e85a840"},
    {file = "autoflake-2.3.1.tar.gz", hash = "sha256:c98b75dc5b0a86459c4f01a1d32ac7eb4338ec4317a4469515ff1e687ecd909e"},
]

[package.dependencies]
pyflakes = ">=3.0.0"

[[package]]
name = "autopep8"
version = "2.3.1"
description = "A tool that automatically formats Python code to conform to the PEP 8 style guide"
optional = false
python-versions = ">=3.8"
files = [
    {file = "autopep8-2.3.1-py2.py3-none-any.whl", hash = "sha256:a203fe0fcad7939987422140ab17a930f684763bf7335bdb6709991dd7ef6c2d"},
    {file = "autopep8-2.3.1.tar.gz", hash = "sha256:8d6c87eba648fdcfc83e29b788910b8643171c395d9c4bcf115ece035b9c9dda"},
]

[package.dependencies]
pycodestyle = ">=2.12.0"

[[package]]
name = "babel"
version = "2.16.0"
description = "Internationalization utilities"
optional = false
python-versions = ">=3.8"
files = [
    {file = "babel-2.16.0-py3-none-any.whl", hash = "sha256:368b5b98b37c06b7daf6696391c3240c938b37767d4584413e8438c5c435fa8b"},
    {file = "babel-2.16.0.tar.gz", hash = "sha256:d1f3554ca26605fe173f3de0c65f750f5a42f924499bf134de6423582298e316"},
]

[package.extras]
dev = ["freezegun (>=1.0,<2.0)", "pytest (>=6.0)", "pytest-cov"]

[[package]]
name = "beautifulsoup4"
version = "4.12.3"
description = "Screen-scraping library"
optional = false
python-versions = ">=3.6.0"
files = [
    {file = "beautifulsoup4-4.12.3-py3-none-any.whl", hash = "sha256:b80878c9f40111313e55da8ba20bdba06d8fa3969fc68304167741bbf9e082ed"},
    {file = "beautifulsoup4-4.12.3.tar.gz", hash = "sha256:74e3d1928edc070d21748185c46e3fb33490f22f52a3addee9aee0f4f7781051"},
]

[package.dependencies]
soupsieve = ">1.2"

[package.extras]
cchardet = ["cchardet"]
chardet = ["chardet"]
charset-normalizer = ["charset-normalizer"]
html5lib = ["html5lib"]
lxml = ["lxml"]

[[package]]
name = "black"
version = "24.10.0"
description = "The uncompromising code formatter."
optional = false
python-versions = ">=3.9"
files = [
    {file = "black-24.10.0-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:e6668650ea4b685440857138e5fe40cde4d652633b1bdffc62933d0db4ed9812"},
    {file = "black-24.10.0-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:1c536fcf674217e87b8cc3657b81809d3c085d7bf3ef262ead700da345bfa6ea"},
    {file = "black-24.10.0-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:649fff99a20bd06c6f727d2a27f401331dc0cc861fb69cde910fe95b01b5928f"},
    {file = "black-24.10.0-cp310-cp310-win_amd64.whl", hash = "sha256:fe4d6476887de70546212c99ac9bd803d90b42fc4767f058a0baa895013fbb3e"},
    {file = "black-24.10.0-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:5a2221696a8224e335c28816a9d331a6c2ae15a2ee34ec857dcf3e45dbfa99ad"},
    {file = "black-24.10.0-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:f9da3333530dbcecc1be13e69c250ed8dfa67f43c4005fb537bb426e19200d50"},
    {file = "black-24.10.0-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:4007b1393d902b48b36958a216c20c4482f601569d19ed1df294a496eb366392"},
    {file = "black-24.10.0-cp311-cp311-win_amd64.whl", hash = "sha256:394d4ddc64782e51153eadcaaca95144ac4c35e27ef9b0a42e121ae7e57a9175"},
    {file = "black-24.10.0-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:b5e39e0fae001df40f95bd8cc36b9165c5e2ea88900167bddf258bacef9bbdc3"},
    {file = "black-24.10.0-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:d37d422772111794b26757c5b55a3eade028aa3fde43121ab7b673d050949d65"},
    {file = "black-24.10.0-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:14b3502784f09ce2443830e3133dacf2c0110d45191ed470ecb04d0f5f6fcb0f"},
    {file = "black-24.10.0-cp312-cp312-win_amd64.whl", hash = "sha256:30d2c30dc5139211dda799758559d1b049f7f14c580c409d6ad925b74a4208a8"},
    {file = "black-24.10.0-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:1cbacacb19e922a1d75ef2b6ccaefcd6e93a2c05ede32f06a21386a04cedb981"},
    {file = "black-24.10.0-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:1f93102e0c5bb3907451063e08b9876dbeac810e7da5a8bfb7aeb5a9ef89066b"},
    {file = "black-24.10.0-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:ddacb691cdcdf77b96f549cf9591701d8db36b2f19519373d60d31746068dbf2"},
    {file = "black-24.10.0-cp313-cp313-win_amd64.whl", hash = "sha256:680359d932801c76d2e9c9068d05c6b107f2584b2a5b88831c83962eb9984c1b"},
    {file = "black-24.10.0-cp39-cp39-macosx_10_9_x86_64.whl", hash = "sha256:17374989640fbca88b6a448129cd1745c5eb8d9547b464f281b251dd00155ccd"},
    {file = "black-24.10.0-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:63f626344343083322233f175aaf372d326de8436f5928c042639a4afbbf1d3f"},
    {file = "black-24.10.0-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:ccfa1d0cb6200857f1923b602f978386a3a2758a65b52e0950299ea014be6800"},
    {file = "black-24.10.0-cp39-cp39-win_amd64.whl", hash = "sha256:2cd9c95431d94adc56600710f8813ee27eea544dd118d45896bb734e9d7a0dc7"},
    {file = "black-24.10.0-py3-none-any.whl", hash = "sha256:3bb2b7a1f7b685f85b11fed1ef10f8a9148bceb49853e47a294a3dd963c1dd7d"},
    {file = "black-24.10.0.tar.gz", hash = "sha256:846ea64c97afe3bc677b761787993be4991810ecc7a4a937816dd6bddedc4875"},
]

[package.dependencies]
click = ">=8.0.0"
mypy-extensions = ">=0.4.3"
packaging = ">=22.0"
pathspec = ">=0.9.0"
platformdirs = ">=2"

[package.extras]
colorama = ["colorama (>=0.4.3)"]
d = ["aiohttp (>=3.10)"]
jupyter = ["ipython (>=7.8.0)", "tokenize-rt (>=3.2.0)"]
uvloop = ["uvloop (>=0.15.2)"]

[[package]]
name = "bleach"
version = "6.2.0"
description = "An easy safelist-based HTML-sanitizing tool."
optional = false
python-versions = ">=3.9"
files = [
    {file = "bleach-6.2.0-py3-none-any.whl", hash = "sha256:117d9c6097a7c3d22fd578fcd8d35ff1e125df6736f554da4e432fdd63f31e5e"},
    {file = "bleach-6.2.0.tar.gz", hash = "sha256:123e894118b8a599fd80d3ec1a6d4cc7ce4e5882b1317a7e1ba69b56e95f991f"},
]

[package.dependencies]
webencodings = "*"

[package.extras]
css = ["tinycss2 (>=1.1.0,<1.5)"]

[[package]]
name = "certifi"
version = "2024.12.14"
description = "Python package for providing Mozilla's CA Bundle."
optional = false
python-versions = ">=3.6"
files = [
    {file = "certifi-2024.12.14-py3-none-any.whl", hash = "sha256:1275f7a45be9464efc1173084eaa30f866fe2e47d389406136d332ed4967ec56"},
    {file = "certifi-2024.12.14.tar.gz", hash = "sha256:b650d30f370c2b724812bee08008be0c4163b163ddaec3f2546c1caf65f191db"},
]

[[package]]
name = "cffi"
version = "1.17.1"
description = "Foreign Function Interface for Python calling C code."
optional = false
python-versions = ">=3.8"
files = [
    {file = "cffi-1.17.1-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:df8b1c11f177bc2313ec4b2d46baec87a5f3e71fc8b45dab2ee7cae86d9aba14"},
    {file = "cffi-1.17.1-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:8f2cdc858323644ab277e9bb925ad72ae0e67f69e804f4898c070998d50b1a67"},
    {file = "cffi-1.17.1-cp310-cp310-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:edae79245293e15384b51f88b00613ba9f7198016a5948b5dddf4917d4d26382"},
    {file = "cffi-1.17.1-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:45398b671ac6d70e67da8e4224a065cec6a93541bb7aebe1b198a61b58c7b702"},
    {file = "cffi-1.17.1-cp310-cp310-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:ad9413ccdeda48c5afdae7e4fa2192157e991ff761e7ab8fdd8926f40b160cc3"},
    {file = "cffi-1.17.1-cp310-cp310-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:5da5719280082ac6bd9aa7becb3938dc9f9cbd57fac7d2871717b1feb0902ab6"},
    {file = "cffi-1.17.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:2bb1a08b8008b281856e5971307cc386a8e9c5b625ac297e853d36da6efe9c17"},
    {file = "cffi-1.17.1-cp310-cp310-musllinux_1_1_aarch64.whl", hash = "sha256:045d61c734659cc045141be4bae381a41d89b741f795af1dd018bfb532fd0df8"},
    {file = "cffi-1.17.1-cp310-cp310-musllinux_1_1_i686.whl", hash = "sha256:6883e737d7d9e4899a8a695e00ec36bd4e5e4f18fabe0aca0efe0a4b44cdb13e"},
    {file = "cffi-1.17.1-cp310-cp310-musllinux_1_1_x86_64.whl", hash = "sha256:6b8b4a92e1c65048ff98cfe1f735ef8f1ceb72e3d5f0c25fdb12087a23da22be"},
    {file = "cffi-1.17.1-cp310-cp310-win32.whl", hash = "sha256:c9c3d058ebabb74db66e431095118094d06abf53284d9c81f27300d0e0d8bc7c"},
    {file = "cffi-1.17.1-cp310-cp310-win_amd64.whl", hash = "sha256:0f048dcf80db46f0098ccac01132761580d28e28bc0f78ae0d58048063317e15"},
    {file = "cffi-1.17.1-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:a45e3c6913c5b87b3ff120dcdc03f6131fa0065027d0ed7ee6190736a74cd401"},
    {file = "cffi-1.17.1-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:30c5e0cb5ae493c04c8b42916e52ca38079f1b235c2f8ae5f4527b963c401caf"},
    {file = "cffi-1.17.1-cp311-cp311-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:f75c7ab1f9e4aca5414ed4d8e5c0e303a34f4421f8a0d47a4d019ceff0ab6af4"},
    {file = "cffi-1.17.1-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:a1ed2dd2972641495a3ec98445e09766f077aee98a1c896dcb4ad0d303628e41"},
    {file = "cffi-1.17.1-cp311-cp311-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:46bf43160c1a35f7ec506d254e5c890f3c03648a4dbac12d624e4490a7046cd1"},
    {file = "cffi-1.17.1-cp311-cp311-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:a24ed04c8ffd54b0729c07cee15a81d964e6fee0e3d4d342a27b020d22959dc6"},
    {file = "cffi-1.17.1-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:610faea79c43e44c71e1ec53a554553fa22321b65fae24889706c0a84d4ad86d"},
    {file = "cffi-1.17.1-cp311-cp311-musllinux_1_1_aarch64.whl", hash = "sha256:a9b15d491f3ad5d692e11f6b71f7857e7835eb677955c00cc0aefcd0669adaf6"},
    {file = "cffi-1.17.1-cp311-cp311-musllinux_1_1_i686.whl", hash = "sha256:de2ea4b5833625383e464549fec1bc395c1bdeeb5f25c4a3a82b5a8c756ec22f"},
    {file = "cffi-1.17.1-cp311-cp311-musllinux_1_1_x86_64.whl", hash = "sha256:fc48c783f9c87e60831201f2cce7f3b2e4846bf4d8728eabe54d60700b318a0b"},
    {file = "cffi-1.17.1-cp311-cp311-win32.whl", hash = "sha256:85a950a4ac9c359340d5963966e3e0a94a676bd6245a4b55bc43949eee26a655"},
    {file = "cffi-1.17.1-cp311-cp311-win_amd64.whl", hash = "sha256:caaf0640ef5f5517f49bc275eca1406b0ffa6aa184892812030f04c2abf589a0"},
    {file = "cffi-1.17.1-cp312-cp312-macosx_10_9_x86_64.whl", hash = "sha256:805b4371bf7197c329fcb3ead37e710d1bca9da5d583f5073b799d5c5bd1eee4"},
    {file = "cffi-1.17.1-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:733e99bc2df47476e3848417c5a4540522f234dfd4ef3ab7fafdf555b082ec0c"},
    {file = "cffi-1.17.1-cp312-cp312-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:1257bdabf294dceb59f5e70c64a3e2f462c30c7ad68092d01bbbfb1c16b1ba36"},
    {file = "cffi-1.17.1-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:da95af8214998d77a98cc14e3a3bd00aa191526343078b530ceb0bd710fb48a5"},
    {file = "cffi-1.17.1-cp312-cp312-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:d63afe322132c194cf832bfec0dc69a99fb9bb6bbd550f161a49e9e855cc78ff"},
    {file = "cffi-1.17.1-cp312-cp312-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:f79fc4fc25f1c8698ff97788206bb3c2598949bfe0fef03d299eb1b5356ada99"},
    {file = "cffi-1.17.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:b62ce867176a75d03a665bad002af8e6d54644fad99a3c70905c543130e39d93"},
    {file = "cffi-1.17.1-cp312-cp312-musllinux_1_1_aarch64.whl", hash = "sha256:386c8bf53c502fff58903061338ce4f4950cbdcb23e2902d86c0f722b786bbe3"},
    {file = "cffi-1.17.1-cp312-cp312-musllinux_1_1_x86_64.whl", hash = "sha256:4ceb10419a9adf4460ea14cfd6bc43d08701f0835e979bf821052f1805850fe8"},
    {file = "cffi-1.17.1-cp312-cp312-win32.whl", hash = "sha256:a08d7e755f8ed21095a310a693525137cfe756ce62d066e53f502a83dc550f65"},
    {file = "cffi-1.17.1-cp312-cp312-win_amd64.whl", hash = "sha256:51392eae71afec0d0c8fb1a53b204dbb3bcabcb3c9b807eedf3e1e6ccf2de903"},
    {file = "cffi-1.17.1-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:f3a2b4222ce6b60e2e8b337bb9596923045681d71e5a082783484d845390938e"},
    {file = "cffi-1.17.1-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:0984a4925a435b1da406122d4d7968dd861c1385afe3b45ba82b750f229811e2"},
    {file = "cffi-1.17.1-cp313-cp313-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:d01b12eeeb4427d3110de311e1774046ad344f5b1a7403101878976ecd7a10f3"},
    {file = "cffi-1.17.1-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:706510fe141c86a69c8ddc029c7910003a17353970cff3b904ff0686a5927683"},
    {file = "cffi-1.17.1-cp313-cp313-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:de55b766c7aa2e2a3092c51e0483d700341182f08e67c63630d5b6f200bb28e5"},
    {file = "cffi-1.17.1-cp313-cp313-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:c59d6e989d07460165cc5ad3c61f9fd8f1b4796eacbd81cee78957842b834af4"},
    {file = "cffi-1.17.1-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:dd398dbc6773384a17fe0d3e7eeb8d1a21c2200473ee6806bb5e6a8e62bb73dd"},
    {file = "cffi-1.17.1-cp313-cp313-musllinux_1_1_aarch64.whl", hash = "sha256:3edc8d958eb099c634dace3c7e16560ae474aa3803a5df240542b305d14e14ed"},
    {file = "cffi-1.17.1-cp313-cp313-musllinux_1_1_x86_64.whl", hash = "sha256:72e72408cad3d5419375fc87d289076ee319835bdfa2caad331e377589aebba9"},
    {file = "cffi-1.17.1-cp313-cp313-win32.whl", hash = "sha256:e03eab0a8677fa80d646b5ddece1cbeaf556c313dcfac435ba11f107ba117b5d"},
    {file = "cffi-1.17.1-cp313-cp313-win_amd64.whl", hash = "sha256:f6a16c31041f09ead72d69f583767292f750d24913dadacf5756b966aacb3f1a"},
    {file = "cffi-1.17.1-cp38-cp38-macosx_10_9_x86_64.whl", hash = "sha256:636062ea65bd0195bc012fea9321aca499c0504409f413dc88af450b57ffd03b"},
    {file = "cffi-1.17.1-cp38-cp38-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:c7eac2ef9b63c79431bc4b25f1cd649d7f061a28808cbc6c47b534bd789ef964"},
    {file = "cffi-1.17.1-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:e221cf152cff04059d011ee126477f0d9588303eb57e88923578ace7baad17f9"},
    {file = "cffi-1.17.1-cp38-cp38-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:31000ec67d4221a71bd3f67df918b1f88f676f1c3b535a7eb473255fdc0b83fc"},
    {file = "cffi-1.17.1-cp38-cp38-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:6f17be4345073b0a7b8ea599688f692ac3ef23ce28e5df79c04de519dbc4912c"},
    {file = "cffi-1.17.1-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:0e2b1fac190ae3ebfe37b979cc1ce69c81f4e4fe5746bb401dca63a9062cdaf1"},
    {file = "cffi-1.17.1-cp38-cp38-win32.whl", hash = "sha256:7596d6620d3fa590f677e9ee430df2958d2d6d6de2feeae5b20e82c00b76fbf8"},
    {file = "cffi-1.17.1-cp38-cp38-win_amd64.whl", hash = "sha256:78122be759c3f8a014ce010908ae03364d00a1f81ab5c7f4a7a5120607ea56e1"},
    {file = "cffi-1.17.1-cp39-cp39-macosx_10_9_x86_64.whl", hash = "sha256:b2ab587605f4ba0bf81dc0cb08a41bd1c0a5906bd59243d56bad7668a6fc6c16"},
    {file = "cffi-1.17.1-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:28b16024becceed8c6dfbc75629e27788d8a3f9030691a1dbf9821a128b22c36"},
    {file = "cffi-1.17.1-cp39-cp39-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:1d599671f396c4723d016dbddb72fe8e0397082b0a77a4fab8028923bec050e8"},
    {file = "cffi-1.17.1-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:ca74b8dbe6e8e8263c0ffd60277de77dcee6c837a3d0881d8c1ead7268c9e576"},
    {file = "cffi-1.17.1-cp39-cp39-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:f7f5baafcc48261359e14bcd6d9bff6d4b28d9103847c9e136694cb0501aef87"},
    {file = "cffi-1.17.1-cp39-cp39-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:98e3969bcff97cae1b2def8ba499ea3d6f31ddfdb7635374834cf89a1a08ecf0"},
    {file = "cffi-1.17.1-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:cdf5ce3acdfd1661132f2a9c19cac174758dc2352bfe37d98aa7512c6b7178b3"},
    {file = "cffi-1.17.1-cp39-cp39-musllinux_1_1_aarch64.whl", hash = "sha256:9755e4345d1ec879e3849e62222a18c7174d65a6a92d5b346b1863912168b595"},
    {file = "cffi-1.17.1-cp39-cp39-musllinux_1_1_i686.whl", hash = "sha256:f1e22e8c4419538cb197e4dd60acc919d7696e5ef98ee4da4e01d3f8cfa4cc5a"},
    {file = "cffi-1.17.1-cp39-cp39-musllinux_1_1_x86_64.whl", hash = "sha256:c03e868a0b3bc35839ba98e74211ed2b05d2119be4e8a0f224fba9384f1fe02e"},
    {file = "cffi-1.17.1-cp39-cp39-win32.whl", hash = "sha256:e31ae45bc2e29f6b2abd0de1cc3b9d5205aa847cafaecb8af1476a609a2f6eb7"},
    {file = "cffi-1.17.1-cp39-cp39-win_amd64.whl", hash = "sha256:d016c76bdd850f3c626af19b0542c9677ba156e4ee4fccfdd7848803533ef662"},
    {file = "cffi-1.17.1.tar.gz", hash = "sha256:1c39c6016c32bc48dd54561950ebd6836e1670f2ae46128f67cf49e789c52824"},
]

[package.dependencies]
pycparser = "*"

[[package]]
name = "charset-normalizer"
version = "3.4.1"
description = "The Real First Universal Charset Detector. Open, modern and actively maintained alternative to Chardet."
optional = false
python-versions = ">=3.7"
files = [
    {file = "charset_normalizer-3.4.1-cp310-cp310-macosx_10_9_universal2.whl", hash = "sha256:91b36a978b5ae0ee86c394f5a54d6ef44db1de0815eb43de826d41d21e4af3de"},
    {file = "charset_normalizer-3.4.1-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:7461baadb4dc00fd9e0acbe254e3d7d2112e7f92ced2adc96e54ef6501c5f176"},
    {file = "charset_normalizer-3.4.1-cp310-cp310-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:e218488cd232553829be0664c2292d3af2eeeb94b32bea483cf79ac6a694e037"},
    {file = "charset_normalizer-3.4.1-cp310-cp310-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:80ed5e856eb7f30115aaf94e4a08114ccc8813e6ed1b5efa74f9f82e8509858f"},
    {file = "charset_normalizer-3.4.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:b010a7a4fd316c3c484d482922d13044979e78d1861f0e0650423144c616a46a"},
    {file = "charset_normalizer-3.4.1-cp310-cp310-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:4532bff1b8421fd0a320463030c7520f56a79c9024a4e88f01c537316019005a"},
    {file = "charset_normalizer-3.4.1-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:d973f03c0cb71c5ed99037b870f2be986c3c05e63622c017ea9816881d2dd247"},
    {file = "charset_normalizer-3.4.1-cp310-cp310-musllinux_1_2_i686.whl", hash = "sha256:3a3bd0dcd373514dcec91c411ddb9632c0d7d92aed7093b8c3bbb6d69ca74408"},
    {file = "charset_normalizer-3.4.1-cp310-cp310-musllinux_1_2_ppc64le.whl", hash = "sha256:d9c3cdf5390dcd29aa8056d13e8e99526cda0305acc038b96b30352aff5ff2bb"},
    {file = "charset_normalizer-3.4.1-cp310-cp310-musllinux_1_2_s390x.whl", hash = "sha256:2bdfe3ac2e1bbe5b59a1a63721eb3b95fc9b6817ae4a46debbb4e11f6232428d"},
    {file = "charset_normalizer-3.4.1-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:eab677309cdb30d047996b36d34caeda1dc91149e4fdca0b1a039b3f79d9a807"},
    {file = "charset_normalizer-3.4.1-cp310-cp310-win32.whl", hash = "sha256:c0429126cf75e16c4f0ad00ee0eae4242dc652290f940152ca8c75c3a4b6ee8f"},
    {file = "charset_normalizer-3.4.1-cp310-cp310-win_amd64.whl", hash = "sha256:9f0b8b1c6d84c8034a44893aba5e767bf9c7a211e313a9605d9c617d7083829f"},
    {file = "charset_normalizer-3.4.1-cp311-cp311-macosx_10_9_universal2.whl", hash = "sha256:8bfa33f4f2672964266e940dd22a195989ba31669bd84629f05fab3ef4e2d125"},
    {file = "charset_normalizer-3.4.1-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:28bf57629c75e810b6ae989f03c0828d64d6b26a5e205535585f96093e405ed1"},
    {file = "charset_normalizer-3.4.1-cp311-cp311-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:f08ff5e948271dc7e18a35641d2f11a4cd8dfd5634f55228b691e62b37125eb3"},
    {file = "charset_normalizer-3.4.1-cp311-cp311-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:234ac59ea147c59ee4da87a0c0f098e9c8d169f4dc2a159ef720f1a61bbe27cd"},
    {file = "charset_normalizer-3.4.1-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:fd4ec41f914fa74ad1b8304bbc634b3de73d2a0889bd32076342a573e0779e00"},
    {file = "charset_normalizer-3.4.1-cp311-cp311-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:eea6ee1db730b3483adf394ea72f808b6e18cf3cb6454b4d86e04fa8c4327a12"},
    {file = "charset_normalizer-3.4.1-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:c96836c97b1238e9c9e3fe90844c947d5afbf4f4c92762679acfe19927d81d77"},
    {file = "charset_normalizer-3.4.1-cp311-cp311-musllinux_1_2_i686.whl", hash = "sha256:4d86f7aff21ee58f26dcf5ae81a9addbd914115cdebcbb2217e4f0ed8982e146"},
    {file = "charset_normalizer-3.4.1-cp311-cp311-musllinux_1_2_ppc64le.whl", hash = "sha256:09b5e6733cbd160dcc09589227187e242a30a49ca5cefa5a7edd3f9d19ed53fd"},
    {file = "charset_normalizer-3.4.1-cp311-cp311-musllinux_1_2_s390x.whl", hash = "sha256:5777ee0881f9499ed0f71cc82cf873d9a0ca8af166dfa0af8ec4e675b7df48e6"},
    {file = "charset_normalizer-3.4.1-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:237bdbe6159cff53b4f24f397d43c6336c6b0b42affbe857970cefbb620911c8"},
    {file = "charset_normalizer-3.4.1-cp311-cp311-win32.whl", hash = "sha256:8417cb1f36cc0bc7eaba8ccb0e04d55f0ee52df06df3ad55259b9a323555fc8b"},
    {file = "charset_normalizer-3.4.1-cp311-cp311-win_amd64.whl", hash = "sha256:d7f50a1f8c450f3925cb367d011448c39239bb3eb4117c36a6d354794de4ce76"},
    {file = "charset_normalizer-3.4.1-cp312-cp312-macosx_10_13_universal2.whl", hash = "sha256:73d94b58ec7fecbc7366247d3b0b10a21681004153238750bb67bd9012414545"},
    {file = "charset_normalizer-3.4.1-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:dad3e487649f498dd991eeb901125411559b22e8d7ab25d3aeb1af367df5efd7"},
    {file = "charset_normalizer-3.4.1-cp312-cp312-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:c30197aa96e8eed02200a83fba2657b4c3acd0f0aa4bdc9f6c1af8e8962e0757"},
    {file = "charset_normalizer-3.4.1-cp312-cp312-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:2369eea1ee4a7610a860d88f268eb39b95cb588acd7235e02fd5a5601773d4fa"},
    {file = "charset_normalizer-3.4.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:bc2722592d8998c870fa4e290c2eec2c1569b87fe58618e67d38b4665dfa680d"},
    {file = "charset_normalizer-3.4.1-cp312-cp312-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:ffc9202a29ab3920fa812879e95a9e78b2465fd10be7fcbd042899695d75e616"},
    {file = "charset_normalizer-3.4.1-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:804a4d582ba6e5b747c625bf1255e6b1507465494a40a2130978bda7b932c90b"},
    {file = "charset_normalizer-3.4.1-cp312-cp312-musllinux_1_2_i686.whl", hash = "sha256:0f55e69f030f7163dffe9fd0752b32f070566451afe180f99dbeeb81f511ad8d"},
    {file = "charset_normalizer-3.4.1-cp312-cp312-musllinux_1_2_ppc64le.whl", hash = "sha256:c4c3e6da02df6fa1410a7680bd3f63d4f710232d3139089536310d027950696a"},
    {file = "charset_normalizer-3.4.1-cp312-cp312-musllinux_1_2_s390x.whl", hash = "sha256:5df196eb874dae23dcfb968c83d4f8fdccb333330fe1fc278ac5ceeb101003a9"},
    {file = "charset_normalizer-3.4.1-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:e358e64305fe12299a08e08978f51fc21fac060dcfcddd95453eabe5b93ed0e1"},
    {file = "charset_normalizer-3.4.1-cp312-cp312-win32.whl", hash = "sha256:9b23ca7ef998bc739bf6ffc077c2116917eabcc901f88da1b9856b210ef63f35"},
    {file = "charset_normalizer-3.4.1-cp312-cp312-win_amd64.whl", hash = "sha256:6ff8a4a60c227ad87030d76e99cd1698345d4491638dfa6673027c48b3cd395f"},
    {file = "charset_normalizer-3.4.1-cp313-cp313-macosx_10_13_universal2.whl", hash = "sha256:aabfa34badd18f1da5ec1bc2715cadc8dca465868a4e73a0173466b688f29dda"},
    {file = "charset_normalizer-3.4.1-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:22e14b5d70560b8dd51ec22863f370d1e595ac3d024cb8ad7d308b4cd95f8313"},
    {file = "charset_normalizer-3.4.1-cp313-cp313-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:8436c508b408b82d87dc5f62496973a1805cd46727c34440b0d29d8a2f50a6c9"},
    {file = "charset_normalizer-3.4.1-cp313-cp313-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:2d074908e1aecee37a7635990b2c6d504cd4766c7bc9fc86d63f9c09af3fa11b"},
    {file = "charset_normalizer-3.4.1-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:955f8851919303c92343d2f66165294848d57e9bba6cf6e3625485a70a038d11"},
    {file = "charset_normalizer-3.4.1-cp313-cp313-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:44ecbf16649486d4aebafeaa7ec4c9fed8b88101f4dd612dcaf65d5e815f837f"},
    {file = "charset_normalizer-3.4.1-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:0924e81d3d5e70f8126529951dac65c1010cdf117bb75eb02dd12339b57749dd"},
    {file = "charset_normalizer-3.4.1-cp313-cp313-musllinux_1_2_i686.whl", hash = "sha256:2967f74ad52c3b98de4c3b32e1a44e32975e008a9cd2a8cc8966d6a5218c5cb2"},
    {file = "charset_normalizer-3.4.1-cp313-cp313-musllinux_1_2_ppc64le.whl", hash = "sha256:c75cb2a3e389853835e84a2d8fb2b81a10645b503eca9bcb98df6b5a43eb8886"},
    {file = "charset_normalizer-3.4.1-cp313-cp313-musllinux_1_2_s390x.whl", hash = "sha256:09b26ae6b1abf0d27570633b2b078a2a20419c99d66fb2823173d73f188ce601"},
    {file = "charset_normalizer-3.4.1-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:fa88b843d6e211393a37219e6a1c1df99d35e8fd90446f1118f4216e307e48cd"},
    {file = "charset_normalizer-3.4.1-cp313-cp313-win32.whl", hash = "sha256:eb8178fe3dba6450a3e024e95ac49ed3400e506fd4e9e5c32d30adda88cbd407"},
    {file = "charset_normalizer-3.4.1-cp313-cp313-win_amd64.whl", hash = "sha256:b1ac5992a838106edb89654e0aebfc24f5848ae2547d22c2c3f66454daa11971"},
    {file = "charset_normalizer-3.4.1-cp37-cp37m-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:f30bf9fd9be89ecb2360c7d94a711f00c09b976258846efe40db3d05828e8089"},
    {file = "charset_normalizer-3.4.1-cp37-cp37m-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:97f68b8d6831127e4787ad15e6757232e14e12060bec17091b85eb1486b91d8d"},
    {file = "charset_normalizer-3.4.1-cp37-cp37m-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:7974a0b5ecd505609e3b19742b60cee7aa2aa2fb3151bc917e6e2646d7667dcf"},
    {file = "charset_normalizer-3.4.1-cp37-cp37m-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:fc54db6c8593ef7d4b2a331b58653356cf04f67c960f584edb7c3d8c97e8f39e"},
    {file = "charset_normalizer-3.4.1-cp37-cp37m-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:311f30128d7d333eebd7896965bfcfbd0065f1716ec92bd5638d7748eb6f936a"},
    {file = "charset_normalizer-3.4.1-cp37-cp37m-musllinux_1_2_aarch64.whl", hash = "sha256:7d053096f67cd1241601111b698f5cad775f97ab25d81567d3f59219b5f1adbd"},
    {file = "charset_normalizer-3.4.1-cp37-cp37m-musllinux_1_2_i686.whl", hash = "sha256:807f52c1f798eef6cf26beb819eeb8819b1622ddfeef9d0977a8502d4db6d534"},
    {file = "charset_normalizer-3.4.1-cp37-cp37m-musllinux_1_2_ppc64le.whl", hash = "sha256:dccbe65bd2f7f7ec22c4ff99ed56faa1e9f785482b9bbd7c717e26fd723a1d1e"},
    {file = "charset_normalizer-3.4.1-cp37-cp37m-musllinux_1_2_s390x.whl", hash = "sha256:2fb9bd477fdea8684f78791a6de97a953c51831ee2981f8e4f583ff3b9d9687e"},
    {file = "charset_normalizer-3.4.1-cp37-cp37m-musllinux_1_2_x86_64.whl", hash = "sha256:01732659ba9b5b873fc117534143e4feefecf3b2078b0a6a2e925271bb6f4cfa"},
    {file = "charset_normalizer-3.4.1-cp37-cp37m-win32.whl", hash = "sha256:7a4f97a081603d2050bfaffdefa5b02a9ec823f8348a572e39032caa8404a487"},
    {file = "charset_normalizer-3.4.1-cp37-cp37m-win_amd64.whl", hash = "sha256:7b1bef6280950ee6c177b326508f86cad7ad4dff12454483b51d8b7d673a2c5d"},
    {file = "charset_normalizer-3.4.1-cp38-cp38-macosx_10_9_universal2.whl", hash = "sha256:ecddf25bee22fe4fe3737a399d0d177d72bc22be6913acfab364b40bce1ba83c"},
    {file = "charset_normalizer-3.4.1-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:8c60ca7339acd497a55b0ea5d506b2a2612afb2826560416f6894e8b5770d4a9"},
    {file = "charset_normalizer-3.4.1-cp38-cp38-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:b7b2d86dd06bfc2ade3312a83a5c364c7ec2e3498f8734282c6c3d4b07b346b8"},
    {file = "charset_normalizer-3.4.1-cp38-cp38-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:dd78cfcda14a1ef52584dbb008f7ac81c1328c0f58184bf9a84c49c605002da6"},
    {file = "charset_normalizer-3.4.1-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:6e27f48bcd0957c6d4cb9d6fa6b61d192d0b13d5ef563e5f2ae35feafc0d179c"},
    {file = "charset_normalizer-3.4.1-cp38-cp38-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:01ad647cdd609225c5350561d084b42ddf732f4eeefe6e678765636791e78b9a"},
    {file = "charset_normalizer-3.4.1-cp38-cp38-musllinux_1_2_aarch64.whl", hash = "sha256:619a609aa74ae43d90ed2e89bdd784765de0a25ca761b93e196d938b8fd1dbbd"},
    {file = "charset_normalizer-3.4.1-cp38-cp38-musllinux_1_2_i686.whl", hash = "sha256:89149166622f4db9b4b6a449256291dc87a99ee53151c74cbd82a53c8c2f6ccd"},
    {file = "charset_normalizer-3.4.1-cp38-cp38-musllinux_1_2_ppc64le.whl", hash = "sha256:7709f51f5f7c853f0fb938bcd3bc59cdfdc5203635ffd18bf354f6967ea0f824"},
    {file = "charset_normalizer-3.4.1-cp38-cp38-musllinux_1_2_s390x.whl", hash = "sha256:345b0426edd4e18138d6528aed636de7a9ed169b4aaf9d61a8c19e39d26838ca"},
    {file = "charset_normalizer-3.4.1-cp38-cp38-musllinux_1_2_x86_64.whl", hash = "sha256:0907f11d019260cdc3f94fbdb23ff9125f6b5d1039b76003b5b0ac9d6a6c9d5b"},
    {file = "charset_normalizer-3.4.1-cp38-cp38-win32.whl", hash = "sha256:ea0d8d539afa5eb2728aa1932a988a9a7af94f18582ffae4bc10b3fbdad0626e"},
    {file = "charset_normalizer-3.4.1-cp38-cp38-win_amd64.whl", hash = "sha256:329ce159e82018d646c7ac45b01a430369d526569ec08516081727a20e9e4af4"},
    {file = "charset_normalizer-3.4.1-cp39-cp39-macosx_10_9_universal2.whl", hash = "sha256:b97e690a2118911e39b4042088092771b4ae3fc3aa86518f84b8cf6888dbdb41"},
    {file = "charset_normalizer-3.4.1-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:78baa6d91634dfb69ec52a463534bc0df05dbd546209b79a3880a34487f4b84f"},
    {file = "charset_normalizer-3.4.1-cp39-cp39-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:1a2bc9f351a75ef49d664206d51f8e5ede9da246602dc2d2726837620ea034b2"},
    {file = "charset_normalizer-3.4.1-cp39-cp39-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:75832c08354f595c760a804588b9357d34ec00ba1c940c15e31e96d902093770"},
    {file = "charset_normalizer-3.4.1-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:0af291f4fe114be0280cdd29d533696a77b5b49cfde5467176ecab32353395c4"},
    {file = "charset_normalizer-3.4.1-cp39-cp39-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:0167ddc8ab6508fe81860a57dd472b2ef4060e8d378f0cc555707126830f2537"},
    {file = "charset_normalizer-3.4.1-cp39-cp39-musllinux_1_2_aarch64.whl", hash = "sha256:2a75d49014d118e4198bcee5ee0a6f25856b29b12dbf7cd012791f8a6cc5c496"},
    {file = "charset_normalizer-3.4.1-cp39-cp39-musllinux_1_2_i686.whl", hash = "sha256:363e2f92b0f0174b2f8238240a1a30142e3db7b957a5dd5689b0e75fb717cc78"},
    {file = "charset_normalizer-3.4.1-cp39-cp39-musllinux_1_2_ppc64le.whl", hash = "sha256:ab36c8eb7e454e34e60eb55ca5d241a5d18b2c6244f6827a30e451c42410b5f7"},
    {file = "charset_normalizer-3.4.1-cp39-cp39-musllinux_1_2_s390x.whl", hash = "sha256:4c0907b1928a36d5a998d72d64d8eaa7244989f7aaaf947500d3a800c83a3fd6"},
    {file = "charset_normalizer-3.4.1-cp39-cp39-musllinux_1_2_x86_64.whl", hash = "sha256:04432ad9479fa40ec0f387795ddad4437a2b50417c69fa275e212933519ff294"},
    {file = "charset_normalizer-3.4.1-cp39-cp39-win32.whl", hash = "sha256:3bed14e9c89dcb10e8f3a29f9ccac4955aebe93c71ae803af79265c9ca5644c5"},
    {file = "charset_normalizer-3.4.1-cp39-cp39-win_amd64.whl", hash = "sha256:49402233c892a461407c512a19435d1ce275543138294f7ef013f0b63d5d3765"},
    {file = "charset_normalizer-3.4.1-py3-none-any.whl", hash = "sha256:d98b1668f06378c6dbefec3b92299716b931cd4e6061f3c875a71ced1780ab85"},
    {file = "charset_normalizer-3.4.1.tar.gz", hash = "sha256:44251f18cd68a75b56585dd00dae26183e102cd5e0f9f1466e6df5da2ed64ea3"},
]

[[package]]
name = "click"
version = "8.1.8"
description = "Composable command line interface toolkit"
optional = false
python-versions = ">=3.7"
files = [
    {file = "click-8.1.8-py3-none-any.whl", hash = "sha256:63c132bbbed01578a06712a2d1f497bb62d9c1c0d329b7903a866228027263b2"},
    {file = "click-8.1.8.tar.gz", hash = "sha256:ed53c9d8990d83c2a27deae68e4ee337473f6330c040a31d4225c9574d16096a"},
]

[package.dependencies]
colorama = {version = "*", markers = "platform_system == \"Windows\""}

[[package]]
name = "colorama"
version = "0.4.6"
description = "Cross-platform colored terminal text."
optional = false
python-versions = "!=3.0.*,!=3.1.*,!=3.2.*,!=3.3.*,!=3.4.*,!=3.5.*,!=3.6.*,>=2.7"
files = [
    {file = "colorama-0.4.6-py2.py3-none-any.whl", hash = "sha256:4f1d9991f5acc0ca119f9d443620b77f9d6b33703e51011c16baf57afb285fc6"},
    {file = "colorama-0.4.6.tar.gz", hash = "sha256:08695f5cb7ed6e0531a20572697297273c47b8cae5a63ffc6d6ed5c201be6e44"},
]

[[package]]
name = "comm"
version = "0.2.2"
description = "Jupyter Python Comm implementation, for usage in ipykernel, xeus-python etc."
optional = false
python-versions = ">=3.8"
files = [
    {file = "comm-0.2.2-py3-none-any.whl", hash = "sha256:e6fb86cb70ff661ee8c9c14e7d36d6de3b4066f1441be4063df9c5009f0a64d3"},
    {file = "comm-0.2.2.tar.gz", hash = "sha256:3fd7a84065306e07bea1773df6eb8282de51ba82f77c72f9c85716ab11fe980e"},
]

[package.dependencies]
traitlets = ">=4"

[package.extras]
test = ["pytest"]

[[package]]
name = "contourpy"
version = "1.3.1"
description = "Python library for calculating contours of 2D quadrilateral grids"
optional = false
python-versions = ">=3.10"
files = [
    {file = "contourpy-1.3.1-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:a045f341a77b77e1c5de31e74e966537bba9f3c4099b35bf4c2e3939dd54cdab"},
    {file = "contourpy-1.3.1-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:500360b77259914f7805af7462e41f9cb7ca92ad38e9f94d6c8641b089338124"},
    {file = "contourpy-1.3.1-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:b2f926efda994cdf3c8d3fdb40b9962f86edbc4457e739277b961eced3d0b4c1"},
    {file = "contourpy-1.3.1-cp310-cp310-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:adce39d67c0edf383647a3a007de0a45fd1b08dedaa5318404f1a73059c2512b"},
    {file = "contourpy-1.3.1-cp310-cp310-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:abbb49fb7dac584e5abc6636b7b2a7227111c4f771005853e7d25176daaf8453"},
    {file = "contourpy-1.3.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:a0cffcbede75c059f535725c1680dfb17b6ba8753f0c74b14e6a9c68c29d7ea3"},
    {file = "contourpy-1.3.1-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:ab29962927945d89d9b293eabd0d59aea28d887d4f3be6c22deaefbb938a7277"},
    {file = "contourpy-1.3.1-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:974d8145f8ca354498005b5b981165b74a195abfae9a8129df3e56771961d595"},
    {file = "contourpy-1.3.1-cp310-cp310-win32.whl", hash = "sha256:ac4578ac281983f63b400f7fe6c101bedc10651650eef012be1ccffcbacf3697"},
    {file = "contourpy-1.3.1-cp310-cp310-win_amd64.whl", hash = "sha256:174e758c66bbc1c8576992cec9599ce8b6672b741b5d336b5c74e35ac382b18e"},
    {file = "contourpy-1.3.1-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:3e8b974d8db2c5610fb4e76307e265de0edb655ae8169e8b21f41807ccbeec4b"},
    {file = "contourpy-1.3.1-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:20914c8c973f41456337652a6eeca26d2148aa96dd7ac323b74516988bea89fc"},
    {file = "contourpy-1.3.1-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:19d40d37c1c3a4961b4619dd9d77b12124a453cc3d02bb31a07d58ef684d3d86"},
    {file = "contourpy-1.3.1-cp311-cp311-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:113231fe3825ebf6f15eaa8bc1f5b0ddc19d42b733345eae0934cb291beb88b6"},
    {file = "contourpy-1.3.1-cp311-cp311-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:4dbbc03a40f916a8420e420d63e96a1258d3d1b58cbdfd8d1f07b49fcbd38e85"},
    {file = "contourpy-1.3.1-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:3a04ecd68acbd77fa2d39723ceca4c3197cb2969633836ced1bea14e219d077c"},
    {file = "contourpy-1.3.1-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:c414fc1ed8ee1dbd5da626cf3710c6013d3d27456651d156711fa24f24bd1291"},
    {file = "contourpy-1.3.1-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:31c1b55c1f34f80557d3830d3dd93ba722ce7e33a0b472cba0ec3b6535684d8f"},
    {file = "contourpy-1.3.1-cp311-cp311-win32.whl", hash = "sha256:f611e628ef06670df83fce17805c344710ca5cde01edfdc72751311da8585375"},
    {file = "contourpy-1.3.1-cp311-cp311-win_amd64.whl", hash = "sha256:b2bdca22a27e35f16794cf585832e542123296b4687f9fd96822db6bae17bfc9"},
    {file = "contourpy-1.3.1-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:0ffa84be8e0bd33410b17189f7164c3589c229ce5db85798076a3fa136d0e509"},
    {file = "contourpy-1.3.1-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:805617228ba7e2cbbfb6c503858e626ab528ac2a32a04a2fe88ffaf6b02c32bc"},
    {file = "contourpy-1.3.1-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:ade08d343436a94e633db932e7e8407fe7de8083967962b46bdfc1b0ced39454"},
    {file = "contourpy-1.3.1-cp312-cp312-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:47734d7073fb4590b4a40122b35917cd77be5722d80683b249dac1de266aac80"},
    {file = "contourpy-1.3.1-cp312-cp312-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:2ba94a401342fc0f8b948e57d977557fbf4d515f03c67682dd5c6191cb2d16ec"},
    {file = "contourpy-1.3.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:efa874e87e4a647fd2e4f514d5e91c7d493697127beb95e77d2f7561f6905bd9"},
    {file = "contourpy-1.3.1-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:1bf98051f1045b15c87868dbaea84f92408337d4f81d0e449ee41920ea121d3b"},
    {file = "contourpy-1.3.1-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:61332c87493b00091423e747ea78200659dc09bdf7fd69edd5e98cef5d3e9a8d"},
    {file = "contourpy-1.3.1-cp312-cp312-win32.whl", hash = "sha256:e914a8cb05ce5c809dd0fe350cfbb4e881bde5e2a38dc04e3afe1b3e58bd158e"},
    {file = "contourpy-1.3.1-cp312-cp312-win_amd64.whl", hash = "sha256:08d9d449a61cf53033612cb368f3a1b26cd7835d9b8cd326647efe43bca7568d"},
    {file = "contourpy-1.3.1-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:a761d9ccfc5e2ecd1bf05534eda382aa14c3e4f9205ba5b1684ecfe400716ef2"},
    {file = "contourpy-1.3.1-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:523a8ee12edfa36f6d2a49407f705a6ef4c5098de4f498619787e272de93f2d5"},
    {file = "contourpy-1.3.1-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:ece6df05e2c41bd46776fbc712e0996f7c94e0d0543af1656956d150c4ca7c81"},
    {file = "contourpy-1.3.1-cp313-cp313-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:573abb30e0e05bf31ed067d2f82500ecfdaec15627a59d63ea2d95714790f5c2"},
    {file = "contourpy-1.3.1-cp313-cp313-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:a9fa36448e6a3a1a9a2ba23c02012c43ed88905ec80163f2ffe2421c7192a5d7"},
    {file = "contourpy-1.3.1-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:3ea9924d28fc5586bf0b42d15f590b10c224117e74409dd7a0be3b62b74a501c"},
    {file = "contourpy-1.3.1-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:5b75aa69cb4d6f137b36f7eb2ace9280cfb60c55dc5f61c731fdf6f037f958a3"},
    {file = "contourpy-1.3.1-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:041b640d4ec01922083645a94bb3b2e777e6b626788f4095cf21abbe266413c1"},
    {file = "contourpy-1.3.1-cp313-cp313-win32.whl", hash = "sha256:36987a15e8ace5f58d4d5da9dca82d498c2bbb28dff6e5d04fbfcc35a9cb3a82"},
    {file = "contourpy-1.3.1-cp313-cp313-win_amd64.whl", hash = "sha256:a7895f46d47671fa7ceec40f31fae721da51ad34bdca0bee83e38870b1f47ffd"},
    {file = "contourpy-1.3.1-cp313-cp313t-macosx_10_13_x86_64.whl", hash = "sha256:9ddeb796389dadcd884c7eb07bd14ef12408aaae358f0e2ae24114d797eede30"},
    {file = "contourpy-1.3.1-cp313-cp313t-macosx_11_0_arm64.whl", hash = "sha256:19c1555a6801c2f084c7ddc1c6e11f02eb6a6016ca1318dd5452ba3f613a1751"},
    {file = "contourpy-1.3.1-cp313-cp313t-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:841ad858cff65c2c04bf93875e384ccb82b654574a6d7f30453a04f04af71342"},
    {file = "contourpy-1.3.1-cp313-cp313t-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:4318af1c925fb9a4fb190559ef3eec206845f63e80fb603d47f2d6d67683901c"},
    {file = "contourpy-1.3.1-cp313-cp313t-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:14c102b0eab282427b662cb590f2e9340a9d91a1c297f48729431f2dcd16e14f"},
    {file = "contourpy-1.3.1-cp313-cp313t-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:05e806338bfeaa006acbdeba0ad681a10be63b26e1b17317bfac3c5d98f36cda"},
    {file = "contourpy-1.3.1-cp313-cp313t-musllinux_1_2_aarch64.whl", hash = "sha256:4d76d5993a34ef3df5181ba3c92fabb93f1eaa5729504fb03423fcd9f3177242"},
    {file = "contourpy-1.3.1-cp313-cp313t-musllinux_1_2_x86_64.whl", hash = "sha256:89785bb2a1980c1bd87f0cb1517a71cde374776a5f150936b82580ae6ead44a1"},
    {file = "contourpy-1.3.1-cp313-cp313t-win32.whl", hash = "sha256:8eb96e79b9f3dcadbad2a3891672f81cdcab7f95b27f28f1c67d75f045b6b4f1"},
    {file = "contourpy-1.3.1-cp313-cp313t-win_amd64.whl", hash = "sha256:287ccc248c9e0d0566934e7d606201abd74761b5703d804ff3df8935f523d546"},
    {file = "contourpy-1.3.1-pp310-pypy310_pp73-macosx_10_15_x86_64.whl", hash = "sha256:b457d6430833cee8e4b8e9b6f07aa1c161e5e0d52e118dc102c8f9bd7dd060d6"},
    {file = "contourpy-1.3.1-pp310-pypy310_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:cb76c1a154b83991a3cbbf0dfeb26ec2833ad56f95540b442c73950af2013750"},
    {file = "contourpy-1.3.1-pp310-pypy310_pp73-win_amd64.whl", hash = "sha256:44a29502ca9c7b5ba389e620d44f2fbe792b1fb5734e8b931ad307071ec58c53"},
    {file = "contourpy-1.3.1.tar.gz", hash = "sha256:dfd97abd83335045a913e3bcc4a09c0ceadbe66580cf573fe961f4a825efa699"},
]

[package.dependencies]
numpy = ">=1.23"

[package.extras]
bokeh = ["bokeh", "selenium"]
docs = ["furo", "sphinx (>=7.2)", "sphinx-copybutton"]
mypy = ["contourpy[bokeh,docs]", "docutils-stubs", "mypy (==1.11.1)", "types-Pillow"]
test = ["Pillow", "contourpy[test-no-images]", "matplotlib"]
test-no-images = ["pytest", "pytest-cov", "pytest-rerunfailures", "pytest-xdist", "wurlitzer"]

[[package]]
name = "coverage"
version = "7.6.9"
description = "Code coverage measurement for Python"
optional = false
python-versions = ">=3.9"
files = [
    {file = "coverage-7.6.9-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:85d9636f72e8991a1706b2b55b06c27545448baf9f6dbf51c4004609aacd7dcb"},
    {file = "coverage-7.6.9-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:608a7fd78c67bee8936378299a6cb9f5149bb80238c7a566fc3e6717a4e68710"},
    {file = "coverage-7.6.9-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:96d636c77af18b5cb664ddf12dab9b15a0cfe9c0bde715da38698c8cea748bfa"},
    {file = "coverage-7.6.9-cp310-cp310-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:d75cded8a3cff93da9edc31446872d2997e327921d8eed86641efafd350e1df1"},
    {file = "coverage-7.6.9-cp310-cp310-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:f7b15f589593110ae767ce997775d645b47e5cbbf54fd322f8ebea6277466cec"},
    {file = "coverage-7.6.9-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:44349150f6811b44b25574839b39ae35291f6496eb795b7366fef3bd3cf112d3"},
    {file = "coverage-7.6.9-cp310-cp310-musllinux_1_2_i686.whl", hash = "sha256:d891c136b5b310d0e702e186d70cd16d1119ea8927347045124cb286b29297e5"},
    {file = "coverage-7.6.9-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:db1dab894cc139f67822a92910466531de5ea6034ddfd2b11c0d4c6257168073"},
    {file = "coverage-7.6.9-cp310-cp310-win32.whl", hash = "sha256:41ff7b0da5af71a51b53f501a3bac65fb0ec311ebed1632e58fc6107f03b9198"},
    {file = "coverage-7.6.9-cp310-cp310-win_amd64.whl", hash = "sha256:35371f8438028fdccfaf3570b31d98e8d9eda8bb1d6ab9473f5a390969e98717"},
    {file = "coverage-7.6.9-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:932fc826442132dde42ee52cf66d941f581c685a6313feebed358411238f60f9"},
    {file = "coverage-7.6.9-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:085161be5f3b30fd9b3e7b9a8c301f935c8313dcf928a07b116324abea2c1c2c"},
    {file = "coverage-7.6.9-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:ccc660a77e1c2bf24ddbce969af9447a9474790160cfb23de6be4fa88e3951c7"},
    {file = "coverage-7.6.9-cp311-cp311-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:c69e42c892c018cd3c8d90da61d845f50a8243062b19d228189b0224150018a9"},
    {file = "coverage-7.6.9-cp311-cp311-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:0824a28ec542a0be22f60c6ac36d679e0e262e5353203bea81d44ee81fe9c6d4"},
    {file = "coverage-7.6.9-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:4401ae5fc52ad8d26d2a5d8a7428b0f0c72431683f8e63e42e70606374c311a1"},
    {file = "coverage-7.6.9-cp311-cp311-musllinux_1_2_i686.whl", hash = "sha256:98caba4476a6c8d59ec1eb00c7dd862ba9beca34085642d46ed503cc2d440d4b"},
    {file = "coverage-7.6.9-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:ee5defd1733fd6ec08b168bd4f5387d5b322f45ca9e0e6c817ea6c4cd36313e3"},
    {file = "coverage-7.6.9-cp311-cp311-win32.whl", hash = "sha256:f2d1ec60d6d256bdf298cb86b78dd715980828f50c46701abc3b0a2b3f8a0dc0"},
    {file = "coverage-7.6.9-cp311-cp311-win_amd64.whl", hash = "sha256:0d59fd927b1f04de57a2ba0137166d31c1a6dd9e764ad4af552912d70428c92b"},
    {file = "coverage-7.6.9-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:99e266ae0b5d15f1ca8d278a668df6f51cc4b854513daab5cae695ed7b721cf8"},
    {file = "coverage-7.6.9-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:9901d36492009a0a9b94b20e52ebfc8453bf49bb2b27bca2c9706f8b4f5a554a"},
    {file = "coverage-7.6.9-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:abd3e72dd5b97e3af4246cdada7738ef0e608168de952b837b8dd7e90341f015"},
    {file = "coverage-7.6.9-cp312-cp312-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:ff74026a461eb0660366fb01c650c1d00f833a086b336bdad7ab00cc952072b3"},
    {file = "coverage-7.6.9-cp312-cp312-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:65dad5a248823a4996724a88eb51d4b31587aa7aa428562dbe459c684e5787ae"},
    {file = "coverage-7.6.9-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:22be16571504c9ccea919fcedb459d5ab20d41172056206eb2994e2ff06118a4"},
    {file = "coverage-7.6.9-cp312-cp312-musllinux_1_2_i686.whl", hash = "sha256:0f957943bc718b87144ecaee70762bc2bc3f1a7a53c7b861103546d3a403f0a6"},
    {file = "coverage-7.6.9-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:0ae1387db4aecb1f485fb70a6c0148c6cdaebb6038f1d40089b1fc84a5db556f"},
    {file = "coverage-7.6.9-cp312-cp312-win32.whl", hash = "sha256:1a330812d9cc7ac2182586f6d41b4d0fadf9be9049f350e0efb275c8ee8eb692"},
    {file = "coverage-7.6.9-cp312-cp312-win_amd64.whl", hash = "sha256:b12c6b18269ca471eedd41c1b6a1065b2f7827508edb9a7ed5555e9a56dcfc97"},
    {file = "coverage-7.6.9-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:899b8cd4781c400454f2f64f7776a5d87bbd7b3e7f7bda0cb18f857bb1334664"},
    {file = "coverage-7.6.9-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:61f70dc68bd36810972e55bbbe83674ea073dd1dcc121040a08cdf3416c5349c"},
    {file = "coverage-7.6.9-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:8a289d23d4c46f1a82d5db4abeb40b9b5be91731ee19a379d15790e53031c014"},
    {file = "coverage-7.6.9-cp313-cp313-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:7e216d8044a356fc0337c7a2a0536d6de07888d7bcda76febcb8adc50bdbbd00"},
    {file = "coverage-7.6.9-cp313-cp313-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:3c026eb44f744acaa2bda7493dad903aa5bf5fc4f2554293a798d5606710055d"},
    {file = "coverage-7.6.9-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:e77363e8425325384f9d49272c54045bbed2f478e9dd698dbc65dbc37860eb0a"},
    {file = "coverage-7.6.9-cp313-cp313-musllinux_1_2_i686.whl", hash = "sha256:777abfab476cf83b5177b84d7486497e034eb9eaea0d746ce0c1268c71652077"},
    {file = "coverage-7.6.9-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:447af20e25fdbe16f26e84eb714ba21d98868705cb138252d28bc400381f6ffb"},
    {file = "coverage-7.6.9-cp313-cp313-win32.whl", hash = "sha256:d872ec5aeb086cbea771c573600d47944eea2dcba8be5f3ee649bfe3cb8dc9ba"},
    {file = "coverage-7.6.9-cp313-cp313-win_amd64.whl", hash = "sha256:fd1213c86e48dfdc5a0cc676551db467495a95a662d2396ecd58e719191446e1"},
    {file = "coverage-7.6.9-cp313-cp313t-macosx_10_13_x86_64.whl", hash = "sha256:ba9e7484d286cd5a43744e5f47b0b3fb457865baf07bafc6bee91896364e1419"},
    {file = "coverage-7.6.9-cp313-cp313t-macosx_11_0_arm64.whl", hash = "sha256:e5ea1cf0872ee455c03e5674b5bca5e3e68e159379c1af0903e89f5eba9ccc3a"},
    {file = "coverage-7.6.9-cp313-cp313t-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:2d10e07aa2b91835d6abec555ec8b2733347956991901eea6ffac295f83a30e4"},
    {file = "coverage-7.6.9-cp313-cp313t-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:13a9e2d3ee855db3dd6ea1ba5203316a1b1fd8eaeffc37c5b54987e61e4194ae"},
    {file = "coverage-7.6.9-cp313-cp313t-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:9c38bf15a40ccf5619fa2fe8f26106c7e8e080d7760aeccb3722664c8656b030"},
    {file = "coverage-7.6.9-cp313-cp313t-musllinux_1_2_aarch64.whl", hash = "sha256:d5275455b3e4627c8e7154feaf7ee0743c2e7af82f6e3b561967b1cca755a0be"},
    {file = "coverage-7.6.9-cp313-cp313t-musllinux_1_2_i686.whl", hash = "sha256:8f8770dfc6e2c6a2d4569f411015c8d751c980d17a14b0530da2d7f27ffdd88e"},
    {file = "coverage-7.6.9-cp313-cp313t-musllinux_1_2_x86_64.whl", hash = "sha256:8d2dfa71665a29b153a9681edb1c8d9c1ea50dfc2375fb4dac99ea7e21a0bcd9"},
    {file = "coverage-7.6.9-cp313-cp313t-win32.whl", hash = "sha256:5e6b86b5847a016d0fbd31ffe1001b63355ed309651851295315031ea7eb5a9b"},
    {file = "coverage-7.6.9-cp313-cp313t-win_amd64.whl", hash = "sha256:97ddc94d46088304772d21b060041c97fc16bdda13c6c7f9d8fcd8d5ae0d8611"},
    {file = "coverage-7.6.9-cp39-cp39-macosx_10_9_x86_64.whl", hash = "sha256:adb697c0bd35100dc690de83154627fbab1f4f3c0386df266dded865fc50a902"},
    {file = "coverage-7.6.9-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:be57b6d56e49c2739cdf776839a92330e933dd5e5d929966fbbd380c77f060be"},
    {file = "coverage-7.6.9-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:f1592791f8204ae9166de22ba7e6705fa4ebd02936c09436a1bb85aabca3e599"},
    {file = "coverage-7.6.9-cp39-cp39-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:4e12ae8cc979cf83d258acb5e1f1cf2f3f83524d1564a49d20b8bec14b637f08"},
    {file = "coverage-7.6.9-cp39-cp39-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:bb5555cff66c4d3d6213a296b360f9e1a8e323e74e0426b6c10ed7f4d021e464"},
    {file = "coverage-7.6.9-cp39-cp39-musllinux_1_2_aarch64.whl", hash = "sha256:b9389a429e0e5142e69d5bf4a435dd688c14478a19bb901735cdf75e57b13845"},
    {file = "coverage-7.6.9-cp39-cp39-musllinux_1_2_i686.whl", hash = "sha256:592ac539812e9b46046620341498caf09ca21023c41c893e1eb9dbda00a70cbf"},
    {file = "coverage-7.6.9-cp39-cp39-musllinux_1_2_x86_64.whl", hash = "sha256:a27801adef24cc30871da98a105f77995e13a25a505a0161911f6aafbd66e678"},
    {file = "coverage-7.6.9-cp39-cp39-win32.whl", hash = "sha256:8e3c3e38930cfb729cb8137d7f055e5a473ddaf1217966aa6238c88bd9fd50e6"},
    {file = "coverage-7.6.9-cp39-cp39-win_amd64.whl", hash = "sha256:e28bf44afa2b187cc9f41749138a64435bf340adfcacb5b2290c070ce99839d4"},
    {file = "coverage-7.6.9-pp39.pp310-none-any.whl", hash = "sha256:f3ca78518bc6bc92828cd11867b121891d75cae4ea9e908d72030609b996db1b"},
    {file = "coverage-7.6.9.tar.gz", hash = "sha256:4a8d8977b0c6ef5aeadcb644da9e69ae0dcfe66ec7f368c89c72e058bd71164d"},
]

[package.extras]
toml = ["tomli"]

[[package]]
name = "cycler"
version = "0.12.1"
description = "Composable style cycles"
optional = false
python-versions = ">=3.8"
files = [
    {file = "cycler-0.12.1-py3-none-any.whl", hash = "sha256:85cef7cff222d8644161529808465972e51340599459b8ac3ccbac5a854e0d30"},
    {file = "cycler-0.12.1.tar.gz", hash = "sha256:88bb128f02ba341da8ef447245a9e138fae777f6a23943da4540077d3601eb1c"},
]

[package.extras]
docs = ["ipython", "matplotlib", "numpydoc", "sphinx"]
tests = ["pytest", "pytest-cov", "pytest-xdist"]

[[package]]
name = "debugpy"
version = "1.8.11"
description = "An implementation of the Debug Adapter Protocol for Python"
optional = false
python-versions = ">=3.8"
files = [
    {file = "debugpy-1.8.11-cp310-cp310-macosx_14_0_x86_64.whl", hash = "sha256:2b26fefc4e31ff85593d68b9022e35e8925714a10ab4858fb1b577a8a48cb8cd"},
    {file = "debugpy-1.8.11-cp310-cp310-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:61bc8b3b265e6949855300e84dc93d02d7a3a637f2aec6d382afd4ceb9120c9f"},
    {file = "debugpy-1.8.11-cp310-cp310-win32.whl", hash = "sha256:c928bbf47f65288574b78518449edaa46c82572d340e2750889bbf8cd92f3737"},
    {file = "debugpy-1.8.11-cp310-cp310-win_amd64.whl", hash = "sha256:8da1db4ca4f22583e834dcabdc7832e56fe16275253ee53ba66627b86e304da1"},
    {file = "debugpy-1.8.11-cp311-cp311-macosx_14_0_universal2.whl", hash = "sha256:85de8474ad53ad546ff1c7c7c89230db215b9b8a02754d41cb5a76f70d0be296"},
    {file = "debugpy-1.8.11-cp311-cp311-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:8ffc382e4afa4aee367bf413f55ed17bd91b191dcaf979890af239dda435f2a1"},
    {file = "debugpy-1.8.11-cp311-cp311-win32.whl", hash = "sha256:40499a9979c55f72f4eb2fc38695419546b62594f8af194b879d2a18439c97a9"},
    {file = "debugpy-1.8.11-cp311-cp311-win_amd64.whl", hash = "sha256:987bce16e86efa86f747d5151c54e91b3c1e36acc03ce1ddb50f9d09d16ded0e"},
    {file = "debugpy-1.8.11-cp312-cp312-macosx_14_0_universal2.whl", hash = "sha256:84e511a7545d11683d32cdb8f809ef63fc17ea2a00455cc62d0a4dbb4ed1c308"},
    {file = "debugpy-1.8.11-cp312-cp312-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:ce291a5aca4985d82875d6779f61375e959208cdf09fcec40001e65fb0a54768"},
    {file = "debugpy-1.8.11-cp312-cp312-win32.whl", hash = "sha256:28e45b3f827d3bf2592f3cf7ae63282e859f3259db44ed2b129093ca0ac7940b"},
    {file = "debugpy-1.8.11-cp312-cp312-win_amd64.whl", hash = "sha256:44b1b8e6253bceada11f714acf4309ffb98bfa9ac55e4fce14f9e5d4484287a1"},
    {file = "debugpy-1.8.11-cp313-cp313-macosx_14_0_universal2.whl", hash = "sha256:8988f7163e4381b0da7696f37eec7aca19deb02e500245df68a7159739bbd0d3"},
    {file = "debugpy-1.8.11-cp313-cp313-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:6c1f6a173d1140e557347419767d2b14ac1c9cd847e0b4c5444c7f3144697e4e"},
    {file = "debugpy-1.8.11-cp313-cp313-win32.whl", hash = "sha256:bb3b15e25891f38da3ca0740271e63ab9db61f41d4d8541745cfc1824252cb28"},
    {file = "debugpy-1.8.11-cp313-cp313-win_amd64.whl", hash = "sha256:d8768edcbeb34da9e11bcb8b5c2e0958d25218df7a6e56adf415ef262cd7b6d1"},
    {file = "debugpy-1.8.11-cp38-cp38-macosx_14_0_x86_64.whl", hash = "sha256:ad7efe588c8f5cf940f40c3de0cd683cc5b76819446abaa50dc0829a30c094db"},
    {file = "debugpy-1.8.11-cp38-cp38-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:189058d03a40103a57144752652b3ab08ff02b7595d0ce1f651b9acc3a3a35a0"},
    {file = "debugpy-1.8.11-cp38-cp38-win32.whl", hash = "sha256:32db46ba45849daed7ccf3f2e26f7a386867b077f39b2a974bb5c4c2c3b0a280"},
    {file = "debugpy-1.8.11-cp38-cp38-win_amd64.whl", hash = "sha256:116bf8342062246ca749013df4f6ea106f23bc159305843491f64672a55af2e5"},
    {file = "debugpy-1.8.11-cp39-cp39-macosx_14_0_x86_64.whl", hash = "sha256:654130ca6ad5de73d978057eaf9e582244ff72d4574b3e106fb8d3d2a0d32458"},
    {file = "debugpy-1.8.11-cp39-cp39-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:23dc34c5e03b0212fa3c49a874df2b8b1b8fda95160bd79c01eb3ab51ea8d851"},
    {file = "debugpy-1.8.11-cp39-cp39-win32.whl", hash = "sha256:52d8a3166c9f2815bfae05f386114b0b2d274456980d41f320299a8d9a5615a7"},
    {file = "debugpy-1.8.11-cp39-cp39-win_amd64.whl", hash = "sha256:52c3cf9ecda273a19cc092961ee34eb9ba8687d67ba34cc7b79a521c1c64c4c0"},
    {file = "debugpy-1.8.11-py2.py3-none-any.whl", hash = "sha256:0e22f846f4211383e6a416d04b4c13ed174d24cc5d43f5fd52e7821d0ebc8920"},
    {file = "debugpy-1.8.11.tar.gz", hash = "sha256:6ad2688b69235c43b020e04fecccdf6a96c8943ca9c2fb340b8adc103c655e57"},
]

[[package]]
name = "decorator"
version = "5.1.1"
description = "Decorators for Humans"
optional = false
python-versions = ">=3.5"
files = [
    {file = "decorator-5.1.1-py3-none-any.whl", hash = "sha256:b8c3f85900b9dc423225913c5aace94729fe1fa9763b38939a95226f02d37186"},
    {file = "decorator-5.1.1.tar.gz", hash = "sha256:637996211036b6385ef91435e4fae22989472f9d571faba8927ba8253acbc330"},
]

[[package]]
name = "defusedxml"
version = "0.7.1"
description = "XML bomb protection for Python stdlib modules"
optional = false
python-versions = ">=2.7, !=3.0.*, !=3.1.*, !=3.2.*, !=3.3.*, !=3.4.*"
files = [
    {file = "defusedxml-0.7.1-py2.py3-none-any.whl", hash = "sha256:a352e7e428770286cc899e2542b6cdaedb2b4953ff269a210103ec58f6198a61"},
    {file = "defusedxml-0.7.1.tar.gz", hash = "sha256:1bb3032db185915b62d7c6209c5a8792be6a32ab2fedacc84e01b52c51aa3e69"},
]

[[package]]
name = "dill"
version = "0.3.9"
description = "serialize all of Python"
optional = false
python-versions = ">=3.8"
files = [
    {file = "dill-0.3.9-py3-none-any.whl", hash = "sha256:468dff3b89520b474c0397703366b7b95eebe6303f108adf9b19da1f702be87a"},
    {file = "dill-0.3.9.tar.gz", hash = "sha256:81aa267dddf68cbfe8029c42ca9ec6a4ab3b22371d1c450abc54422577b4512c"},
]

[package.extras]
graph = ["objgraph (>=1.7.2)"]
profile = ["gprof2dot (>=2022.7.29)"]

[[package]]
name = "dnspython"
version = "2.7.0"
description = "DNS toolkit"
optional = false
python-versions = ">=3.9"
files = [
    {file = "dnspython-2.7.0-py3-none-any.whl", hash = "sha256:b4c34b7d10b51bcc3a5071e7b8dee77939f1e878477eeecc965e9835f63c6c86"},
    {file = "dnspython-2.7.0.tar.gz", hash = "sha256:ce9c432eda0dc91cf618a5cedf1a4e142651196bbcd2c80e89ed5a907e5cfaf1"},
]

[package.extras]
dev = ["black (>=23.1.0)", "coverage (>=7.0)", "flake8 (>=7)", "hypercorn (>=0.16.0)", "mypy (>=1.8)", "pylint (>=3)", "pytest (>=7.4)", "pytest-cov (>=4.1.0)", "quart-trio (>=0.11.0)", "sphinx (>=7.2.0)", "sphinx-rtd-theme (>=2.0.0)", "twine (>=4.0.0)", "wheel (>=0.42.0)"]
dnssec = ["cryptography (>=43)"]
doh = ["h2 (>=4.1.0)", "httpcore (>=1.0.0)", "httpx (>=0.26.0)"]
doq = ["aioquic (>=1.0.0)"]
idna = ["idna (>=3.7)"]
trio = ["trio (>=0.23)"]
wmi = ["wmi (>=1.5.1)"]

[[package]]
name = "email-validator"
version = "2.2.0"
description = "A robust email address syntax and deliverability validation library."
optional = false
python-versions = ">=3.8"
files = [
    {file = "email_validator-2.2.0-py3-none-any.whl", hash = "sha256:561977c2d73ce3611850a06fa56b414621e0c8faa9d66f2611407d87465da631"},
    {file = "email_validator-2.2.0.tar.gz", hash = "sha256:cb690f344c617a714f22e66ae771445a1ceb46821152df8e165c5f9a364582b7"},
]

[package.dependencies]
dnspython = ">=2.0.0"
idna = ">=2.0.0"

[[package]]
name = "execnet"
version = "2.1.1"
description = "execnet: rapid multi-Python deployment"
optional = false
python-versions = ">=3.8"
files = [
    {file = "execnet-2.1.1-py3-none-any.whl", hash = "sha256:26dee51f1b80cebd6d0ca8e74dd8745419761d3bef34163928cbebbdc4749fdc"},
    {file = "execnet-2.1.1.tar.gz", hash = "sha256:5189b52c6121c24feae288166ab41b32549c7e2348652736540b9e6e7d4e72e3"},
]

[package.extras]
testing = ["hatch", "pre-commit", "pytest", "tox"]

[[package]]
name = "executing"
version = "2.1.0"
description = "Get the currently executing AST node of a frame, and other information"
optional = false
python-versions = ">=3.8"
files = [
    {file = "executing-2.1.0-py2.py3-none-any.whl", hash = "sha256:8d63781349375b5ebccc3142f4b30350c0cd9c79f921cde38be2be4637e98eaf"},
    {file = "executing-2.1.0.tar.gz", hash = "sha256:8ea27ddd260da8150fa5a708269c4a10e76161e2496ec3e587da9e3c0fe4b9ab"},
]

[package.extras]
tests = ["asttokens (>=2.1.0)", "coverage", "coverage-enable-subprocess", "ipython", "littleutils", "pytest", "rich"]

[[package]]
name = "fastapi"
version = "0.113.0"
description = "FastAPI framework, high performance, easy to learn, fast to code, ready for production"
optional = false
python-versions = ">=3.8"
files = [
    {file = "fastapi-0.113.0-py3-none-any.whl", hash = "sha256:c8d364485b6361fb643d53920a18d58a696e189abcb901ec03b487e35774c476"},
    {file = "fastapi-0.113.0.tar.gz", hash = "sha256:b7cf9684dc154dfc93f8b718e5850577b529889096518df44defa41e73caf50f"},
]

[package.dependencies]
email-validator = {version = ">=2.0.0", optional = true, markers = "extra == \"all\""}
fastapi-cli = {version = ">=0.0.5", extras = ["standard"], optional = true, markers = "extra == \"all\""}
httpx = {version = ">=0.23.0", optional = true, markers = "extra == \"all\""}
itsdangerous = {version = ">=1.1.0", optional = true, markers = "extra == \"all\""}
jinja2 = {version = ">=2.11.2", optional = true, markers = "extra == \"all\""}
orjson = {version = ">=3.2.1", optional = true, markers = "extra == \"all\""}
pydantic = ">=1.7.4,<1.8 || >1.8,<1.8.1 || >1.8.1,<2.0.0 || >2.0.0,<2.0.1 || >2.0.1,<2.1.0 || >2.1.0,<3.0.0"
pydantic-extra-types = {version = ">=2.0.0", optional = true, markers = "extra == \"all\""}
pydantic-settings = {version = ">=2.0.0", optional = true, markers = "extra == \"all\""}
python-multipart = {version = ">=0.0.7", optional = true, markers = "extra == \"all\""}
pyyaml = {version = ">=5.3.1", optional = true, markers = "extra == \"all\""}
starlette = ">=0.37.2,<0.39.0"
typing-extensions = ">=4.8.0"
ujson = {version = ">=4.0.1,<4.0.2 || >4.0.2,<4.1.0 || >4.1.0,<4.2.0 || >4.2.0,<4.3.0 || >4.3.0,<5.0.0 || >5.0.0,<5.1.0 || >5.1.0", optional = true, markers = "extra == \"all\""}
uvicorn = {version = ">=0.12.0", extras = ["standard"], optional = true, markers = "extra == \"all\""}

[package.extras]
all = ["email-validator (>=2.0.0)", "fastapi-cli[standard] (>=0.0.5)", "httpx (>=0.23.0)", "itsdangerous (>=1.1.0)", "jinja2 (>=2.11.2)", "orjson (>=3.2.1)", "pydantic-extra-types (>=2.0.0)", "pydantic-settings (>=2.0.0)", "python-multipart (>=0.0.7)", "pyyaml (>=5.3.1)", "ujson (>=4.0.1,!=4.0.2,!=4.1.0,!=4.2.0,!=4.3.0,!=5.0.0,!=5.1.0)", "uvicorn[standard] (>=0.12.0)"]
standard = ["email-validator (>=2.0.0)", "fastapi-cli[standard] (>=0.0.5)", "httpx (>=0.23.0)", "jinja2 (>=2.11.2)", "python-multipart (>=0.0.7)", "uvicorn[standard] (>=0.12.0)"]

[[package]]
name = "fastapi-cli"
version = "0.0.7"
description = "Run and manage FastAPI apps from the command line with FastAPI CLI. 🚀"
optional = false
python-versions = ">=3.8"
files = [
    {file = "fastapi_cli-0.0.7-py3-none-any.whl", hash = "sha256:d549368ff584b2804336c61f192d86ddea080c11255f375959627911944804f4"},
    {file = "fastapi_cli-0.0.7.tar.gz", hash = "sha256:02b3b65956f526412515907a0793c9094abd4bfb5457b389f645b0ea6ba3605e"},
]

[package.dependencies]
rich-toolkit = ">=0.11.1"
typer = ">=0.12.3"
uvicorn = {version = ">=0.15.0", extras = ["standard"]}

[package.extras]
standard = ["uvicorn[standard] (>=0.15.0)"]

[[package]]
name = "fastjsonschema"
version = "2.21.1"
description = "Fastest Python implementation of JSON schema"
optional = false
python-versions = "*"
files = [
    {file = "fastjsonschema-2.21.1-py3-none-any.whl", hash = "sha256:c9e5b7e908310918cf494a434eeb31384dd84a98b57a30bcb1f535015b554667"},
    {file = "fastjsonschema-2.21.1.tar.gz", hash = "sha256:794d4f0a58f848961ba16af7b9c85a3e88cd360df008c59aac6fc5ae9323b5d4"},
]

[package.extras]
devel = ["colorama", "json-spec", "jsonschema", "pylint", "pytest", "pytest-benchmark", "pytest-cache", "validictory"]

[[package]]
name = "fonttools"
version = "4.55.3"
description = "Tools to manipulate font files"
optional = false
python-versions = ">=3.8"
files = [
    {file = "fonttools-4.55.3-cp310-cp310-macosx_10_9_universal2.whl", hash = "sha256:1dcc07934a2165ccdc3a5a608db56fb3c24b609658a5b340aee4ecf3ba679dc0"},
    {file = "fonttools-4.55.3-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:f7d66c15ba875432a2d2fb419523f5d3d347f91f48f57b8b08a2dfc3c39b8a3f"},
    {file = "fonttools-4.55.3-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:27e4ae3592e62eba83cd2c4ccd9462dcfa603ff78e09110680a5444c6925d841"},
    {file = "fonttools-4.55.3-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:62d65a3022c35e404d19ca14f291c89cc5890032ff04f6c17af0bd1927299674"},
    {file = "fonttools-4.55.3-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:d342e88764fb201286d185093781bf6628bbe380a913c24adf772d901baa8276"},
    {file = "fonttools-4.55.3-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:dd68c87a2bfe37c5b33bcda0fba39b65a353876d3b9006fde3adae31f97b3ef5"},
    {file = "fonttools-4.55.3-cp310-cp310-win32.whl", hash = "sha256:1bc7ad24ff98846282eef1cbeac05d013c2154f977a79886bb943015d2b1b261"},
    {file = "fonttools-4.55.3-cp310-cp310-win_amd64.whl", hash = "sha256:b54baf65c52952db65df39fcd4820668d0ef4766c0ccdf32879b77f7c804d5c5"},
    {file = "fonttools-4.55.3-cp311-cp311-macosx_10_9_universal2.whl", hash = "sha256:8c4491699bad88efe95772543cd49870cf756b019ad56294f6498982408ab03e"},
    {file = "fonttools-4.55.3-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:5323a22eabddf4b24f66d26894f1229261021dacd9d29e89f7872dd8c63f0b8b"},
    {file = "fonttools-4.55.3-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:5480673f599ad410695ca2ddef2dfefe9df779a9a5cda89503881e503c9c7d90"},
    {file = "fonttools-4.55.3-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:da9da6d65cd7aa6b0f806556f4985bcbf603bf0c5c590e61b43aa3e5a0f822d0"},
    {file = "fonttools-4.55.3-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:e894b5bd60d9f473bed7a8f506515549cc194de08064d829464088d23097331b"},
    {file = "fonttools-4.55.3-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:aee3b57643827e237ff6ec6d28d9ff9766bd8b21e08cd13bff479e13d4b14765"},
    {file = "fonttools-4.55.3-cp311-cp311-win32.whl", hash = "sha256:eb6ca911c4c17eb51853143624d8dc87cdcdf12a711fc38bf5bd21521e79715f"},
    {file = "fonttools-4.55.3-cp311-cp311-win_amd64.whl", hash = "sha256:6314bf82c54c53c71805318fcf6786d986461622dd926d92a465199ff54b1b72"},
    {file = "fonttools-4.55.3-cp312-cp312-macosx_10_13_universal2.whl", hash = "sha256:f9e736f60f4911061235603a6119e72053073a12c6d7904011df2d8fad2c0e35"},
    {file = "fonttools-4.55.3-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:7a8aa2c5e5b8b3bcb2e4538d929f6589a5c6bdb84fd16e2ed92649fb5454f11c"},
    {file = "fonttools-4.55.3-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:07f8288aacf0a38d174445fc78377a97fb0b83cfe352a90c9d9c1400571963c7"},
    {file = "fonttools-4.55.3-cp312-cp312-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:b8d5e8916c0970fbc0f6f1bece0063363bb5857a7f170121a4493e31c3db3314"},
    {file = "fonttools-4.55.3-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:ae3b6600565b2d80b7c05acb8e24d2b26ac407b27a3f2e078229721ba5698427"},
    {file = "fonttools-4.55.3-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:54153c49913f45065c8d9e6d0c101396725c5621c8aee744719300f79771d75a"},
    {file = "fonttools-4.55.3-cp312-cp312-win32.whl", hash = "sha256:827e95fdbbd3e51f8b459af5ea10ecb4e30af50221ca103bea68218e9615de07"},
    {file = "fonttools-4.55.3-cp312-cp312-win_amd64.whl", hash = "sha256:e6e8766eeeb2de759e862004aa11a9ea3d6f6d5ec710551a88b476192b64fd54"},
    {file = "fonttools-4.55.3-cp313-cp313-macosx_10_13_universal2.whl", hash = "sha256:a430178ad3e650e695167cb53242dae3477b35c95bef6525b074d87493c4bf29"},
    {file = "fonttools-4.55.3-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:529cef2ce91dc44f8e407cc567fae6e49a1786f2fefefa73a294704c415322a4"},
    {file = "fonttools-4.55.3-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:8e75f12c82127486fac2d8bfbf5bf058202f54bf4f158d367e41647b972342ca"},
    {file = "fonttools-4.55.3-cp313-cp313-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:859c358ebf41db18fb72342d3080bce67c02b39e86b9fbcf1610cca14984841b"},
    {file = "fonttools-4.55.3-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:546565028e244a701f73df6d8dd6be489d01617863ec0c6a42fa25bf45d43048"},
    {file = "fonttools-4.55.3-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:aca318b77f23523309eec4475d1fbbb00a6b133eb766a8bdc401faba91261abe"},
    {file = "fonttools-4.55.3-cp313-cp313-win32.whl", hash = "sha256:8c5ec45428edaa7022f1c949a632a6f298edc7b481312fc7dc258921e9399628"},
    {file = "fonttools-4.55.3-cp313-cp313-win_amd64.whl", hash = "sha256:11e5de1ee0d95af4ae23c1a138b184b7f06e0b6abacabf1d0db41c90b03d834b"},
    {file = "fonttools-4.55.3-cp38-cp38-macosx_10_9_universal2.whl", hash = "sha256:caf8230f3e10f8f5d7593eb6d252a37caf58c480b19a17e250a63dad63834cf3"},
    {file = "fonttools-4.55.3-cp38-cp38-macosx_10_9_x86_64.whl", hash = "sha256:b586ab5b15b6097f2fb71cafa3c98edfd0dba1ad8027229e7b1e204a58b0e09d"},
    {file = "fonttools-4.55.3-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:a8c2794ded89399cc2169c4d0bf7941247b8d5932b2659e09834adfbb01589aa"},
    {file = "fonttools-4.55.3-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:cf4fe7c124aa3f4e4c1940880156e13f2f4d98170d35c749e6b4f119a872551e"},
    {file = "fonttools-4.55.3-cp38-cp38-musllinux_1_2_aarch64.whl", hash = "sha256:86721fbc389ef5cc1e2f477019e5069e8e4421e8d9576e9c26f840dbb04678de"},
    {file = "fonttools-4.55.3-cp38-cp38-musllinux_1_2_x86_64.whl", hash = "sha256:89bdc5d88bdeec1b15af790810e267e8332d92561dce4f0748c2b95c9bdf3926"},
    {file = "fonttools-4.55.3-cp38-cp38-win32.whl", hash = "sha256:bc5dbb4685e51235ef487e4bd501ddfc49be5aede5e40f4cefcccabc6e60fb4b"},
    {file = "fonttools-4.55.3-cp38-cp38-win_amd64.whl", hash = "sha256:cd70de1a52a8ee2d1877b6293af8a2484ac82514f10b1c67c1c5762d38073e56"},
    {file = "fonttools-4.55.3-cp39-cp39-macosx_10_9_universal2.whl", hash = "sha256:bdcc9f04b36c6c20978d3f060e5323a43f6222accc4e7fcbef3f428e216d96af"},
    {file = "fonttools-4.55.3-cp39-cp39-macosx_10_9_x86_64.whl", hash = "sha256:c3ca99e0d460eff46e033cd3992a969658c3169ffcd533e0a39c63a38beb6831"},
    {file = "fonttools-4.55.3-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:22f38464daa6cdb7b6aebd14ab06609328fe1e9705bb0fcc7d1e69de7109ee02"},
    {file = "fonttools-4.55.3-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:ed63959d00b61959b035c7d47f9313c2c1ece090ff63afea702fe86de00dbed4"},
    {file = "fonttools-4.55.3-cp39-cp39-musllinux_1_2_aarch64.whl", hash = "sha256:5e8d657cd7326eeaba27de2740e847c6b39dde2f8d7cd7cc56f6aad404ddf0bd"},
    {file = "fonttools-4.55.3-cp39-cp39-musllinux_1_2_x86_64.whl", hash = "sha256:fb594b5a99943042c702c550d5494bdd7577f6ef19b0bc73877c948a63184a32"},
    {file = "fonttools-4.55.3-cp39-cp39-win32.whl", hash = "sha256:dc5294a3d5c84226e3dbba1b6f61d7ad813a8c0238fceea4e09aa04848c3d851"},
    {file = "fonttools-4.55.3-cp39-cp39-win_amd64.whl", hash = "sha256:aedbeb1db64496d098e6be92b2e63b5fac4e53b1b92032dfc6988e1ea9134a4d"},
    {file = "fonttools-4.55.3-py3-none-any.whl", hash = "sha256:f412604ccbeee81b091b420272841e5ec5ef68967a9790e80bffd0e30b8e2977"},
    {file = "fonttools-4.55.3.tar.gz", hash = "sha256:3983313c2a04d6cc1fe9251f8fc647754cf49a61dac6cb1e7249ae67afaafc45"},
]

[package.extras]
all = ["brotli (>=1.0.1)", "brotlicffi (>=0.8.0)", "fs (>=2.2.0,<3)", "lxml (>=4.0)", "lz4 (>=1.7.4.2)", "matplotlib", "munkres", "pycairo", "scipy", "skia-pathops (>=0.5.0)", "sympy", "uharfbuzz (>=0.23.0)", "unicodedata2 (>=15.1.0)", "xattr", "zopfli (>=0.1.4)"]
graphite = ["lz4 (>=1.7.4.2)"]
interpolatable = ["munkres", "pycairo", "scipy"]
lxml = ["lxml (>=4.0)"]
pathops = ["skia-pathops (>=0.5.0)"]
plot = ["matplotlib"]
repacker = ["uharfbuzz (>=0.23.0)"]
symfont = ["sympy"]
type1 = ["xattr"]
ufo = ["fs (>=2.2.0,<3)"]
unicode = ["unicodedata2 (>=15.1.0)"]
woff = ["brotli (>=1.0.1)", "brotlicffi (>=0.8.0)", "zopfli (>=0.1.4)"]

[[package]]
name = "fqdn"
version = "1.5.1"
description = "Validates fully-qualified domain names against RFC 1123, so that they are acceptable to modern bowsers"
optional = false
python-versions = ">=2.7, !=3.0, !=3.1, !=3.2, !=3.3, !=3.4, <4"
files = [
    {file = "fqdn-1.5.1-py3-none-any.whl", hash = "sha256:3a179af3761e4df6eb2e026ff9e1a3033d3587bf980a0b1b2e1e5d08d7358014"},
    {file = "fqdn-1.5.1.tar.gz", hash = "sha256:105ed3677e767fb5ca086a0c1f4bb66ebc3c100be518f0e0d755d9eae164d89f"},
]

[[package]]
name = "gitdb"
version = "4.0.11"
description = "Git Object Database"
optional = false
python-versions = ">=3.7"
files = [
    {file = "gitdb-4.0.11-py3-none-any.whl", hash = "sha256:81a3407ddd2ee8df444cbacea00e2d038e40150acfa3001696fe0dcf1d3adfa4"},
    {file = "gitdb-4.0.11.tar.gz", hash = "sha256:bf5421126136d6d0af55bc1e7c1af1c397a34f5b7bd79e776cd3e89785c2b04b"},
]

[package.dependencies]
smmap = ">=3.0.1,<6"

[[package]]
name = "gitpython"
version = "3.1.43"
description = "GitPython is a Python library used to interact with Git repositories"
optional = false
python-versions = ">=3.7"
files = [
    {file = "GitPython-3.1.43-py3-none-any.whl", hash = "sha256:eec7ec56b92aad751f9912a73404bc02ba212a23adb2c7098ee668417051a1ff"},
    {file = "GitPython-3.1.43.tar.gz", hash = "sha256:35f314a9f878467f5453cc1fee295c3e18e52f1b99f10f6cf5b1682e968a9e7c"},
]

[package.dependencies]
gitdb = ">=4.0.1,<5"

[package.extras]
doc = ["sphinx (==4.3.2)", "sphinx-autodoc-typehints", "sphinx-rtd-theme", "sphinxcontrib-applehelp (>=1.0.2,<=1.0.4)", "sphinxcontrib-devhelp (==1.0.2)", "sphinxcontrib-htmlhelp (>=2.0.0,<=2.0.1)", "sphinxcontrib-qthelp (==1.0.3)", "sphinxcontrib-serializinghtml (==1.1.5)"]
test = ["coverage[toml]", "ddt (>=1.1.1,!=1.4.3)", "mock", "mypy", "pre-commit", "pytest (>=7.3.1)", "pytest-cov", "pytest-instafail", "pytest-mock", "pytest-sugar", "typing-extensions"]

[[package]]
name = "h11"
version = "0.14.0"
description = "A pure-Python, bring-your-own-I/O implementation of HTTP/1.1"
optional = false
python-versions = ">=3.7"
files = [
    {file = "h11-0.14.0-py3-none-any.whl", hash = "sha256:e3fe4ac4b851c468cc8363d500db52c2ead036020723024a109d37346efaa761"},
    {file = "h11-0.14.0.tar.gz", hash = "sha256:8f19fbbe99e72420ff35c00b27a34cb9937e902a8b810e2c88300c6f0a3b699d"},
]

[[package]]
name = "httpcore"
version = "1.0.7"
description = "A minimal low-level HTTP client."
optional = false
python-versions = ">=3.8"
files = [
    {file = "httpcore-1.0.7-py3-none-any.whl", hash = "sha256:a3fff8f43dc260d5bd363d9f9cf1830fa3a458b332856f34282de498ed420edd"},
    {file = "httpcore-1.0.7.tar.gz", hash = "sha256:8551cb62a169ec7162ac7be8d4817d561f60e08eaa485234898414bb5a8a0b4c"},
]

[package.dependencies]
certifi = "*"
h11 = ">=0.13,<0.15"

[package.extras]
asyncio = ["anyio (>=4.0,<5.0)"]
http2 = ["h2 (>=3,<5)"]
socks = ["socksio (==1.*)"]
trio = ["trio (>=0.22.0,<1.0)"]

[[package]]
name = "httptools"
version = "0.6.4"
description = "A collection of framework independent HTTP protocol utils."
optional = false
python-versions = ">=3.8.0"
files = [
    {file = "httptools-0.6.4-cp310-cp310-macosx_10_9_universal2.whl", hash = "sha256:3c73ce323711a6ffb0d247dcd5a550b8babf0f757e86a52558fe5b86d6fefcc0"},
    {file = "httptools-0.6.4-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:345c288418f0944a6fe67be8e6afa9262b18c7626c3ef3c28adc5eabc06a68da"},
    {file = "httptools-0.6.4-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:deee0e3343f98ee8047e9f4c5bc7cedbf69f5734454a94c38ee829fb2d5fa3c1"},
    {file = "httptools-0.6.4-cp310-cp310-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:ca80b7485c76f768a3bc83ea58373f8db7b015551117375e4918e2aa77ea9b50"},
    {file = "httptools-0.6.4-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:90d96a385fa941283ebd231464045187a31ad932ebfa541be8edf5b3c2328959"},
    {file = "httptools-0.6.4-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:59e724f8b332319e2875efd360e61ac07f33b492889284a3e05e6d13746876f4"},
    {file = "httptools-0.6.4-cp310-cp310-win_amd64.whl", hash = "sha256:c26f313951f6e26147833fc923f78f95604bbec812a43e5ee37f26dc9e5a686c"},
    {file = "httptools-0.6.4-cp311-cp311-macosx_10_9_universal2.whl", hash = "sha256:f47f8ed67cc0ff862b84a1189831d1d33c963fb3ce1ee0c65d3b0cbe7b711069"},
    {file = "httptools-0.6.4-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:0614154d5454c21b6410fdf5262b4a3ddb0f53f1e1721cfd59d55f32138c578a"},
    {file = "httptools-0.6.4-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:f8787367fbdfccae38e35abf7641dafc5310310a5987b689f4c32cc8cc3ee975"},
    {file = "httptools-0.6.4-cp311-cp311-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:40b0f7fe4fd38e6a507bdb751db0379df1e99120c65fbdc8ee6c1d044897a636"},
    {file = "httptools-0.6.4-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:40a5ec98d3f49904b9fe36827dcf1aadfef3b89e2bd05b0e35e94f97c2b14721"},
    {file = "httptools-0.6.4-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:dacdd3d10ea1b4ca9df97a0a303cbacafc04b5cd375fa98732678151643d4988"},
    {file = "httptools-0.6.4-cp311-cp311-win_amd64.whl", hash = "sha256:288cd628406cc53f9a541cfaf06041b4c71d751856bab45e3702191f931ccd17"},
    {file = "httptools-0.6.4-cp312-cp312-macosx_10_13_universal2.whl", hash = "sha256:df017d6c780287d5c80601dafa31f17bddb170232d85c066604d8558683711a2"},
    {file = "httptools-0.6.4-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:85071a1e8c2d051b507161f6c3e26155b5c790e4e28d7f236422dbacc2a9cc44"},
    {file = "httptools-0.6.4-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:69422b7f458c5af875922cdb5bd586cc1f1033295aa9ff63ee196a87519ac8e1"},
    {file = "httptools-0.6.4-cp312-cp312-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:16e603a3bff50db08cd578d54f07032ca1631450ceb972c2f834c2b860c28ea2"},
    {file = "httptools-0.6.4-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:ec4f178901fa1834d4a060320d2f3abc5c9e39766953d038f1458cb885f47e81"},
    {file = "httptools-0.6.4-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:f9eb89ecf8b290f2e293325c646a211ff1c2493222798bb80a530c5e7502494f"},
    {file = "httptools-0.6.4-cp312-cp312-win_amd64.whl", hash = "sha256:db78cb9ca56b59b016e64b6031eda5653be0589dba2b1b43453f6e8b405a0970"},
    {file = "httptools-0.6.4-cp313-cp313-macosx_10_13_universal2.whl", hash = "sha256:ade273d7e767d5fae13fa637f4d53b6e961fb7fd93c7797562663f0171c26660"},
    {file = "httptools-0.6.4-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:856f4bc0478ae143bad54a4242fccb1f3f86a6e1be5548fecfd4102061b3a083"},
    {file = "httptools-0.6.4-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:322d20ea9cdd1fa98bd6a74b77e2ec5b818abdc3d36695ab402a0de8ef2865a3"},
    {file = "httptools-0.6.4-cp313-cp313-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:4d87b29bd4486c0093fc64dea80231f7c7f7eb4dc70ae394d70a495ab8436071"},
    {file = "httptools-0.6.4-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:342dd6946aa6bda4b8f18c734576106b8a31f2fe31492881a9a160ec84ff4bd5"},
    {file = "httptools-0.6.4-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:4b36913ba52008249223042dca46e69967985fb4051951f94357ea681e1f5dc0"},
    {file = "httptools-0.6.4-cp313-cp313-win_amd64.whl", hash = "sha256:28908df1b9bb8187393d5b5db91435ccc9c8e891657f9cbb42a2541b44c82fc8"},
    {file = "httptools-0.6.4-cp38-cp38-macosx_10_9_universal2.whl", hash = "sha256:d3f0d369e7ffbe59c4b6116a44d6a8eb4783aae027f2c0b366cf0aa964185dba"},
    {file = "httptools-0.6.4-cp38-cp38-macosx_11_0_arm64.whl", hash = "sha256:94978a49b8f4569ad607cd4946b759d90b285e39c0d4640c6b36ca7a3ddf2efc"},
    {file = "httptools-0.6.4-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:40dc6a8e399e15ea525305a2ddba998b0af5caa2566bcd79dcbe8948181eeaff"},
    {file = "httptools-0.6.4-cp38-cp38-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:ab9ba8dcf59de5181f6be44a77458e45a578fc99c31510b8c65b7d5acc3cf490"},
    {file = "httptools-0.6.4-cp38-cp38-musllinux_1_2_aarch64.whl", hash = "sha256:fc411e1c0a7dcd2f902c7c48cf079947a7e65b5485dea9decb82b9105ca71a43"},
    {file = "httptools-0.6.4-cp38-cp38-musllinux_1_2_x86_64.whl", hash = "sha256:d54efd20338ac52ba31e7da78e4a72570cf729fac82bc31ff9199bedf1dc7440"},
    {file = "httptools-0.6.4-cp38-cp38-win_amd64.whl", hash = "sha256:df959752a0c2748a65ab5387d08287abf6779ae9165916fe053e68ae1fbdc47f"},
    {file = "httptools-0.6.4-cp39-cp39-macosx_10_9_universal2.whl", hash = "sha256:85797e37e8eeaa5439d33e556662cc370e474445d5fab24dcadc65a8ffb04003"},
    {file = "httptools-0.6.4-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:db353d22843cf1028f43c3651581e4bb49374d85692a85f95f7b9a130e1b2cab"},
    {file = "httptools-0.6.4-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:d1ffd262a73d7c28424252381a5b854c19d9de5f56f075445d33919a637e3547"},
    {file = "httptools-0.6.4-cp39-cp39-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:703c346571fa50d2e9856a37d7cd9435a25e7fd15e236c397bf224afaa355fe9"},
    {file = "httptools-0.6.4-cp39-cp39-musllinux_1_2_aarch64.whl", hash = "sha256:aafe0f1918ed07b67c1e838f950b1c1fabc683030477e60b335649b8020e1076"},
    {file = "httptools-0.6.4-cp39-cp39-musllinux_1_2_x86_64.whl", hash = "sha256:0e563e54979e97b6d13f1bbc05a96109923e76b901f786a5eae36e99c01237bd"},
    {file = "httptools-0.6.4-cp39-cp39-win_amd64.whl", hash = "sha256:b799de31416ecc589ad79dd85a0b2657a8fe39327944998dea368c1d4c9e55e6"},
    {file = "httptools-0.6.4.tar.gz", hash = "sha256:4e93eee4add6493b59a5c514da98c939b244fce4a0d8879cd3f466562f4b7d5c"},
]

[package.extras]
test = ["Cython (>=0.29.24)"]

[[package]]
name = "httpx"
version = "0.28.1"
description = "The next generation HTTP client."
optional = false
python-versions = ">=3.8"
files = [
    {file = "httpx-0.28.1-py3-none-any.whl", hash = "sha256:d909fcccc110f8c7faf814ca82a9a4d816bc5a6dbfea25d6591d6985b8ba59ad"},
    {file = "httpx-0.28.1.tar.gz", hash = "sha256:75e98c5f16b0f35b567856f597f06ff2270a374470a5c2392242528e3e3e42fc"},
]

[package.dependencies]
anyio = "*"
certifi = "*"
httpcore = "==1.*"
idna = "*"

[package.extras]
brotli = ["brotli", "brotlicffi"]
cli = ["click (==8.*)", "pygments (==2.*)", "rich (>=10,<14)"]
http2 = ["h2 (>=3,<5)"]
socks = ["socksio (==1.*)"]
zstd = ["zstandard (>=0.18.0)"]

[[package]]
name = "hypothesis"
version = "6.123.1"
description = "A library for property-based testing"
optional = false
python-versions = ">=3.9"
files = [
    {file = "hypothesis-6.123.1-py3-none-any.whl", hash = "sha256:acf177faeff578f02afef2744c00b9ec3ce03f3b72ffbb6d7013f7baeebaac5e"},
    {file = "hypothesis-6.123.1.tar.gz", hash = "sha256:eb2bf646537ad818270feff6428c34f57813a4ef78781861ec1693b0840ab1d8"},
]

[package.dependencies]
attrs = ">=22.2.0"
sortedcontainers = ">=2.1.0,<3.0.0"

[package.extras]
all = ["black (>=19.10b0)", "click (>=7.0)", "crosshair-tool (>=0.0.78)", "django (>=4.2)", "dpcontracts (>=0.4)", "hypothesis-crosshair (>=0.0.18)", "lark (>=0.10.1)", "libcst (>=0.3.16)", "numpy (>=1.19.3)", "pandas (>=1.1)", "pytest (>=4.6)", "python-dateutil (>=1.4)", "pytz (>=2014.1)", "redis (>=3.0.0)", "rich (>=9.0.0)", "tzdata (>=2024.2)"]
cli = ["black (>=19.10b0)", "click (>=7.0)", "rich (>=9.0.0)"]
codemods = ["libcst (>=0.3.16)"]
crosshair = ["crosshair-tool (>=0.0.78)", "hypothesis-crosshair (>=0.0.18)"]
dateutil = ["python-dateutil (>=1.4)"]
django = ["django (>=4.2)"]
dpcontracts = ["dpcontracts (>=0.4)"]
ghostwriter = ["black (>=19.10b0)"]
lark = ["lark (>=0.10.1)"]
numpy = ["numpy (>=1.19.3)"]
pandas = ["pandas (>=1.1)"]
pytest = ["pytest (>=4.6)"]
pytz = ["pytz (>=2014.1)"]
redis = ["redis (>=3.0.0)"]
zoneinfo = ["tzdata (>=2024.2)"]

[[package]]
name = "icdiff"
version = "2.0.7"
description = "improved colored diff"
optional = false
python-versions = "*"
files = [
    {file = "icdiff-2.0.7-py3-none-any.whl", hash = "sha256:f05d1b3623223dd1c70f7848da7d699de3d9a2550b902a8234d9026292fb5762"},
    {file = "icdiff-2.0.7.tar.gz", hash = "sha256:f79a318891adbf59a45e3a7694f5e1f18c5407065264637072ac8363b759866f"},
]

[[package]]
name = "idna"
version = "3.10"
description = "Internationalized Domain Names in Applications (IDNA)"
optional = false
python-versions = ">=3.6"
files = [
    {file = "idna-3.10-py3-none-any.whl", hash = "sha256:946d195a0d259cbba61165e88e65941f16e9b36ea6ddb97f00452bae8b1287d3"},
    {file = "idna-3.10.tar.gz", hash = "sha256:12f65c9b470abda6dc35cf8e63cc574b1c52b11df2c86030af0ac09b01b13ea9"},
]

[package.extras]
all = ["flake8 (>=7.1.1)", "mypy (>=1.11.2)", "pytest (>=8.3.2)", "ruff (>=0.6.2)"]

[[package]]
name = "importlib-metadata"
version = "8.5.0"
description = "Read metadata from Python packages"
optional = false
python-versions = ">=3.8"
files = [
    {file = "importlib_metadata-8.5.0-py3-none-any.whl", hash = "sha256:45e54197d28b7a7f1559e60b95e7c567032b602131fbd588f1497f47880aa68b"},
    {file = "importlib_metadata-8.5.0.tar.gz", hash = "sha256:71522656f0abace1d072b9e5481a48f07c138e00f079c38c8f883823f9c26bd7"},
]

[package.dependencies]
zipp = ">=3.20"

[package.extras]
check = ["pytest-checkdocs (>=2.4)", "pytest-ruff (>=0.2.1)"]
cover = ["pytest-cov"]
doc = ["furo", "jaraco.packaging (>=9.3)", "jaraco.tidelift (>=1.4)", "rst.linker (>=1.9)", "sphinx (>=3.5)", "sphinx-lint"]
enabler = ["pytest-enabler (>=2.2)"]
perf = ["ipython"]
test = ["flufl.flake8", "importlib-resources (>=1.3)", "jaraco.test (>=5.4)", "packaging", "pyfakefs", "pytest (>=6,!=8.1.*)", "pytest-perf (>=0.9.2)"]
type = ["pytest-mypy"]

[[package]]
name = "iniconfig"
version = "2.0.0"
description = "brain-dead simple config-ini parsing"
optional = false
python-versions = ">=3.7"
files = [
    {file = "iniconfig-2.0.0-py3-none-any.whl", hash = "sha256:b6a85871a79d2e3b22d2d1b94ac2824226a63c6b741c88f7ae975f18b6778374"},
    {file = "iniconfig-2.0.0.tar.gz", hash = "sha256:2d91e135bf72d31a410b17c16da610a82cb55f6b0477d1a902134b24a455b8b3"},
]

[[package]]
name = "ipykernel"
version = "6.29.5"
description = "IPython Kernel for Jupyter"
optional = false
python-versions = ">=3.8"
files = [
    {file = "ipykernel-6.29.5-py3-none-any.whl", hash = "sha256:afdb66ba5aa354b09b91379bac28ae4afebbb30e8b39510c9690afb7a10421b5"},
    {file = "ipykernel-6.29.5.tar.gz", hash = "sha256:f093a22c4a40f8828f8e330a9c297cb93dcab13bd9678ded6de8e5cf81c56215"},
]

[package.dependencies]
appnope = {version = "*", markers = "platform_system == \"Darwin\""}
comm = ">=0.1.1"
debugpy = ">=1.6.5"
ipython = ">=7.23.1"
jupyter-client = ">=6.1.12"
jupyter-core = ">=4.12,<5.0.dev0 || >=5.1.dev0"
matplotlib-inline = ">=0.1"
nest-asyncio = "*"
packaging = "*"
psutil = "*"
pyzmq = ">=24"
tornado = ">=6.1"
traitlets = ">=5.4.0"

[package.extras]
cov = ["coverage[toml]", "curio", "matplotlib", "pytest-cov", "trio"]
docs = ["myst-parser", "pydata-sphinx-theme", "sphinx", "sphinx-autodoc-typehints", "sphinxcontrib-github-alt", "sphinxcontrib-spelling", "trio"]
pyqt5 = ["pyqt5"]
pyside6 = ["pyside6"]
test = ["flaky", "ipyparallel", "pre-commit", "pytest (>=7.0)", "pytest-asyncio (>=0.23.5)", "pytest-cov", "pytest-timeout"]

[[package]]
name = "ipython"
version = "8.31.0"
description = "IPython: Productive Interactive Computing"
optional = false
python-versions = ">=3.10"
files = [
    {file = "ipython-8.31.0-py3-none-any.whl", hash = "sha256:46ec58f8d3d076a61d128fe517a51eb730e3aaf0c184ea8c17d16e366660c6a6"},
    {file = "ipython-8.31.0.tar.gz", hash = "sha256:b6a2274606bec6166405ff05e54932ed6e5cfecaca1fc05f2cacde7bb074d70b"},
]

[package.dependencies]
colorama = {version = "*", markers = "sys_platform == \"win32\""}
decorator = "*"
jedi = ">=0.16"
matplotlib-inline = "*"
pexpect = {version = ">4.3", markers = "sys_platform != \"win32\" and sys_platform != \"emscripten\""}
prompt_toolkit = ">=3.0.41,<3.1.0"
pygments = ">=2.4.0"
stack_data = "*"
traitlets = ">=5.13.0"

[package.extras]
all = ["ipython[black,doc,kernel,matplotlib,nbconvert,nbformat,notebook,parallel,qtconsole]", "ipython[test,test-extra]"]
black = ["black"]
doc = ["docrepr", "exceptiongroup", "intersphinx_registry", "ipykernel", "ipython[test]", "matplotlib", "setuptools (>=18.5)", "sphinx (>=1.3)", "sphinx-rtd-theme", "sphinxcontrib-jquery", "tomli", "typing_extensions"]
kernel = ["ipykernel"]
matplotlib = ["matplotlib"]
nbconvert = ["nbconvert"]
nbformat = ["nbformat"]
notebook = ["ipywidgets", "notebook"]
parallel = ["ipyparallel"]
qtconsole = ["qtconsole"]
test = ["packaging", "pickleshare", "pytest", "pytest-asyncio (<0.22)", "testpath"]
test-extra = ["curio", "ipython[test]", "matplotlib (!=3.2.0)", "nbformat", "numpy (>=1.23)", "pandas", "trio"]

[[package]]
name = "ipython-genutils"
version = "0.2.0"
description = "Vestigial utilities from IPython"
optional = false
python-versions = "*"
files = [
    {file = "ipython_genutils-0.2.0-py2.py3-none-any.whl", hash = "sha256:72dd37233799e619666c9f639a9da83c34013a73e8bbc79a7a6348d93c61fab8"},
    {file = "ipython_genutils-0.2.0.tar.gz", hash = "sha256:eb2e116e75ecef9d4d228fdc66af54269afa26ab4463042e33785b887c628ba8"},
]

[[package]]
name = "ipython-sql"
version = "0.5.0"
description = "RDBMS access via IPython"
optional = false
python-versions = "*"
files = [
    {file = "ipython-sql-0.5.0.tar.gz", hash = "sha256:3db3ce7f9a95dfaf43876fa80b16bdbb9a04de0ba12042cab13e9f596fcffdbe"},
    {file = "ipython_sql-0.5.0-py3-none-any.whl", hash = "sha256:61b46ecffb956f62dbc17b5744cf70c649104c8db9afd821aa39b31f7cbb5d5b"},
]

[package.dependencies]
ipython = "*"
ipython-genutils = "*"
prettytable = "*"
six = "*"
sqlalchemy = ">=2.0"
sqlparse = "*"

[[package]]
name = "ipywidgets"
version = "8.1.5"
description = "Jupyter interactive widgets"
optional = false
python-versions = ">=3.7"
files = [
    {file = "ipywidgets-8.1.5-py3-none-any.whl", hash = "sha256:3290f526f87ae6e77655555baba4f36681c555b8bdbbff430b70e52c34c86245"},
    {file = "ipywidgets-8.1.5.tar.gz", hash = "sha256:870e43b1a35656a80c18c9503bbf2d16802db1cb487eec6fab27d683381dde17"},
]

[package.dependencies]
comm = ">=0.1.3"
ipython = ">=6.1.0"
jupyterlab-widgets = ">=3.0.12,<3.1.0"
traitlets = ">=4.3.1"
widgetsnbextension = ">=4.0.12,<4.1.0"

[package.extras]
test = ["ipykernel", "jsonschema", "pytest (>=3.6.0)", "pytest-cov", "pytz"]

[[package]]
name = "isoduration"
version = "20.11.0"
description = "Operations with ISO 8601 durations"
optional = false
python-versions = ">=3.7"
files = [
    {file = "isoduration-20.11.0-py3-none-any.whl", hash = "sha256:b2904c2a4228c3d44f409c8ae8e2370eb21a26f7ac2ec5446df141dde3452042"},
    {file = "isoduration-20.11.0.tar.gz", hash = "sha256:ac2f9015137935279eac671f94f89eb00584f940f5dc49462a0c4ee692ba1bd9"},
]

[package.dependencies]
arrow = ">=0.15.0"

[[package]]
name = "isort"
version = "5.13.2"
description = "A Python utility / library to sort Python imports."
optional = false
python-versions = ">=3.8.0"
files = [
    {file = "isort-5.13.2-py3-none-any.whl", hash = "sha256:8ca5e72a8d85860d5a3fa69b8745237f2939afe12dbf656afbcb47fe72d947a6"},
    {file = "isort-5.13.2.tar.gz", hash = "sha256:48fdfcb9face5d58a4f6dde2e72a1fb8dcaf8ab26f95ab49fab84c2ddefb0109"},
]

[package.extras]
colors = ["colorama (>=0.4.6)"]

[[package]]
name = "itsdangerous"
version = "2.2.0"
description = "Safely pass data to untrusted environments and back."
optional = false
python-versions = ">=3.8"
files = [
    {file = "itsdangerous-2.2.0-py3-none-any.whl", hash = "sha256:c6242fc49e35958c8b15141343aa660db5fc54d4f13a1db01a3f5891b98700ef"},
    {file = "itsdangerous-2.2.0.tar.gz", hash = "sha256:e0050c0b7da1eea53ffaf149c0cfbb5c6e2e2b69c4bef22c81fa6eb73e5f6173"},
]

[[package]]
name = "jedi"
version = "0.19.2"
description = "An autocompletion tool for Python that can be used for text editors."
optional = false
python-versions = ">=3.6"
files = [
    {file = "jedi-0.19.2-py2.py3-none-any.whl", hash = "sha256:a8ef22bde8490f57fe5c7681a3c83cb58874daf72b4784de3cce5b6ef6edb5b9"},
    {file = "jedi-0.19.2.tar.gz", hash = "sha256:4770dc3de41bde3966b02eb84fbcf557fb33cce26ad23da12c742fb50ecb11f0"},
]

[package.dependencies]
parso = ">=0.8.4,<0.9.0"

[package.extras]
docs = ["Jinja2 (==2.11.3)", "MarkupSafe (==1.1.1)", "Pygments (==2.8.1)", "alabaster (==0.7.12)", "babel (==2.9.1)", "chardet (==4.0.0)", "commonmark (==0.8.1)", "docutils (==0.17.1)", "future (==0.18.2)", "idna (==2.10)", "imagesize (==1.2.0)", "mock (==1.0.1)", "packaging (==20.9)", "pyparsing (==2.4.7)", "pytz (==2021.1)", "readthedocs-sphinx-ext (==2.1.4)", "recommonmark (==0.5.0)", "requests (==2.25.1)", "six (==1.15.0)", "snowballstemmer (==2.1.0)", "sphinx (==1.8.5)", "sphinx-rtd-theme (==0.4.3)", "sphinxcontrib-serializinghtml (==1.1.4)", "sphinxcontrib-websupport (==1.2.4)", "urllib3 (==1.26.4)"]
qa = ["flake8 (==5.0.4)", "mypy (==0.971)", "types-setuptools (==67.2.0.1)"]
testing = ["Django", "attrs", "colorama", "docopt", "pytest (<9.0.0)"]

[[package]]
name = "jinja2"
version = "3.1.5"
description = "A very fast and expressive template engine."
optional = false
python-versions = ">=3.7"
files = [
    {file = "jinja2-3.1.5-py3-none-any.whl", hash = "sha256:aba0f4dc9ed8013c424088f68a5c226f7d6097ed89b246d7749c2ec4175c6adb"},
    {file = "jinja2-3.1.5.tar.gz", hash = "sha256:8fefff8dc3034e27bb80d67c671eb8a9bc424c0ef4c0826edbff304cceff43bb"},
]

[package.dependencies]
MarkupSafe = ">=2.0"

[package.extras]
i18n = ["Babel (>=2.7)"]

[[package]]
name = "json5"
version = "0.10.0"
description = "A Python implementation of the JSON5 data format."
optional = false
python-versions = ">=3.8.0"
files = [
    {file = "json5-0.10.0-py3-none-any.whl", hash = "sha256:19b23410220a7271e8377f81ba8aacba2fdd56947fbb137ee5977cbe1f5e8dfa"},
    {file = "json5-0.10.0.tar.gz", hash = "sha256:e66941c8f0a02026943c52c2eb34ebeb2a6f819a0be05920a6f5243cd30fd559"},
]

[package.extras]
dev = ["build (==1.2.2.post1)", "coverage (==7.5.3)", "mypy (==1.13.0)", "pip (==24.3.1)", "pylint (==3.2.3)", "ruff (==0.7.3)", "twine (==5.1.1)", "uv (==0.5.1)"]

[[package]]
name = "jsonpointer"
version = "3.0.0"
description = "Identify specific nodes in a JSON document (RFC 6901)"
optional = false
python-versions = ">=3.7"
files = [
    {file = "jsonpointer-3.0.0-py2.py3-none-any.whl", hash = "sha256:13e088adc14fca8b6aa8177c044e12701e6ad4b28ff10e65f2267a90109c9942"},
    {file = "jsonpointer-3.0.0.tar.gz", hash = "sha256:2b2d729f2091522d61c3b31f82e11870f60b68f43fbc705cb76bf4b832af59ef"},
]

[[package]]
name = "jsonschema"
version = "4.23.0"
description = "An implementation of JSON Schema validation for Python"
optional = false
python-versions = ">=3.8"
files = [
    {file = "jsonschema-4.23.0-py3-none-any.whl", hash = "sha256:fbadb6f8b144a8f8cf9f0b89ba94501d143e50411a1278633f56a7acf7fd5566"},
    {file = "jsonschema-4.23.0.tar.gz", hash = "sha256:d71497fef26351a33265337fa77ffeb82423f3ea21283cd9467bb03999266bc4"},
]

[package.dependencies]
attrs = ">=22.2.0"
fqdn = {version = "*", optional = true, markers = "extra == \"format-nongpl\""}
idna = {version = "*", optional = true, markers = "extra == \"format-nongpl\""}
isoduration = {version = "*", optional = true, markers = "extra == \"format-nongpl\""}
jsonpointer = {version = ">1.13", optional = true, markers = "extra == \"format-nongpl\""}
jsonschema-specifications = ">=2023.03.6"
referencing = ">=0.28.4"
rfc3339-validator = {version = "*", optional = true, markers = "extra == \"format-nongpl\""}
rfc3986-validator = {version = ">0.1.0", optional = true, markers = "extra == \"format-nongpl\""}
rpds-py = ">=0.7.1"
uri-template = {version = "*", optional = true, markers = "extra == \"format-nongpl\""}
webcolors = {version = ">=24.6.0", optional = true, markers = "extra == \"format-nongpl\""}

[package.extras]
format = ["fqdn", "idna", "isoduration", "jsonpointer (>1.13)", "rfc3339-validator", "rfc3987", "uri-template", "webcolors (>=1.11)"]
format-nongpl = ["fqdn", "idna", "isoduration", "jsonpointer (>1.13)", "rfc3339-validator", "rfc3986-validator (>0.1.0)", "uri-template", "webcolors (>=24.6.0)"]

[[package]]
name = "jsonschema-specifications"
version = "2024.10.1"
description = "The JSON Schema meta-schemas and vocabularies, exposed as a Registry"
optional = false
python-versions = ">=3.9"
files = [
    {file = "jsonschema_specifications-2024.10.1-py3-none-any.whl", hash = "sha256:a09a0680616357d9a0ecf05c12ad234479f549239d0f5b55f3deea67475da9bf"},
    {file = "jsonschema_specifications-2024.10.1.tar.gz", hash = "sha256:0f38b83639958ce1152d02a7f062902c41c8fd20d558b0c34344292d417ae272"},
]

[package.dependencies]
referencing = ">=0.31.0"

[[package]]
name = "jupyter-client"
version = "8.6.3"
description = "Jupyter protocol implementation and client libraries"
optional = false
python-versions = ">=3.8"
files = [
    {file = "jupyter_client-8.6.3-py3-none-any.whl", hash = "sha256:e8a19cc986cc45905ac3362915f410f3af85424b4c0905e94fa5f2cb08e8f23f"},
    {file = "jupyter_client-8.6.3.tar.gz", hash = "sha256:35b3a0947c4a6e9d589eb97d7d4cd5e90f910ee73101611f01283732bd6d9419"},
]

[package.dependencies]
jupyter-core = ">=4.12,<5.0.dev0 || >=5.1.dev0"
python-dateutil = ">=2.8.2"
pyzmq = ">=23.0"
tornado = ">=6.2"
traitlets = ">=5.3"

[package.extras]
docs = ["ipykernel", "myst-parser", "pydata-sphinx-theme", "sphinx (>=4)", "sphinx-autodoc-typehints", "sphinxcontrib-github-alt", "sphinxcontrib-spelling"]
test = ["coverage", "ipykernel (>=6.14)", "mypy", "paramiko", "pre-commit", "pytest (<8.2.0)", "pytest-cov", "pytest-jupyter[client] (>=0.4.1)", "pytest-timeout"]

[[package]]
name = "jupyter-contrib-core"
version = "0.4.2"
description = "Common utilities for jupyter-contrib projects."
optional = false
python-versions = "*"
files = [
    {file = "jupyter_contrib_core-0.4.2.tar.gz", hash = "sha256:1887212f3ca9d4487d624c0705c20dfdf03d5a0b9ea2557d3aaeeb4c38bdcabb"},
]

[package.dependencies]
jupyter_core = "*"
notebook = ">=4.0"
setuptools = "*"
tornado = "*"
traitlets = "*"

[package.extras]
testing-utils = ["mock", "nose"]

[[package]]
name = "jupyter-contrib-nbextensions"
version = "0.7.0"
description = "A collection of Jupyter nbextensions."
optional = false
python-versions = "*"
files = [
    {file = "jupyter_contrib_nbextensions-0.7.0.tar.gz", hash = "sha256:06e33f005885eb92f89cbe82711e921278201298d08ab0d886d1ba09e8c3e9ca"},
]

[package.dependencies]
ipython_genutils = "*"
jupyter_contrib_core = ">=0.3.3"
jupyter_core = "*"
jupyter_highlight_selected_word = ">=0.1.1"
jupyter_nbextensions_configurator = ">=0.4.0"
lxml = "*"
nbconvert = ">=6.0"
notebook = ">=6.0"
tornado = "*"
traitlets = ">=4.1"

[package.extras]
test = ["mock", "nbformat", "nose", "pip", "requests"]

[[package]]
name = "jupyter-core"
version = "5.7.2"
description = "Jupyter core package. A base package on which Jupyter projects rely."
optional = false
python-versions = ">=3.8"
files = [
    {file = "jupyter_core-5.7.2-py3-none-any.whl", hash = "sha256:4f7315d2f6b4bcf2e3e7cb6e46772eba760ae459cd1f59d29eb57b0a01bd7409"},
    {file = "jupyter_core-5.7.2.tar.gz", hash = "sha256:aa5f8d32bbf6b431ac830496da7392035d6f61b4f54872f15c4bd2a9c3f536d9"},
]

[package.dependencies]
platformdirs = ">=2.5"
pywin32 = {version = ">=300", markers = "sys_platform == \"win32\" and platform_python_implementation != \"PyPy\""}
traitlets = ">=5.3"

[package.extras]
docs = ["myst-parser", "pydata-sphinx-theme", "sphinx-autodoc-typehints", "sphinxcontrib-github-alt", "sphinxcontrib-spelling", "traitlets"]
test = ["ipykernel", "pre-commit", "pytest (<8)", "pytest-cov", "pytest-timeout"]

[[package]]
name = "jupyter-events"
version = "0.11.0"
description = "Jupyter Event System library"
optional = false
python-versions = ">=3.9"
files = [
    {file = "jupyter_events-0.11.0-py3-none-any.whl", hash = "sha256:36399b41ce1ca45fe8b8271067d6a140ffa54cec4028e95491c93b78a855cacf"},
    {file = "jupyter_events-0.11.0.tar.gz", hash = "sha256:c0bc56a37aac29c1fbc3bcfbddb8c8c49533f9cf11f1c4e6adadba936574ab90"},
]

[package.dependencies]
jsonschema = {version = ">=4.18.0", extras = ["format-nongpl"]}
python-json-logger = ">=2.0.4"
pyyaml = ">=5.3"
referencing = "*"
rfc3339-validator = "*"
rfc3986-validator = ">=0.1.1"
traitlets = ">=5.3"

[package.extras]
cli = ["click", "rich"]
docs = ["jupyterlite-sphinx", "myst-parser", "pydata-sphinx-theme (>=0.16)", "sphinx (>=8)", "sphinxcontrib-spelling"]
test = ["click", "pre-commit", "pytest (>=7.0)", "pytest-asyncio (>=0.19.0)", "pytest-console-scripts", "rich"]

[[package]]
name = "jupyter-highlight-selected-word"
version = "0.2.0"
description = "Jupyter notebook extension that enables highlighting every instance of the current word in the notebook."
optional = false
python-versions = "*"
files = [
    {file = "jupyter_highlight_selected_word-0.2.0-py2.py3-none-any.whl", hash = "sha256:9545dfa9cb057eebe3a5795604dcd3a5294ea18637e553f61a0b67c1b5903c58"},
    {file = "jupyter_highlight_selected_word-0.2.0.tar.gz", hash = "sha256:9fa740424859a807950ca08d2bfd28a35154cd32dd6d50ac4e0950022adc0e7b"},
]

[[package]]
name = "jupyter-lsp"
version = "2.2.5"
description = "Multi-Language Server WebSocket proxy for Jupyter Notebook/Lab server"
optional = false
python-versions = ">=3.8"
files = [
    {file = "jupyter-lsp-2.2.5.tar.gz", hash = "sha256:793147a05ad446f809fd53ef1cd19a9f5256fd0a2d6b7ce943a982cb4f545001"},
    {file = "jupyter_lsp-2.2.5-py3-none-any.whl", hash = "sha256:45fbddbd505f3fbfb0b6cb2f1bc5e15e83ab7c79cd6e89416b248cb3c00c11da"},
]

[package.dependencies]
jupyter-server = ">=1.1.2"

[[package]]
name = "jupyter-nbextensions-configurator"
version = "0.6.4"
description = "jupyter serverextension providing configuration interfaces for nbextensions."
optional = false
python-versions = "*"
files = [
    {file = "jupyter_nbextensions_configurator-0.6.4-py2.py3-none-any.whl", hash = "sha256:fe7a7b0805b5926449692fb077e0e659bab8b27563bc68cba26854532fdf99c7"},
]

[package.dependencies]
jupyter-contrib-core = ">=0.3.3"
jupyter-core = "*"
jupyter-server = "*"
notebook = ">=6.0"
pyyaml = "*"
tornado = "*"
traitlets = "*"

[package.extras]
test = ["jupyter-contrib-core[testing-utils]", "mock", "nose", "requests", "selenium"]

[[package]]
name = "jupyter-server"
version = "2.15.0"
description = "The backend—i.e. core services, APIs, and REST endpoints—to Jupyter web applications."
optional = false
python-versions = ">=3.9"
files = [
    {file = "jupyter_server-2.15.0-py3-none-any.whl", hash = "sha256:872d989becf83517012ee669f09604aa4a28097c0bd90b2f424310156c2cdae3"},
    {file = "jupyter_server-2.15.0.tar.gz", hash = "sha256:9d446b8697b4f7337a1b7cdcac40778babdd93ba614b6d68ab1c0c918f1c4084"},
]

[package.dependencies]
anyio = ">=3.1.0"
argon2-cffi = ">=21.1"
jinja2 = ">=3.0.3"
jupyter-client = ">=7.4.4"
jupyter-core = ">=4.12,<5.0.dev0 || >=5.1.dev0"
jupyter-events = ">=0.11.0"
jupyter-server-terminals = ">=0.4.4"
nbconvert = ">=6.4.4"
nbformat = ">=5.3.0"
overrides = ">=5.0"
packaging = ">=22.0"
prometheus-client = ">=0.9"
pywinpty = {version = ">=2.0.1", markers = "os_name == \"nt\""}
pyzmq = ">=24"
send2trash = ">=1.8.2"
terminado = ">=0.8.3"
tornado = ">=6.2.0"
traitlets = ">=5.6.0"
websocket-client = ">=1.7"

[package.extras]
docs = ["ipykernel", "jinja2", "jupyter-client", "myst-parser", "nbformat", "prometheus-client", "pydata-sphinx-theme", "send2trash", "sphinx-autodoc-typehints", "sphinxcontrib-github-alt", "sphinxcontrib-openapi (>=0.8.0)", "sphinxcontrib-spelling", "sphinxemoji", "tornado", "typing-extensions"]
test = ["flaky", "ipykernel", "pre-commit", "pytest (>=7.0,<9)", "pytest-console-scripts", "pytest-jupyter[server] (>=0.7)", "pytest-timeout", "requests"]

[[package]]
name = "jupyter-server-mathjax"
version = "0.2.6"
description = "MathJax resources as a Jupyter Server Extension."
optional = false
python-versions = ">=3.7"
files = [
    {file = "jupyter_server_mathjax-0.2.6-py3-none-any.whl", hash = "sha256:416389dde2010df46d5fbbb7adb087a5607111070af65a1445391040f2babb5e"},
    {file = "jupyter_server_mathjax-0.2.6.tar.gz", hash = "sha256:bb1e6b6dc0686c1fe386a22b5886163db548893a99c2810c36399e9c4ca23943"},
]

[package.dependencies]
jupyter-server = ">=1.1"

[package.extras]
test = ["jupyter-server[test]", "pytest"]

[[package]]
name = "jupyter-server-terminals"
version = "0.5.3"
description = "A Jupyter Server Extension Providing Terminals."
optional = false
python-versions = ">=3.8"
files = [
    {file = "jupyter_server_terminals-0.5.3-py3-none-any.whl", hash = "sha256:41ee0d7dc0ebf2809c668e0fc726dfaf258fcd3e769568996ca731b6194ae9aa"},
    {file = "jupyter_server_terminals-0.5.3.tar.gz", hash = "sha256:5ae0295167220e9ace0edcfdb212afd2b01ee8d179fe6f23c899590e9b8a5269"},
]

[package.dependencies]
pywinpty = {version = ">=2.0.3", markers = "os_name == \"nt\""}
terminado = ">=0.8.3"

[package.extras]
docs = ["jinja2", "jupyter-server", "mistune (<4.0)", "myst-parser", "nbformat", "packaging", "pydata-sphinx-theme", "sphinxcontrib-github-alt", "sphinxcontrib-openapi", "sphinxcontrib-spelling", "sphinxemoji", "tornado"]
test = ["jupyter-server (>=2.0.0)", "pytest (>=7.0)", "pytest-jupyter[server] (>=0.5.3)", "pytest-timeout"]

[[package]]
name = "jupyterlab"
version = "4.3.4"
description = "JupyterLab computational environment"
optional = false
python-versions = ">=3.8"
files = [
    {file = "jupyterlab-4.3.4-py3-none-any.whl", hash = "sha256:b754c2601c5be6adf87cb5a1d8495d653ffb945f021939f77776acaa94dae952"},
    {file = "jupyterlab-4.3.4.tar.gz", hash = "sha256:f0bb9b09a04766e3423cccc2fc23169aa2ffedcdf8713e9e0fb33cac0b6859d0"},
]

[package.dependencies]
async-lru = ">=1.0.0"
httpx = ">=0.25.0"
ipykernel = ">=6.5.0"
jinja2 = ">=3.0.3"
jupyter-core = "*"
jupyter-lsp = ">=2.0.0"
jupyter-server = ">=2.4.0,<3"
jupyterlab-server = ">=2.27.1,<3"
notebook-shim = ">=0.2"
packaging = "*"
setuptools = ">=40.8.0"
tornado = ">=6.2.0"
traitlets = "*"

[package.extras]
dev = ["build", "bump2version", "coverage", "hatch", "pre-commit", "pytest-cov", "ruff (==0.6.9)"]
docs = ["jsx-lexer", "myst-parser", "pydata-sphinx-theme (>=0.13.0)", "pytest", "pytest-check-links", "pytest-jupyter", "sphinx (>=1.8,<8.1.0)", "sphinx-copybutton"]
docs-screenshots = ["altair (==5.4.1)", "ipython (==8.16.1)", "ipywidgets (==8.1.5)", "jupyterlab-geojson (==3.4.0)", "jupyterlab-language-pack-zh-cn (==4.2.post3)", "matplotlib (==3.9.2)", "nbconvert (>=7.0.0)", "pandas (==2.2.3)", "scipy (==1.14.1)", "vega-datasets (==0.9.0)"]
test = ["coverage", "pytest (>=7.0)", "pytest-check-links (>=0.7)", "pytest-console-scripts", "pytest-cov", "pytest-jupyter (>=0.5.3)", "pytest-timeout", "pytest-tornasync", "requests", "requests-cache", "virtualenv"]
upgrade-extension = ["copier (>=9,<10)", "jinja2-time (<0.3)", "pydantic (<3.0)", "pyyaml-include (<3.0)", "tomli-w (<2.0)"]

[[package]]
name = "jupyterlab-git"
version = "0.50.2"
description = "A JupyterLab extension for version control using git"
optional = false
python-versions = ">=3.8"
files = [
    {file = "jupyterlab_git-0.50.2-py3-none-any.whl", hash = "sha256:059114d19fcb5560f82914b070ed7654fab62392e3c1fdd5946c5dc460ae3697"},
    {file = "jupyterlab_git-0.50.2.tar.gz", hash = "sha256:ceefdc85632caf499a048b434459d03b5e621665801e9db7197bd486f5d33433"},
]

[package.dependencies]
jupyter-server = ">=2.0.1,<3"
nbdime = ">=4.0.1,<4.1.0"
nbformat = "*"
packaging = "*"
pexpect = "*"
traitlets = ">=5.0,<6.0"

[package.extras]
dev = ["black", "jupyterlab (>=4.0,<5.0)", "pre-commit"]
test = ["coverage", "jupytext", "pytest", "pytest-asyncio", "pytest-cov", "pytest-jupyter[server] (>=0.6.0)"]
ui-tests = ["jupyter-archive"]

[[package]]
name = "jupyterlab-github"
version = "4.0.0"
description = "JupyterLab viewer for GitHub repositories"
optional = false
python-versions = ">=3.8"
files = [
    {file = "jupyterlab_github-4.0.0-py3-none-any.whl", hash = "sha256:7189533c3d50f7aa11729ffc3302dd10e3783a526a719b292249cce92a034b79"},
    {file = "jupyterlab_github-4.0.0.tar.gz", hash = "sha256:fab84e981684753f35bc5b55f8e84a30ffbe5b642d7b2d2721e4a048896afa54"},
]

[package.dependencies]
jupyterlab = ">=4.0.0,<5"

[[package]]
name = "jupyterlab-pygments"
version = "0.3.0"
description = "Pygments theme using JupyterLab CSS variables"
optional = false
python-versions = ">=3.8"
files = [
    {file = "jupyterlab_pygments-0.3.0-py3-none-any.whl", hash = "sha256:841a89020971da1d8693f1a99997aefc5dc424bb1b251fd6322462a1b8842780"},
    {file = "jupyterlab_pygments-0.3.0.tar.gz", hash = "sha256:721aca4d9029252b11cfa9d185e5b5af4d54772bb8072f9b7036f4170054d35d"},
]

[[package]]
name = "jupyterlab-server"
version = "2.27.3"
description = "A set of server components for JupyterLab and JupyterLab like applications."
optional = false
python-versions = ">=3.8"
files = [
    {file = "jupyterlab_server-2.27.3-py3-none-any.whl", hash = "sha256:e697488f66c3db49df675158a77b3b017520d772c6e1548c7d9bcc5df7944ee4"},
    {file = "jupyterlab_server-2.27.3.tar.gz", hash = "sha256:eb36caca59e74471988f0ae25c77945610b887f777255aa21f8065def9e51ed4"},
]

[package.dependencies]
babel = ">=2.10"
jinja2 = ">=3.0.3"
json5 = ">=0.9.0"
jsonschema = ">=4.18.0"
jupyter-server = ">=1.21,<3"
packaging = ">=21.3"
requests = ">=2.31"

[package.extras]
docs = ["autodoc-traits", "jinja2 (<3.2.0)", "mistune (<4)", "myst-parser", "pydata-sphinx-theme", "sphinx", "sphinx-copybutton", "sphinxcontrib-openapi (>0.8)"]
openapi = ["openapi-core (>=0.18.0,<0.19.0)", "ruamel-yaml"]
test = ["hatch", "ipykernel", "openapi-core (>=0.18.0,<0.19.0)", "openapi-spec-validator (>=0.6.0,<0.8.0)", "pytest (>=7.0,<8)", "pytest-console-scripts", "pytest-cov", "pytest-jupyter[server] (>=0.6.2)", "pytest-timeout", "requests-mock", "ruamel-yaml", "sphinxcontrib-spelling", "strict-rfc3339", "werkzeug"]

[[package]]
name = "jupyterlab-widgets"
version = "3.0.13"
description = "Jupyter interactive widgets for JupyterLab"
optional = false
python-versions = ">=3.7"
files = [
    {file = "jupyterlab_widgets-3.0.13-py3-none-any.whl", hash = "sha256:e3cda2c233ce144192f1e29914ad522b2f4c40e77214b0cc97377ca3d323db54"},
    {file = "jupyterlab_widgets-3.0.13.tar.gz", hash = "sha256:a2966d385328c1942b683a8cd96b89b8dd82c8b8f81dda902bb2bc06d46f5bed"},
]

[[package]]
name = "kiwisolver"
version = "1.4.8"
description = "A fast implementation of the Cassowary constraint solver"
optional = false
python-versions = ">=3.10"
files = [
    {file = "kiwisolver-1.4.8-cp310-cp310-macosx_10_9_universal2.whl", hash = "sha256:88c6f252f6816a73b1f8c904f7bbe02fd67c09a69f7cb8a0eecdbf5ce78e63db"},
    {file = "kiwisolver-1.4.8-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:c72941acb7b67138f35b879bbe85be0f6c6a70cab78fe3ef6db9c024d9223e5b"},
    {file = "kiwisolver-1.4.8-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:ce2cf1e5688edcb727fdf7cd1bbd0b6416758996826a8be1d958f91880d0809d"},
    {file = "kiwisolver-1.4.8-cp310-cp310-manylinux_2_12_i686.manylinux2010_i686.whl", hash = "sha256:c8bf637892dc6e6aad2bc6d4d69d08764166e5e3f69d469e55427b6ac001b19d"},
    {file = "kiwisolver-1.4.8-cp310-cp310-manylinux_2_12_x86_64.manylinux2010_x86_64.whl", hash = "sha256:034d2c891f76bd3edbdb3ea11140d8510dca675443da7304205a2eaa45d8334c"},
    {file = "kiwisolver-1.4.8-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:d47b28d1dfe0793d5e96bce90835e17edf9a499b53969b03c6c47ea5985844c3"},
    {file = "kiwisolver-1.4.8-cp310-cp310-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:eb158fe28ca0c29f2260cca8c43005329ad58452c36f0edf298204de32a9a3ed"},
    {file = "kiwisolver-1.4.8-cp310-cp310-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:d5536185fce131780ebd809f8e623bf4030ce1b161353166c49a3c74c287897f"},
    {file = "kiwisolver-1.4.8-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:369b75d40abedc1da2c1f4de13f3482cb99e3237b38726710f4a793432b1c5ff"},
    {file = "kiwisolver-1.4.8-cp310-cp310-musllinux_1_2_i686.whl", hash = "sha256:641f2ddf9358c80faa22e22eb4c9f54bd3f0e442e038728f500e3b978d00aa7d"},
    {file = "kiwisolver-1.4.8-cp310-cp310-musllinux_1_2_ppc64le.whl", hash = "sha256:d561d2d8883e0819445cfe58d7ddd673e4015c3c57261d7bdcd3710d0d14005c"},
    {file = "kiwisolver-1.4.8-cp310-cp310-musllinux_1_2_s390x.whl", hash = "sha256:1732e065704b47c9afca7ffa272f845300a4eb959276bf6970dc07265e73b605"},
    {file = "kiwisolver-1.4.8-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:bcb1ebc3547619c3b58a39e2448af089ea2ef44b37988caf432447374941574e"},
    {file = "kiwisolver-1.4.8-cp310-cp310-win_amd64.whl", hash = "sha256:89c107041f7b27844179ea9c85d6da275aa55ecf28413e87624d033cf1f6b751"},
    {file = "kiwisolver-1.4.8-cp310-cp310-win_arm64.whl", hash = "sha256:b5773efa2be9eb9fcf5415ea3ab70fc785d598729fd6057bea38d539ead28271"},
    {file = "kiwisolver-1.4.8-cp311-cp311-macosx_10_9_universal2.whl", hash = "sha256:a4d3601908c560bdf880f07d94f31d734afd1bb71e96585cace0e38ef44c6d84"},
    {file = "kiwisolver-1.4.8-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:856b269c4d28a5c0d5e6c1955ec36ebfd1651ac00e1ce0afa3e28da95293b561"},
    {file = "kiwisolver-1.4.8-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:c2b9a96e0f326205af81a15718a9073328df1173a2619a68553decb7097fd5d7"},
    {file = "kiwisolver-1.4.8-cp311-cp311-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:c5020c83e8553f770cb3b5fc13faac40f17e0b205bd237aebd21d53d733adb03"},
    {file = "kiwisolver-1.4.8-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:dace81d28c787956bfbfbbfd72fdcef014f37d9b48830829e488fdb32b49d954"},
    {file = "kiwisolver-1.4.8-cp311-cp311-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:11e1022b524bd48ae56c9b4f9296bce77e15a2e42a502cceba602f804b32bb79"},
    {file = "kiwisolver-1.4.8-cp311-cp311-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:3b9b4d2892fefc886f30301cdd80debd8bb01ecdf165a449eb6e78f79f0fabd6"},
    {file = "kiwisolver-1.4.8-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:3a96c0e790ee875d65e340ab383700e2b4891677b7fcd30a699146f9384a2bb0"},
    {file = "kiwisolver-1.4.8-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:23454ff084b07ac54ca8be535f4174170c1094a4cff78fbae4f73a4bcc0d4dab"},
    {file = "kiwisolver-1.4.8-cp311-cp311-musllinux_1_2_i686.whl", hash = "sha256:87b287251ad6488e95b4f0b4a79a6d04d3ea35fde6340eb38fbd1ca9cd35bbbc"},
    {file = "kiwisolver-1.4.8-cp311-cp311-musllinux_1_2_ppc64le.whl", hash = "sha256:b21dbe165081142b1232a240fc6383fd32cdd877ca6cc89eab93e5f5883e1c25"},
    {file = "kiwisolver-1.4.8-cp311-cp311-musllinux_1_2_s390x.whl", hash = "sha256:768cade2c2df13db52475bd28d3a3fac8c9eff04b0e9e2fda0f3760f20b3f7fc"},
    {file = "kiwisolver-1.4.8-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:d47cfb2650f0e103d4bf68b0b5804c68da97272c84bb12850d877a95c056bd67"},
    {file = "kiwisolver-1.4.8-cp311-cp311-win_amd64.whl", hash = "sha256:ed33ca2002a779a2e20eeb06aea7721b6e47f2d4b8a8ece979d8ba9e2a167e34"},
    {file = "kiwisolver-1.4.8-cp311-cp311-win_arm64.whl", hash = "sha256:16523b40aab60426ffdebe33ac374457cf62863e330a90a0383639ce14bf44b2"},
    {file = "kiwisolver-1.4.8-cp312-cp312-macosx_10_13_universal2.whl", hash = "sha256:d6af5e8815fd02997cb6ad9bbed0ee1e60014438ee1a5c2444c96f87b8843502"},
    {file = "kiwisolver-1.4.8-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:bade438f86e21d91e0cf5dd7c0ed00cda0f77c8c1616bd83f9fc157fa6760d31"},
    {file = "kiwisolver-1.4.8-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:b83dc6769ddbc57613280118fb4ce3cd08899cc3369f7d0e0fab518a7cf37fdb"},
    {file = "kiwisolver-1.4.8-cp312-cp312-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:111793b232842991be367ed828076b03d96202c19221b5ebab421ce8bcad016f"},
    {file = "kiwisolver-1.4.8-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:257af1622860e51b1a9d0ce387bf5c2c4f36a90594cb9514f55b074bcc787cfc"},
    {file = "kiwisolver-1.4.8-cp312-cp312-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:69b5637c3f316cab1ec1c9a12b8c5f4750a4c4b71af9157645bf32830e39c03a"},
    {file = "kiwisolver-1.4.8-cp312-cp312-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:782bb86f245ec18009890e7cb8d13a5ef54dcf2ebe18ed65f795e635a96a1c6a"},
    {file = "kiwisolver-1.4.8-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:cc978a80a0db3a66d25767b03688f1147a69e6237175c0f4ffffaaedf744055a"},
    {file = "kiwisolver-1.4.8-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:36dbbfd34838500a31f52c9786990d00150860e46cd5041386f217101350f0d3"},
    {file = "kiwisolver-1.4.8-cp312-cp312-musllinux_1_2_i686.whl", hash = "sha256:eaa973f1e05131de5ff3569bbba7f5fd07ea0595d3870ed4a526d486fe57fa1b"},
    {file = "kiwisolver-1.4.8-cp312-cp312-musllinux_1_2_ppc64le.whl", hash = "sha256:a66f60f8d0c87ab7f59b6fb80e642ebb29fec354a4dfad687ca4092ae69d04f4"},
    {file = "kiwisolver-1.4.8-cp312-cp312-musllinux_1_2_s390x.whl", hash = "sha256:858416b7fb777a53f0c59ca08190ce24e9abbd3cffa18886a5781b8e3e26f65d"},
    {file = "kiwisolver-1.4.8-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:085940635c62697391baafaaeabdf3dd7a6c3643577dde337f4d66eba021b2b8"},
    {file = "kiwisolver-1.4.8-cp312-cp312-win_amd64.whl", hash = "sha256:01c3d31902c7db5fb6182832713d3b4122ad9317c2c5877d0539227d96bb2e50"},
    {file = "kiwisolver-1.4.8-cp312-cp312-win_arm64.whl", hash = "sha256:a3c44cb68861de93f0c4a8175fbaa691f0aa22550c331fefef02b618a9dcb476"},
    {file = "kiwisolver-1.4.8-cp313-cp313-macosx_10_13_universal2.whl", hash = "sha256:1c8ceb754339793c24aee1c9fb2485b5b1f5bb1c2c214ff13368431e51fc9a09"},
    {file = "kiwisolver-1.4.8-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:54a62808ac74b5e55a04a408cda6156f986cefbcf0ada13572696b507cc92fa1"},
    {file = "kiwisolver-1.4.8-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:68269e60ee4929893aad82666821aaacbd455284124817af45c11e50a4b42e3c"},
    {file = "kiwisolver-1.4.8-cp313-cp313-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:34d142fba9c464bc3bbfeff15c96eab0e7310343d6aefb62a79d51421fcc5f1b"},
    {file = "kiwisolver-1.4.8-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:3ddc373e0eef45b59197de815b1b28ef89ae3955e7722cc9710fb91cd77b7f47"},
    {file = "kiwisolver-1.4.8-cp313-cp313-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:77e6f57a20b9bd4e1e2cedda4d0b986ebd0216236f0106e55c28aea3d3d69b16"},
    {file = "kiwisolver-1.4.8-cp313-cp313-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:08e77738ed7538f036cd1170cbed942ef749137b1311fa2bbe2a7fda2f6bf3cc"},
    {file = "kiwisolver-1.4.8-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:a5ce1e481a74b44dd5e92ff03ea0cb371ae7a0268318e202be06c8f04f4f1246"},
    {file = "kiwisolver-1.4.8-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:fc2ace710ba7c1dfd1a3b42530b62b9ceed115f19a1656adefce7b1782a37794"},
    {file = "kiwisolver-1.4.8-cp313-cp313-musllinux_1_2_i686.whl", hash = "sha256:3452046c37c7692bd52b0e752b87954ef86ee2224e624ef7ce6cb21e8c41cc1b"},
    {file = "kiwisolver-1.4.8-cp313-cp313-musllinux_1_2_ppc64le.whl", hash = "sha256:7e9a60b50fe8b2ec6f448fe8d81b07e40141bfced7f896309df271a0b92f80f3"},
    {file = "kiwisolver-1.4.8-cp313-cp313-musllinux_1_2_s390x.whl", hash = "sha256:918139571133f366e8362fa4a297aeba86c7816b7ecf0bc79168080e2bd79957"},
    {file = "kiwisolver-1.4.8-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:e063ef9f89885a1d68dd8b2e18f5ead48653176d10a0e324e3b0030e3a69adeb"},
    {file = "kiwisolver-1.4.8-cp313-cp313-win_amd64.whl", hash = "sha256:a17b7c4f5b2c51bb68ed379defd608a03954a1845dfed7cc0117f1cc8a9b7fd2"},
    {file = "kiwisolver-1.4.8-cp313-cp313-win_arm64.whl", hash = "sha256:3cd3bc628b25f74aedc6d374d5babf0166a92ff1317f46267f12d2ed54bc1d30"},
    {file = "kiwisolver-1.4.8-cp313-cp313t-macosx_10_13_universal2.whl", hash = "sha256:370fd2df41660ed4e26b8c9d6bbcad668fbe2560462cba151a721d49e5b6628c"},
    {file = "kiwisolver-1.4.8-cp313-cp313t-macosx_10_13_x86_64.whl", hash = "sha256:84a2f830d42707de1d191b9490ac186bf7997a9495d4e9072210a1296345f7dc"},
    {file = "kiwisolver-1.4.8-cp313-cp313t-macosx_11_0_arm64.whl", hash = "sha256:7a3ad337add5148cf51ce0b55642dc551c0b9d6248458a757f98796ca7348712"},
    {file = "kiwisolver-1.4.8-cp313-cp313t-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:7506488470f41169b86d8c9aeff587293f530a23a23a49d6bc64dab66bedc71e"},
    {file = "kiwisolver-1.4.8-cp313-cp313t-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:2f0121b07b356a22fb0414cec4666bbe36fd6d0d759db3d37228f496ed67c880"},
    {file = "kiwisolver-1.4.8-cp313-cp313t-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:d6d6bd87df62c27d4185de7c511c6248040afae67028a8a22012b010bc7ad062"},
    {file = "kiwisolver-1.4.8-cp313-cp313t-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:291331973c64bb9cce50bbe871fb2e675c4331dab4f31abe89f175ad7679a4d7"},
    {file = "kiwisolver-1.4.8-cp313-cp313t-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:893f5525bb92d3d735878ec00f781b2de998333659507d29ea4466208df37bed"},
    {file = "kiwisolver-1.4.8-cp313-cp313t-musllinux_1_2_aarch64.whl", hash = "sha256:b47a465040146981dc9db8647981b8cb96366fbc8d452b031e4f8fdffec3f26d"},
    {file = "kiwisolver-1.4.8-cp313-cp313t-musllinux_1_2_i686.whl", hash = "sha256:99cea8b9dd34ff80c521aef46a1dddb0dcc0283cf18bde6d756f1e6f31772165"},
    {file = "kiwisolver-1.4.8-cp313-cp313t-musllinux_1_2_ppc64le.whl", hash = "sha256:151dffc4865e5fe6dafce5480fab84f950d14566c480c08a53c663a0020504b6"},
    {file = "kiwisolver-1.4.8-cp313-cp313t-musllinux_1_2_s390x.whl", hash = "sha256:577facaa411c10421314598b50413aa1ebcf5126f704f1e5d72d7e4e9f020d90"},
    {file = "kiwisolver-1.4.8-cp313-cp313t-musllinux_1_2_x86_64.whl", hash = "sha256:be4816dc51c8a471749d664161b434912eee82f2ea66bd7628bd14583a833e85"},
    {file = "kiwisolver-1.4.8-pp310-pypy310_pp73-macosx_10_15_x86_64.whl", hash = "sha256:e7a019419b7b510f0f7c9dceff8c5eae2392037eae483a7f9162625233802b0a"},
    {file = "kiwisolver-1.4.8-pp310-pypy310_pp73-macosx_11_0_arm64.whl", hash = "sha256:286b18e86682fd2217a48fc6be6b0f20c1d0ed10958d8dc53453ad58d7be0bf8"},
    {file = "kiwisolver-1.4.8-pp310-pypy310_pp73-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:4191ee8dfd0be1c3666ccbac178c5a05d5f8d689bbe3fc92f3c4abec817f8fe0"},
    {file = "kiwisolver-1.4.8-pp310-pypy310_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:7cd2785b9391f2873ad46088ed7599a6a71e762e1ea33e87514b1a441ed1da1c"},
    {file = "kiwisolver-1.4.8-pp310-pypy310_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:c07b29089b7ba090b6f1a669f1411f27221c3662b3a1b7010e67b59bb5a6f10b"},
    {file = "kiwisolver-1.4.8-pp310-pypy310_pp73-win_amd64.whl", hash = "sha256:65ea09a5a3faadd59c2ce96dc7bf0f364986a315949dc6374f04396b0d60e09b"},
    {file = "kiwisolver-1.4.8.tar.gz", hash = "sha256:23d5f023bdc8c7e54eb65f03ca5d5bb25b601eac4d7f1a042888a1f45237987e"},
]

[[package]]
name = "loguru"
version = "0.7.3"
description = "Python logging made (stupidly) simple"
optional = false
python-versions = "<4.0,>=3.5"
files = [
    {file = "loguru-0.7.3-py3-none-any.whl", hash = "sha256:31a33c10c8e1e10422bfd431aeb5d351c7cf7fa671e3c4df004162264b28220c"},
    {file = "loguru-0.7.3.tar.gz", hash = "sha256:19480589e77d47b8d85b2c827ad95d49bf31b0dcde16593892eb51dd18706eb6"},
]

[package.dependencies]
colorama = {version = ">=0.3.4", markers = "sys_platform == \"win32\""}
win32-setctime = {version = ">=1.0.0", markers = "sys_platform == \"win32\""}

[package.extras]
dev = ["Sphinx (==8.1.3)", "build (==1.2.2)", "colorama (==0.4.5)", "colorama (==0.4.6)", "exceptiongroup (==1.1.3)", "freezegun (==1.1.0)", "freezegun (==1.5.0)", "mypy (==v0.910)", "mypy (==v0.971)", "mypy (==v1.13.0)", "mypy (==v1.4.1)", "myst-parser (==4.0.0)", "pre-commit (==4.0.1)", "pytest (==6.1.2)", "pytest (==8.3.2)", "pytest-cov (==2.12.1)", "pytest-cov (==5.0.0)", "pytest-cov (==6.0.0)", "pytest-mypy-plugins (==1.9.3)", "pytest-mypy-plugins (==3.1.0)", "sphinx-rtd-theme (==3.0.2)", "tox (==3.27.1)", "tox (==4.23.2)", "twine (==6.0.1)"]

[[package]]
name = "lxml"
version = "5.3.0"
description = "Powerful and Pythonic XML processing library combining libxml2/libxslt with the ElementTree API."
optional = false
python-versions = ">=3.6"
files = [
    {file = "lxml-5.3.0-cp310-cp310-macosx_10_9_universal2.whl", hash = "sha256:dd36439be765e2dde7660212b5275641edbc813e7b24668831a5c8ac91180656"},
    {file = "lxml-5.3.0-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:ae5fe5c4b525aa82b8076c1a59d642c17b6e8739ecf852522c6321852178119d"},
    {file = "lxml-5.3.0-cp310-cp310-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:501d0d7e26b4d261fca8132854d845e4988097611ba2531408ec91cf3fd9d20a"},
    {file = "lxml-5.3.0-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:fb66442c2546446944437df74379e9cf9e9db353e61301d1a0e26482f43f0dd8"},
    {file = "lxml-5.3.0-cp310-cp310-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:9e41506fec7a7f9405b14aa2d5c8abbb4dbbd09d88f9496958b6d00cb4d45330"},
    {file = "lxml-5.3.0-cp310-cp310-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:f7d4a670107d75dfe5ad080bed6c341d18c4442f9378c9f58e5851e86eb79965"},
    {file = "lxml-5.3.0-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:41ce1f1e2c7755abfc7e759dc34d7d05fd221723ff822947132dc934d122fe22"},
    {file = "lxml-5.3.0-cp310-cp310-manylinux_2_28_aarch64.whl", hash = "sha256:44264ecae91b30e5633013fb66f6ddd05c006d3e0e884f75ce0b4755b3e3847b"},
    {file = "lxml-5.3.0-cp310-cp310-manylinux_2_28_ppc64le.whl", hash = "sha256:3c174dc350d3ec52deb77f2faf05c439331d6ed5e702fc247ccb4e6b62d884b7"},
    {file = "lxml-5.3.0-cp310-cp310-manylinux_2_28_s390x.whl", hash = "sha256:2dfab5fa6a28a0b60a20638dc48e6343c02ea9933e3279ccb132f555a62323d8"},
    {file = "lxml-5.3.0-cp310-cp310-manylinux_2_28_x86_64.whl", hash = "sha256:b1c8c20847b9f34e98080da785bb2336ea982e7f913eed5809e5a3c872900f32"},
    {file = "lxml-5.3.0-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:2c86bf781b12ba417f64f3422cfc302523ac9cd1d8ae8c0f92a1c66e56ef2e86"},
    {file = "lxml-5.3.0-cp310-cp310-musllinux_1_2_ppc64le.whl", hash = "sha256:c162b216070f280fa7da844531169be0baf9ccb17263cf5a8bf876fcd3117fa5"},
    {file = "lxml-5.3.0-cp310-cp310-musllinux_1_2_s390x.whl", hash = "sha256:36aef61a1678cb778097b4a6eeae96a69875d51d1e8f4d4b491ab3cfb54b5a03"},
    {file = "lxml-5.3.0-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:f65e5120863c2b266dbcc927b306c5b78e502c71edf3295dfcb9501ec96e5fc7"},
    {file = "lxml-5.3.0-cp310-cp310-win32.whl", hash = "sha256:ef0c1fe22171dd7c7c27147f2e9c3e86f8bdf473fed75f16b0c2e84a5030ce80"},
    {file = "lxml-5.3.0-cp310-cp310-win_amd64.whl", hash = "sha256:052d99051e77a4f3e8482c65014cf6372e61b0a6f4fe9edb98503bb5364cfee3"},
    {file = "lxml-5.3.0-cp311-cp311-macosx_10_9_universal2.whl", hash = "sha256:74bcb423462233bc5d6066e4e98b0264e7c1bed7541fff2f4e34fe6b21563c8b"},
    {file = "lxml-5.3.0-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:a3d819eb6f9b8677f57f9664265d0a10dd6551d227afb4af2b9cd7bdc2ccbf18"},
    {file = "lxml-5.3.0-cp311-cp311-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:5b8f5db71b28b8c404956ddf79575ea77aa8b1538e8b2ef9ec877945b3f46442"},
    {file = "lxml-5.3.0-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:2c3406b63232fc7e9b8783ab0b765d7c59e7c59ff96759d8ef9632fca27c7ee4"},
    {file = "lxml-5.3.0-cp311-cp311-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:2ecdd78ab768f844c7a1d4a03595038c166b609f6395e25af9b0f3f26ae1230f"},
    {file = "lxml-5.3.0-cp311-cp311-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:168f2dfcfdedf611eb285efac1516c8454c8c99caf271dccda8943576b67552e"},
    {file = "lxml-5.3.0-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:aa617107a410245b8660028a7483b68e7914304a6d4882b5ff3d2d3eb5948d8c"},
    {file = "lxml-5.3.0-cp311-cp311-manylinux_2_28_aarch64.whl", hash = "sha256:69959bd3167b993e6e710b99051265654133a98f20cec1d9b493b931942e9c16"},
    {file = "lxml-5.3.0-cp311-cp311-manylinux_2_28_ppc64le.whl", hash = "sha256:bd96517ef76c8654446fc3db9242d019a1bb5fe8b751ba414765d59f99210b79"},
    {file = "lxml-5.3.0-cp311-cp311-manylinux_2_28_s390x.whl", hash = "sha256:ab6dd83b970dc97c2d10bc71aa925b84788c7c05de30241b9e96f9b6d9ea3080"},
    {file = "lxml-5.3.0-cp311-cp311-manylinux_2_28_x86_64.whl", hash = "sha256:eec1bb8cdbba2925bedc887bc0609a80e599c75b12d87ae42ac23fd199445654"},
    {file = "lxml-5.3.0-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:6a7095eeec6f89111d03dabfe5883a1fd54da319c94e0fb104ee8f23616b572d"},
    {file = "lxml-5.3.0-cp311-cp311-musllinux_1_2_ppc64le.whl", hash = "sha256:6f651ebd0b21ec65dfca93aa629610a0dbc13dbc13554f19b0113da2e61a4763"},
    {file = "lxml-5.3.0-cp311-cp311-musllinux_1_2_s390x.whl", hash = "sha256:f422a209d2455c56849442ae42f25dbaaba1c6c3f501d58761c619c7836642ec"},
    {file = "lxml-5.3.0-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:62f7fdb0d1ed2065451f086519865b4c90aa19aed51081979ecd05a21eb4d1be"},
    {file = "lxml-5.3.0-cp311-cp311-win32.whl", hash = "sha256:c6379f35350b655fd817cd0d6cbeef7f265f3ae5fedb1caae2eb442bbeae9ab9"},
    {file = "lxml-5.3.0-cp311-cp311-win_amd64.whl", hash = "sha256:9c52100e2c2dbb0649b90467935c4b0de5528833c76a35ea1a2691ec9f1ee7a1"},
    {file = "lxml-5.3.0-cp312-cp312-macosx_10_9_universal2.whl", hash = "sha256:e99f5507401436fdcc85036a2e7dc2e28d962550afe1cbfc07c40e454256a859"},
    {file = "lxml-5.3.0-cp312-cp312-macosx_10_9_x86_64.whl", hash = "sha256:384aacddf2e5813a36495233b64cb96b1949da72bef933918ba5c84e06af8f0e"},
    {file = "lxml-5.3.0-cp312-cp312-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:874a216bf6afaf97c263b56371434e47e2c652d215788396f60477540298218f"},
    {file = "lxml-5.3.0-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:65ab5685d56914b9a2a34d67dd5488b83213d680b0c5d10b47f81da5a16b0b0e"},
    {file = "lxml-5.3.0-cp312-cp312-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:aac0bbd3e8dd2d9c45ceb82249e8bdd3ac99131a32b4d35c8af3cc9db1657179"},
    {file = "lxml-5.3.0-cp312-cp312-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:b369d3db3c22ed14c75ccd5af429086f166a19627e84a8fdade3f8f31426e52a"},
    {file = "lxml-5.3.0-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:c24037349665434f375645fa9d1f5304800cec574d0310f618490c871fd902b3"},
    {file = "lxml-5.3.0-cp312-cp312-manylinux_2_28_aarch64.whl", hash = "sha256:62d172f358f33a26d6b41b28c170c63886742f5b6772a42b59b4f0fa10526cb1"},
    {file = "lxml-5.3.0-cp312-cp312-manylinux_2_28_ppc64le.whl", hash = "sha256:c1f794c02903c2824fccce5b20c339a1a14b114e83b306ff11b597c5f71a1c8d"},
    {file = "lxml-5.3.0-cp312-cp312-manylinux_2_28_s390x.whl", hash = "sha256:5d6a6972b93c426ace71e0be9a6f4b2cfae9b1baed2eed2006076a746692288c"},
    {file = "lxml-5.3.0-cp312-cp312-manylinux_2_28_x86_64.whl", hash = "sha256:3879cc6ce938ff4eb4900d901ed63555c778731a96365e53fadb36437a131a99"},
    {file = "lxml-5.3.0-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:74068c601baff6ff021c70f0935b0c7bc528baa8ea210c202e03757c68c5a4ff"},
    {file = "lxml-5.3.0-cp312-cp312-musllinux_1_2_ppc64le.whl", hash = "sha256:ecd4ad8453ac17bc7ba3868371bffb46f628161ad0eefbd0a855d2c8c32dd81a"},
    {file = "lxml-5.3.0-cp312-cp312-musllinux_1_2_s390x.whl", hash = "sha256:7e2f58095acc211eb9d8b5771bf04df9ff37d6b87618d1cbf85f92399c98dae8"},
    {file = "lxml-5.3.0-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:e63601ad5cd8f860aa99d109889b5ac34de571c7ee902d6812d5d9ddcc77fa7d"},
    {file = "lxml-5.3.0-cp312-cp312-win32.whl", hash = "sha256:17e8d968d04a37c50ad9c456a286b525d78c4a1c15dd53aa46c1d8e06bf6fa30"},
    {file = "lxml-5.3.0-cp312-cp312-win_amd64.whl", hash = "sha256:c1a69e58a6bb2de65902051d57fde951febad631a20a64572677a1052690482f"},
    {file = "lxml-5.3.0-cp313-cp313-macosx_10_13_universal2.whl", hash = "sha256:8c72e9563347c7395910de6a3100a4840a75a6f60e05af5e58566868d5eb2d6a"},
    {file = "lxml-5.3.0-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:e92ce66cd919d18d14b3856906a61d3f6b6a8500e0794142338da644260595cd"},
    {file = "lxml-5.3.0-cp313-cp313-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:1d04f064bebdfef9240478f7a779e8c5dc32b8b7b0b2fc6a62e39b928d428e51"},
    {file = "lxml-5.3.0-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:5c2fb570d7823c2bbaf8b419ba6e5662137f8166e364a8b2b91051a1fb40ab8b"},
    {file = "lxml-5.3.0-cp313-cp313-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:0c120f43553ec759f8de1fee2f4794452b0946773299d44c36bfe18e83caf002"},
    {file = "lxml-5.3.0-cp313-cp313-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:562e7494778a69086f0312ec9689f6b6ac1c6b65670ed7d0267e49f57ffa08c4"},
    {file = "lxml-5.3.0-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:423b121f7e6fa514ba0c7918e56955a1d4470ed35faa03e3d9f0e3baa4c7e492"},
    {file = "lxml-5.3.0-cp313-cp313-manylinux_2_28_aarch64.whl", hash = "sha256:c00f323cc00576df6165cc9d21a4c21285fa6b9989c5c39830c3903dc4303ef3"},
    {file = "lxml-5.3.0-cp313-cp313-manylinux_2_28_ppc64le.whl", hash = "sha256:1fdc9fae8dd4c763e8a31e7630afef517eab9f5d5d31a278df087f307bf601f4"},
    {file = "lxml-5.3.0-cp313-cp313-manylinux_2_28_s390x.whl", hash = "sha256:658f2aa69d31e09699705949b5fc4719cbecbd4a97f9656a232e7d6c7be1a367"},
    {file = "lxml-5.3.0-cp313-cp313-manylinux_2_28_x86_64.whl", hash = "sha256:1473427aff3d66a3fa2199004c3e601e6c4500ab86696edffdbc84954c72d832"},
    {file = "lxml-5.3.0-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:a87de7dd873bf9a792bf1e58b1c3887b9264036629a5bf2d2e6579fe8e73edff"},
    {file = "lxml-5.3.0-cp313-cp313-musllinux_1_2_ppc64le.whl", hash = "sha256:0d7b36afa46c97875303a94e8f3ad932bf78bace9e18e603f2085b652422edcd"},
    {file = "lxml-5.3.0-cp313-cp313-musllinux_1_2_s390x.whl", hash = "sha256:cf120cce539453ae086eacc0130a324e7026113510efa83ab42ef3fcfccac7fb"},
    {file = "lxml-5.3.0-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:df5c7333167b9674aa8ae1d4008fa4bc17a313cc490b2cca27838bbdcc6bb15b"},
    {file = "lxml-5.3.0-cp313-cp313-win32.whl", hash = "sha256:c802e1c2ed9f0c06a65bc4ed0189d000ada8049312cfeab6ca635e39c9608957"},
    {file = "lxml-5.3.0-cp313-cp313-win_amd64.whl", hash = "sha256:406246b96d552e0503e17a1006fd27edac678b3fcc9f1be71a2f94b4ff61528d"},
    {file = "lxml-5.3.0-cp36-cp36m-macosx_10_9_x86_64.whl", hash = "sha256:8f0de2d390af441fe8b2c12626d103540b5d850d585b18fcada58d972b74a74e"},
    {file = "lxml-5.3.0-cp36-cp36m-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:1afe0a8c353746e610bd9031a630a95bcfb1a720684c3f2b36c4710a0a96528f"},
    {file = "lxml-5.3.0-cp36-cp36m-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:56b9861a71575f5795bde89256e7467ece3d339c9b43141dbdd54544566b3b94"},
    {file = "lxml-5.3.0-cp36-cp36m-manylinux_2_28_x86_64.whl", hash = "sha256:9fb81d2824dff4f2e297a276297e9031f46d2682cafc484f49de182aa5e5df99"},
    {file = "lxml-5.3.0-cp36-cp36m-manylinux_2_5_x86_64.manylinux1_x86_64.whl", hash = "sha256:2c226a06ecb8cdef28845ae976da407917542c5e6e75dcac7cc33eb04aaeb237"},
    {file = "lxml-5.3.0-cp36-cp36m-musllinux_1_2_x86_64.whl", hash = "sha256:7d3d1ca42870cdb6d0d29939630dbe48fa511c203724820fc0fd507b2fb46577"},
    {file = "lxml-5.3.0-cp36-cp36m-win32.whl", hash = "sha256:094cb601ba9f55296774c2d57ad68730daa0b13dc260e1f941b4d13678239e70"},
    {file = "lxml-5.3.0-cp36-cp36m-win_amd64.whl", hash = "sha256:eafa2c8658f4e560b098fe9fc54539f86528651f61849b22111a9b107d18910c"},
    {file = "lxml-5.3.0-cp37-cp37m-macosx_10_9_x86_64.whl", hash = "sha256:cb83f8a875b3d9b458cada4f880fa498646874ba4011dc974e071a0a84a1b033"},
    {file = "lxml-5.3.0-cp37-cp37m-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:25f1b69d41656b05885aa185f5fdf822cb01a586d1b32739633679699f220391"},
    {file = "lxml-5.3.0-cp37-cp37m-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:23e0553b8055600b3bf4a00b255ec5c92e1e4aebf8c2c09334f8368e8bd174d6"},
    {file = "lxml-5.3.0-cp37-cp37m-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:9ada35dd21dc6c039259596b358caab6b13f4db4d4a7f8665764d616daf9cc1d"},
    {file = "lxml-5.3.0-cp37-cp37m-manylinux_2_28_aarch64.whl", hash = "sha256:81b4e48da4c69313192d8c8d4311e5d818b8be1afe68ee20f6385d0e96fc9512"},
    {file = "lxml-5.3.0-cp37-cp37m-manylinux_2_28_x86_64.whl", hash = "sha256:2bc9fd5ca4729af796f9f59cd8ff160fe06a474da40aca03fcc79655ddee1a8b"},
    {file = "lxml-5.3.0-cp37-cp37m-musllinux_1_2_aarch64.whl", hash = "sha256:07da23d7ee08577760f0a71d67a861019103e4812c87e2fab26b039054594cc5"},
    {file = "lxml-5.3.0-cp37-cp37m-musllinux_1_2_x86_64.whl", hash = "sha256:ea2e2f6f801696ad7de8aec061044d6c8c0dd4037608c7cab38a9a4d316bfb11"},
    {file = "lxml-5.3.0-cp37-cp37m-win32.whl", hash = "sha256:5c54afdcbb0182d06836cc3d1be921e540be3ebdf8b8a51ee3ef987537455f84"},
    {file = "lxml-5.3.0-cp37-cp37m-win_amd64.whl", hash = "sha256:f2901429da1e645ce548bf9171784c0f74f0718c3f6150ce166be39e4dd66c3e"},
    {file = "lxml-5.3.0-cp38-cp38-macosx_10_9_x86_64.whl", hash = "sha256:c56a1d43b2f9ee4786e4658c7903f05da35b923fb53c11025712562d5cc02753"},
    {file = "lxml-5.3.0-cp38-cp38-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:6ee8c39582d2652dcd516d1b879451500f8db3fe3607ce45d7c5957ab2596040"},
    {file = "lxml-5.3.0-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:0fdf3a3059611f7585a78ee10399a15566356116a4288380921a4b598d807a22"},
    {file = "lxml-5.3.0-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:146173654d79eb1fc97498b4280c1d3e1e5d58c398fa530905c9ea50ea849b22"},
    {file = "lxml-5.3.0-cp38-cp38-manylinux_2_28_aarch64.whl", hash = "sha256:0a7056921edbdd7560746f4221dca89bb7a3fe457d3d74267995253f46343f15"},
    {file = "lxml-5.3.0-cp38-cp38-manylinux_2_28_x86_64.whl", hash = "sha256:9e4b47ac0f5e749cfc618efdf4726269441014ae1d5583e047b452a32e221920"},
    {file = "lxml-5.3.0-cp38-cp38-musllinux_1_2_aarch64.whl", hash = "sha256:f914c03e6a31deb632e2daa881fe198461f4d06e57ac3d0e05bbcab8eae01945"},
    {file = "lxml-5.3.0-cp38-cp38-musllinux_1_2_x86_64.whl", hash = "sha256:213261f168c5e1d9b7535a67e68b1f59f92398dd17a56d934550837143f79c42"},
    {file = "lxml-5.3.0-cp38-cp38-win32.whl", hash = "sha256:218c1b2e17a710e363855594230f44060e2025b05c80d1f0661258142b2add2e"},
    {file = "lxml-5.3.0-cp38-cp38-win_amd64.whl", hash = "sha256:315f9542011b2c4e1d280e4a20ddcca1761993dda3afc7a73b01235f8641e903"},
    {file = "lxml-5.3.0-cp39-cp39-macosx_10_9_universal2.whl", hash = "sha256:1ffc23010330c2ab67fac02781df60998ca8fe759e8efde6f8b756a20599c5de"},
    {file = "lxml-5.3.0-cp39-cp39-macosx_10_9_x86_64.whl", hash = "sha256:2b3778cb38212f52fac9fe913017deea2fdf4eb1a4f8e4cfc6b009a13a6d3fcc"},
    {file = "lxml-5.3.0-cp39-cp39-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:4b0c7a688944891086ba192e21c5229dea54382f4836a209ff8d0a660fac06be"},
    {file = "lxml-5.3.0-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:747a3d3e98e24597981ca0be0fd922aebd471fa99d0043a3842d00cdcad7ad6a"},
    {file = "lxml-5.3.0-cp39-cp39-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:86a6b24b19eaebc448dc56b87c4865527855145d851f9fc3891673ff97950540"},
    {file = "lxml-5.3.0-cp39-cp39-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:b11a5d918a6216e521c715b02749240fb07ae5a1fefd4b7bf12f833bc8b4fe70"},
    {file = "lxml-5.3.0-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:68b87753c784d6acb8a25b05cb526c3406913c9d988d51f80adecc2b0775d6aa"},
    {file = "lxml-5.3.0-cp39-cp39-manylinux_2_28_aarch64.whl", hash = "sha256:109fa6fede314cc50eed29e6e56c540075e63d922455346f11e4d7a036d2b8cf"},
    {file = "lxml-5.3.0-cp39-cp39-manylinux_2_28_ppc64le.whl", hash = "sha256:02ced472497b8362c8e902ade23e3300479f4f43e45f4105c85ef43b8db85229"},
    {file = "lxml-5.3.0-cp39-cp39-manylinux_2_28_s390x.whl", hash = "sha256:6b038cc86b285e4f9fea2ba5ee76e89f21ed1ea898e287dc277a25884f3a7dfe"},
    {file = "lxml-5.3.0-cp39-cp39-manylinux_2_28_x86_64.whl", hash = "sha256:7437237c6a66b7ca341e868cda48be24b8701862757426852c9b3186de1da8a2"},
    {file = "lxml-5.3.0-cp39-cp39-musllinux_1_2_aarch64.whl", hash = "sha256:7f41026c1d64043a36fda21d64c5026762d53a77043e73e94b71f0521939cc71"},
    {file = "lxml-5.3.0-cp39-cp39-musllinux_1_2_ppc64le.whl", hash = "sha256:482c2f67761868f0108b1743098640fbb2a28a8e15bf3f47ada9fa59d9fe08c3"},
    {file = "lxml-5.3.0-cp39-cp39-musllinux_1_2_s390x.whl", hash = "sha256:1483fd3358963cc5c1c9b122c80606a3a79ee0875bcac0204149fa09d6ff2727"},
    {file = "lxml-5.3.0-cp39-cp39-musllinux_1_2_x86_64.whl", hash = "sha256:2dec2d1130a9cda5b904696cec33b2cfb451304ba9081eeda7f90f724097300a"},
    {file = "lxml-5.3.0-cp39-cp39-win32.whl", hash = "sha256:a0eabd0a81625049c5df745209dc7fcef6e2aea7793e5f003ba363610aa0a3ff"},
    {file = "lxml-5.3.0-cp39-cp39-win_amd64.whl", hash = "sha256:89e043f1d9d341c52bf2af6d02e6adde62e0a46e6755d5eb60dc6e4f0b8aeca2"},
    {file = "lxml-5.3.0-pp310-pypy310_pp73-macosx_10_15_x86_64.whl", hash = "sha256:7b1cd427cb0d5f7393c31b7496419da594fe600e6fdc4b105a54f82405e6626c"},
    {file = "lxml-5.3.0-pp310-pypy310_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:51806cfe0279e06ed8500ce19479d757db42a30fd509940b1701be9c86a5ff9a"},
    {file = "lxml-5.3.0-pp310-pypy310_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:ee70d08fd60c9565ba8190f41a46a54096afa0eeb8f76bd66f2c25d3b1b83005"},
    {file = "lxml-5.3.0-pp310-pypy310_pp73-manylinux_2_28_aarch64.whl", hash = "sha256:8dc2c0395bea8254d8daebc76dcf8eb3a95ec2a46fa6fae5eaccee366bfe02ce"},
    {file = "lxml-5.3.0-pp310-pypy310_pp73-manylinux_2_28_x86_64.whl", hash = "sha256:6ba0d3dcac281aad8a0e5b14c7ed6f9fa89c8612b47939fc94f80b16e2e9bc83"},
    {file = "lxml-5.3.0-pp310-pypy310_pp73-win_amd64.whl", hash = "sha256:6e91cf736959057f7aac7adfc83481e03615a8e8dd5758aa1d95ea69e8931dba"},
    {file = "lxml-5.3.0-pp37-pypy37_pp73-macosx_10_9_x86_64.whl", hash = "sha256:94d6c3782907b5e40e21cadf94b13b0842ac421192f26b84c45f13f3c9d5dc27"},
    {file = "lxml-5.3.0-pp37-pypy37_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:c300306673aa0f3ed5ed9372b21867690a17dba38c68c44b287437c362ce486b"},
    {file = "lxml-5.3.0-pp37-pypy37_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:78d9b952e07aed35fe2e1a7ad26e929595412db48535921c5013edc8aa4a35ce"},
    {file = "lxml-5.3.0-pp37-pypy37_pp73-manylinux_2_28_aarch64.whl", hash = "sha256:01220dca0d066d1349bd6a1726856a78f7929f3878f7e2ee83c296c69495309e"},
    {file = "lxml-5.3.0-pp37-pypy37_pp73-manylinux_2_28_x86_64.whl", hash = "sha256:2d9b8d9177afaef80c53c0a9e30fa252ff3036fb1c6494d427c066a4ce6a282f"},
    {file = "lxml-5.3.0-pp37-pypy37_pp73-win_amd64.whl", hash = "sha256:20094fc3f21ea0a8669dc4c61ed7fa8263bd37d97d93b90f28fc613371e7a875"},
    {file = "lxml-5.3.0-pp38-pypy38_pp73-macosx_10_9_x86_64.whl", hash = "sha256:ace2c2326a319a0bb8a8b0e5b570c764962e95818de9f259ce814ee666603f19"},
    {file = "lxml-5.3.0-pp38-pypy38_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:92e67a0be1639c251d21e35fe74df6bcc40cba445c2cda7c4a967656733249e2"},
    {file = "lxml-5.3.0-pp38-pypy38_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:dd5350b55f9fecddc51385463a4f67a5da829bc741e38cf689f38ec9023f54ab"},
    {file = "lxml-5.3.0-pp38-pypy38_pp73-manylinux_2_28_aarch64.whl", hash = "sha256:4c1fefd7e3d00921c44dc9ca80a775af49698bbfd92ea84498e56acffd4c5469"},
    {file = "lxml-5.3.0-pp38-pypy38_pp73-manylinux_2_28_x86_64.whl", hash = "sha256:71a8dd38fbd2f2319136d4ae855a7078c69c9a38ae06e0c17c73fd70fc6caad8"},
    {file = "lxml-5.3.0-pp38-pypy38_pp73-win_amd64.whl", hash = "sha256:97acf1e1fd66ab53dacd2c35b319d7e548380c2e9e8c54525c6e76d21b1ae3b1"},
    {file = "lxml-5.3.0-pp39-pypy39_pp73-macosx_10_15_x86_64.whl", hash = "sha256:68934b242c51eb02907c5b81d138cb977b2129a0a75a8f8b60b01cb8586c7b21"},
    {file = "lxml-5.3.0-pp39-pypy39_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:b710bc2b8292966b23a6a0121f7a6c51d45d2347edcc75f016ac123b8054d3f2"},
    {file = "lxml-5.3.0-pp39-pypy39_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:18feb4b93302091b1541221196a2155aa296c363fd233814fa11e181adebc52f"},
    {file = "lxml-5.3.0-pp39-pypy39_pp73-manylinux_2_28_aarch64.whl", hash = "sha256:3eb44520c4724c2e1a57c0af33a379eee41792595023f367ba3952a2d96c2aab"},
    {file = "lxml-5.3.0-pp39-pypy39_pp73-manylinux_2_28_x86_64.whl", hash = "sha256:609251a0ca4770e5a8768ff902aa02bf636339c5a93f9349b48eb1f606f7f3e9"},
    {file = "lxml-5.3.0-pp39-pypy39_pp73-win_amd64.whl", hash = "sha256:516f491c834eb320d6c843156440fe7fc0d50b33e44387fcec5b02f0bc118a4c"},
    {file = "lxml-5.3.0.tar.gz", hash = "sha256:4e109ca30d1edec1ac60cdbe341905dc3b8f55b16855e03a54aaf59e51ec8c6f"},
]

[package.extras]
cssselect = ["cssselect (>=0.7)"]
html-clean = ["lxml-html-clean"]
html5 = ["html5lib"]
htmlsoup = ["BeautifulSoup4"]
source = ["Cython (>=3.0.11)"]

[[package]]
name = "markdown-it-py"
version = "3.0.0"
description = "Python port of markdown-it. Markdown parsing, done right!"
optional = false
python-versions = ">=3.8"
files = [
    {file = "markdown-it-py-3.0.0.tar.gz", hash = "sha256:e3f60a94fa066dc52ec76661e37c851cb232d92f9886b15cb560aaada2df8feb"},
    {file = "markdown_it_py-3.0.0-py3-none-any.whl", hash = "sha256:355216845c60bd96232cd8d8c40e8f9765cc86f46880e43a8fd22dc1a1a8cab1"},
]

[package.dependencies]
mdurl = ">=0.1,<1.0"

[package.extras]
benchmarking = ["psutil", "pytest", "pytest-benchmark"]
code-style = ["pre-commit (>=3.0,<4.0)"]
compare = ["commonmark (>=0.9,<1.0)", "markdown (>=3.4,<4.0)", "mistletoe (>=1.0,<2.0)", "mistune (>=2.0,<3.0)", "panflute (>=2.3,<3.0)"]
linkify = ["linkify-it-py (>=1,<3)"]
plugins = ["mdit-py-plugins"]
profiling = ["gprof2dot"]
rtd = ["jupyter_sphinx", "mdit-py-plugins", "myst-parser", "pyyaml", "sphinx", "sphinx-copybutton", "sphinx-design", "sphinx_book_theme"]
testing = ["coverage", "pytest", "pytest-cov", "pytest-regressions"]

[[package]]
name = "markupsafe"
version = "3.0.2"
description = "Safely add untrusted strings to HTML/XML markup."
optional = false
python-versions = ">=3.9"
files = [
    {file = "MarkupSafe-3.0.2-cp310-cp310-macosx_10_9_universal2.whl", hash = "sha256:7e94c425039cde14257288fd61dcfb01963e658efbc0ff54f5306b06054700f8"},
    {file = "MarkupSafe-3.0.2-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:9e2d922824181480953426608b81967de705c3cef4d1af983af849d7bd619158"},
    {file = "MarkupSafe-3.0.2-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:38a9ef736c01fccdd6600705b09dc574584b89bea478200c5fbf112a6b0d5579"},
    {file = "MarkupSafe-3.0.2-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:bbcb445fa71794da8f178f0f6d66789a28d7319071af7a496d4d507ed566270d"},
    {file = "MarkupSafe-3.0.2-cp310-cp310-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:57cb5a3cf367aeb1d316576250f65edec5bb3be939e9247ae594b4bcbc317dfb"},
    {file = "MarkupSafe-3.0.2-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:3809ede931876f5b2ec92eef964286840ed3540dadf803dd570c3b7e13141a3b"},
    {file = "MarkupSafe-3.0.2-cp310-cp310-musllinux_1_2_i686.whl", hash = "sha256:e07c3764494e3776c602c1e78e298937c3315ccc9043ead7e685b7f2b8d47b3c"},
    {file = "MarkupSafe-3.0.2-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:b424c77b206d63d500bcb69fa55ed8d0e6a3774056bdc4839fc9298a7edca171"},
    {file = "MarkupSafe-3.0.2-cp310-cp310-win32.whl", hash = "sha256:fcabf5ff6eea076f859677f5f0b6b5c1a51e70a376b0579e0eadef8db48c6b50"},
    {file = "MarkupSafe-3.0.2-cp310-cp310-win_amd64.whl", hash = "sha256:6af100e168aa82a50e186c82875a5893c5597a0c1ccdb0d8b40240b1f28b969a"},
    {file = "MarkupSafe-3.0.2-cp311-cp311-macosx_10_9_universal2.whl", hash = "sha256:9025b4018f3a1314059769c7bf15441064b2207cb3f065e6ea1e7359cb46db9d"},
    {file = "MarkupSafe-3.0.2-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:93335ca3812df2f366e80509ae119189886b0f3c2b81325d39efdb84a1e2ae93"},
    {file = "MarkupSafe-3.0.2-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:2cb8438c3cbb25e220c2ab33bb226559e7afb3baec11c4f218ffa7308603c832"},
    {file = "MarkupSafe-3.0.2-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:a123e330ef0853c6e822384873bef7507557d8e4a082961e1defa947aa59ba84"},
    {file = "MarkupSafe-3.0.2-cp311-cp311-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:1e084f686b92e5b83186b07e8a17fc09e38fff551f3602b249881fec658d3eca"},
    {file = "MarkupSafe-3.0.2-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:d8213e09c917a951de9d09ecee036d5c7d36cb6cb7dbaece4c71a60d79fb9798"},
    {file = "MarkupSafe-3.0.2-cp311-cp311-musllinux_1_2_i686.whl", hash = "sha256:5b02fb34468b6aaa40dfc198d813a641e3a63b98c2b05a16b9f80b7ec314185e"},
    {file = "MarkupSafe-3.0.2-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:0bff5e0ae4ef2e1ae4fdf2dfd5b76c75e5c2fa4132d05fc1b0dabcd20c7e28c4"},
    {file = "MarkupSafe-3.0.2-cp311-cp311-win32.whl", hash = "sha256:6c89876f41da747c8d3677a2b540fb32ef5715f97b66eeb0c6b66f5e3ef6f59d"},
    {file = "MarkupSafe-3.0.2-cp311-cp311-win_amd64.whl", hash = "sha256:70a87b411535ccad5ef2f1df5136506a10775d267e197e4cf531ced10537bd6b"},
    {file = "MarkupSafe-3.0.2-cp312-cp312-macosx_10_13_universal2.whl", hash = "sha256:9778bd8ab0a994ebf6f84c2b949e65736d5575320a17ae8984a77fab08db94cf"},
    {file = "MarkupSafe-3.0.2-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:846ade7b71e3536c4e56b386c2a47adf5741d2d8b94ec9dc3e92e5e1ee1e2225"},
    {file = "MarkupSafe-3.0.2-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:1c99d261bd2d5f6b59325c92c73df481e05e57f19837bdca8413b9eac4bd8028"},
    {file = "MarkupSafe-3.0.2-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:e17c96c14e19278594aa4841ec148115f9c7615a47382ecb6b82bd8fea3ab0c8"},
    {file = "MarkupSafe-3.0.2-cp312-cp312-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:88416bd1e65dcea10bc7569faacb2c20ce071dd1f87539ca2ab364bf6231393c"},
    {file = "MarkupSafe-3.0.2-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:2181e67807fc2fa785d0592dc2d6206c019b9502410671cc905d132a92866557"},
    {file = "MarkupSafe-3.0.2-cp312-cp312-musllinux_1_2_i686.whl", hash = "sha256:52305740fe773d09cffb16f8ed0427942901f00adedac82ec8b67752f58a1b22"},
    {file = "MarkupSafe-3.0.2-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:ad10d3ded218f1039f11a75f8091880239651b52e9bb592ca27de44eed242a48"},
    {file = "MarkupSafe-3.0.2-cp312-cp312-win32.whl", hash = "sha256:0f4ca02bea9a23221c0182836703cbf8930c5e9454bacce27e767509fa286a30"},
    {file = "MarkupSafe-3.0.2-cp312-cp312-win_amd64.whl", hash = "sha256:8e06879fc22a25ca47312fbe7c8264eb0b662f6db27cb2d3bbbc74b1df4b9b87"},
    {file = "MarkupSafe-3.0.2-cp313-cp313-macosx_10_13_universal2.whl", hash = "sha256:ba9527cdd4c926ed0760bc301f6728ef34d841f405abf9d4f959c478421e4efd"},
    {file = "MarkupSafe-3.0.2-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:f8b3d067f2e40fe93e1ccdd6b2e1d16c43140e76f02fb1319a05cf2b79d99430"},
    {file = "MarkupSafe-3.0.2-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:569511d3b58c8791ab4c2e1285575265991e6d8f8700c7be0e88f86cb0672094"},
    {file = "MarkupSafe-3.0.2-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:15ab75ef81add55874e7ab7055e9c397312385bd9ced94920f2802310c930396"},
    {file = "MarkupSafe-3.0.2-cp313-cp313-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:f3818cb119498c0678015754eba762e0d61e5b52d34c8b13d770f0719f7b1d79"},
    {file = "MarkupSafe-3.0.2-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:cdb82a876c47801bb54a690c5ae105a46b392ac6099881cdfb9f6e95e4014c6a"},
    {file = "MarkupSafe-3.0.2-cp313-cp313-musllinux_1_2_i686.whl", hash = "sha256:cabc348d87e913db6ab4aa100f01b08f481097838bdddf7c7a84b7575b7309ca"},
    {file = "MarkupSafe-3.0.2-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:444dcda765c8a838eaae23112db52f1efaf750daddb2d9ca300bcae1039adc5c"},
    {file = "MarkupSafe-3.0.2-cp313-cp313-win32.whl", hash = "sha256:bcf3e58998965654fdaff38e58584d8937aa3096ab5354d493c77d1fdd66d7a1"},
    {file = "MarkupSafe-3.0.2-cp313-cp313-win_amd64.whl", hash = "sha256:e6a2a455bd412959b57a172ce6328d2dd1f01cb2135efda2e4576e8a23fa3b0f"},
    {file = "MarkupSafe-3.0.2-cp313-cp313t-macosx_10_13_universal2.whl", hash = "sha256:b5a6b3ada725cea8a5e634536b1b01c30bcdcd7f9c6fff4151548d5bf6b3a36c"},
    {file = "MarkupSafe-3.0.2-cp313-cp313t-macosx_11_0_arm64.whl", hash = "sha256:a904af0a6162c73e3edcb969eeeb53a63ceeb5d8cf642fade7d39e7963a22ddb"},
    {file = "MarkupSafe-3.0.2-cp313-cp313t-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:4aa4e5faecf353ed117801a068ebab7b7e09ffb6e1d5e412dc852e0da018126c"},
    {file = "MarkupSafe-3.0.2-cp313-cp313t-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:c0ef13eaeee5b615fb07c9a7dadb38eac06a0608b41570d8ade51c56539e509d"},
    {file = "MarkupSafe-3.0.2-cp313-cp313t-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:d16a81a06776313e817c951135cf7340a3e91e8c1ff2fac444cfd75fffa04afe"},
    {file = "MarkupSafe-3.0.2-cp313-cp313t-musllinux_1_2_aarch64.whl", hash = "sha256:6381026f158fdb7c72a168278597a5e3a5222e83ea18f543112b2662a9b699c5"},
    {file = "MarkupSafe-3.0.2-cp313-cp313t-musllinux_1_2_i686.whl", hash = "sha256:3d79d162e7be8f996986c064d1c7c817f6df3a77fe3d6859f6f9e7be4b8c213a"},
    {file = "MarkupSafe-3.0.2-cp313-cp313t-musllinux_1_2_x86_64.whl", hash = "sha256:131a3c7689c85f5ad20f9f6fb1b866f402c445b220c19fe4308c0b147ccd2ad9"},
    {file = "MarkupSafe-3.0.2-cp313-cp313t-win32.whl", hash = "sha256:ba8062ed2cf21c07a9e295d5b8a2a5ce678b913b45fdf68c32d95d6c1291e0b6"},
    {file = "MarkupSafe-3.0.2-cp313-cp313t-win_amd64.whl", hash = "sha256:e444a31f8db13eb18ada366ab3cf45fd4b31e4db1236a4448f68778c1d1a5a2f"},
    {file = "MarkupSafe-3.0.2-cp39-cp39-macosx_10_9_universal2.whl", hash = "sha256:eaa0a10b7f72326f1372a713e73c3f739b524b3af41feb43e4921cb529f5929a"},
    {file = "MarkupSafe-3.0.2-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:48032821bbdf20f5799ff537c7ac3d1fba0ba032cfc06194faffa8cda8b560ff"},
    {file = "MarkupSafe-3.0.2-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:1a9d3f5f0901fdec14d8d2f66ef7d035f2157240a433441719ac9a3fba440b13"},
    {file = "MarkupSafe-3.0.2-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:88b49a3b9ff31e19998750c38e030fc7bb937398b1f78cfa599aaef92d693144"},
    {file = "MarkupSafe-3.0.2-cp39-cp39-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:cfad01eed2c2e0c01fd0ecd2ef42c492f7f93902e39a42fc9ee1692961443a29"},
    {file = "MarkupSafe-3.0.2-cp39-cp39-musllinux_1_2_aarch64.whl", hash = "sha256:1225beacc926f536dc82e45f8a4d68502949dc67eea90eab715dea3a21c1b5f0"},
    {file = "MarkupSafe-3.0.2-cp39-cp39-musllinux_1_2_i686.whl", hash = "sha256:3169b1eefae027567d1ce6ee7cae382c57fe26e82775f460f0b2778beaad66c0"},
    {file = "MarkupSafe-3.0.2-cp39-cp39-musllinux_1_2_x86_64.whl", hash = "sha256:eb7972a85c54febfb25b5c4b4f3af4dcc731994c7da0d8a0b4a6eb0640e1d178"},
    {file = "MarkupSafe-3.0.2-cp39-cp39-win32.whl", hash = "sha256:8c4e8c3ce11e1f92f6536ff07154f9d49677ebaaafc32db9db4620bc11ed480f"},
    {file = "MarkupSafe-3.0.2-cp39-cp39-win_amd64.whl", hash = "sha256:6e296a513ca3d94054c2c881cc913116e90fd030ad1c656b3869762b754f5f8a"},
    {file = "markupsafe-3.0.2.tar.gz", hash = "sha256:ee55d3edf80167e48ea11a923c7386f4669df67d7994554387f84e7d8b0a2bf0"},
]

[[package]]
name = "matplotlib"
version = "3.10.0"
description = "Python plotting package"
optional = false
python-versions = ">=3.10"
files = [
    {file = "matplotlib-3.10.0-cp310-cp310-macosx_10_12_x86_64.whl", hash = "sha256:2c5829a5a1dd5a71f0e31e6e8bb449bc0ee9dbfb05ad28fc0c6b55101b3a4be6"},
    {file = "matplotlib-3.10.0-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:a2a43cbefe22d653ab34bb55d42384ed30f611bcbdea1f8d7f431011a2e1c62e"},
    {file = "matplotlib-3.10.0-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:607b16c8a73943df110f99ee2e940b8a1cbf9714b65307c040d422558397dac5"},
    {file = "matplotlib-3.10.0-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:01d2b19f13aeec2e759414d3bfe19ddfb16b13a1250add08d46d5ff6f9be83c6"},
    {file = "matplotlib-3.10.0-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:5e6c6461e1fc63df30bf6f80f0b93f5b6784299f721bc28530477acd51bfc3d1"},
    {file = "matplotlib-3.10.0-cp310-cp310-win_amd64.whl", hash = "sha256:994c07b9d9fe8d25951e3202a68c17900679274dadfc1248738dcfa1bd40d7f3"},
    {file = "matplotlib-3.10.0-cp311-cp311-macosx_10_12_x86_64.whl", hash = "sha256:fd44fc75522f58612ec4a33958a7e5552562b7705b42ef1b4f8c0818e304a363"},
    {file = "matplotlib-3.10.0-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:c58a9622d5dbeb668f407f35f4e6bfac34bb9ecdcc81680c04d0258169747997"},
    {file = "matplotlib-3.10.0-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:845d96568ec873be63f25fa80e9e7fae4be854a66a7e2f0c8ccc99e94a8bd4ef"},
    {file = "matplotlib-3.10.0-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:5439f4c5a3e2e8eab18e2f8c3ef929772fd5641876db71f08127eed95ab64683"},
    {file = "matplotlib-3.10.0-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:4673ff67a36152c48ddeaf1135e74ce0d4bce1bbf836ae40ed39c29edf7e2765"},
    {file = "matplotlib-3.10.0-cp311-cp311-win_amd64.whl", hash = "sha256:7e8632baebb058555ac0cde75db885c61f1212e47723d63921879806b40bec6a"},
    {file = "matplotlib-3.10.0-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:4659665bc7c9b58f8c00317c3c2a299f7f258eeae5a5d56b4c64226fca2f7c59"},
    {file = "matplotlib-3.10.0-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:d44cb942af1693cced2604c33a9abcef6205601c445f6d0dc531d813af8a2f5a"},
    {file = "matplotlib-3.10.0-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:a994f29e968ca002b50982b27168addfd65f0105610b6be7fa515ca4b5307c95"},
    {file = "matplotlib-3.10.0-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:9b0558bae37f154fffda54d779a592bc97ca8b4701f1c710055b609a3bac44c8"},
    {file = "matplotlib-3.10.0-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:503feb23bd8c8acc75541548a1d709c059b7184cde26314896e10a9f14df5f12"},
    {file = "matplotlib-3.10.0-cp312-cp312-win_amd64.whl", hash = "sha256:c40ba2eb08b3f5de88152c2333c58cee7edcead0a2a0d60fcafa116b17117adc"},
    {file = "matplotlib-3.10.0-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:96f2886f5c1e466f21cc41b70c5a0cd47bfa0015eb2d5793c88ebce658600e25"},
    {file = "matplotlib-3.10.0-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:12eaf48463b472c3c0f8dbacdbf906e573013df81a0ab82f0616ea4b11281908"},
    {file = "matplotlib-3.10.0-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:2fbbabc82fde51391c4da5006f965e36d86d95f6ee83fb594b279564a4c5d0d2"},
    {file = "matplotlib-3.10.0-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:ad2e15300530c1a94c63cfa546e3b7864bd18ea2901317bae8bbf06a5ade6dcf"},
    {file = "matplotlib-3.10.0-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:3547d153d70233a8496859097ef0312212e2689cdf8d7ed764441c77604095ae"},
    {file = "matplotlib-3.10.0-cp313-cp313-win_amd64.whl", hash = "sha256:c55b20591ced744aa04e8c3e4b7543ea4d650b6c3c4b208c08a05b4010e8b442"},
    {file = "matplotlib-3.10.0-cp313-cp313t-macosx_10_13_x86_64.whl", hash = "sha256:9ade1003376731a971e398cc4ef38bb83ee8caf0aee46ac6daa4b0506db1fd06"},
    {file = "matplotlib-3.10.0-cp313-cp313t-macosx_11_0_arm64.whl", hash = "sha256:95b710fea129c76d30be72c3b38f330269363fbc6e570a5dd43580487380b5ff"},
    {file = "matplotlib-3.10.0-cp313-cp313t-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:5cdbaf909887373c3e094b0318d7ff230b2ad9dcb64da7ade654182872ab2593"},
    {file = "matplotlib-3.10.0-cp313-cp313t-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:d907fddb39f923d011875452ff1eca29a9e7f21722b873e90db32e5d8ddff12e"},
    {file = "matplotlib-3.10.0-cp313-cp313t-musllinux_1_2_x86_64.whl", hash = "sha256:3b427392354d10975c1d0f4ee18aa5844640b512d5311ef32efd4dd7db106ede"},
    {file = "matplotlib-3.10.0-cp313-cp313t-win_amd64.whl", hash = "sha256:5fd41b0ec7ee45cd960a8e71aea7c946a28a0b8a4dcee47d2856b2af051f334c"},
    {file = "matplotlib-3.10.0-pp310-pypy310_pp73-macosx_10_15_x86_64.whl", hash = "sha256:81713dd0d103b379de4516b861d964b1d789a144103277769238c732229d7f03"},
    {file = "matplotlib-3.10.0-pp310-pypy310_pp73-macosx_11_0_arm64.whl", hash = "sha256:359f87baedb1f836ce307f0e850d12bb5f1936f70d035561f90d41d305fdacea"},
    {file = "matplotlib-3.10.0-pp310-pypy310_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:ae80dc3a4add4665cf2faa90138384a7ffe2a4e37c58d83e115b54287c4f06ef"},
    {file = "matplotlib-3.10.0.tar.gz", hash = "sha256:b886d02a581b96704c9d1ffe55709e49b4d2d52709ccebc4be42db856e511278"},
]

[package.dependencies]
contourpy = ">=1.0.1"
cycler = ">=0.10"
fonttools = ">=4.22.0"
kiwisolver = ">=1.3.1"
numpy = ">=1.23"
packaging = ">=20.0"
pillow = ">=8"
pyparsing = ">=2.3.1"
python-dateutil = ">=2.7"

[package.extras]
dev = ["meson-python (>=0.13.1,<0.17.0)", "pybind11 (>=2.13.2,!=2.13.3)", "setuptools (>=64)", "setuptools_scm (>=7)"]

[[package]]
name = "matplotlib-inline"
version = "0.1.7"
description = "Inline Matplotlib backend for Jupyter"
optional = false
python-versions = ">=3.8"
files = [
    {file = "matplotlib_inline-0.1.7-py3-none-any.whl", hash = "sha256:df192d39a4ff8f21b1895d72e6a13f5fcc5099f00fa84384e0ea28c2cc0653ca"},
    {file = "matplotlib_inline-0.1.7.tar.gz", hash = "sha256:8423b23ec666be3d16e16b60bdd8ac4e86e840ebd1dd11a30b9f117f2fa0ab90"},
]

[package.dependencies]
traitlets = "*"

[[package]]
name = "mccabe"
version = "0.7.0"
description = "McCabe checker, plugin for flake8"
optional = false
python-versions = ">=3.6"
files = [
    {file = "mccabe-0.7.0-py2.py3-none-any.whl", hash = "sha256:6c2d30ab6be0e4a46919781807b4f0d834ebdd6c6e3dca0bda5a15f863427b6e"},
    {file = "mccabe-0.7.0.tar.gz", hash = "sha256:348e0240c33b60bbdf4e523192ef919f28cb2c3d7d5c7794f74009290f236325"},
]

[[package]]
name = "mdurl"
version = "0.1.2"
description = "Markdown URL utilities"
optional = false
python-versions = ">=3.7"
files = [
    {file = "mdurl-0.1.2-py3-none-any.whl", hash = "sha256:84008a41e51615a49fc9966191ff91509e3c40b939176e643fd50a5c2196b8f8"},
    {file = "mdurl-0.1.2.tar.gz", hash = "sha256:bb413d29f5eea38f31dd4754dd7377d4465116fb207585f97bf925588687c1ba"},
]

[[package]]
name = "mistune"
version = "3.0.2"
description = "A sane and fast Markdown parser with useful plugins and renderers"
optional = false
python-versions = ">=3.7"
files = [
    {file = "mistune-3.0.2-py3-none-any.whl", hash = "sha256:71481854c30fdbc938963d3605b72501f5c10a9320ecd412c121c163a1c7d205"},
    {file = "mistune-3.0.2.tar.gz", hash = "sha256:fc7f93ded930c92394ef2cb6f04a8aabab4117a91449e72dcc8dfa646a508be8"},
]

[[package]]
name = "mypy"
version = "1.14.0"
description = "Optional static typing for Python"
optional = false
python-versions = ">=3.8"
files = [
    {file = "mypy-1.14.0-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:e971c1c667007f9f2b397ffa80fa8e1e0adccff336e5e77e74cb5f22868bee87"},
    {file = "mypy-1.14.0-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:e86aaeaa3221a278c66d3d673b297232947d873773d61ca3ee0e28b2ff027179"},
    {file = "mypy-1.14.0-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:1628c5c3ce823d296e41e2984ff88c5861499041cb416a8809615d0c1f41740e"},
    {file = "mypy-1.14.0-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:7fadb29b77fc14a0dd81304ed73c828c3e5cde0016c7e668a86a3e0dfc9f3af3"},
    {file = "mypy-1.14.0-cp310-cp310-win_amd64.whl", hash = "sha256:3fa76988dc760da377c1e5069200a50d9eaaccf34f4ea18428a3337034ab5a44"},
    {file = "mypy-1.14.0-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:6e73c8a154eed31db3445fe28f63ad2d97b674b911c00191416cf7f6459fd49a"},
    {file = "mypy-1.14.0-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:273e70fcb2e38c5405a188425aa60b984ffdcef65d6c746ea5813024b68c73dc"},
    {file = "mypy-1.14.0-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:1daca283d732943731a6a9f20fdbcaa927f160bc51602b1d4ef880a6fb252015"},
    {file = "mypy-1.14.0-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:7e68047bedb04c1c25bba9901ea46ff60d5eaac2d71b1f2161f33107e2b368eb"},
    {file = "mypy-1.14.0-cp311-cp311-win_amd64.whl", hash = "sha256:7a52f26b9c9b1664a60d87675f3bae00b5c7f2806e0c2800545a32c325920bcc"},
    {file = "mypy-1.14.0-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:d5326ab70a6db8e856d59ad4cb72741124950cbbf32e7b70e30166ba7bbf61dd"},
    {file = "mypy-1.14.0-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:bf4ec4980bec1e0e24e5075f449d014011527ae0055884c7e3abc6a99cd2c7f1"},
    {file = "mypy-1.14.0-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:390dfb898239c25289495500f12fa73aa7f24a4c6d90ccdc165762462b998d63"},
    {file = "mypy-1.14.0-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:7e026d55ddcd76e29e87865c08cbe2d0104e2b3153a523c529de584759379d3d"},
    {file = "mypy-1.14.0-cp312-cp312-win_amd64.whl", hash = "sha256:585ed36031d0b3ee362e5107ef449a8b5dfd4e9c90ccbe36414ee405ee6b32ba"},
    {file = "mypy-1.14.0-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:e9f6f4c0b27401d14c483c622bc5105eff3911634d576bbdf6695b9a7c1ba741"},
    {file = "mypy-1.14.0-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:56b2280cedcb312c7a79f5001ae5325582d0d339bce684e4a529069d0e7ca1e7"},
    {file = "mypy-1.14.0-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:342de51c48bab326bfc77ce056ba08c076d82ce4f5a86621f972ed39970f94d8"},
    {file = "mypy-1.14.0-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:00df23b42e533e02a6f0055e54de9a6ed491cd8b7ea738647364fd3a39ea7efc"},
    {file = "mypy-1.14.0-cp313-cp313-win_amd64.whl", hash = "sha256:e8c8387e5d9dff80e7daf961df357c80e694e942d9755f3ad77d69b0957b8e3f"},
    {file = "mypy-1.14.0-cp38-cp38-macosx_10_9_x86_64.whl", hash = "sha256:0b16738b1d80ec4334654e89e798eb705ac0c36c8a5c4798496cd3623aa02286"},
    {file = "mypy-1.14.0-cp38-cp38-macosx_11_0_arm64.whl", hash = "sha256:10065fcebb7c66df04b05fc799a854b1ae24d9963c8bb27e9064a9bdb43aa8ad"},
    {file = "mypy-1.14.0-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:fbb7d683fa6bdecaa106e8368aa973ecc0ddb79a9eaeb4b821591ecd07e9e03c"},
    {file = "mypy-1.14.0-cp38-cp38-musllinux_1_2_x86_64.whl", hash = "sha256:3498cb55448dc5533e438cd13d6ddd28654559c8c4d1fd4b5ca57a31b81bac01"},
    {file = "mypy-1.14.0-cp38-cp38-win_amd64.whl", hash = "sha256:c7b243408ea43755f3a21a0a08e5c5ae30eddb4c58a80f415ca6b118816e60aa"},
    {file = "mypy-1.14.0-cp39-cp39-macosx_10_9_x86_64.whl", hash = "sha256:14117b9da3305b39860d0aa34b8f1ff74d209a368829a584eb77524389a9c13e"},
    {file = "mypy-1.14.0-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:af98c5a958f9c37404bd4eef2f920b94874507e146ed6ee559f185b8809c44cc"},
    {file = "mypy-1.14.0-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:f0b343a1d3989547024377c2ba0dca9c74a2428ad6ed24283c213af8dbb0710b"},
    {file = "mypy-1.14.0-cp39-cp39-musllinux_1_2_x86_64.whl", hash = "sha256:cdb5563c1726c85fb201be383168f8c866032db95e1095600806625b3a648cb7"},
    {file = "mypy-1.14.0-cp39-cp39-win_amd64.whl", hash = "sha256:74e925649c1ee0a79aa7448baf2668d81cc287dc5782cff6a04ee93f40fb8d3f"},
    {file = "mypy-1.14.0-py3-none-any.whl", hash = "sha256:2238d7f93fc4027ed1efc944507683df3ba406445a2b6c96e79666a045aadfab"},
    {file = "mypy-1.14.0.tar.gz", hash = "sha256:822dbd184d4a9804df5a7d5335a68cf7662930e70b8c1bc976645d1509f9a9d6"},
]

[package.dependencies]
mypy_extensions = ">=1.0.0"
typing_extensions = ">=4.6.0"

[package.extras]
dmypy = ["psutil (>=4.0)"]
faster-cache = ["orjson"]
install-types = ["pip"]
mypyc = ["setuptools (>=50)"]
reports = ["lxml"]

[[package]]
name = "mypy-extensions"
version = "1.0.0"
description = "Type system extensions for programs checked with the mypy type checker."
optional = false
python-versions = ">=3.5"
files = [
    {file = "mypy_extensions-1.0.0-py3-none-any.whl", hash = "sha256:4392f6c0eb8a5668a69e23d168ffa70f0be9ccfd32b5cc2d26a34ae5b844552d"},
    {file = "mypy_extensions-1.0.0.tar.gz", hash = "sha256:75dbf8955dc00442a438fc4d0666508a9a97b6bd41aa2f0ffe9d2f2725af0782"},
]

[[package]]
name = "nbclient"
version = "0.10.2"
description = "A client library for executing notebooks. Formerly nbconvert's ExecutePreprocessor."
optional = false
python-versions = ">=3.9.0"
files = [
    {file = "nbclient-0.10.2-py3-none-any.whl", hash = "sha256:4ffee11e788b4a27fabeb7955547e4318a5298f34342a4bfd01f2e1faaeadc3d"},
    {file = "nbclient-0.10.2.tar.gz", hash = "sha256:90b7fc6b810630db87a6d0c2250b1f0ab4cf4d3c27a299b0cde78a4ed3fd9193"},
]

[package.dependencies]
jupyter-client = ">=6.1.12"
jupyter-core = ">=4.12,<5.0.dev0 || >=5.1.dev0"
nbformat = ">=5.1"
traitlets = ">=5.4"

[package.extras]
dev = ["pre-commit"]
docs = ["autodoc-traits", "flaky", "ipykernel (>=6.19.3)", "ipython", "ipywidgets", "mock", "moto", "myst-parser", "nbconvert (>=7.1.0)", "pytest (>=7.0,<8)", "pytest-asyncio", "pytest-cov (>=4.0)", "sphinx (>=1.7)", "sphinx-book-theme", "sphinxcontrib-spelling", "testpath", "xmltodict"]
test = ["flaky", "ipykernel (>=6.19.3)", "ipython", "ipywidgets", "nbconvert (>=7.1.0)", "pytest (>=7.0,<8)", "pytest-asyncio", "pytest-cov (>=4.0)", "testpath", "xmltodict"]

[[package]]
name = "nbconvert"
version = "7.16.4"
description = "Converting Jupyter Notebooks (.ipynb files) to other formats.  Output formats include asciidoc, html, latex, markdown, pdf, py, rst, script.  nbconvert can be used both as a Python library (`import nbconvert`) or as a command line tool (invoked as `jupyter nbconvert ...`)."
optional = false
python-versions = ">=3.8"
files = [
    {file = "nbconvert-7.16.4-py3-none-any.whl", hash = "sha256:05873c620fe520b6322bf8a5ad562692343fe3452abda5765c7a34b7d1aa3eb3"},
    {file = "nbconvert-7.16.4.tar.gz", hash = "sha256:86ca91ba266b0a448dc96fa6c5b9d98affabde2867b363258703536807f9f7f4"},
]

[package.dependencies]
beautifulsoup4 = "*"
bleach = "!=5.0.0"
defusedxml = "*"
jinja2 = ">=3.0"
jupyter-core = ">=4.7"
jupyterlab-pygments = "*"
markupsafe = ">=2.0"
mistune = ">=2.0.3,<4"
nbclient = ">=0.5.0"
nbformat = ">=5.7"
packaging = "*"
pandocfilters = ">=1.4.1"
pygments = ">=2.4.1"
tinycss2 = "*"
traitlets = ">=5.1"

[package.extras]
all = ["flaky", "ipykernel", "ipython", "ipywidgets (>=7.5)", "myst-parser", "nbsphinx (>=0.2.12)", "playwright", "pydata-sphinx-theme", "pyqtwebengine (>=5.15)", "pytest (>=7)", "sphinx (==5.0.2)", "sphinxcontrib-spelling", "tornado (>=6.1)"]
docs = ["ipykernel", "ipython", "myst-parser", "nbsphinx (>=0.2.12)", "pydata-sphinx-theme", "sphinx (==5.0.2)", "sphinxcontrib-spelling"]
qtpdf = ["pyqtwebengine (>=5.15)"]
qtpng = ["pyqtwebengine (>=5.15)"]
serve = ["tornado (>=6.1)"]
test = ["flaky", "ipykernel", "ipywidgets (>=7.5)", "pytest (>=7)"]
webpdf = ["playwright"]

[[package]]
name = "nbdime"
version = "4.0.2"
description = "Diff and merge of Jupyter Notebooks"
optional = false
python-versions = ">=3.6"
files = [
    {file = "nbdime-4.0.2-py3-none-any.whl", hash = "sha256:e5a43aca669c576c66e757071c0e882de05ac305311d79aded99bfb5a3e9419e"},
    {file = "nbdime-4.0.2.tar.gz", hash = "sha256:d8279f8f4b236c0b253b20d60c4831bb67843ed8dbd6e09f234eb011d36f1bf2"},
]

[package.dependencies]
colorama = "*"
gitpython = "<2.1.4 || >2.1.4,<2.1.5 || >2.1.5,<2.1.6 || >2.1.6"
jinja2 = ">=2.9"
jupyter-server = "*"
jupyter-server-mathjax = ">=0.2.2"
nbformat = "*"
pygments = "*"
requests = "*"
tornado = "*"

[package.extras]
docs = ["recommonmark", "sphinx", "sphinx-rtd-theme"]
test = ["jsonschema", "jupyter-server[test]", "mock", "notebook", "pytest (>=6.0)", "pytest-cov", "pytest-timeout", "pytest-tornado", "requests", "tabulate"]

[[package]]
name = "nbformat"
version = "5.10.4"
description = "The Jupyter Notebook format"
optional = false
python-versions = ">=3.8"
files = [
    {file = "nbformat-5.10.4-py3-none-any.whl", hash = "sha256:3b48d6c8fbca4b299bf3982ea7db1af21580e4fec269ad087b9e81588891200b"},
    {file = "nbformat-5.10.4.tar.gz", hash = "sha256:322168b14f937a5d11362988ecac2a4952d3d8e3a2cbeb2319584631226d5b3a"},
]

[package.dependencies]
fastjsonschema = ">=2.15"
jsonschema = ">=2.6"
jupyter-core = ">=4.12,<5.0.dev0 || >=5.1.dev0"
traitlets = ">=5.1"

[package.extras]
docs = ["myst-parser", "pydata-sphinx-theme", "sphinx", "sphinxcontrib-github-alt", "sphinxcontrib-spelling"]
test = ["pep440", "pre-commit", "pytest", "testpath"]

[[package]]
name = "nest-asyncio"
version = "1.6.0"
description = "Patch asyncio to allow nested event loops"
optional = false
python-versions = ">=3.5"
files = [
    {file = "nest_asyncio-1.6.0-py3-none-any.whl", hash = "sha256:87af6efd6b5e897c81050477ef65c62e2b2f35d51703cae01aff2905b1852e1c"},
    {file = "nest_asyncio-1.6.0.tar.gz", hash = "sha256:6f172d5449aca15afd6c646851f4e31e02c598d553a667e38cafa997cfec55fe"},
]

[[package]]
name = "notebook"
version = "7.3.2"
description = "Jupyter Notebook - A web-based notebook environment for interactive computing"
optional = false
python-versions = ">=3.8"
files = [
    {file = "notebook-7.3.2-py3-none-any.whl", hash = "sha256:e5f85fc59b69d3618d73cf27544418193ff8e8058d5bf61d315ce4f473556288"},
    {file = "notebook-7.3.2.tar.gz", hash = "sha256:705e83a1785f45b383bf3ee13cb76680b92d24f56fb0c7d2136fe1d850cd3ca8"},
]

[package.dependencies]
jupyter-server = ">=2.4.0,<3"
jupyterlab = ">=4.3.4,<4.4"
jupyterlab-server = ">=2.27.1,<3"
notebook-shim = ">=0.2,<0.3"
tornado = ">=6.2.0"

[package.extras]
dev = ["hatch", "pre-commit"]
docs = ["myst-parser", "nbsphinx", "pydata-sphinx-theme", "sphinx (>=1.3.6)", "sphinxcontrib-github-alt", "sphinxcontrib-spelling"]
test = ["importlib-resources (>=5.0)", "ipykernel", "jupyter-server[test] (>=2.4.0,<3)", "jupyterlab-server[test] (>=2.27.1,<3)", "nbval", "pytest (>=7.0)", "pytest-console-scripts", "pytest-timeout", "pytest-tornasync", "requests"]

[[package]]
name = "notebook-shim"
version = "0.2.4"
description = "A shim layer for notebook traits and config"
optional = false
python-versions = ">=3.7"
files = [
    {file = "notebook_shim-0.2.4-py3-none-any.whl", hash = "sha256:411a5be4e9dc882a074ccbcae671eda64cceb068767e9a3419096986560e1cef"},
    {file = "notebook_shim-0.2.4.tar.gz", hash = "sha256:b4b2cfa1b65d98307ca24361f5b30fe785b53c3fd07b7a47e89acb5e6ac638cb"},
]

[package.dependencies]
jupyter-server = ">=1.8,<3"

[package.extras]
test = ["pytest", "pytest-console-scripts", "pytest-jupyter", "pytest-tornasync"]

[[package]]
name = "numpy"
version = "2.2.1"
description = "Fundamental package for array computing in Python"
optional = false
python-versions = ">=3.10"
files = [
    {file = "numpy-2.2.1-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:5edb4e4caf751c1518e6a26a83501fda79bff41cc59dac48d70e6d65d4ec4440"},
    {file = "numpy-2.2.1-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:aa3017c40d513ccac9621a2364f939d39e550c542eb2a894b4c8da92b38896ab"},
    {file = "numpy-2.2.1-cp310-cp310-macosx_14_0_arm64.whl", hash = "sha256:61048b4a49b1c93fe13426e04e04fdf5a03f456616f6e98c7576144677598675"},
    {file = "numpy-2.2.1-cp310-cp310-macosx_14_0_x86_64.whl", hash = "sha256:7671dc19c7019103ca44e8d94917eba8534c76133523ca8406822efdd19c9308"},
    {file = "numpy-2.2.1-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:4250888bcb96617e00bfa28ac24850a83c9f3a16db471eca2ee1f1714df0f957"},
    {file = "numpy-2.2.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:a7746f235c47abc72b102d3bce9977714c2444bdfaea7888d241b4c4bb6a78bf"},
    {file = "numpy-2.2.1-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:059e6a747ae84fce488c3ee397cee7e5f905fd1bda5fb18c66bc41807ff119b2"},
    {file = "numpy-2.2.1-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:f62aa6ee4eb43b024b0e5a01cf65a0bb078ef8c395e8713c6e8a12a697144528"},
    {file = "numpy-2.2.1-cp310-cp310-win32.whl", hash = "sha256:48fd472630715e1c1c89bf1feab55c29098cb403cc184b4859f9c86d4fcb6a95"},
    {file = "numpy-2.2.1-cp310-cp310-win_amd64.whl", hash = "sha256:b541032178a718c165a49638d28272b771053f628382d5e9d1c93df23ff58dbf"},
    {file = "numpy-2.2.1-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:40f9e544c1c56ba8f1cf7686a8c9b5bb249e665d40d626a23899ba6d5d9e1484"},
    {file = "numpy-2.2.1-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:f9b57eaa3b0cd8db52049ed0330747b0364e899e8a606a624813452b8203d5f7"},
    {file = "numpy-2.2.1-cp311-cp311-macosx_14_0_arm64.whl", hash = "sha256:bc8a37ad5b22c08e2dbd27df2b3ef7e5c0864235805b1e718a235bcb200cf1cb"},
    {file = "numpy-2.2.1-cp311-cp311-macosx_14_0_x86_64.whl", hash = "sha256:9036d6365d13b6cbe8f27a0eaf73ddcc070cae584e5ff94bb45e3e9d729feab5"},
    {file = "numpy-2.2.1-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:51faf345324db860b515d3f364eaa93d0e0551a88d6218a7d61286554d190d73"},
    {file = "numpy-2.2.1-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:38efc1e56b73cc9b182fe55e56e63b044dd26a72128fd2fbd502f75555d92591"},
    {file = "numpy-2.2.1-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:31b89fa67a8042e96715c68e071a1200c4e172f93b0fbe01a14c0ff3ff820fc8"},
    {file = "numpy-2.2.1-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:4c86e2a209199ead7ee0af65e1d9992d1dce7e1f63c4b9a616500f93820658d0"},
    {file = "numpy-2.2.1-cp311-cp311-win32.whl", hash = "sha256:b34d87e8a3090ea626003f87f9392b3929a7bbf4104a05b6667348b6bd4bf1cd"},
    {file = "numpy-2.2.1-cp311-cp311-win_amd64.whl", hash = "sha256:360137f8fb1b753c5cde3ac388597ad680eccbbbb3865ab65efea062c4a1fd16"},
    {file = "numpy-2.2.1-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:694f9e921a0c8f252980e85bce61ebbd07ed2b7d4fa72d0e4246f2f8aa6642ab"},
    {file = "numpy-2.2.1-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:3683a8d166f2692664262fd4900f207791d005fb088d7fdb973cc8d663626faa"},
    {file = "numpy-2.2.1-cp312-cp312-macosx_14_0_arm64.whl", hash = "sha256:780077d95eafc2ccc3ced969db22377b3864e5b9a0ea5eb347cc93b3ea900315"},
    {file = "numpy-2.2.1-cp312-cp312-macosx_14_0_x86_64.whl", hash = "sha256:55ba24ebe208344aa7a00e4482f65742969a039c2acfcb910bc6fcd776eb4355"},
    {file = "numpy-2.2.1-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:9b1d07b53b78bf84a96898c1bc139ad7f10fda7423f5fd158fd0f47ec5e01ac7"},
    {file = "numpy-2.2.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:5062dc1a4e32a10dc2b8b13cedd58988261416e811c1dc4dbdea4f57eea61b0d"},
    {file = "numpy-2.2.1-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:fce4f615f8ca31b2e61aa0eb5865a21e14f5629515c9151850aa936c02a1ee51"},
    {file = "numpy-2.2.1-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:67d4cda6fa6ffa073b08c8372aa5fa767ceb10c9a0587c707505a6d426f4e046"},
    {file = "numpy-2.2.1-cp312-cp312-win32.whl", hash = "sha256:32cb94448be47c500d2c7a95f93e2f21a01f1fd05dd2beea1ccd049bb6001cd2"},
    {file = "numpy-2.2.1-cp312-cp312-win_amd64.whl", hash = "sha256:ba5511d8f31c033a5fcbda22dd5c813630af98c70b2661f2d2c654ae3cdfcfc8"},
    {file = "numpy-2.2.1-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:f1d09e520217618e76396377c81fba6f290d5f926f50c35f3a5f72b01a0da780"},
    {file = "numpy-2.2.1-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:3ecc47cd7f6ea0336042be87d9e7da378e5c7e9b3c8ad0f7c966f714fc10d821"},
    {file = "numpy-2.2.1-cp313-cp313-macosx_14_0_arm64.whl", hash = "sha256:f419290bc8968a46c4933158c91a0012b7a99bb2e465d5ef5293879742f8797e"},
    {file = "numpy-2.2.1-cp313-cp313-macosx_14_0_x86_64.whl", hash = "sha256:5b6c390bfaef8c45a260554888966618328d30e72173697e5cabe6b285fb2348"},
    {file = "numpy-2.2.1-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:526fc406ab991a340744aad7e25251dd47a6720a685fa3331e5c59fef5282a59"},
    {file = "numpy-2.2.1-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:f74e6fdeb9a265624ec3a3918430205dff1df7e95a230779746a6af78bc615af"},
    {file = "numpy-2.2.1-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:53c09385ff0b72ba79d8715683c1168c12e0b6e84fb0372e97553d1ea91efe51"},
    {file = "numpy-2.2.1-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:f3eac17d9ec51be534685ba877b6ab5edc3ab7ec95c8f163e5d7b39859524716"},
    {file = "numpy-2.2.1-cp313-cp313-win32.whl", hash = "sha256:9ad014faa93dbb52c80d8f4d3dcf855865c876c9660cb9bd7553843dd03a4b1e"},
    {file = "numpy-2.2.1-cp313-cp313-win_amd64.whl", hash = "sha256:164a829b6aacf79ca47ba4814b130c4020b202522a93d7bff2202bfb33b61c60"},
    {file = "numpy-2.2.1-cp313-cp313t-macosx_10_13_x86_64.whl", hash = "sha256:4dfda918a13cc4f81e9118dea249e192ab167a0bb1966272d5503e39234d694e"},
    {file = "numpy-2.2.1-cp313-cp313t-macosx_11_0_arm64.whl", hash = "sha256:733585f9f4b62e9b3528dd1070ec4f52b8acf64215b60a845fa13ebd73cd0712"},
    {file = "numpy-2.2.1-cp313-cp313t-macosx_14_0_arm64.whl", hash = "sha256:89b16a18e7bba224ce5114db863e7029803c179979e1af6ad6a6b11f70545008"},
    {file = "numpy-2.2.1-cp313-cp313t-macosx_14_0_x86_64.whl", hash = "sha256:676f4eebf6b2d430300f1f4f4c2461685f8269f94c89698d832cdf9277f30b84"},
    {file = "numpy-2.2.1-cp313-cp313t-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:27f5cdf9f493b35f7e41e8368e7d7b4bbafaf9660cba53fb21d2cd174ec09631"},
    {file = "numpy-2.2.1-cp313-cp313t-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:c1ad395cf254c4fbb5b2132fee391f361a6e8c1adbd28f2cd8e79308a615fe9d"},
    {file = "numpy-2.2.1-cp313-cp313t-musllinux_1_2_aarch64.whl", hash = "sha256:08ef779aed40dbc52729d6ffe7dd51df85796a702afbf68a4f4e41fafdc8bda5"},
    {file = "numpy-2.2.1-cp313-cp313t-musllinux_1_2_x86_64.whl", hash = "sha256:26c9c4382b19fcfbbed3238a14abf7ff223890ea1936b8890f058e7ba35e8d71"},
    {file = "numpy-2.2.1-cp313-cp313t-win32.whl", hash = "sha256:93cf4e045bae74c90ca833cba583c14b62cb4ba2cba0abd2b141ab52548247e2"},
    {file = "numpy-2.2.1-cp313-cp313t-win_amd64.whl", hash = "sha256:bff7d8ec20f5f42607599f9994770fa65d76edca264a87b5e4ea5629bce12268"},
    {file = "numpy-2.2.1-pp310-pypy310_pp73-macosx_10_15_x86_64.whl", hash = "sha256:7ba9cc93a91d86365a5d270dee221fdc04fb68d7478e6bf6af650de78a8339e3"},
    {file = "numpy-2.2.1-pp310-pypy310_pp73-macosx_14_0_x86_64.whl", hash = "sha256:3d03883435a19794e41f147612a77a8f56d4e52822337844fff3d4040a142964"},
    {file = "numpy-2.2.1-pp310-pypy310_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:4511d9e6071452b944207c8ce46ad2f897307910b402ea5fa975da32e0102800"},
    {file = "numpy-2.2.1-pp310-pypy310_pp73-win_amd64.whl", hash = "sha256:5c5cc0cbabe9452038ed984d05ac87910f89370b9242371bd9079cb4af61811e"},
    {file = "numpy-2.2.1.tar.gz", hash = "sha256:45681fd7128c8ad1c379f0ca0776a8b0c6583d2f69889ddac01559dfe4390918"},
]

[[package]]
name = "orjson"
version = "3.10.12"
description = "Fast, correct Python JSON library supporting dataclasses, datetimes, and numpy"
optional = false
python-versions = ">=3.8"
files = [
    {file = "orjson-3.10.12-cp310-cp310-macosx_10_15_x86_64.macosx_11_0_arm64.macosx_10_15_universal2.whl", hash = "sha256:ece01a7ec71d9940cc654c482907a6b65df27251255097629d0dea781f255c6d"},
    {file = "orjson-3.10.12-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:c34ec9aebc04f11f4b978dd6caf697a2df2dd9b47d35aa4cc606cabcb9df69d7"},
    {file = "orjson-3.10.12-cp310-cp310-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:fd6ec8658da3480939c79b9e9e27e0db31dffcd4ba69c334e98c9976ac29140e"},
    {file = "orjson-3.10.12-cp310-cp310-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:f17e6baf4cf01534c9de8a16c0c611f3d94925d1701bf5f4aff17003677d8ced"},
    {file = "orjson-3.10.12-cp310-cp310-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:6402ebb74a14ef96f94a868569f5dccf70d791de49feb73180eb3c6fda2ade56"},
    {file = "orjson-3.10.12-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:0000758ae7c7853e0a4a6063f534c61656ebff644391e1f81698c1b2d2fc8cd2"},
    {file = "orjson-3.10.12-cp310-cp310-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:888442dcee99fd1e5bd37a4abb94930915ca6af4db50e23e746cdf4d1e63db13"},
    {file = "orjson-3.10.12-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:c1f7a3ce79246aa0e92f5458d86c54f257fb5dfdc14a192651ba7ec2c00f8a05"},
    {file = "orjson-3.10.12-cp310-cp310-musllinux_1_2_armv7l.whl", hash = "sha256:802a3935f45605c66fb4a586488a38af63cb37aaad1c1d94c982c40dcc452e85"},
    {file = "orjson-3.10.12-cp310-cp310-musllinux_1_2_i686.whl", hash = "sha256:1da1ef0113a2be19bb6c557fb0ec2d79c92ebd2fed4cfb1b26bab93f021fb885"},
    {file = "orjson-3.10.12-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:7a3273e99f367f137d5b3fecb5e9f45bcdbfac2a8b2f32fbc72129bbd48789c2"},
    {file = "orjson-3.10.12-cp310-none-win32.whl", hash = "sha256:475661bf249fd7907d9b0a2a2421b4e684355a77ceef85b8352439a9163418c3"},
    {file = "orjson-3.10.12-cp310-none-win_amd64.whl", hash = "sha256:87251dc1fb2b9e5ab91ce65d8f4caf21910d99ba8fb24b49fd0c118b2362d509"},
    {file = "orjson-3.10.12-cp311-cp311-macosx_10_15_x86_64.macosx_11_0_arm64.macosx_10_15_universal2.whl", hash = "sha256:a734c62efa42e7df94926d70fe7d37621c783dea9f707a98cdea796964d4cf74"},
    {file = "orjson-3.10.12-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:750f8b27259d3409eda8350c2919a58b0cfcd2054ddc1bd317a643afc646ef23"},
    {file = "orjson-3.10.12-cp311-cp311-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:bb52c22bfffe2857e7aa13b4622afd0dd9d16ea7cc65fd2bf318d3223b1b6252"},
    {file = "orjson-3.10.12-cp311-cp311-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:440d9a337ac8c199ff8251e100c62e9488924c92852362cd27af0e67308c16ef"},
    {file = "orjson-3.10.12-cp311-cp311-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:a9e15c06491c69997dfa067369baab3bf094ecb74be9912bdc4339972323f252"},
    {file = "orjson-3.10.12-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:362d204ad4b0b8724cf370d0cd917bb2dc913c394030da748a3bb632445ce7c4"},
    {file = "orjson-3.10.12-cp311-cp311-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:2b57cbb4031153db37b41622eac67329c7810e5f480fda4cfd30542186f006ae"},
    {file = "orjson-3.10.12-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:165c89b53ef03ce0d7c59ca5c82fa65fe13ddf52eeb22e859e58c237d4e33b9b"},
    {file = "orjson-3.10.12-cp311-cp311-musllinux_1_2_armv7l.whl", hash = "sha256:5dee91b8dfd54557c1a1596eb90bcd47dbcd26b0baaed919e6861f076583e9da"},
    {file = "orjson-3.10.12-cp311-cp311-musllinux_1_2_i686.whl", hash = "sha256:77a4e1cfb72de6f905bdff061172adfb3caf7a4578ebf481d8f0530879476c07"},
    {file = "orjson-3.10.12-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:038d42c7bc0606443459b8fe2d1f121db474c49067d8d14c6a075bbea8bf14dd"},
    {file = "orjson-3.10.12-cp311-none-win32.whl", hash = "sha256:03b553c02ab39bed249bedd4abe37b2118324d1674e639b33fab3d1dafdf4d79"},
    {file = "orjson-3.10.12-cp311-none-win_amd64.whl", hash = "sha256:8b8713b9e46a45b2af6b96f559bfb13b1e02006f4242c156cbadef27800a55a8"},
    {file = "orjson-3.10.12-cp312-cp312-macosx_10_15_x86_64.macosx_11_0_arm64.macosx_10_15_universal2.whl", hash = "sha256:53206d72eb656ca5ac7d3a7141e83c5bbd3ac30d5eccfe019409177a57634b0d"},
    {file = "orjson-3.10.12-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:ac8010afc2150d417ebda810e8df08dd3f544e0dd2acab5370cfa6bcc0662f8f"},
    {file = "orjson-3.10.12-cp312-cp312-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:ed459b46012ae950dd2e17150e838ab08215421487371fa79d0eced8d1461d70"},
    {file = "orjson-3.10.12-cp312-cp312-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:8dcb9673f108a93c1b52bfc51b0af422c2d08d4fc710ce9c839faad25020bb69"},
    {file = "orjson-3.10.12-cp312-cp312-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:22a51ae77680c5c4652ebc63a83d5255ac7d65582891d9424b566fb3b5375ee9"},
    {file = "orjson-3.10.12-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:910fdf2ac0637b9a77d1aad65f803bac414f0b06f720073438a7bd8906298192"},
    {file = "orjson-3.10.12-cp312-cp312-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:24ce85f7100160936bc2116c09d1a8492639418633119a2224114f67f63a4559"},
    {file = "orjson-3.10.12-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:8a76ba5fc8dd9c913640292df27bff80a685bed3a3c990d59aa6ce24c352f8fc"},
    {file = "orjson-3.10.12-cp312-cp312-musllinux_1_2_armv7l.whl", hash = "sha256:ff70ef093895fd53f4055ca75f93f047e088d1430888ca1229393a7c0521100f"},
    {file = "orjson-3.10.12-cp312-cp312-musllinux_1_2_i686.whl", hash = "sha256:f4244b7018b5753ecd10a6d324ec1f347da130c953a9c88432c7fbc8875d13be"},
    {file = "orjson-3.10.12-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:16135ccca03445f37921fa4b585cff9a58aa8d81ebcb27622e69bfadd220b32c"},
    {file = "orjson-3.10.12-cp312-none-win32.whl", hash = "sha256:2d879c81172d583e34153d524fcba5d4adafbab8349a7b9f16ae511c2cee8708"},
    {file = "orjson-3.10.12-cp312-none-win_amd64.whl", hash = "sha256:fc23f691fa0f5c140576b8c365bc942d577d861a9ee1142e4db468e4e17094fb"},
    {file = "orjson-3.10.12-cp313-cp313-macosx_10_15_x86_64.macosx_11_0_arm64.macosx_10_15_universal2.whl", hash = "sha256:47962841b2a8aa9a258b377f5188db31ba49af47d4003a32f55d6f8b19006543"},
    {file = "orjson-3.10.12-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:6334730e2532e77b6054e87ca84f3072bee308a45a452ea0bffbbbc40a67e296"},
    {file = "orjson-3.10.12-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:accfe93f42713c899fdac2747e8d0d5c659592df2792888c6c5f829472e4f85e"},
    {file = "orjson-3.10.12-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:a7974c490c014c48810d1dede6c754c3cc46598da758c25ca3b4001ac45b703f"},
    {file = "orjson-3.10.12-cp313-cp313-musllinux_1_2_armv7l.whl", hash = "sha256:3f250ce7727b0b2682f834a3facff88e310f52f07a5dcfd852d99637d386e79e"},
    {file = "orjson-3.10.12-cp313-cp313-musllinux_1_2_i686.whl", hash = "sha256:f31422ff9486ae484f10ffc51b5ab2a60359e92d0716fcce1b3593d7bb8a9af6"},
    {file = "orjson-3.10.12-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:5f29c5d282bb2d577c2a6bbde88d8fdcc4919c593f806aac50133f01b733846e"},
    {file = "orjson-3.10.12-cp313-none-win32.whl", hash = "sha256:f45653775f38f63dc0e6cd4f14323984c3149c05d6007b58cb154dd080ddc0dc"},
    {file = "orjson-3.10.12-cp313-none-win_amd64.whl", hash = "sha256:229994d0c376d5bdc91d92b3c9e6be2f1fbabd4cc1b59daae1443a46ee5e9825"},
    {file = "orjson-3.10.12-cp38-cp38-macosx_10_15_x86_64.macosx_11_0_arm64.macosx_10_15_universal2.whl", hash = "sha256:7d69af5b54617a5fac5c8e5ed0859eb798e2ce8913262eb522590239db6c6763"},
    {file = "orjson-3.10.12-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:7ed119ea7d2953365724a7059231a44830eb6bbb0cfead33fcbc562f5fd8f935"},
    {file = "orjson-3.10.12-cp38-cp38-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:9c5fc1238ef197e7cad5c91415f524aaa51e004be5a9b35a1b8a84ade196f73f"},
    {file = "orjson-3.10.12-cp38-cp38-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:43509843990439b05f848539d6f6198d4ac86ff01dd024b2f9a795c0daeeab60"},
    {file = "orjson-3.10.12-cp38-cp38-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:f72e27a62041cfb37a3de512247ece9f240a561e6c8662276beaf4d53d406db4"},
    {file = "orjson-3.10.12-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:9a904f9572092bb6742ab7c16c623f0cdccbad9eeb2d14d4aa06284867bddd31"},
    {file = "orjson-3.10.12-cp38-cp38-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:855c0833999ed5dc62f64552db26f9be767434917d8348d77bacaab84f787d7b"},
    {file = "orjson-3.10.12-cp38-cp38-musllinux_1_2_aarch64.whl", hash = "sha256:897830244e2320f6184699f598df7fb9db9f5087d6f3f03666ae89d607e4f8ed"},
    {file = "orjson-3.10.12-cp38-cp38-musllinux_1_2_armv7l.whl", hash = "sha256:0b32652eaa4a7539f6f04abc6243619c56f8530c53bf9b023e1269df5f7816dd"},
    {file = "orjson-3.10.12-cp38-cp38-musllinux_1_2_i686.whl", hash = "sha256:36b4aa31e0f6a1aeeb6f8377769ca5d125db000f05c20e54163aef1d3fe8e833"},
    {file = "orjson-3.10.12-cp38-cp38-musllinux_1_2_x86_64.whl", hash = "sha256:5535163054d6cbf2796f93e4f0dbc800f61914c0e3c4ed8499cf6ece22b4a3da"},
    {file = "orjson-3.10.12-cp38-none-win32.whl", hash = "sha256:90a5551f6f5a5fa07010bf3d0b4ca2de21adafbbc0af6cb700b63cd767266cb9"},
    {file = "orjson-3.10.12-cp38-none-win_amd64.whl", hash = "sha256:703a2fb35a06cdd45adf5d733cf613cbc0cb3ae57643472b16bc22d325b5fb6c"},
    {file = "orjson-3.10.12-cp39-cp39-macosx_10_15_x86_64.macosx_11_0_arm64.macosx_10_15_universal2.whl", hash = "sha256:f29de3ef71a42a5822765def1febfb36e0859d33abf5c2ad240acad5c6a1b78d"},
    {file = "orjson-3.10.12-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:de365a42acc65d74953f05e4772c974dad6c51cfc13c3240899f534d611be967"},
    {file = "orjson-3.10.12-cp39-cp39-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:91a5a0158648a67ff0004cb0df5df7dcc55bfc9ca154d9c01597a23ad54c8d0c"},
    {file = "orjson-3.10.12-cp39-cp39-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:c47ce6b8d90fe9646a25b6fb52284a14ff215c9595914af63a5933a49972ce36"},
    {file = "orjson-3.10.12-cp39-cp39-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:0eee4c2c5bfb5c1b47a5db80d2ac7aaa7e938956ae88089f098aff2c0f35d5d8"},
    {file = "orjson-3.10.12-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:35d3081bbe8b86587eb5c98a73b97f13d8f9fea685cf91a579beddacc0d10566"},
    {file = "orjson-3.10.12-cp39-cp39-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:73c23a6e90383884068bc2dba83d5222c9fcc3b99a0ed2411d38150734236755"},
    {file = "orjson-3.10.12-cp39-cp39-musllinux_1_2_aarch64.whl", hash = "sha256:5472be7dc3269b4b52acba1433dac239215366f89dc1d8d0e64029abac4e714e"},
    {file = "orjson-3.10.12-cp39-cp39-musllinux_1_2_armv7l.whl", hash = "sha256:7319cda750fca96ae5973efb31b17d97a5c5225ae0bc79bf5bf84df9e1ec2ab6"},
    {file = "orjson-3.10.12-cp39-cp39-musllinux_1_2_i686.whl", hash = "sha256:74d5ca5a255bf20b8def6a2b96b1e18ad37b4a122d59b154c458ee9494377f80"},
    {file = "orjson-3.10.12-cp39-cp39-musllinux_1_2_x86_64.whl", hash = "sha256:ff31d22ecc5fb85ef62c7d4afe8301d10c558d00dd24274d4bbe464380d3cd69"},
    {file = "orjson-3.10.12-cp39-none-win32.whl", hash = "sha256:c22c3ea6fba91d84fcb4cda30e64aff548fcf0c44c876e681f47d61d24b12e6b"},
    {file = "orjson-3.10.12-cp39-none-win_amd64.whl", hash = "sha256:be604f60d45ace6b0b33dd990a66b4526f1a7a186ac411c942674625456ca548"},
    {file = "orjson-3.10.12.tar.gz", hash = "sha256:0a78bbda3aea0f9f079057ee1ee8a1ecf790d4f1af88dd67493c6b8ee52506ff"},
]

[[package]]
name = "overrides"
version = "7.7.0"
description = "A decorator to automatically detect mismatch when overriding a method."
optional = false
python-versions = ">=3.6"
files = [
    {file = "overrides-7.7.0-py3-none-any.whl", hash = "sha256:c7ed9d062f78b8e4c1a7b70bd8796b35ead4d9f510227ef9c5dc7626c60d7e49"},
    {file = "overrides-7.7.0.tar.gz", hash = "sha256:55158fa3d93b98cc75299b1e67078ad9003ca27945c76162c1c0766d6f91820a"},
]

[[package]]
name = "packaging"
version = "24.2"
description = "Core utilities for Python packages"
optional = false
python-versions = ">=3.8"
files = [
    {file = "packaging-24.2-py3-none-any.whl", hash = "sha256:09abb1bccd265c01f4a3aa3f7a7db064b36514d2cba19a2f694fe6150451a759"},
    {file = "packaging-24.2.tar.gz", hash = "sha256:c228a6dc5e932d346bc5739379109d49e8853dd8223571c7c5b55260edc0b97f"},
]

[[package]]
name = "pandas"
version = "2.2.3"
description = "Powerful data structures for data analysis, time series, and statistics"
optional = false
python-versions = ">=3.9"
files = [
    {file = "pandas-2.2.3-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:1948ddde24197a0f7add2bdc4ca83bf2b1ef84a1bc8ccffd95eda17fd836ecb5"},
    {file = "pandas-2.2.3-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:381175499d3802cde0eabbaf6324cce0c4f5d52ca6f8c377c29ad442f50f6348"},
    {file = "pandas-2.2.3-cp310-cp310-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:d9c45366def9a3dd85a6454c0e7908f2b3b8e9c138f5dc38fed7ce720d8453ed"},
    {file = "pandas-2.2.3-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:86976a1c5b25ae3f8ccae3a5306e443569ee3c3faf444dfd0f41cda24667ad57"},
    {file = "pandas-2.2.3-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:b8661b0238a69d7aafe156b7fa86c44b881387509653fdf857bebc5e4008ad42"},
    {file = "pandas-2.2.3-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:37e0aced3e8f539eccf2e099f65cdb9c8aa85109b0be6e93e2baff94264bdc6f"},
    {file = "pandas-2.2.3-cp310-cp310-win_amd64.whl", hash = "sha256:56534ce0746a58afaf7942ba4863e0ef81c9c50d3f0ae93e9497d6a41a057645"},
    {file = "pandas-2.2.3-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:66108071e1b935240e74525006034333f98bcdb87ea116de573a6a0dccb6c039"},
    {file = "pandas-2.2.3-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:7c2875855b0ff77b2a64a0365e24455d9990730d6431b9e0ee18ad8acee13dbd"},
    {file = "pandas-2.2.3-cp311-cp311-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:cd8d0c3be0515c12fed0bdbae072551c8b54b7192c7b1fda0ba56059a0179698"},
    {file = "pandas-2.2.3-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:c124333816c3a9b03fbeef3a9f230ba9a737e9e5bb4060aa2107a86cc0a497fc"},
    {file = "pandas-2.2.3-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:63cc132e40a2e084cf01adf0775b15ac515ba905d7dcca47e9a251819c575ef3"},
    {file = "pandas-2.2.3-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:29401dbfa9ad77319367d36940cd8a0b3a11aba16063e39632d98b0e931ddf32"},
    {file = "pandas-2.2.3-cp311-cp311-win_amd64.whl", hash = "sha256:3fc6873a41186404dad67245896a6e440baacc92f5b716ccd1bc9ed2995ab2c5"},
    {file = "pandas-2.2.3-cp312-cp312-macosx_10_9_x86_64.whl", hash = "sha256:b1d432e8d08679a40e2a6d8b2f9770a5c21793a6f9f47fdd52c5ce1948a5a8a9"},
    {file = "pandas-2.2.3-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:a5a1595fe639f5988ba6a8e5bc9649af3baf26df3998a0abe56c02609392e0a4"},
    {file = "pandas-2.2.3-cp312-cp312-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:5de54125a92bb4d1c051c0659e6fcb75256bf799a732a87184e5ea503965bce3"},
    {file = "pandas-2.2.3-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:fffb8ae78d8af97f849404f21411c95062db1496aeb3e56f146f0355c9989319"},
    {file = "pandas-2.2.3-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:6dfcb5ee8d4d50c06a51c2fffa6cff6272098ad6540aed1a76d15fb9318194d8"},
    {file = "pandas-2.2.3-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:062309c1b9ea12a50e8ce661145c6aab431b1e99530d3cd60640e255778bd43a"},
    {file = "pandas-2.2.3-cp312-cp312-win_amd64.whl", hash = "sha256:59ef3764d0fe818125a5097d2ae867ca3fa64df032331b7e0917cf5d7bf66b13"},
    {file = "pandas-2.2.3-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:f00d1345d84d8c86a63e476bb4955e46458b304b9575dcf71102b5c705320015"},
    {file = "pandas-2.2.3-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:3508d914817e153ad359d7e069d752cdd736a247c322d932eb89e6bc84217f28"},
    {file = "pandas-2.2.3-cp313-cp313-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:22a9d949bfc9a502d320aa04e5d02feab689d61da4e7764b62c30b991c42c5f0"},
    {file = "pandas-2.2.3-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:f3a255b2c19987fbbe62a9dfd6cff7ff2aa9ccab3fc75218fd4b7530f01efa24"},
    {file = "pandas-2.2.3-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:800250ecdadb6d9c78eae4990da62743b857b470883fa27f652db8bdde7f6659"},
    {file = "pandas-2.2.3-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:6374c452ff3ec675a8f46fd9ab25c4ad0ba590b71cf0656f8b6daa5202bca3fb"},
    {file = "pandas-2.2.3-cp313-cp313-win_amd64.whl", hash = "sha256:61c5ad4043f791b61dd4752191d9f07f0ae412515d59ba8f005832a532f8736d"},
    {file = "pandas-2.2.3-cp313-cp313t-macosx_10_13_x86_64.whl", hash = "sha256:3b71f27954685ee685317063bf13c7709a7ba74fc996b84fc6821c59b0f06468"},
    {file = "pandas-2.2.3-cp313-cp313t-macosx_11_0_arm64.whl", hash = "sha256:38cf8125c40dae9d5acc10fa66af8ea6fdf760b2714ee482ca691fc66e6fcb18"},
    {file = "pandas-2.2.3-cp313-cp313t-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:ba96630bc17c875161df3818780af30e43be9b166ce51c9a18c1feae342906c2"},
    {file = "pandas-2.2.3-cp313-cp313t-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:1db71525a1538b30142094edb9adc10be3f3e176748cd7acc2240c2f2e5aa3a4"},
    {file = "pandas-2.2.3-cp313-cp313t-musllinux_1_2_aarch64.whl", hash = "sha256:15c0e1e02e93116177d29ff83e8b1619c93ddc9c49083f237d4312337a61165d"},
    {file = "pandas-2.2.3-cp313-cp313t-musllinux_1_2_x86_64.whl", hash = "sha256:ad5b65698ab28ed8d7f18790a0dc58005c7629f227be9ecc1072aa74c0c1d43a"},
    {file = "pandas-2.2.3-cp39-cp39-macosx_10_9_x86_64.whl", hash = "sha256:bc6b93f9b966093cb0fd62ff1a7e4c09e6d546ad7c1de191767baffc57628f39"},
    {file = "pandas-2.2.3-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:5dbca4c1acd72e8eeef4753eeca07de9b1db4f398669d5994086f788a5d7cc30"},
    {file = "pandas-2.2.3-cp39-cp39-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:8cd6d7cc958a3910f934ea8dbdf17b2364827bb4dafc38ce6eef6bb3d65ff09c"},
    {file = "pandas-2.2.3-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:99df71520d25fade9db7c1076ac94eb994f4d2673ef2aa2e86ee039b6746d20c"},
    {file = "pandas-2.2.3-cp39-cp39-musllinux_1_2_aarch64.whl", hash = "sha256:31d0ced62d4ea3e231a9f228366919a5ea0b07440d9d4dac345376fd8e1477ea"},
    {file = "pandas-2.2.3-cp39-cp39-musllinux_1_2_x86_64.whl", hash = "sha256:7eee9e7cea6adf3e3d24e304ac6b8300646e2a5d1cd3a3c2abed9101b0846761"},
    {file = "pandas-2.2.3-cp39-cp39-win_amd64.whl", hash = "sha256:4850ba03528b6dd51d6c5d273c46f183f39a9baf3f0143e566b89450965b105e"},
    {file = "pandas-2.2.3.tar.gz", hash = "sha256:4f18ba62b61d7e192368b84517265a99b4d7ee8912f8708660fb4a366cc82667"},
]

[package.dependencies]
numpy = {version = ">=1.26.0", markers = "python_version >= \"3.12\""}
python-dateutil = ">=2.8.2"
pytz = ">=2020.1"
tzdata = ">=2022.7"

[package.extras]
all = ["PyQt5 (>=5.15.9)", "SQLAlchemy (>=2.0.0)", "adbc-driver-postgresql (>=0.8.0)", "adbc-driver-sqlite (>=0.8.0)", "beautifulsoup4 (>=4.11.2)", "bottleneck (>=1.3.6)", "dataframe-api-compat (>=0.1.7)", "fastparquet (>=2022.12.0)", "fsspec (>=2022.11.0)", "gcsfs (>=2022.11.0)", "html5lib (>=1.1)", "hypothesis (>=6.46.1)", "jinja2 (>=3.1.2)", "lxml (>=4.9.2)", "matplotlib (>=3.6.3)", "numba (>=0.56.4)", "numexpr (>=2.8.4)", "odfpy (>=1.4.1)", "openpyxl (>=3.1.0)", "pandas-gbq (>=0.19.0)", "psycopg2 (>=2.9.6)", "pyarrow (>=10.0.1)", "pymysql (>=1.0.2)", "pyreadstat (>=1.2.0)", "pytest (>=7.3.2)", "pytest-xdist (>=2.2.0)", "python-calamine (>=0.1.7)", "pyxlsb (>=1.0.10)", "qtpy (>=2.3.0)", "s3fs (>=2022.11.0)", "scipy (>=1.10.0)", "tables (>=3.8.0)", "tabulate (>=0.9.0)", "xarray (>=2022.12.0)", "xlrd (>=2.0.1)", "xlsxwriter (>=3.0.5)", "zstandard (>=0.19.0)"]
aws = ["s3fs (>=2022.11.0)"]
clipboard = ["PyQt5 (>=5.15.9)", "qtpy (>=2.3.0)"]
compression = ["zstandard (>=0.19.0)"]
computation = ["scipy (>=1.10.0)", "xarray (>=2022.12.0)"]
consortium-standard = ["dataframe-api-compat (>=0.1.7)"]
excel = ["odfpy (>=1.4.1)", "openpyxl (>=3.1.0)", "python-calamine (>=0.1.7)", "pyxlsb (>=1.0.10)", "xlrd (>=2.0.1)", "xlsxwriter (>=3.0.5)"]
feather = ["pyarrow (>=10.0.1)"]
fss = ["fsspec (>=2022.11.0)"]
gcp = ["gcsfs (>=2022.11.0)", "pandas-gbq (>=0.19.0)"]
hdf5 = ["tables (>=3.8.0)"]
html = ["beautifulsoup4 (>=4.11.2)", "html5lib (>=1.1)", "lxml (>=4.9.2)"]
mysql = ["SQLAlchemy (>=2.0.0)", "pymysql (>=1.0.2)"]
output-formatting = ["jinja2 (>=3.1.2)", "tabulate (>=0.9.0)"]
parquet = ["pyarrow (>=10.0.1)"]
performance = ["bottleneck (>=1.3.6)", "numba (>=0.56.4)", "numexpr (>=2.8.4)"]
plot = ["matplotlib (>=3.6.3)"]
postgresql = ["SQLAlchemy (>=2.0.0)", "adbc-driver-postgresql (>=0.8.0)", "psycopg2 (>=2.9.6)"]
pyarrow = ["pyarrow (>=10.0.1)"]
spss = ["pyreadstat (>=1.2.0)"]
sql-other = ["SQLAlchemy (>=2.0.0)", "adbc-driver-postgresql (>=0.8.0)", "adbc-driver-sqlite (>=0.8.0)"]
test = ["hypothesis (>=6.46.1)", "pytest (>=7.3.2)", "pytest-xdist (>=2.2.0)"]
xml = ["lxml (>=4.9.2)"]

[[package]]
name = "pandocfilters"
version = "1.5.1"
description = "Utilities for writing pandoc filters in python"
optional = false
python-versions = ">=2.7, !=3.0.*, !=3.1.*, !=3.2.*, !=3.3.*"
files = [
    {file = "pandocfilters-1.5.1-py2.py3-none-any.whl", hash = "sha256:93be382804a9cdb0a7267585f157e5d1731bbe5545a85b268d6f5fe6232de2bc"},
    {file = "pandocfilters-1.5.1.tar.gz", hash = "sha256:002b4a555ee4ebc03f8b66307e287fa492e4a77b4ea14d3f934328297bb4939e"},
]

[[package]]
name = "parso"
version = "0.8.4"
description = "A Python Parser"
optional = false
python-versions = ">=3.6"
files = [
    {file = "parso-0.8.4-py2.py3-none-any.whl", hash = "sha256:a418670a20291dacd2dddc80c377c5c3791378ee1e8d12bffc35420643d43f18"},
    {file = "parso-0.8.4.tar.gz", hash = "sha256:eb3a7b58240fb99099a345571deecc0f9540ea5f4dd2fe14c2a99d6b281ab92d"},
]

[package.extras]
qa = ["flake8 (==5.0.4)", "mypy (==0.971)", "types-setuptools (==67.2.0.1)"]
testing = ["docopt", "pytest"]

[[package]]
name = "pathspec"
version = "0.12.1"
description = "Utility library for gitignore style pattern matching of file paths."
optional = false
python-versions = ">=3.8"
files = [
    {file = "pathspec-0.12.1-py3-none-any.whl", hash = "sha256:a0d503e138a4c123b27490a4f7beda6a01c6f288df0e4a8b79c7eb0dc7b4cc08"},
    {file = "pathspec-0.12.1.tar.gz", hash = "sha256:a482d51503a1ab33b1c67a6c3813a26953dbdc71c31dacaef9a838c4e29f5712"},
]

[[package]]
name = "pexpect"
version = "4.9.0"
description = "Pexpect allows easy control of interactive console applications."
optional = false
python-versions = "*"
files = [
    {file = "pexpect-4.9.0-py2.py3-none-any.whl", hash = "sha256:7236d1e080e4936be2dc3e326cec0af72acf9212a7e1d060210e70a47e253523"},
    {file = "pexpect-4.9.0.tar.gz", hash = "sha256:ee7d41123f3c9911050ea2c2dac107568dc43b2d3b0c7557a33212c398ead30f"},
]

[package.dependencies]
ptyprocess = ">=0.5"

[[package]]
name = "pillow"
version = "11.0.0"
description = "Python Imaging Library (Fork)"
optional = false
python-versions = ">=3.9"
files = [
    {file = "pillow-11.0.0-cp310-cp310-macosx_10_10_x86_64.whl", hash = "sha256:6619654954dc4936fcff82db8eb6401d3159ec6be81e33c6000dfd76ae189947"},
    {file = "pillow-11.0.0-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:b3c5ac4bed7519088103d9450a1107f76308ecf91d6dabc8a33a2fcfb18d0fba"},
    {file = "pillow-11.0.0-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:a65149d8ada1055029fcb665452b2814fe7d7082fcb0c5bed6db851cb69b2086"},
    {file = "pillow-11.0.0-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:88a58d8ac0cc0e7f3a014509f0455248a76629ca9b604eca7dc5927cc593c5e9"},
    {file = "pillow-11.0.0-cp310-cp310-manylinux_2_28_aarch64.whl", hash = "sha256:c26845094b1af3c91852745ae78e3ea47abf3dbcd1cf962f16b9a5fbe3ee8488"},
    {file = "pillow-11.0.0-cp310-cp310-manylinux_2_28_x86_64.whl", hash = "sha256:1a61b54f87ab5786b8479f81c4b11f4d61702830354520837f8cc791ebba0f5f"},
    {file = "pillow-11.0.0-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:674629ff60030d144b7bca2b8330225a9b11c482ed408813924619c6f302fdbb"},
    {file = "pillow-11.0.0-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:598b4e238f13276e0008299bd2482003f48158e2b11826862b1eb2ad7c768b97"},
    {file = "pillow-11.0.0-cp310-cp310-win32.whl", hash = "sha256:9a0f748eaa434a41fccf8e1ee7a3eed68af1b690e75328fd7a60af123c193b50"},
    {file = "pillow-11.0.0-cp310-cp310-win_amd64.whl", hash = "sha256:a5629742881bcbc1f42e840af185fd4d83a5edeb96475a575f4da50d6ede337c"},
    {file = "pillow-11.0.0-cp310-cp310-win_arm64.whl", hash = "sha256:ee217c198f2e41f184f3869f3e485557296d505b5195c513b2bfe0062dc537f1"},
    {file = "pillow-11.0.0-cp311-cp311-macosx_10_10_x86_64.whl", hash = "sha256:1c1d72714f429a521d8d2d018badc42414c3077eb187a59579f28e4270b4b0fc"},
    {file = "pillow-11.0.0-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:499c3a1b0d6fc8213519e193796eb1a86a1be4b1877d678b30f83fd979811d1a"},
    {file = "pillow-11.0.0-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:c8b2351c85d855293a299038e1f89db92a2f35e8d2f783489c6f0b2b5f3fe8a3"},
    {file = "pillow-11.0.0-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:6f4dba50cfa56f910241eb7f883c20f1e7b1d8f7d91c750cd0b318bad443f4d5"},
    {file = "pillow-11.0.0-cp311-cp311-manylinux_2_28_aarch64.whl", hash = "sha256:5ddbfd761ee00c12ee1be86c9c0683ecf5bb14c9772ddbd782085779a63dd55b"},
    {file = "pillow-11.0.0-cp311-cp311-manylinux_2_28_x86_64.whl", hash = "sha256:45c566eb10b8967d71bf1ab8e4a525e5a93519e29ea071459ce517f6b903d7fa"},
    {file = "pillow-11.0.0-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:b4fd7bd29610a83a8c9b564d457cf5bd92b4e11e79a4ee4716a63c959699b306"},
    {file = "pillow-11.0.0-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:cb929ca942d0ec4fac404cbf520ee6cac37bf35be479b970c4ffadf2b6a1cad9"},
    {file = "pillow-11.0.0-cp311-cp311-win32.whl", hash = "sha256:006bcdd307cc47ba43e924099a038cbf9591062e6c50e570819743f5607404f5"},
    {file = "pillow-11.0.0-cp311-cp311-win_amd64.whl", hash = "sha256:52a2d8323a465f84faaba5236567d212c3668f2ab53e1c74c15583cf507a0291"},
    {file = "pillow-11.0.0-cp311-cp311-win_arm64.whl", hash = "sha256:16095692a253047fe3ec028e951fa4221a1f3ed3d80c397e83541a3037ff67c9"},
    {file = "pillow-11.0.0-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:d2c0a187a92a1cb5ef2c8ed5412dd8d4334272617f532d4ad4de31e0495bd923"},
    {file = "pillow-11.0.0-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:084a07ef0821cfe4858fe86652fffac8e187b6ae677e9906e192aafcc1b69903"},
    {file = "pillow-11.0.0-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:8069c5179902dcdce0be9bfc8235347fdbac249d23bd90514b7a47a72d9fecf4"},
    {file = "pillow-11.0.0-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:f02541ef64077f22bf4924f225c0fd1248c168f86e4b7abdedd87d6ebaceab0f"},
    {file = "pillow-11.0.0-cp312-cp312-manylinux_2_28_aarch64.whl", hash = "sha256:fcb4621042ac4b7865c179bb972ed0da0218a076dc1820ffc48b1d74c1e37fe9"},
    {file = "pillow-11.0.0-cp312-cp312-manylinux_2_28_x86_64.whl", hash = "sha256:00177a63030d612148e659b55ba99527803288cea7c75fb05766ab7981a8c1b7"},
    {file = "pillow-11.0.0-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:8853a3bf12afddfdf15f57c4b02d7ded92c7a75a5d7331d19f4f9572a89c17e6"},
    {file = "pillow-11.0.0-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:3107c66e43bda25359d5ef446f59c497de2b5ed4c7fdba0894f8d6cf3822dafc"},
    {file = "pillow-11.0.0-cp312-cp312-win32.whl", hash = "sha256:86510e3f5eca0ab87429dd77fafc04693195eec7fd6a137c389c3eeb4cfb77c6"},
    {file = "pillow-11.0.0-cp312-cp312-win_amd64.whl", hash = "sha256:8ec4a89295cd6cd4d1058a5e6aec6bf51e0eaaf9714774e1bfac7cfc9051db47"},
    {file = "pillow-11.0.0-cp312-cp312-win_arm64.whl", hash = "sha256:27a7860107500d813fcd203b4ea19b04babe79448268403172782754870dac25"},
    {file = "pillow-11.0.0-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:bcd1fb5bb7b07f64c15618c89efcc2cfa3e95f0e3bcdbaf4642509de1942a699"},
    {file = "pillow-11.0.0-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:0e038b0745997c7dcaae350d35859c9715c71e92ffb7e0f4a8e8a16732150f38"},
    {file = "pillow-11.0.0-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:0ae08bd8ffc41aebf578c2af2f9d8749d91f448b3bfd41d7d9ff573d74f2a6b2"},
    {file = "pillow-11.0.0-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:d69bfd8ec3219ae71bcde1f942b728903cad25fafe3100ba2258b973bd2bc1b2"},
    {file = "pillow-11.0.0-cp313-cp313-manylinux_2_28_aarch64.whl", hash = "sha256:61b887f9ddba63ddf62fd02a3ba7add935d053b6dd7d58998c630e6dbade8527"},
    {file = "pillow-11.0.0-cp313-cp313-manylinux_2_28_x86_64.whl", hash = "sha256:c6a660307ca9d4867caa8d9ca2c2658ab685de83792d1876274991adec7b93fa"},
    {file = "pillow-11.0.0-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:73e3a0200cdda995c7e43dd47436c1548f87a30bb27fb871f352a22ab8dcf45f"},
    {file = "pillow-11.0.0-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:fba162b8872d30fea8c52b258a542c5dfd7b235fb5cb352240c8d63b414013eb"},
    {file = "pillow-11.0.0-cp313-cp313-win32.whl", hash = "sha256:f1b82c27e89fffc6da125d5eb0ca6e68017faf5efc078128cfaa42cf5cb38798"},
    {file = "pillow-11.0.0-cp313-cp313-win_amd64.whl", hash = "sha256:8ba470552b48e5835f1d23ecb936bb7f71d206f9dfeee64245f30c3270b994de"},
    {file = "pillow-11.0.0-cp313-cp313-win_arm64.whl", hash = "sha256:846e193e103b41e984ac921b335df59195356ce3f71dcfd155aa79c603873b84"},
    {file = "pillow-11.0.0-cp313-cp313t-macosx_10_13_x86_64.whl", hash = "sha256:4ad70c4214f67d7466bea6a08061eba35c01b1b89eaa098040a35272a8efb22b"},
    {file = "pillow-11.0.0-cp313-cp313t-macosx_11_0_arm64.whl", hash = "sha256:6ec0d5af64f2e3d64a165f490d96368bb5dea8b8f9ad04487f9ab60dc4bb6003"},
    {file = "pillow-11.0.0-cp313-cp313t-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:c809a70e43c7977c4a42aefd62f0131823ebf7dd73556fa5d5950f5b354087e2"},
    {file = "pillow-11.0.0-cp313-cp313t-manylinux_2_28_x86_64.whl", hash = "sha256:4b60c9520f7207aaf2e1d94de026682fc227806c6e1f55bba7606d1c94dd623a"},
    {file = "pillow-11.0.0-cp313-cp313t-musllinux_1_2_x86_64.whl", hash = "sha256:1e2688958a840c822279fda0086fec1fdab2f95bf2b717b66871c4ad9859d7e8"},
    {file = "pillow-11.0.0-cp313-cp313t-win32.whl", hash = "sha256:607bbe123c74e272e381a8d1957083a9463401f7bd01287f50521ecb05a313f8"},
    {file = "pillow-11.0.0-cp313-cp313t-win_amd64.whl", hash = "sha256:5c39ed17edea3bc69c743a8dd3e9853b7509625c2462532e62baa0732163a904"},
    {file = "pillow-11.0.0-cp313-cp313t-win_arm64.whl", hash = "sha256:75acbbeb05b86bc53cbe7b7e6fe00fbcf82ad7c684b3ad82e3d711da9ba287d3"},
    {file = "pillow-11.0.0-cp39-cp39-macosx_10_10_x86_64.whl", hash = "sha256:2e46773dc9f35a1dd28bd6981332fd7f27bec001a918a72a79b4133cf5291dba"},
    {file = "pillow-11.0.0-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:2679d2258b7f1192b378e2893a8a0a0ca472234d4c2c0e6bdd3380e8dfa21b6a"},
    {file = "pillow-11.0.0-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:eda2616eb2313cbb3eebbe51f19362eb434b18e3bb599466a1ffa76a033fb916"},
    {file = "pillow-11.0.0-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:20ec184af98a121fb2da42642dea8a29ec80fc3efbaefb86d8fdd2606619045d"},
    {file = "pillow-11.0.0-cp39-cp39-manylinux_2_28_aarch64.whl", hash = "sha256:8594f42df584e5b4bb9281799698403f7af489fba84c34d53d1c4bfb71b7c4e7"},
    {file = "pillow-11.0.0-cp39-cp39-manylinux_2_28_x86_64.whl", hash = "sha256:c12b5ae868897c7338519c03049a806af85b9b8c237b7d675b8c5e089e4a618e"},
    {file = "pillow-11.0.0-cp39-cp39-musllinux_1_2_aarch64.whl", hash = "sha256:70fbbdacd1d271b77b7721fe3cdd2d537bbbd75d29e6300c672ec6bb38d9672f"},
    {file = "pillow-11.0.0-cp39-cp39-musllinux_1_2_x86_64.whl", hash = "sha256:5178952973e588b3f1360868847334e9e3bf49d19e169bbbdfaf8398002419ae"},
    {file = "pillow-11.0.0-cp39-cp39-win32.whl", hash = "sha256:8c676b587da5673d3c75bd67dd2a8cdfeb282ca38a30f37950511766b26858c4"},
    {file = "pillow-11.0.0-cp39-cp39-win_amd64.whl", hash = "sha256:94f3e1780abb45062287b4614a5bc0874519c86a777d4a7ad34978e86428b8dd"},
    {file = "pillow-11.0.0-cp39-cp39-win_arm64.whl", hash = "sha256:290f2cc809f9da7d6d622550bbf4c1e57518212da51b6a30fe8e0a270a5b78bd"},
    {file = "pillow-11.0.0-pp310-pypy310_pp73-macosx_10_15_x86_64.whl", hash = "sha256:1187739620f2b365de756ce086fdb3604573337cc28a0d3ac4a01ab6b2d2a6d2"},
    {file = "pillow-11.0.0-pp310-pypy310_pp73-macosx_11_0_arm64.whl", hash = "sha256:fbbcb7b57dc9c794843e3d1258c0fbf0f48656d46ffe9e09b63bbd6e8cd5d0a2"},
    {file = "pillow-11.0.0-pp310-pypy310_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:5d203af30149ae339ad1b4f710d9844ed8796e97fda23ffbc4cc472968a47d0b"},
    {file = "pillow-11.0.0-pp310-pypy310_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:21a0d3b115009ebb8ac3d2ebec5c2982cc693da935f4ab7bb5c8ebe2f47d36f2"},
    {file = "pillow-11.0.0-pp310-pypy310_pp73-manylinux_2_28_aarch64.whl", hash = "sha256:73853108f56df97baf2bb8b522f3578221e56f646ba345a372c78326710d3830"},
    {file = "pillow-11.0.0-pp310-pypy310_pp73-manylinux_2_28_x86_64.whl", hash = "sha256:e58876c91f97b0952eb766123bfef372792ab3f4e3e1f1a2267834c2ab131734"},
    {file = "pillow-11.0.0-pp310-pypy310_pp73-win_amd64.whl", hash = "sha256:224aaa38177597bb179f3ec87eeefcce8e4f85e608025e9cfac60de237ba6316"},
    {file = "pillow-11.0.0-pp39-pypy39_pp73-macosx_11_0_arm64.whl", hash = "sha256:5bd2d3bdb846d757055910f0a59792d33b555800813c3b39ada1829c372ccb06"},
    {file = "pillow-11.0.0-pp39-pypy39_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:375b8dd15a1f5d2feafff536d47e22f69625c1aa92f12b339ec0b2ca40263273"},
    {file = "pillow-11.0.0-pp39-pypy39_pp73-manylinux_2_28_x86_64.whl", hash = "sha256:daffdf51ee5db69a82dd127eabecce20729e21f7a3680cf7cbb23f0829189790"},
    {file = "pillow-11.0.0-pp39-pypy39_pp73-win_amd64.whl", hash = "sha256:7326a1787e3c7b0429659e0a944725e1b03eeaa10edd945a86dead1913383944"},
    {file = "pillow-11.0.0.tar.gz", hash = "sha256:72bacbaf24ac003fea9bff9837d1eedb6088758d41e100c1552930151f677739"},
]

[package.extras]
docs = ["furo", "olefile", "sphinx (>=8.1)", "sphinx-copybutton", "sphinx-inline-tabs", "sphinxext-opengraph"]
fpx = ["olefile"]
mic = ["olefile"]
tests = ["check-manifest", "coverage", "defusedxml", "markdown2", "olefile", "packaging", "pyroma", "pytest", "pytest-cov", "pytest-timeout"]
typing = ["typing-extensions"]
xmp = ["defusedxml"]

[[package]]
name = "platformdirs"
version = "4.3.6"
description = "A small Python package for determining appropriate platform-specific dirs, e.g. a `user data dir`."
optional = false
python-versions = ">=3.8"
files = [
    {file = "platformdirs-4.3.6-py3-none-any.whl", hash = "sha256:73e575e1408ab8103900836b97580d5307456908a03e92031bab39e4554cc3fb"},
    {file = "platformdirs-4.3.6.tar.gz", hash = "sha256:357fb2acbc885b0419afd3ce3ed34564c13c9b95c89360cd9563f73aa5e2b907"},
]

[package.extras]
docs = ["furo (>=2024.8.6)", "proselint (>=0.14)", "sphinx (>=8.0.2)", "sphinx-autodoc-typehints (>=2.4)"]
test = ["appdirs (==1.4.4)", "covdefaults (>=2.3)", "pytest (>=8.3.2)", "pytest-cov (>=5)", "pytest-mock (>=3.14)"]
type = ["mypy (>=1.11.2)"]

[[package]]
name = "plotly"
version = "5.24.1"
description = "An open-source, interactive data visualization library for Python"
optional = false
python-versions = ">=3.8"
files = [
    {file = "plotly-5.24.1-py3-none-any.whl", hash = "sha256:f67073a1e637eb0dc3e46324d9d51e2fe76e9727c892dde64ddf1e1b51f29089"},
    {file = "plotly-5.24.1.tar.gz", hash = "sha256:dbc8ac8339d248a4bcc36e08a5659bacfe1b079390b8953533f4eb22169b4bae"},
]

[package.dependencies]
packaging = "*"
tenacity = ">=6.2.0"

[[package]]
name = "pluggy"
version = "1.5.0"
description = "plugin and hook calling mechanisms for python"
optional = false
python-versions = ">=3.8"
files = [
    {file = "pluggy-1.5.0-py3-none-any.whl", hash = "sha256:44e1ad92c8ca002de6377e165f3e0f1be63266ab4d554740532335b9d75ea669"},
    {file = "pluggy-1.5.0.tar.gz", hash = "sha256:2cffa88e94fdc978c4c574f15f9e59b7f4201d439195c3715ca9e2486f1d0cf1"},
]

[package.extras]
dev = ["pre-commit", "tox"]
testing = ["pytest", "pytest-benchmark"]

[[package]]
name = "pprintpp"
version = "0.4.0"
description = "A drop-in replacement for pprint that's actually pretty"
optional = false
python-versions = "*"
files = [
    {file = "pprintpp-0.4.0-py2.py3-none-any.whl", hash = "sha256:b6b4dcdd0c0c0d75e4d7b2f21a9e933e5b2ce62b26e1a54537f9651ae5a5c01d"},
    {file = "pprintpp-0.4.0.tar.gz", hash = "sha256:ea826108e2c7f49dc6d66c752973c3fc9749142a798d6b254e1e301cfdbc6403"},
]

[[package]]
name = "prettytable"
version = "3.12.0"
description = "A simple Python library for easily displaying tabular data in a visually appealing ASCII table format"
optional = false
python-versions = ">=3.9"
files = [
    {file = "prettytable-3.12.0-py3-none-any.whl", hash = "sha256:77ca0ad1c435b6e363d7e8623d7cc4fcf2cf15513bf77a1c1b2e814930ac57cc"},
    {file = "prettytable-3.12.0.tar.gz", hash = "sha256:f04b3e1ba35747ac86e96ec33e3bb9748ce08e254dc2a1c6253945901beec804"},
]

[package.dependencies]
wcwidth = "*"

[package.extras]
tests = ["pytest", "pytest-cov", "pytest-lazy-fixtures"]

[[package]]
name = "prometheus-client"
version = "0.21.1"
description = "Python client for the Prometheus monitoring system."
optional = false
python-versions = ">=3.8"
files = [
    {file = "prometheus_client-0.21.1-py3-none-any.whl", hash = "sha256:594b45c410d6f4f8888940fe80b5cc2521b305a1fafe1c58609ef715a001f301"},
    {file = "prometheus_client-0.21.1.tar.gz", hash = "sha256:252505a722ac04b0456be05c05f75f45d760c2911ffc45f2a06bcaed9f3ae3fb"},
]

[package.extras]
twisted = ["twisted"]

[[package]]
name = "prompt-toolkit"
version = "3.0.48"
description = "Library for building powerful interactive command lines in Python"
optional = false
python-versions = ">=3.7.0"
files = [
    {file = "prompt_toolkit-3.0.48-py3-none-any.whl", hash = "sha256:f49a827f90062e411f1ce1f854f2aedb3c23353244f8108b89283587397ac10e"},
    {file = "prompt_toolkit-3.0.48.tar.gz", hash = "sha256:d6623ab0477a80df74e646bdbc93621143f5caf104206aa29294d53de1a03d90"},
]

[package.dependencies]
wcwidth = "*"

[[package]]
name = "psutil"
version = "6.1.1"
description = "Cross-platform lib for process and system monitoring in Python."
optional = false
python-versions = "!=3.0.*,!=3.1.*,!=3.2.*,!=3.3.*,!=3.4.*,!=3.5.*,>=2.7"
files = [
    {file = "psutil-6.1.1-cp27-cp27m-macosx_10_9_x86_64.whl", hash = "sha256:9ccc4316f24409159897799b83004cb1e24f9819b0dcf9c0b68bdcb6cefee6a8"},
    {file = "psutil-6.1.1-cp27-cp27m-manylinux2010_i686.whl", hash = "sha256:ca9609c77ea3b8481ab005da74ed894035936223422dc591d6772b147421f777"},
    {file = "psutil-6.1.1-cp27-cp27m-manylinux2010_x86_64.whl", hash = "sha256:8df0178ba8a9e5bc84fed9cfa61d54601b371fbec5c8eebad27575f1e105c0d4"},
    {file = "psutil-6.1.1-cp27-cp27mu-manylinux2010_i686.whl", hash = "sha256:1924e659d6c19c647e763e78670a05dbb7feaf44a0e9c94bf9e14dfc6ba50468"},
    {file = "psutil-6.1.1-cp27-cp27mu-manylinux2010_x86_64.whl", hash = "sha256:018aeae2af92d943fdf1da6b58665124897cfc94faa2ca92098838f83e1b1bca"},
    {file = "psutil-6.1.1-cp27-none-win32.whl", hash = "sha256:6d4281f5bbca041e2292be3380ec56a9413b790579b8e593b1784499d0005dac"},
    {file = "psutil-6.1.1-cp27-none-win_amd64.whl", hash = "sha256:c777eb75bb33c47377c9af68f30e9f11bc78e0f07fbf907be4a5d70b2fe5f030"},
    {file = "psutil-6.1.1-cp36-abi3-macosx_10_9_x86_64.whl", hash = "sha256:fc0ed7fe2231a444fc219b9c42d0376e0a9a1a72f16c5cfa0f68d19f1a0663e8"},
    {file = "psutil-6.1.1-cp36-abi3-macosx_11_0_arm64.whl", hash = "sha256:0bdd4eab935276290ad3cb718e9809412895ca6b5b334f5a9111ee6d9aff9377"},
    {file = "psutil-6.1.1-cp36-abi3-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:b6e06c20c05fe95a3d7302d74e7097756d4ba1247975ad6905441ae1b5b66003"},
    {file = "psutil-6.1.1-cp36-abi3-manylinux_2_12_x86_64.manylinux2010_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:97f7cb9921fbec4904f522d972f0c0e1f4fabbdd4e0287813b21215074a0f160"},
    {file = "psutil-6.1.1-cp36-abi3-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:33431e84fee02bc84ea36d9e2c4a6d395d479c9dd9bba2376c1f6ee8f3a4e0b3"},
    {file = "psutil-6.1.1-cp36-cp36m-win32.whl", hash = "sha256:384636b1a64b47814437d1173be1427a7c83681b17a450bfc309a1953e329603"},
    {file = "psutil-6.1.1-cp36-cp36m-win_amd64.whl", hash = "sha256:8be07491f6ebe1a693f17d4f11e69d0dc1811fa082736500f649f79df7735303"},
    {file = "psutil-6.1.1-cp37-abi3-win32.whl", hash = "sha256:eaa912e0b11848c4d9279a93d7e2783df352b082f40111e078388701fd479e53"},
    {file = "psutil-6.1.1-cp37-abi3-win_amd64.whl", hash = "sha256:f35cfccb065fff93529d2afb4a2e89e363fe63ca1e4a5da22b603a85833c2649"},
    {file = "psutil-6.1.1.tar.gz", hash = "sha256:cf8496728c18f2d0b45198f06895be52f36611711746b7f30c464b422b50e2f5"},
]

[package.extras]
dev = ["abi3audit", "black", "check-manifest", "coverage", "packaging", "pylint", "pyperf", "pypinfo", "pytest-cov", "requests", "rstcheck", "ruff", "sphinx", "sphinx_rtd_theme", "toml-sort", "twine", "virtualenv", "vulture", "wheel"]
test = ["pytest", "pytest-xdist", "setuptools"]

[[package]]
name = "ptyprocess"
version = "0.7.0"
description = "Run a subprocess in a pseudo terminal"
optional = false
python-versions = "*"
files = [
    {file = "ptyprocess-0.7.0-py2.py3-none-any.whl", hash = "sha256:4b41f3967fce3af57cc7e94b888626c18bf37a083e3651ca8feeb66d492fef35"},
    {file = "ptyprocess-0.7.0.tar.gz", hash = "sha256:5c5d0a3b48ceee0b48485e0c26037c0acd7d29765ca3fbb5cb3831d347423220"},
]

[[package]]
name = "pure-eval"
version = "0.2.3"
description = "Safely evaluate AST nodes without side effects"
optional = false
python-versions = "*"
files = [
    {file = "pure_eval-0.2.3-py3-none-any.whl", hash = "sha256:1db8e35b67b3d218d818ae653e27f06c3aa420901fa7b081ca98cbedc874e0d0"},
    {file = "pure_eval-0.2.3.tar.gz", hash = "sha256:5f4e983f40564c576c7c8635ae88db5956bb2229d7e9237d03b3c0b0190eaf42"},
]

[package.extras]
tests = ["pytest"]

[[package]]
name = "py-cpuinfo"
version = "9.0.0"
description = "Get CPU info with pure Python"
optional = false
python-versions = "*"
files = [
    {file = "py-cpuinfo-9.0.0.tar.gz", hash = "sha256:3cdbbf3fac90dc6f118bfd64384f309edeadd902d7c8fb17f02ffa1fc3f49690"},
    {file = "py_cpuinfo-9.0.0-py3-none-any.whl", hash = "sha256:859625bc251f64e21f077d099d4162689c762b5d6a4c3c97553d56241c9674d5"},
]

[[package]]
name = "pycodestyle"
version = "2.12.1"
description = "Python style guide checker"
optional = false
python-versions = ">=3.8"
files = [
    {file = "pycodestyle-2.12.1-py2.py3-none-any.whl", hash = "sha256:46f0fb92069a7c28ab7bb558f05bfc0110dac69a0cd23c61ea0040283a9d78b3"},
    {file = "pycodestyle-2.12.1.tar.gz", hash = "sha256:6838eae08bbce4f6accd5d5572075c63626a15ee3e6f842df996bf62f6d73521"},
]

[[package]]
name = "pycparser"
version = "2.22"
description = "C parser in Python"
optional = false
python-versions = ">=3.8"
files = [
    {file = "pycparser-2.22-py3-none-any.whl", hash = "sha256:c3702b6d3dd8c7abc1afa565d7e63d53a1d0bd86cdc24edd75470f4de499cfcc"},
    {file = "pycparser-2.22.tar.gz", hash = "sha256:491c8be9c040f5390f5bf44a5b07752bd07f56edf992381b05c701439eec10f6"},
]

[[package]]
name = "pydantic"
version = "2.10.4"
description = "Data validation using Python type hints"
optional = false
python-versions = ">=3.8"
files = [
    {file = "pydantic-2.10.4-py3-none-any.whl", hash = "sha256:597e135ea68be3a37552fb524bc7d0d66dcf93d395acd93a00682f1efcb8ee3d"},
    {file = "pydantic-2.10.4.tar.gz", hash = "sha256:82f12e9723da6de4fe2ba888b5971157b3be7ad914267dea8f05f82b28254f06"},
]

[package.dependencies]
annotated-types = ">=0.6.0"
pydantic-core = "2.27.2"
typing-extensions = ">=4.12.2"

[package.extras]
email = ["email-validator (>=2.0.0)"]
timezone = ["tzdata"]

[[package]]
name = "pydantic-core"
version = "2.27.2"
description = "Core functionality for Pydantic validation and serialization"
optional = false
python-versions = ">=3.8"
files = [
    {file = "pydantic_core-2.27.2-cp310-cp310-macosx_10_12_x86_64.whl", hash = "sha256:2d367ca20b2f14095a8f4fa1210f5a7b78b8a20009ecced6b12818f455b1e9fa"},
    {file = "pydantic_core-2.27.2-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:491a2b73db93fab69731eaee494f320faa4e093dbed776be1a829c2eb222c34c"},
    {file = "pydantic_core-2.27.2-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:7969e133a6f183be60e9f6f56bfae753585680f3b7307a8e555a948d443cc05a"},
    {file = "pydantic_core-2.27.2-cp310-cp310-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:3de9961f2a346257caf0aa508a4da705467f53778e9ef6fe744c038119737ef5"},
    {file = "pydantic_core-2.27.2-cp310-cp310-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:e2bb4d3e5873c37bb3dd58714d4cd0b0e6238cebc4177ac8fe878f8b3aa8e74c"},
    {file = "pydantic_core-2.27.2-cp310-cp310-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:280d219beebb0752699480fe8f1dc61ab6615c2046d76b7ab7ee38858de0a4e7"},
    {file = "pydantic_core-2.27.2-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:47956ae78b6422cbd46f772f1746799cbb862de838fd8d1fbd34a82e05b0983a"},
    {file = "pydantic_core-2.27.2-cp310-cp310-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:14d4a5c49d2f009d62a2a7140d3064f686d17a5d1a268bc641954ba181880236"},
    {file = "pydantic_core-2.27.2-cp310-cp310-musllinux_1_1_aarch64.whl", hash = "sha256:337b443af21d488716f8d0b6164de833e788aa6bd7e3a39c005febc1284f4962"},
    {file = "pydantic_core-2.27.2-cp310-cp310-musllinux_1_1_armv7l.whl", hash = "sha256:03d0f86ea3184a12f41a2d23f7ccb79cdb5a18e06993f8a45baa8dfec746f0e9"},
    {file = "pydantic_core-2.27.2-cp310-cp310-musllinux_1_1_x86_64.whl", hash = "sha256:7041c36f5680c6e0f08d922aed302e98b3745d97fe1589db0a3eebf6624523af"},
    {file = "pydantic_core-2.27.2-cp310-cp310-win32.whl", hash = "sha256:50a68f3e3819077be2c98110c1f9dcb3817e93f267ba80a2c05bb4f8799e2ff4"},
    {file = "pydantic_core-2.27.2-cp310-cp310-win_amd64.whl", hash = "sha256:e0fd26b16394ead34a424eecf8a31a1f5137094cabe84a1bcb10fa6ba39d3d31"},
    {file = "pydantic_core-2.27.2-cp311-cp311-macosx_10_12_x86_64.whl", hash = "sha256:8e10c99ef58cfdf2a66fc15d66b16c4a04f62bca39db589ae8cba08bc55331bc"},
    {file = "pydantic_core-2.27.2-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:26f32e0adf166a84d0cb63be85c562ca8a6fa8de28e5f0d92250c6b7e9e2aff7"},
    {file = "pydantic_core-2.27.2-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:8c19d1ea0673cd13cc2f872f6c9ab42acc4e4f492a7ca9d3795ce2b112dd7e15"},
    {file = "pydantic_core-2.27.2-cp311-cp311-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:5e68c4446fe0810e959cdff46ab0a41ce2f2c86d227d96dc3847af0ba7def306"},
    {file = "pydantic_core-2.27.2-cp311-cp311-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:d9640b0059ff4f14d1f37321b94061c6db164fbe49b334b31643e0528d100d99"},
    {file = "pydantic_core-2.27.2-cp311-cp311-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:40d02e7d45c9f8af700f3452f329ead92da4c5f4317ca9b896de7ce7199ea459"},
    {file = "pydantic_core-2.27.2-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:1c1fd185014191700554795c99b347d64f2bb637966c4cfc16998a0ca700d048"},
    {file = "pydantic_core-2.27.2-cp311-cp311-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:d81d2068e1c1228a565af076598f9e7451712700b673de8f502f0334f281387d"},
    {file = "pydantic_core-2.27.2-cp311-cp311-musllinux_1_1_aarch64.whl", hash = "sha256:1a4207639fb02ec2dbb76227d7c751a20b1a6b4bc52850568e52260cae64ca3b"},
    {file = "pydantic_core-2.27.2-cp311-cp311-musllinux_1_1_armv7l.whl", hash = "sha256:3de3ce3c9ddc8bbd88f6e0e304dea0e66d843ec9de1b0042b0911c1663ffd474"},
    {file = "pydantic_core-2.27.2-cp311-cp311-musllinux_1_1_x86_64.whl", hash = "sha256:30c5f68ded0c36466acede341551106821043e9afaad516adfb6e8fa80a4e6a6"},
    {file = "pydantic_core-2.27.2-cp311-cp311-win32.whl", hash = "sha256:c70c26d2c99f78b125a3459f8afe1aed4d9687c24fd677c6a4436bc042e50d6c"},
    {file = "pydantic_core-2.27.2-cp311-cp311-win_amd64.whl", hash = "sha256:08e125dbdc505fa69ca7d9c499639ab6407cfa909214d500897d02afb816e7cc"},
    {file = "pydantic_core-2.27.2-cp311-cp311-win_arm64.whl", hash = "sha256:26f0d68d4b235a2bae0c3fc585c585b4ecc51382db0e3ba402a22cbc440915e4"},
    {file = "pydantic_core-2.27.2-cp312-cp312-macosx_10_12_x86_64.whl", hash = "sha256:9e0c8cfefa0ef83b4da9588448b6d8d2a2bf1a53c3f1ae5fca39eb3061e2f0b0"},
    {file = "pydantic_core-2.27.2-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:83097677b8e3bd7eaa6775720ec8e0405f1575015a463285a92bfdfe254529ef"},
    {file = "pydantic_core-2.27.2-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:172fce187655fece0c90d90a678424b013f8fbb0ca8b036ac266749c09438cb7"},
    {file = "pydantic_core-2.27.2-cp312-cp312-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:519f29f5213271eeeeb3093f662ba2fd512b91c5f188f3bb7b27bc5973816934"},
    {file = "pydantic_core-2.27.2-cp312-cp312-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:05e3a55d124407fffba0dd6b0c0cd056d10e983ceb4e5dbd10dda135c31071d6"},
    {file = "pydantic_core-2.27.2-cp312-cp312-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:9c3ed807c7b91de05e63930188f19e921d1fe90de6b4f5cd43ee7fcc3525cb8c"},
    {file = "pydantic_core-2.27.2-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:6fb4aadc0b9a0c063206846d603b92030eb6f03069151a625667f982887153e2"},
    {file = "pydantic_core-2.27.2-cp312-cp312-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:28ccb213807e037460326424ceb8b5245acb88f32f3d2777427476e1b32c48c4"},
    {file = "pydantic_core-2.27.2-cp312-cp312-musllinux_1_1_aarch64.whl", hash = "sha256:de3cd1899e2c279b140adde9357c4495ed9d47131b4a4eaff9052f23398076b3"},
    {file = "pydantic_core-2.27.2-cp312-cp312-musllinux_1_1_armv7l.whl", hash = "sha256:220f892729375e2d736b97d0e51466252ad84c51857d4d15f5e9692f9ef12be4"},
    {file = "pydantic_core-2.27.2-cp312-cp312-musllinux_1_1_x86_64.whl", hash = "sha256:a0fcd29cd6b4e74fe8ddd2c90330fd8edf2e30cb52acda47f06dd615ae72da57"},
    {file = "pydantic_core-2.27.2-cp312-cp312-win32.whl", hash = "sha256:1e2cb691ed9834cd6a8be61228471d0a503731abfb42f82458ff27be7b2186fc"},
    {file = "pydantic_core-2.27.2-cp312-cp312-win_amd64.whl", hash = "sha256:cc3f1a99a4f4f9dd1de4fe0312c114e740b5ddead65bb4102884b384c15d8bc9"},
    {file = "pydantic_core-2.27.2-cp312-cp312-win_arm64.whl", hash = "sha256:3911ac9284cd8a1792d3cb26a2da18f3ca26c6908cc434a18f730dc0db7bfa3b"},
    {file = "pydantic_core-2.27.2-cp313-cp313-macosx_10_12_x86_64.whl", hash = "sha256:7d14bd329640e63852364c306f4d23eb744e0f8193148d4044dd3dacdaacbd8b"},
    {file = "pydantic_core-2.27.2-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:82f91663004eb8ed30ff478d77c4d1179b3563df6cdb15c0817cd1cdaf34d154"},
    {file = "pydantic_core-2.27.2-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:71b24c7d61131bb83df10cc7e687433609963a944ccf45190cfc21e0887b08c9"},
    {file = "pydantic_core-2.27.2-cp313-cp313-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:fa8e459d4954f608fa26116118bb67f56b93b209c39b008277ace29937453dc9"},
    {file = "pydantic_core-2.27.2-cp313-cp313-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:ce8918cbebc8da707ba805b7fd0b382816858728ae7fe19a942080c24e5b7cd1"},
    {file = "pydantic_core-2.27.2-cp313-cp313-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:eda3f5c2a021bbc5d976107bb302e0131351c2ba54343f8a496dc8783d3d3a6a"},
    {file = "pydantic_core-2.27.2-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:bd8086fa684c4775c27f03f062cbb9eaa6e17f064307e86b21b9e0abc9c0f02e"},
    {file = "pydantic_core-2.27.2-cp313-cp313-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:8d9b3388db186ba0c099a6d20f0604a44eabdeef1777ddd94786cdae158729e4"},
    {file = "pydantic_core-2.27.2-cp313-cp313-musllinux_1_1_aarch64.whl", hash = "sha256:7a66efda2387de898c8f38c0cf7f14fca0b51a8ef0b24bfea5849f1b3c95af27"},
    {file = "pydantic_core-2.27.2-cp313-cp313-musllinux_1_1_armv7l.whl", hash = "sha256:18a101c168e4e092ab40dbc2503bdc0f62010e95d292b27827871dc85450d7ee"},
    {file = "pydantic_core-2.27.2-cp313-cp313-musllinux_1_1_x86_64.whl", hash = "sha256:ba5dd002f88b78a4215ed2f8ddbdf85e8513382820ba15ad5ad8955ce0ca19a1"},
    {file = "pydantic_core-2.27.2-cp313-cp313-win32.whl", hash = "sha256:1ebaf1d0481914d004a573394f4be3a7616334be70261007e47c2a6fe7e50130"},
    {file = "pydantic_core-2.27.2-cp313-cp313-win_amd64.whl", hash = "sha256:953101387ecf2f5652883208769a79e48db18c6df442568a0b5ccd8c2723abee"},
    {file = "pydantic_core-2.27.2-cp313-cp313-win_arm64.whl", hash = "sha256:ac4dbfd1691affb8f48c2c13241a2e3b60ff23247cbcf981759c768b6633cf8b"},
    {file = "pydantic_core-2.27.2-cp38-cp38-macosx_10_12_x86_64.whl", hash = "sha256:d3e8d504bdd3f10835468f29008d72fc8359d95c9c415ce6e767203db6127506"},
    {file = "pydantic_core-2.27.2-cp38-cp38-macosx_11_0_arm64.whl", hash = "sha256:521eb9b7f036c9b6187f0b47318ab0d7ca14bd87f776240b90b21c1f4f149320"},
    {file = "pydantic_core-2.27.2-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:85210c4d99a0114f5a9481b44560d7d1e35e32cc5634c656bc48e590b669b145"},
    {file = "pydantic_core-2.27.2-cp38-cp38-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:d716e2e30c6f140d7560ef1538953a5cd1a87264c737643d481f2779fc247fe1"},
    {file = "pydantic_core-2.27.2-cp38-cp38-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:f66d89ba397d92f840f8654756196d93804278457b5fbede59598a1f9f90b228"},
    {file = "pydantic_core-2.27.2-cp38-cp38-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:669e193c1c576a58f132e3158f9dfa9662969edb1a250c54d8fa52590045f046"},
    {file = "pydantic_core-2.27.2-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:9fdbe7629b996647b99c01b37f11170a57ae675375b14b8c13b8518b8320ced5"},
    {file = "pydantic_core-2.27.2-cp38-cp38-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:d262606bf386a5ba0b0af3b97f37c83d7011439e3dc1a9298f21efb292e42f1a"},
    {file = "pydantic_core-2.27.2-cp38-cp38-musllinux_1_1_aarch64.whl", hash = "sha256:cabb9bcb7e0d97f74df8646f34fc76fbf793b7f6dc2438517d7a9e50eee4f14d"},
    {file = "pydantic_core-2.27.2-cp38-cp38-musllinux_1_1_armv7l.whl", hash = "sha256:d2d63f1215638d28221f664596b1ccb3944f6e25dd18cd3b86b0a4c408d5ebb9"},
    {file = "pydantic_core-2.27.2-cp38-cp38-musllinux_1_1_x86_64.whl", hash = "sha256:bca101c00bff0adb45a833f8451b9105d9df18accb8743b08107d7ada14bd7da"},
    {file = "pydantic_core-2.27.2-cp38-cp38-win32.whl", hash = "sha256:f6f8e111843bbb0dee4cb6594cdc73e79b3329b526037ec242a3e49012495b3b"},
    {file = "pydantic_core-2.27.2-cp38-cp38-win_amd64.whl", hash = "sha256:fd1aea04935a508f62e0d0ef1f5ae968774a32afc306fb8545e06f5ff5cdf3ad"},
    {file = "pydantic_core-2.27.2-cp39-cp39-macosx_10_12_x86_64.whl", hash = "sha256:c10eb4f1659290b523af58fa7cffb452a61ad6ae5613404519aee4bfbf1df993"},
    {file = "pydantic_core-2.27.2-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:ef592d4bad47296fb11f96cd7dc898b92e795032b4894dfb4076cfccd43a9308"},
    {file = "pydantic_core-2.27.2-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:c61709a844acc6bf0b7dce7daae75195a10aac96a596ea1b776996414791ede4"},
    {file = "pydantic_core-2.27.2-cp39-cp39-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:42c5f762659e47fdb7b16956c71598292f60a03aa92f8b6351504359dbdba6cf"},
    {file = "pydantic_core-2.27.2-cp39-cp39-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:4c9775e339e42e79ec99c441d9730fccf07414af63eac2f0e48e08fd38a64d76"},
    {file = "pydantic_core-2.27.2-cp39-cp39-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:57762139821c31847cfb2df63c12f725788bd9f04bc2fb392790959b8f70f118"},
    {file = "pydantic_core-2.27.2-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:0d1e85068e818c73e048fe28cfc769040bb1f475524f4745a5dc621f75ac7630"},
    {file = "pydantic_core-2.27.2-cp39-cp39-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:097830ed52fd9e427942ff3b9bc17fab52913b2f50f2880dc4a5611446606a54"},
    {file = "pydantic_core-2.27.2-cp39-cp39-musllinux_1_1_aarch64.whl", hash = "sha256:044a50963a614ecfae59bb1eaf7ea7efc4bc62f49ed594e18fa1e5d953c40e9f"},
    {file = "pydantic_core-2.27.2-cp39-cp39-musllinux_1_1_armv7l.whl", hash = "sha256:4e0b4220ba5b40d727c7f879eac379b822eee5d8fff418e9d3381ee45b3b0362"},
    {file = "pydantic_core-2.27.2-cp39-cp39-musllinux_1_1_x86_64.whl", hash = "sha256:5e4f4bb20d75e9325cc9696c6802657b58bc1dbbe3022f32cc2b2b632c3fbb96"},
    {file = "pydantic_core-2.27.2-cp39-cp39-win32.whl", hash = "sha256:cca63613e90d001b9f2f9a9ceb276c308bfa2a43fafb75c8031c4f66039e8c6e"},
    {file = "pydantic_core-2.27.2-cp39-cp39-win_amd64.whl", hash = "sha256:77d1bca19b0f7021b3a982e6f903dcd5b2b06076def36a652e3907f596e29f67"},
    {file = "pydantic_core-2.27.2-pp310-pypy310_pp73-macosx_10_12_x86_64.whl", hash = "sha256:2bf14caea37e91198329b828eae1618c068dfb8ef17bb33287a7ad4b61ac314e"},
    {file = "pydantic_core-2.27.2-pp310-pypy310_pp73-macosx_11_0_arm64.whl", hash = "sha256:b0cb791f5b45307caae8810c2023a184c74605ec3bcbb67d13846c28ff731ff8"},
    {file = "pydantic_core-2.27.2-pp310-pypy310_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:688d3fd9fcb71f41c4c015c023d12a79d1c4c0732ec9eb35d96e3388a120dcf3"},
    {file = "pydantic_core-2.27.2-pp310-pypy310_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:3d591580c34f4d731592f0e9fe40f9cc1b430d297eecc70b962e93c5c668f15f"},
    {file = "pydantic_core-2.27.2-pp310-pypy310_pp73-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:82f986faf4e644ffc189a7f1aafc86e46ef70372bb153e7001e8afccc6e54133"},
    {file = "pydantic_core-2.27.2-pp310-pypy310_pp73-musllinux_1_1_aarch64.whl", hash = "sha256:bec317a27290e2537f922639cafd54990551725fc844249e64c523301d0822fc"},
    {file = "pydantic_core-2.27.2-pp310-pypy310_pp73-musllinux_1_1_armv7l.whl", hash = "sha256:0296abcb83a797db256b773f45773da397da75a08f5fcaef41f2044adec05f50"},
    {file = "pydantic_core-2.27.2-pp310-pypy310_pp73-musllinux_1_1_x86_64.whl", hash = "sha256:0d75070718e369e452075a6017fbf187f788e17ed67a3abd47fa934d001863d9"},
    {file = "pydantic_core-2.27.2-pp310-pypy310_pp73-win_amd64.whl", hash = "sha256:7e17b560be3c98a8e3aa66ce828bdebb9e9ac6ad5466fba92eb74c4c95cb1151"},
    {file = "pydantic_core-2.27.2-pp39-pypy39_pp73-macosx_10_12_x86_64.whl", hash = "sha256:c33939a82924da9ed65dab5a65d427205a73181d8098e79b6b426bdf8ad4e656"},
    {file = "pydantic_core-2.27.2-pp39-pypy39_pp73-macosx_11_0_arm64.whl", hash = "sha256:00bad2484fa6bda1e216e7345a798bd37c68fb2d97558edd584942aa41b7d278"},
    {file = "pydantic_core-2.27.2-pp39-pypy39_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:c817e2b40aba42bac6f457498dacabc568c3b7a986fc9ba7c8d9d260b71485fb"},
    {file = "pydantic_core-2.27.2-pp39-pypy39_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:251136cdad0cb722e93732cb45ca5299fb56e1344a833640bf93b2803f8d1bfd"},
    {file = "pydantic_core-2.27.2-pp39-pypy39_pp73-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:d2088237af596f0a524d3afc39ab3b036e8adb054ee57cbb1dcf8e09da5b29cc"},
    {file = "pydantic_core-2.27.2-pp39-pypy39_pp73-musllinux_1_1_aarch64.whl", hash = "sha256:d4041c0b966a84b4ae7a09832eb691a35aec90910cd2dbe7a208de59be77965b"},
    {file = "pydantic_core-2.27.2-pp39-pypy39_pp73-musllinux_1_1_armv7l.whl", hash = "sha256:8083d4e875ebe0b864ffef72a4304827015cff328a1be6e22cc850753bfb122b"},
    {file = "pydantic_core-2.27.2-pp39-pypy39_pp73-musllinux_1_1_x86_64.whl", hash = "sha256:f141ee28a0ad2123b6611b6ceff018039df17f32ada8b534e6aa039545a3efb2"},
    {file = "pydantic_core-2.27.2-pp39-pypy39_pp73-win_amd64.whl", hash = "sha256:7d0c8399fcc1848491f00e0314bd59fb34a9c008761bcb422a057670c3f65e35"},
    {file = "pydantic_core-2.27.2.tar.gz", hash = "sha256:eb026e5a4c1fee05726072337ff51d1efb6f59090b7da90d30ea58625b1ffb39"},
]

[package.dependencies]
typing-extensions = ">=4.6.0,<4.7.0 || >4.7.0"

[[package]]
name = "pydantic-extra-types"
version = "2.10.1"
description = "Extra Pydantic types."
optional = false
python-versions = ">=3.8"
files = [
    {file = "pydantic_extra_types-2.10.1-py3-none-any.whl", hash = "sha256:db2c86c04a837bbac0d2d79bbd6f5d46c4c9253db11ca3fdd36a2b282575f1e2"},
    {file = "pydantic_extra_types-2.10.1.tar.gz", hash = "sha256:e4f937af34a754b8f1fa228a2fac867091a51f56ed0e8a61d5b3a6719b13c923"},
]

[package.dependencies]
pydantic = ">=2.5.2"
typing-extensions = "*"

[package.extras]
all = ["pendulum (>=3.0.0,<4.0.0)", "phonenumbers (>=8,<9)", "pycountry (>=23)", "python-ulid (>=1,<2)", "python-ulid (>=1,<4)", "pytz (>=2024.1)", "semver (>=3.0.2)", "semver (>=3.0.2,<3.1.0)", "tzdata (>=2024.1)"]
pendulum = ["pendulum (>=3.0.0,<4.0.0)"]
phonenumbers = ["phonenumbers (>=8,<9)"]
pycountry = ["pycountry (>=23)"]
python-ulid = ["python-ulid (>=1,<2)", "python-ulid (>=1,<4)"]
semver = ["semver (>=3.0.2)"]

[[package]]
name = "pydantic-settings"
version = "2.7.0"
description = "Settings management using Pydantic"
optional = false
python-versions = ">=3.8"
files = [
    {file = "pydantic_settings-2.7.0-py3-none-any.whl", hash = "sha256:e00c05d5fa6cbbb227c84bd7487c5c1065084119b750df7c8c1a554aed236eb5"},
    {file = "pydantic_settings-2.7.0.tar.gz", hash = "sha256:ac4bfd4a36831a48dbf8b2d9325425b549a0a6f18cea118436d728eb4f1c4d66"},
]

[package.dependencies]
pydantic = ">=2.7.0"
python-dotenv = ">=0.21.0"

[package.extras]
azure-key-vault = ["azure-identity (>=1.16.0)", "azure-keyvault-secrets (>=4.8.0)"]
toml = ["tomli (>=2.0.1)"]
yaml = ["pyyaml (>=6.0.1)"]

[[package]]
name = "pyflakes"
version = "3.2.0"
description = "passive checker of Python programs"
optional = false
python-versions = ">=3.8"
files = [
    {file = "pyflakes-3.2.0-py2.py3-none-any.whl", hash = "sha256:84b5be138a2dfbb40689ca07e2152deb896a65c3a3e24c251c5c62489568074a"},
    {file = "pyflakes-3.2.0.tar.gz", hash = "sha256:1c61603ff154621fb2a9172037d84dca3500def8c8b630657d1701f026f8af3f"},
]

[[package]]
name = "pygments"
version = "2.18.0"
description = "Pygments is a syntax highlighting package written in Python."
optional = false
python-versions = ">=3.8"
files = [
    {file = "pygments-2.18.0-py3-none-any.whl", hash = "sha256:b8e6aca0523f3ab76fee51799c488e38782ac06eafcf95e7ba832985c8e7b13a"},
    {file = "pygments-2.18.0.tar.gz", hash = "sha256:786ff802f32e91311bff3889f6e9a86e81505fe99f2735bb6d60ae0c5004f199"},
]

[package.extras]
windows-terminal = ["colorama (>=0.4.6)"]

[[package]]
name = "pylint"
version = "3.3.3"
description = "python code static checker"
optional = false
python-versions = ">=3.9.0"
files = [
    {file = "pylint-3.3.3-py3-none-any.whl", hash = "sha256:26e271a2bc8bce0fc23833805a9076dd9b4d5194e2a02164942cb3cdc37b4183"},
    {file = "pylint-3.3.3.tar.gz", hash = "sha256:07c607523b17e6d16e2ae0d7ef59602e332caa762af64203c24b41c27139f36a"},
]

[package.dependencies]
astroid = ">=3.3.8,<=3.4.0-dev0"
colorama = {version = ">=0.4.5", markers = "sys_platform == \"win32\""}
dill = {version = ">=0.3.7", markers = "python_version >= \"3.12\""}
isort = ">=4.2.5,<5.13.0 || >5.13.0,<6"
mccabe = ">=0.6,<0.8"
platformdirs = ">=2.2.0"
tomlkit = ">=0.10.1"

[package.extras]
spelling = ["pyenchant (>=3.2,<4.0)"]
testutils = ["gitpython (>3)"]

[[package]]
name = "pyparsing"
version = "3.2.0"
description = "pyparsing module - Classes and methods to define and execute parsing grammars"
optional = false
python-versions = ">=3.9"
files = [
    {file = "pyparsing-3.2.0-py3-none-any.whl", hash = "sha256:93d9577b88da0bbea8cc8334ee8b918ed014968fd2ec383e868fb8afb1ccef84"},
    {file = "pyparsing-3.2.0.tar.gz", hash = "sha256:cbf74e27246d595d9a74b186b810f6fbb86726dbf3b9532efb343f6d7294fe9c"},
]

[package.extras]
diagrams = ["jinja2", "railroad-diagrams"]

[[package]]
name = "pytest"
version = "8.3.4"
description = "pytest: simple powerful testing with Python"
optional = false
python-versions = ">=3.8"
files = [
    {file = "pytest-8.3.4-py3-none-any.whl", hash = "sha256:50e16d954148559c9a74109af1eaf0c945ba2d8f30f0a3d3335edde19788b6f6"},
    {file = "pytest-8.3.4.tar.gz", hash = "sha256:965370d062bce11e73868e0335abac31b4d3de0e82f4007408d242b4f8610761"},
]

[package.dependencies]
colorama = {version = "*", markers = "sys_platform == \"win32\""}
iniconfig = "*"
packaging = "*"
pluggy = ">=1.5,<2"

[package.extras]
dev = ["argcomplete", "attrs (>=19.2)", "hypothesis (>=3.56)", "mock", "pygments (>=2.7.2)", "requests", "setuptools", "xmlschema"]

[[package]]
name = "pytest-benchmark"
version = "4.0.0"
description = "A ``pytest`` fixture for benchmarking code. It will group the tests into rounds that are calibrated to the chosen timer."
optional = false
python-versions = ">=3.7"
files = [
    {file = "pytest-benchmark-4.0.0.tar.gz", hash = "sha256:fb0785b83efe599a6a956361c0691ae1dbb5318018561af10f3e915caa0048d1"},
    {file = "pytest_benchmark-4.0.0-py3-none-any.whl", hash = "sha256:fdb7db64e31c8b277dff9850d2a2556d8b60bcb0ea6524e36e28ffd7c87f71d6"},
]

[package.dependencies]
py-cpuinfo = "*"
pytest = ">=3.8"

[package.extras]
aspect = ["aspectlib"]
elasticsearch = ["elasticsearch"]
histogram = ["pygal", "pygaljs"]

[[package]]
name = "pytest-cov"
version = "5.0.0"
description = "Pytest plugin for measuring coverage."
optional = false
python-versions = ">=3.8"
files = [
    {file = "pytest-cov-5.0.0.tar.gz", hash = "sha256:5837b58e9f6ebd335b0f8060eecce69b662415b16dc503883a02f45dfeb14857"},
    {file = "pytest_cov-5.0.0-py3-none-any.whl", hash = "sha256:4f0764a1219df53214206bf1feea4633c3b558a2925c8b59f144f682861ce652"},
]

[package.dependencies]
coverage = {version = ">=5.2.1", extras = ["toml"]}
pytest = ">=4.6"

[package.extras]
testing = ["fields", "hunter", "process-tests", "pytest-xdist", "virtualenv"]

[[package]]
name = "pytest-emoji"
version = "0.2.0"
description = "A pytest plugin that adds emojis to your test result report"
optional = false
python-versions = ">=3.4"
files = [
    {file = "pytest-emoji-0.2.0.tar.gz", hash = "sha256:e1bd4790d87649c2d09c272c88bdfc4d37c1cc7c7a46583087d7c510944571e8"},
    {file = "pytest_emoji-0.2.0-py3-none-any.whl", hash = "sha256:6e34ed21970fa4b80a56ad11417456bd873eb066c02315fe9df0fafe6d4d4436"},
]

[package.dependencies]
pytest = ">=4.2.1"

[[package]]
name = "pytest-html"
version = "4.1.1"
description = "pytest plugin for generating HTML reports"
optional = false
python-versions = ">=3.8"
files = [
    {file = "pytest_html-4.1.1-py3-none-any.whl", hash = "sha256:c8152cea03bd4e9bee6d525573b67bbc6622967b72b9628dda0ea3e2a0b5dd71"},
    {file = "pytest_html-4.1.1.tar.gz", hash = "sha256:70a01e8ae5800f4a074b56a4cb1025c8f4f9b038bba5fe31e3c98eb996686f07"},
]

[package.dependencies]
jinja2 = ">=3.0.0"
pytest = ">=7.0.0"
pytest-metadata = ">=2.0.0"

[package.extras]
docs = ["pip-tools (>=6.13.0)"]
test = ["assertpy (>=1.1)", "beautifulsoup4 (>=4.11.1)", "black (>=22.1.0)", "flake8 (>=4.0.1)", "pre-commit (>=2.17.0)", "pytest-mock (>=3.7.0)", "pytest-rerunfailures (>=11.1.2)", "pytest-xdist (>=2.4.0)", "selenium (>=4.3.0)", "tox (>=3.24.5)"]

[[package]]
name = "pytest-icdiff"
version = "0.9"
description = "use icdiff for better error messages in pytest assertions"
optional = false
python-versions = ">=3.7"
files = [
    {file = "pytest-icdiff-0.9.tar.gz", hash = "sha256:13aede616202e57fcc882568b64589002ef85438046f012ac30a8d959dac8b75"},
    {file = "pytest_icdiff-0.9-py3-none-any.whl", hash = "sha256:efee0da3bd1b24ef2d923751c5c547fbb8df0a46795553fba08ef57c3ca03d82"},
]

[package.dependencies]
icdiff = "*"
pprintpp = "*"
pytest = "*"

[[package]]
name = "pytest-instafail"
version = "0.5.0"
description = "pytest plugin to show failures instantly"
optional = false
python-versions = ">=3.7"
files = [
    {file = "pytest-instafail-0.5.0.tar.gz", hash = "sha256:33a606f7e0c8e646dc3bfee0d5e3a4b7b78ef7c36168cfa1f3d93af7ca706c9e"},
    {file = "pytest_instafail-0.5.0-py3-none-any.whl", hash = "sha256:6855414487e9e4bb76a118ce952c3c27d3866af15487506c4ded92eb72387819"},
]

[package.dependencies]
pytest = ">=5"

[[package]]
name = "pytest-metadata"
version = "3.1.1"
description = "pytest plugin for test session metadata"
optional = false
python-versions = ">=3.8"
files = [
    {file = "pytest_metadata-3.1.1-py3-none-any.whl", hash = "sha256:c8e0844db684ee1c798cfa38908d20d67d0463ecb6137c72e91f418558dd5f4b"},
    {file = "pytest_metadata-3.1.1.tar.gz", hash = "sha256:d2a29b0355fbc03f168aa96d41ff88b1a3b44a3b02acbe491801c98a048017c8"},
]

[package.dependencies]
pytest = ">=7.0.0"

[package.extras]
test = ["black (>=22.1.0)", "flake8 (>=4.0.1)", "pre-commit (>=2.17.0)", "tox (>=3.24.5)"]

[[package]]
name = "pytest-mock"
version = "3.14.0"
description = "Thin-wrapper around the mock package for easier use with pytest"
optional = false
python-versions = ">=3.8"
files = [
    {file = "pytest-mock-3.14.0.tar.gz", hash = "sha256:2719255a1efeceadbc056d6bf3df3d1c5015530fb40cf347c0f9afac88410bd0"},
    {file = "pytest_mock-3.14.0-py3-none-any.whl", hash = "sha256:0b72c38033392a5f4621342fe11e9219ac11ec9d375f8e2a0c164539e0d70f6f"},
]

[package.dependencies]
pytest = ">=6.2.5"

[package.extras]
dev = ["pre-commit", "pytest-asyncio", "tox"]

[[package]]
name = "pytest-sugar"
version = "1.0.0"
description = "pytest-sugar is a plugin for pytest that changes the default look and feel of pytest (e.g. progressbar, show tests that fail instantly)."
optional = false
python-versions = "*"
files = [
    {file = "pytest-sugar-1.0.0.tar.gz", hash = "sha256:6422e83258f5b0c04ce7c632176c7732cab5fdb909cb39cca5c9139f81276c0a"},
    {file = "pytest_sugar-1.0.0-py3-none-any.whl", hash = "sha256:70ebcd8fc5795dc457ff8b69d266a4e2e8a74ae0c3edc749381c64b5246c8dfd"},
]

[package.dependencies]
packaging = ">=21.3"
pytest = ">=6.2.0"
termcolor = ">=2.1.0"

[package.extras]
dev = ["black", "flake8", "pre-commit"]

[[package]]
name = "pytest-timeout"
version = "2.3.1"
description = "pytest plugin to abort hanging tests"
optional = false
python-versions = ">=3.7"
files = [
    {file = "pytest-timeout-2.3.1.tar.gz", hash = "sha256:12397729125c6ecbdaca01035b9e5239d4db97352320af155b3f5de1ba5165d9"},
    {file = "pytest_timeout-2.3.1-py3-none-any.whl", hash = "sha256:68188cb703edfc6a18fad98dc25a3c61e9f24d644b0b70f33af545219fc7813e"},
]

[package.dependencies]
pytest = ">=7.0.0"

[[package]]
name = "pytest-xdist"
version = "3.6.1"
description = "pytest xdist plugin for distributed testing, most importantly across multiple CPUs"
optional = false
python-versions = ">=3.8"
files = [
    {file = "pytest_xdist-3.6.1-py3-none-any.whl", hash = "sha256:9ed4adfb68a016610848639bb7e02c9352d5d9f03d04809919e2dafc3be4cca7"},
    {file = "pytest_xdist-3.6.1.tar.gz", hash = "sha256:ead156a4db231eec769737f57668ef58a2084a34b2e55c4a8fa20d861107300d"},
]

[package.dependencies]
execnet = ">=2.1"
pytest = ">=7.0.0"

[package.extras]
psutil = ["psutil (>=3.0)"]
setproctitle = ["setproctitle"]
testing = ["filelock"]

[[package]]
name = "python-dateutil"
version = "2.9.0.post0"
description = "Extensions to the standard Python datetime module"
optional = false
python-versions = "!=3.0.*,!=3.1.*,!=3.2.*,>=2.7"
files = [
    {file = "python-dateutil-2.9.0.post0.tar.gz", hash = "sha256:37dd54208da7e1cd875388217d5e00ebd4179249f90fb72437e91a35459a0ad3"},
    {file = "python_dateutil-2.9.0.post0-py2.py3-none-any.whl", hash = "sha256:a8b2bc7bffae282281c8140a97d3aa9c14da0b136dfe83f850eea9a5f7470427"},
]

[package.dependencies]
six = ">=1.5"

[[package]]
name = "python-dotenv"
version = "1.0.1"
description = "Read key-value pairs from a .env file and set them as environment variables"
optional = false
python-versions = ">=3.8"
files = [
    {file = "python-dotenv-1.0.1.tar.gz", hash = "sha256:e324ee90a023d808f1959c46bcbc04446a10ced277783dc6ee09987c37ec10ca"},
    {file = "python_dotenv-1.0.1-py3-none-any.whl", hash = "sha256:f7b63ef50f1b690dddf550d03497b66d609393b40b564ed0d674909a68ebf16a"},
]

[package.extras]
cli = ["click (>=5.0)"]

[[package]]
name = "python-json-logger"
version = "3.2.1"
description = "JSON Log Formatter for the Python Logging Package"
optional = false
python-versions = ">=3.8"
files = [
    {file = "python_json_logger-3.2.1-py3-none-any.whl", hash = "sha256:cdc17047eb5374bd311e748b42f99d71223f3b0e186f4206cc5d52aefe85b090"},
    {file = "python_json_logger-3.2.1.tar.gz", hash = "sha256:8eb0554ea17cb75b05d2848bc14fb02fbdbd9d6972120781b974380bfa162008"},
]

[package.extras]
dev = ["backports.zoneinfo", "black", "build", "freezegun", "mdx_truly_sane_lists", "mike", "mkdocs", "mkdocs-awesome-pages-plugin", "mkdocs-gen-files", "mkdocs-literate-nav", "mkdocs-material (>=8.5)", "mkdocstrings[python]", "msgspec", "msgspec-python313-pre", "mypy", "orjson", "pylint", "pytest", "tzdata", "validate-pyproject[all]"]

[[package]]
name = "python-multipart"
version = "0.0.20"
description = "A streaming multipart parser for Python"
optional = false
python-versions = ">=3.8"
files = [
    {file = "python_multipart-0.0.20-py3-none-any.whl", hash = "sha256:8a62d3a8335e06589fe01f2a3e178cdcc632f3fbe0d492ad9ee0ec35aab1f104"},
    {file = "python_multipart-0.0.20.tar.gz", hash = "sha256:8dd0cab45b8e23064ae09147625994d090fa46f5b0d1e13af944c331a7fa9d13"},
]

[[package]]
name = "pytz"
version = "2024.2"
description = "World timezone definitions, modern and historical"
optional = false
python-versions = "*"
files = [
    {file = "pytz-2024.2-py2.py3-none-any.whl", hash = "sha256:31c7c1817eb7fae7ca4b8c7ee50c72f93aa2dd863de768e1ef4245d426aa0725"},
    {file = "pytz-2024.2.tar.gz", hash = "sha256:2aa355083c50a0f93fa581709deac0c9ad65cca8a9e9beac660adcbd493c798a"},
]

[[package]]
name = "pywin32"
version = "308"
description = "Python for Window Extensions"
optional = false
python-versions = "*"
files = [
    {file = "pywin32-308-cp310-cp310-win32.whl", hash = "sha256:796ff4426437896550d2981b9c2ac0ffd75238ad9ea2d3bfa67a1abd546d262e"},
    {file = "pywin32-308-cp310-cp310-win_amd64.whl", hash = "sha256:4fc888c59b3c0bef905ce7eb7e2106a07712015ea1c8234b703a088d46110e8e"},
    {file = "pywin32-308-cp310-cp310-win_arm64.whl", hash = "sha256:a5ab5381813b40f264fa3495b98af850098f814a25a63589a8e9eb12560f450c"},
    {file = "pywin32-308-cp311-cp311-win32.whl", hash = "sha256:5d8c8015b24a7d6855b1550d8e660d8daa09983c80e5daf89a273e5c6fb5095a"},
    {file = "pywin32-308-cp311-cp311-win_amd64.whl", hash = "sha256:575621b90f0dc2695fec346b2d6302faebd4f0f45c05ea29404cefe35d89442b"},
    {file = "pywin32-308-cp311-cp311-win_arm64.whl", hash = "sha256:100a5442b7332070983c4cd03f2e906a5648a5104b8a7f50175f7906efd16bb6"},
    {file = "pywin32-308-cp312-cp312-win32.whl", hash = "sha256:587f3e19696f4bf96fde9d8a57cec74a57021ad5f204c9e627e15c33ff568897"},
    {file = "pywin32-308-cp312-cp312-win_amd64.whl", hash = "sha256:00b3e11ef09ede56c6a43c71f2d31857cf7c54b0ab6e78ac659497abd2834f47"},
    {file = "pywin32-308-cp312-cp312-win_arm64.whl", hash = "sha256:9b4de86c8d909aed15b7011182c8cab38c8850de36e6afb1f0db22b8959e3091"},
    {file = "pywin32-308-cp313-cp313-win32.whl", hash = "sha256:1c44539a37a5b7b21d02ab34e6a4d314e0788f1690d65b48e9b0b89f31abbbed"},
    {file = "pywin32-308-cp313-cp313-win_amd64.whl", hash = "sha256:fd380990e792eaf6827fcb7e187b2b4b1cede0585e3d0c9e84201ec27b9905e4"},
    {file = "pywin32-308-cp313-cp313-win_arm64.whl", hash = "sha256:ef313c46d4c18dfb82a2431e3051ac8f112ccee1a34f29c263c583c568db63cd"},
    {file = "pywin32-308-cp37-cp37m-win32.whl", hash = "sha256:1f696ab352a2ddd63bd07430080dd598e6369152ea13a25ebcdd2f503a38f1ff"},
    {file = "pywin32-308-cp37-cp37m-win_amd64.whl", hash = "sha256:13dcb914ed4347019fbec6697a01a0aec61019c1046c2b905410d197856326a6"},
    {file = "pywin32-308-cp38-cp38-win32.whl", hash = "sha256:5794e764ebcabf4ff08c555b31bd348c9025929371763b2183172ff4708152f0"},
    {file = "pywin32-308-cp38-cp38-win_amd64.whl", hash = "sha256:3b92622e29d651c6b783e368ba7d6722b1634b8e70bd376fd7610fe1992e19de"},
    {file = "pywin32-308-cp39-cp39-win32.whl", hash = "sha256:7873ca4dc60ab3287919881a7d4f88baee4a6e639aa6962de25a98ba6b193341"},
    {file = "pywin32-308-cp39-cp39-win_amd64.whl", hash = "sha256:71b3322d949b4cc20776436a9c9ba0eeedcbc9c650daa536df63f0ff111bb920"},
]

[[package]]
name = "pywinpty"
version = "2.0.14"
description = "Pseudo terminal support for Windows from Python."
optional = false
python-versions = ">=3.8"
files = [
    {file = "pywinpty-2.0.14-cp310-none-win_amd64.whl", hash = "sha256:0b149c2918c7974f575ba79f5a4aad58bd859a52fa9eb1296cc22aa412aa411f"},
    {file = "pywinpty-2.0.14-cp311-none-win_amd64.whl", hash = "sha256:cf2a43ac7065b3e0dc8510f8c1f13a75fb8fde805efa3b8cff7599a1ef497bc7"},
    {file = "pywinpty-2.0.14-cp312-none-win_amd64.whl", hash = "sha256:55dad362ef3e9408ade68fd173e4f9032b3ce08f68cfe7eacb2c263ea1179737"},
    {file = "pywinpty-2.0.14-cp313-none-win_amd64.whl", hash = "sha256:074fb988a56ec79ca90ed03a896d40707131897cefb8f76f926e3834227f2819"},
    {file = "pywinpty-2.0.14-cp39-none-win_amd64.whl", hash = "sha256:5725fd56f73c0531ec218663bd8c8ff5acc43c78962fab28564871b5fce053fd"},
    {file = "pywinpty-2.0.14.tar.gz", hash = "sha256:18bd9529e4a5daf2d9719aa17788ba6013e594ae94c5a0c27e83df3278b0660e"},
]

[[package]]
name = "pyyaml"
version = "6.0.2"
description = "YAML parser and emitter for Python"
optional = false
python-versions = ">=3.8"
files = [
    {file = "PyYAML-6.0.2-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:0a9a2848a5b7feac301353437eb7d5957887edbf81d56e903999a75a3d743086"},
    {file = "PyYAML-6.0.2-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:29717114e51c84ddfba879543fb232a6ed60086602313ca38cce623c1d62cfbf"},
    {file = "PyYAML-6.0.2-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:8824b5a04a04a047e72eea5cec3bc266db09e35de6bdfe34c9436ac5ee27d237"},
    {file = "PyYAML-6.0.2-cp310-cp310-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:7c36280e6fb8385e520936c3cb3b8042851904eba0e58d277dca80a5cfed590b"},
    {file = "PyYAML-6.0.2-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:ec031d5d2feb36d1d1a24380e4db6d43695f3748343d99434e6f5f9156aaa2ed"},
    {file = "PyYAML-6.0.2-cp310-cp310-musllinux_1_1_aarch64.whl", hash = "sha256:936d68689298c36b53b29f23c6dbb74de12b4ac12ca6cfe0e047bedceea56180"},
    {file = "PyYAML-6.0.2-cp310-cp310-musllinux_1_1_x86_64.whl", hash = "sha256:23502f431948090f597378482b4812b0caae32c22213aecf3b55325e049a6c68"},
    {file = "PyYAML-6.0.2-cp310-cp310-win32.whl", hash = "sha256:2e99c6826ffa974fe6e27cdb5ed0021786b03fc98e5ee3c5bfe1fd5015f42b99"},
    {file = "PyYAML-6.0.2-cp310-cp310-win_amd64.whl", hash = "sha256:a4d3091415f010369ae4ed1fc6b79def9416358877534caf6a0fdd2146c87a3e"},
    {file = "PyYAML-6.0.2-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:cc1c1159b3d456576af7a3e4d1ba7e6924cb39de8f67111c735f6fc832082774"},
    {file = "PyYAML-6.0.2-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:1e2120ef853f59c7419231f3bf4e7021f1b936f6ebd222406c3b60212205d2ee"},
    {file = "PyYAML-6.0.2-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:5d225db5a45f21e78dd9358e58a98702a0302f2659a3c6cd320564b75b86f47c"},
    {file = "PyYAML-6.0.2-cp311-cp311-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:5ac9328ec4831237bec75defaf839f7d4564be1e6b25ac710bd1a96321cc8317"},
    {file = "PyYAML-6.0.2-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:3ad2a3decf9aaba3d29c8f537ac4b243e36bef957511b4766cb0057d32b0be85"},
    {file = "PyYAML-6.0.2-cp311-cp311-musllinux_1_1_aarch64.whl", hash = "sha256:ff3824dc5261f50c9b0dfb3be22b4567a6f938ccce4587b38952d85fd9e9afe4"},
    {file = "PyYAML-6.0.2-cp311-cp311-musllinux_1_1_x86_64.whl", hash = "sha256:797b4f722ffa07cc8d62053e4cff1486fa6dc094105d13fea7b1de7d8bf71c9e"},
    {file = "PyYAML-6.0.2-cp311-cp311-win32.whl", hash = "sha256:11d8f3dd2b9c1207dcaf2ee0bbbfd5991f571186ec9cc78427ba5bd32afae4b5"},
    {file = "PyYAML-6.0.2-cp311-cp311-win_amd64.whl", hash = "sha256:e10ce637b18caea04431ce14fabcf5c64a1c61ec9c56b071a4b7ca131ca52d44"},
    {file = "PyYAML-6.0.2-cp312-cp312-macosx_10_9_x86_64.whl", hash = "sha256:c70c95198c015b85feafc136515252a261a84561b7b1d51e3384e0655ddf25ab"},
    {file = "PyYAML-6.0.2-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:ce826d6ef20b1bc864f0a68340c8b3287705cae2f8b4b1d932177dcc76721725"},
    {file = "PyYAML-6.0.2-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:1f71ea527786de97d1a0cc0eacd1defc0985dcf6b3f17bb77dcfc8c34bec4dc5"},
    {file = "PyYAML-6.0.2-cp312-cp312-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:9b22676e8097e9e22e36d6b7bda33190d0d400f345f23d4065d48f4ca7ae0425"},
    {file = "PyYAML-6.0.2-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:80bab7bfc629882493af4aa31a4cfa43a4c57c83813253626916b8c7ada83476"},
    {file = "PyYAML-6.0.2-cp312-cp312-musllinux_1_1_aarch64.whl", hash = "sha256:0833f8694549e586547b576dcfaba4a6b55b9e96098b36cdc7ebefe667dfed48"},
    {file = "PyYAML-6.0.2-cp312-cp312-musllinux_1_1_x86_64.whl", hash = "sha256:8b9c7197f7cb2738065c481a0461e50ad02f18c78cd75775628afb4d7137fb3b"},
    {file = "PyYAML-6.0.2-cp312-cp312-win32.whl", hash = "sha256:ef6107725bd54b262d6dedcc2af448a266975032bc85ef0172c5f059da6325b4"},
    {file = "PyYAML-6.0.2-cp312-cp312-win_amd64.whl", hash = "sha256:7e7401d0de89a9a855c839bc697c079a4af81cf878373abd7dc625847d25cbd8"},
    {file = "PyYAML-6.0.2-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:efdca5630322a10774e8e98e1af481aad470dd62c3170801852d752aa7a783ba"},
    {file = "PyYAML-6.0.2-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:50187695423ffe49e2deacb8cd10510bc361faac997de9efef88badc3bb9e2d1"},
    {file = "PyYAML-6.0.2-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:0ffe8360bab4910ef1b9e87fb812d8bc0a308b0d0eef8c8f44e0254ab3b07133"},
    {file = "PyYAML-6.0.2-cp313-cp313-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:17e311b6c678207928d649faa7cb0d7b4c26a0ba73d41e99c4fff6b6c3276484"},
    {file = "PyYAML-6.0.2-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:70b189594dbe54f75ab3a1acec5f1e3faa7e8cf2f1e08d9b561cb41b845f69d5"},
    {file = "PyYAML-6.0.2-cp313-cp313-musllinux_1_1_aarch64.whl", hash = "sha256:41e4e3953a79407c794916fa277a82531dd93aad34e29c2a514c2c0c5fe971cc"},
    {file = "PyYAML-6.0.2-cp313-cp313-musllinux_1_1_x86_64.whl", hash = "sha256:68ccc6023a3400877818152ad9a1033e3db8625d899c72eacb5a668902e4d652"},
    {file = "PyYAML-6.0.2-cp313-cp313-win32.whl", hash = "sha256:bc2fa7c6b47d6bc618dd7fb02ef6fdedb1090ec036abab80d4681424b84c1183"},
    {file = "PyYAML-6.0.2-cp313-cp313-win_amd64.whl", hash = "sha256:8388ee1976c416731879ac16da0aff3f63b286ffdd57cdeb95f3f2e085687563"},
    {file = "PyYAML-6.0.2-cp38-cp38-macosx_10_9_x86_64.whl", hash = "sha256:24471b829b3bf607e04e88d79542a9d48bb037c2267d7927a874e6c205ca7e9a"},
    {file = "PyYAML-6.0.2-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:d7fded462629cfa4b685c5416b949ebad6cec74af5e2d42905d41e257e0869f5"},
    {file = "PyYAML-6.0.2-cp38-cp38-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:d84a1718ee396f54f3a086ea0a66d8e552b2ab2017ef8b420e92edbc841c352d"},
    {file = "PyYAML-6.0.2-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:9056c1ecd25795207ad294bcf39f2db3d845767be0ea6e6a34d856f006006083"},
    {file = "PyYAML-6.0.2-cp38-cp38-musllinux_1_1_x86_64.whl", hash = "sha256:82d09873e40955485746739bcb8b4586983670466c23382c19cffecbf1fd8706"},
    {file = "PyYAML-6.0.2-cp38-cp38-win32.whl", hash = "sha256:43fa96a3ca0d6b1812e01ced1044a003533c47f6ee8aca31724f78e93ccc089a"},
    {file = "PyYAML-6.0.2-cp38-cp38-win_amd64.whl", hash = "sha256:01179a4a8559ab5de078078f37e5c1a30d76bb88519906844fd7bdea1b7729ff"},
    {file = "PyYAML-6.0.2-cp39-cp39-macosx_10_9_x86_64.whl", hash = "sha256:688ba32a1cffef67fd2e9398a2efebaea461578b0923624778664cc1c914db5d"},
    {file = "PyYAML-6.0.2-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:a8786accb172bd8afb8be14490a16625cbc387036876ab6ba70912730faf8e1f"},
    {file = "PyYAML-6.0.2-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:d8e03406cac8513435335dbab54c0d385e4a49e4945d2909a581c83647ca0290"},
    {file = "PyYAML-6.0.2-cp39-cp39-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:f753120cb8181e736c57ef7636e83f31b9c0d1722c516f7e86cf15b7aa57ff12"},
    {file = "PyYAML-6.0.2-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:3b1fdb9dc17f5a7677423d508ab4f243a726dea51fa5e70992e59a7411c89d19"},
    {file = "PyYAML-6.0.2-cp39-cp39-musllinux_1_1_aarch64.whl", hash = "sha256:0b69e4ce7a131fe56b7e4d770c67429700908fc0752af059838b1cfb41960e4e"},
    {file = "PyYAML-6.0.2-cp39-cp39-musllinux_1_1_x86_64.whl", hash = "sha256:a9f8c2e67970f13b16084e04f134610fd1d374bf477b17ec1599185cf611d725"},
    {file = "PyYAML-6.0.2-cp39-cp39-win32.whl", hash = "sha256:6395c297d42274772abc367baaa79683958044e5d3835486c16da75d2a694631"},
    {file = "PyYAML-6.0.2-cp39-cp39-win_amd64.whl", hash = "sha256:39693e1f8320ae4f43943590b49779ffb98acb81f788220ea932a6b6c51004d8"},
    {file = "pyyaml-6.0.2.tar.gz", hash = "sha256:d584d9ec91ad65861cc08d42e834324ef890a082e591037abe114850ff7bbc3e"},
]

[[package]]
name = "pyzmq"
version = "26.2.0"
description = "Python bindings for 0MQ"
optional = false
python-versions = ">=3.7"
files = [
    {file = "pyzmq-26.2.0-cp310-cp310-macosx_10_15_universal2.whl", hash = "sha256:ddf33d97d2f52d89f6e6e7ae66ee35a4d9ca6f36eda89c24591b0c40205a3629"},
    {file = "pyzmq-26.2.0-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:dacd995031a01d16eec825bf30802fceb2c3791ef24bcce48fa98ce40918c27b"},
    {file = "pyzmq-26.2.0-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:89289a5ee32ef6c439086184529ae060c741334b8970a6855ec0b6ad3ff28764"},
    {file = "pyzmq-26.2.0-cp310-cp310-manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:5506f06d7dc6ecf1efacb4a013b1f05071bb24b76350832c96449f4a2d95091c"},
    {file = "pyzmq-26.2.0-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:8ea039387c10202ce304af74def5021e9adc6297067f3441d348d2b633e8166a"},
    {file = "pyzmq-26.2.0-cp310-cp310-manylinux_2_28_x86_64.whl", hash = "sha256:a2224fa4a4c2ee872886ed00a571f5e967c85e078e8e8c2530a2fb01b3309b88"},
    {file = "pyzmq-26.2.0-cp310-cp310-musllinux_1_1_aarch64.whl", hash = "sha256:28ad5233e9c3b52d76196c696e362508959741e1a005fb8fa03b51aea156088f"},
    {file = "pyzmq-26.2.0-cp310-cp310-musllinux_1_1_i686.whl", hash = "sha256:1c17211bc037c7d88e85ed8b7d8f7e52db6dc8eca5590d162717c654550f7282"},
    {file = "pyzmq-26.2.0-cp310-cp310-musllinux_1_1_x86_64.whl", hash = "sha256:b8f86dd868d41bea9a5f873ee13bf5551c94cf6bc51baebc6f85075971fe6eea"},
    {file = "pyzmq-26.2.0-cp310-cp310-win32.whl", hash = "sha256:46a446c212e58456b23af260f3d9fb785054f3e3653dbf7279d8f2b5546b21c2"},
    {file = "pyzmq-26.2.0-cp310-cp310-win_amd64.whl", hash = "sha256:49d34ab71db5a9c292a7644ce74190b1dd5a3475612eefb1f8be1d6961441971"},
    {file = "pyzmq-26.2.0-cp310-cp310-win_arm64.whl", hash = "sha256:bfa832bfa540e5b5c27dcf5de5d82ebc431b82c453a43d141afb1e5d2de025fa"},
    {file = "pyzmq-26.2.0-cp311-cp311-macosx_10_15_universal2.whl", hash = "sha256:8f7e66c7113c684c2b3f1c83cdd3376103ee0ce4c49ff80a648643e57fb22218"},
    {file = "pyzmq-26.2.0-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:3a495b30fc91db2db25120df5847d9833af237546fd59170701acd816ccc01c4"},
    {file = "pyzmq-26.2.0-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:77eb0968da535cba0470a5165468b2cac7772cfb569977cff92e240f57e31bef"},
    {file = "pyzmq-26.2.0-cp311-cp311-manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:6ace4f71f1900a548f48407fc9be59c6ba9d9aaf658c2eea6cf2779e72f9f317"},
    {file = "pyzmq-26.2.0-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:92a78853d7280bffb93df0a4a6a2498cba10ee793cc8076ef797ef2f74d107cf"},
    {file = "pyzmq-26.2.0-cp311-cp311-manylinux_2_28_x86_64.whl", hash = "sha256:689c5d781014956a4a6de61d74ba97b23547e431e9e7d64f27d4922ba96e9d6e"},
    {file = "pyzmq-26.2.0-cp311-cp311-musllinux_1_1_aarch64.whl", hash = "sha256:0aca98bc423eb7d153214b2df397c6421ba6373d3397b26c057af3c904452e37"},
    {file = "pyzmq-26.2.0-cp311-cp311-musllinux_1_1_i686.whl", hash = "sha256:1f3496d76b89d9429a656293744ceca4d2ac2a10ae59b84c1da9b5165f429ad3"},
    {file = "pyzmq-26.2.0-cp311-cp311-musllinux_1_1_x86_64.whl", hash = "sha256:5c2b3bfd4b9689919db068ac6c9911f3fcb231c39f7dd30e3138be94896d18e6"},
    {file = "pyzmq-26.2.0-cp311-cp311-win32.whl", hash = "sha256:eac5174677da084abf378739dbf4ad245661635f1600edd1221f150b165343f4"},
    {file = "pyzmq-26.2.0-cp311-cp311-win_amd64.whl", hash = "sha256:5a509df7d0a83a4b178d0f937ef14286659225ef4e8812e05580776c70e155d5"},
    {file = "pyzmq-26.2.0-cp311-cp311-win_arm64.whl", hash = "sha256:c0e6091b157d48cbe37bd67233318dbb53e1e6327d6fc3bb284afd585d141003"},
    {file = "pyzmq-26.2.0-cp312-cp312-macosx_10_15_universal2.whl", hash = "sha256:ded0fc7d90fe93ae0b18059930086c51e640cdd3baebdc783a695c77f123dcd9"},
    {file = "pyzmq-26.2.0-cp312-cp312-macosx_10_9_x86_64.whl", hash = "sha256:17bf5a931c7f6618023cdacc7081f3f266aecb68ca692adac015c383a134ca52"},
    {file = "pyzmq-26.2.0-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:55cf66647e49d4621a7e20c8d13511ef1fe1efbbccf670811864452487007e08"},
    {file = "pyzmq-26.2.0-cp312-cp312-manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:4661c88db4a9e0f958c8abc2b97472e23061f0bc737f6f6179d7a27024e1faa5"},
    {file = "pyzmq-26.2.0-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:ea7f69de383cb47522c9c208aec6dd17697db7875a4674c4af3f8cfdac0bdeae"},
    {file = "pyzmq-26.2.0-cp312-cp312-manylinux_2_28_x86_64.whl", hash = "sha256:7f98f6dfa8b8ccaf39163ce872bddacca38f6a67289116c8937a02e30bbe9711"},
    {file = "pyzmq-26.2.0-cp312-cp312-musllinux_1_1_aarch64.whl", hash = "sha256:e3e0210287329272539eea617830a6a28161fbbd8a3271bf4150ae3e58c5d0e6"},
    {file = "pyzmq-26.2.0-cp312-cp312-musllinux_1_1_i686.whl", hash = "sha256:6b274e0762c33c7471f1a7471d1a2085b1a35eba5cdc48d2ae319f28b6fc4de3"},
    {file = "pyzmq-26.2.0-cp312-cp312-musllinux_1_1_x86_64.whl", hash = "sha256:29c6a4635eef69d68a00321e12a7d2559fe2dfccfa8efae3ffb8e91cd0b36a8b"},
    {file = "pyzmq-26.2.0-cp312-cp312-win32.whl", hash = "sha256:989d842dc06dc59feea09e58c74ca3e1678c812a4a8a2a419046d711031f69c7"},
    {file = "pyzmq-26.2.0-cp312-cp312-win_amd64.whl", hash = "sha256:2a50625acdc7801bc6f74698c5c583a491c61d73c6b7ea4dee3901bb99adb27a"},
    {file = "pyzmq-26.2.0-cp312-cp312-win_arm64.whl", hash = "sha256:4d29ab8592b6ad12ebbf92ac2ed2bedcfd1cec192d8e559e2e099f648570e19b"},
    {file = "pyzmq-26.2.0-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:9dd8cd1aeb00775f527ec60022004d030ddc51d783d056e3e23e74e623e33726"},
    {file = "pyzmq-26.2.0-cp313-cp313-macosx_10_15_universal2.whl", hash = "sha256:28c812d9757fe8acecc910c9ac9dafd2ce968c00f9e619db09e9f8f54c3a68a3"},
    {file = "pyzmq-26.2.0-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:4d80b1dd99c1942f74ed608ddb38b181b87476c6a966a88a950c7dee118fdf50"},
    {file = "pyzmq-26.2.0-cp313-cp313-manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:8c997098cc65e3208eca09303630e84d42718620e83b733d0fd69543a9cab9cb"},
    {file = "pyzmq-26.2.0-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:7ad1bc8d1b7a18497dda9600b12dc193c577beb391beae5cd2349184db40f187"},
    {file = "pyzmq-26.2.0-cp313-cp313-manylinux_2_28_x86_64.whl", hash = "sha256:bea2acdd8ea4275e1278350ced63da0b166421928276c7c8e3f9729d7402a57b"},
    {file = "pyzmq-26.2.0-cp313-cp313-musllinux_1_1_aarch64.whl", hash = "sha256:23f4aad749d13698f3f7b64aad34f5fc02d6f20f05999eebc96b89b01262fb18"},
    {file = "pyzmq-26.2.0-cp313-cp313-musllinux_1_1_i686.whl", hash = "sha256:a4f96f0d88accc3dbe4a9025f785ba830f968e21e3e2c6321ccdfc9aef755115"},
    {file = "pyzmq-26.2.0-cp313-cp313-musllinux_1_1_x86_64.whl", hash = "sha256:ced65e5a985398827cc9276b93ef6dfabe0273c23de8c7931339d7e141c2818e"},
    {file = "pyzmq-26.2.0-cp313-cp313-win32.whl", hash = "sha256:31507f7b47cc1ead1f6e86927f8ebb196a0bab043f6345ce070f412a59bf87b5"},
    {file = "pyzmq-26.2.0-cp313-cp313-win_amd64.whl", hash = "sha256:70fc7fcf0410d16ebdda9b26cbd8bf8d803d220a7f3522e060a69a9c87bf7bad"},
    {file = "pyzmq-26.2.0-cp313-cp313-win_arm64.whl", hash = "sha256:c3789bd5768ab5618ebf09cef6ec2b35fed88709b104351748a63045f0ff9797"},
    {file = "pyzmq-26.2.0-cp313-cp313t-macosx_10_13_x86_64.whl", hash = "sha256:034da5fc55d9f8da09015d368f519478a52675e558c989bfcb5cf6d4e16a7d2a"},
    {file = "pyzmq-26.2.0-cp313-cp313t-macosx_10_15_universal2.whl", hash = "sha256:c92d73464b886931308ccc45b2744e5968cbaade0b1d6aeb40d8ab537765f5bc"},
    {file = "pyzmq-26.2.0-cp313-cp313t-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:794a4562dcb374f7dbbfb3f51d28fb40123b5a2abadee7b4091f93054909add5"},
    {file = "pyzmq-26.2.0-cp313-cp313t-manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:aee22939bb6075e7afededabad1a56a905da0b3c4e3e0c45e75810ebe3a52672"},
    {file = "pyzmq-26.2.0-cp313-cp313t-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:2ae90ff9dad33a1cfe947d2c40cb9cb5e600d759ac4f0fd22616ce6540f72797"},
    {file = "pyzmq-26.2.0-cp313-cp313t-manylinux_2_28_x86_64.whl", hash = "sha256:43a47408ac52647dfabbc66a25b05b6a61700b5165807e3fbd40063fcaf46386"},
    {file = "pyzmq-26.2.0-cp313-cp313t-musllinux_1_1_aarch64.whl", hash = "sha256:25bf2374a2a8433633c65ccb9553350d5e17e60c8eb4de4d92cc6bd60f01d306"},
    {file = "pyzmq-26.2.0-cp313-cp313t-musllinux_1_1_i686.whl", hash = "sha256:007137c9ac9ad5ea21e6ad97d3489af654381324d5d3ba614c323f60dab8fae6"},
    {file = "pyzmq-26.2.0-cp313-cp313t-musllinux_1_1_x86_64.whl", hash = "sha256:470d4a4f6d48fb34e92d768b4e8a5cc3780db0d69107abf1cd7ff734b9766eb0"},
    {file = "pyzmq-26.2.0-cp37-cp37m-macosx_10_9_x86_64.whl", hash = "sha256:3b55a4229ce5da9497dd0452b914556ae58e96a4381bb6f59f1305dfd7e53fc8"},
    {file = "pyzmq-26.2.0-cp37-cp37m-manylinux_2_12_i686.manylinux2010_i686.whl", hash = "sha256:9cb3a6460cdea8fe8194a76de8895707e61ded10ad0be97188cc8463ffa7e3a8"},
    {file = "pyzmq-26.2.0-cp37-cp37m-manylinux_2_12_x86_64.manylinux2010_x86_64.whl", hash = "sha256:8ab5cad923cc95c87bffee098a27856c859bd5d0af31bd346035aa816b081fe1"},
    {file = "pyzmq-26.2.0-cp37-cp37m-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:9ed69074a610fad1c2fda66180e7b2edd4d31c53f2d1872bc2d1211563904cd9"},
    {file = "pyzmq-26.2.0-cp37-cp37m-musllinux_1_1_aarch64.whl", hash = "sha256:cccba051221b916a4f5e538997c45d7d136a5646442b1231b916d0164067ea27"},
    {file = "pyzmq-26.2.0-cp37-cp37m-musllinux_1_1_i686.whl", hash = "sha256:0eaa83fc4c1e271c24eaf8fb083cbccef8fde77ec8cd45f3c35a9a123e6da097"},
    {file = "pyzmq-26.2.0-cp37-cp37m-musllinux_1_1_x86_64.whl", hash = "sha256:9edda2df81daa129b25a39b86cb57dfdfe16f7ec15b42b19bfac503360d27a93"},
    {file = "pyzmq-26.2.0-cp37-cp37m-win32.whl", hash = "sha256:ea0eb6af8a17fa272f7b98d7bebfab7836a0d62738e16ba380f440fceca2d951"},
    {file = "pyzmq-26.2.0-cp37-cp37m-win_amd64.whl", hash = "sha256:4ff9dc6bc1664bb9eec25cd17506ef6672d506115095411e237d571e92a58231"},
    {file = "pyzmq-26.2.0-cp38-cp38-macosx_10_15_universal2.whl", hash = "sha256:2eb7735ee73ca1b0d71e0e67c3739c689067f055c764f73aac4cc8ecf958ee3f"},
    {file = "pyzmq-26.2.0-cp38-cp38-macosx_10_9_x86_64.whl", hash = "sha256:1a534f43bc738181aa7cbbaf48e3eca62c76453a40a746ab95d4b27b1111a7d2"},
    {file = "pyzmq-26.2.0-cp38-cp38-manylinux_2_12_i686.manylinux2010_i686.whl", hash = "sha256:aedd5dd8692635813368e558a05266b995d3d020b23e49581ddd5bbe197a8ab6"},
    {file = "pyzmq-26.2.0-cp38-cp38-manylinux_2_12_x86_64.manylinux2010_x86_64.whl", hash = "sha256:8be4700cd8bb02cc454f630dcdf7cfa99de96788b80c51b60fe2fe1dac480289"},
    {file = "pyzmq-26.2.0-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:1fcc03fa4997c447dce58264e93b5aa2d57714fbe0f06c07b7785ae131512732"},
    {file = "pyzmq-26.2.0-cp38-cp38-musllinux_1_1_aarch64.whl", hash = "sha256:402b190912935d3db15b03e8f7485812db350d271b284ded2b80d2e5704be780"},
    {file = "pyzmq-26.2.0-cp38-cp38-musllinux_1_1_i686.whl", hash = "sha256:8685fa9c25ff00f550c1fec650430c4b71e4e48e8d852f7ddcf2e48308038640"},
    {file = "pyzmq-26.2.0-cp38-cp38-musllinux_1_1_x86_64.whl", hash = "sha256:76589c020680778f06b7e0b193f4b6dd66d470234a16e1df90329f5e14a171cd"},
    {file = "pyzmq-26.2.0-cp38-cp38-win32.whl", hash = "sha256:8423c1877d72c041f2c263b1ec6e34360448decfb323fa8b94e85883043ef988"},
    {file = "pyzmq-26.2.0-cp38-cp38-win_amd64.whl", hash = "sha256:76589f2cd6b77b5bdea4fca5992dc1c23389d68b18ccc26a53680ba2dc80ff2f"},
    {file = "pyzmq-26.2.0-cp39-cp39-macosx_10_15_universal2.whl", hash = "sha256:b1d464cb8d72bfc1a3adc53305a63a8e0cac6bc8c5a07e8ca190ab8d3faa43c2"},
    {file = "pyzmq-26.2.0-cp39-cp39-macosx_10_9_x86_64.whl", hash = "sha256:4da04c48873a6abdd71811c5e163bd656ee1b957971db7f35140a2d573f6949c"},
    {file = "pyzmq-26.2.0-cp39-cp39-manylinux_2_12_i686.manylinux2010_i686.whl", hash = "sha256:d049df610ac811dcffdc147153b414147428567fbbc8be43bb8885f04db39d98"},
    {file = "pyzmq-26.2.0-cp39-cp39-manylinux_2_12_x86_64.manylinux2010_x86_64.whl", hash = "sha256:05590cdbc6b902101d0e65d6a4780af14dc22914cc6ab995d99b85af45362cc9"},
    {file = "pyzmq-26.2.0-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:c811cfcd6a9bf680236c40c6f617187515269ab2912f3d7e8c0174898e2519db"},
    {file = "pyzmq-26.2.0-cp39-cp39-musllinux_1_1_aarch64.whl", hash = "sha256:6835dd60355593de10350394242b5757fbbd88b25287314316f266e24c61d073"},
    {file = "pyzmq-26.2.0-cp39-cp39-musllinux_1_1_i686.whl", hash = "sha256:bc6bee759a6bddea5db78d7dcd609397449cb2d2d6587f48f3ca613b19410cfc"},
    {file = "pyzmq-26.2.0-cp39-cp39-musllinux_1_1_x86_64.whl", hash = "sha256:c530e1eecd036ecc83c3407f77bb86feb79916d4a33d11394b8234f3bd35b940"},
    {file = "pyzmq-26.2.0-cp39-cp39-win32.whl", hash = "sha256:367b4f689786fca726ef7a6c5ba606958b145b9340a5e4808132cc65759abd44"},
    {file = "pyzmq-26.2.0-cp39-cp39-win_amd64.whl", hash = "sha256:e6fa2e3e683f34aea77de8112f6483803c96a44fd726d7358b9888ae5bb394ec"},
    {file = "pyzmq-26.2.0-cp39-cp39-win_arm64.whl", hash = "sha256:7445be39143a8aa4faec43b076e06944b8f9d0701b669df4af200531b21e40bb"},
    {file = "pyzmq-26.2.0-pp310-pypy310_pp73-macosx_10_15_x86_64.whl", hash = "sha256:706e794564bec25819d21a41c31d4df2d48e1cc4b061e8d345d7fb4dd3e94072"},
    {file = "pyzmq-26.2.0-pp310-pypy310_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:8b435f2753621cd36e7c1762156815e21c985c72b19135dac43a7f4f31d28dd1"},
    {file = "pyzmq-26.2.0-pp310-pypy310_pp73-manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:160c7e0a5eb178011e72892f99f918c04a131f36056d10d9c1afb223fc952c2d"},
    {file = "pyzmq-26.2.0-pp310-pypy310_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:2c4a71d5d6e7b28a47a394c0471b7e77a0661e2d651e7ae91e0cab0a587859ca"},
    {file = "pyzmq-26.2.0-pp310-pypy310_pp73-win_amd64.whl", hash = "sha256:90412f2db8c02a3864cbfc67db0e3dcdbda336acf1c469526d3e869394fe001c"},
    {file = "pyzmq-26.2.0-pp37-pypy37_pp73-macosx_10_9_x86_64.whl", hash = "sha256:2ea4ad4e6a12e454de05f2949d4beddb52460f3de7c8b9d5c46fbb7d7222e02c"},
    {file = "pyzmq-26.2.0-pp37-pypy37_pp73-manylinux_2_12_i686.manylinux2010_i686.whl", hash = "sha256:fc4f7a173a5609631bb0c42c23d12c49df3966f89f496a51d3eb0ec81f4519d6"},
    {file = "pyzmq-26.2.0-pp37-pypy37_pp73-manylinux_2_12_x86_64.manylinux2010_x86_64.whl", hash = "sha256:878206a45202247781472a2d99df12a176fef806ca175799e1c6ad263510d57c"},
    {file = "pyzmq-26.2.0-pp37-pypy37_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:17c412bad2eb9468e876f556eb4ee910e62d721d2c7a53c7fa31e643d35352e6"},
    {file = "pyzmq-26.2.0-pp37-pypy37_pp73-win_amd64.whl", hash = "sha256:0d987a3ae5a71c6226b203cfd298720e0086c7fe7c74f35fa8edddfbd6597eed"},
    {file = "pyzmq-26.2.0-pp38-pypy38_pp73-macosx_10_9_x86_64.whl", hash = "sha256:39887ac397ff35b7b775db7201095fc6310a35fdbae85bac4523f7eb3b840e20"},
    {file = "pyzmq-26.2.0-pp38-pypy38_pp73-manylinux_2_12_i686.manylinux2010_i686.whl", hash = "sha256:fdb5b3e311d4d4b0eb8b3e8b4d1b0a512713ad7e6a68791d0923d1aec433d919"},
    {file = "pyzmq-26.2.0-pp38-pypy38_pp73-manylinux_2_12_x86_64.manylinux2010_x86_64.whl", hash = "sha256:226af7dcb51fdb0109f0016449b357e182ea0ceb6b47dfb5999d569e5db161d5"},
    {file = "pyzmq-26.2.0-pp38-pypy38_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:0bed0e799e6120b9c32756203fb9dfe8ca2fb8467fed830c34c877e25638c3fc"},
    {file = "pyzmq-26.2.0-pp38-pypy38_pp73-win_amd64.whl", hash = "sha256:29c7947c594e105cb9e6c466bace8532dc1ca02d498684128b339799f5248277"},
    {file = "pyzmq-26.2.0-pp39-pypy39_pp73-macosx_10_15_x86_64.whl", hash = "sha256:cdeabcff45d1c219636ee2e54d852262e5c2e085d6cb476d938aee8d921356b3"},
    {file = "pyzmq-26.2.0-pp39-pypy39_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:35cffef589bcdc587d06f9149f8d5e9e8859920a071df5a2671de2213bef592a"},
    {file = "pyzmq-26.2.0-pp39-pypy39_pp73-manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:18c8dc3b7468d8b4bdf60ce9d7141897da103c7a4690157b32b60acb45e333e6"},
    {file = "pyzmq-26.2.0-pp39-pypy39_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:7133d0a1677aec369d67dd78520d3fa96dd7f3dcec99d66c1762870e5ea1a50a"},
    {file = "pyzmq-26.2.0-pp39-pypy39_pp73-manylinux_2_28_x86_64.whl", hash = "sha256:6a96179a24b14fa6428cbfc08641c779a53f8fcec43644030328f44034c7f1f4"},
    {file = "pyzmq-26.2.0-pp39-pypy39_pp73-win_amd64.whl", hash = "sha256:4f78c88905461a9203eac9faac157a2a0dbba84a0fd09fd29315db27be40af9f"},
    {file = "pyzmq-26.2.0.tar.gz", hash = "sha256:070672c258581c8e4f640b5159297580a9974b026043bd4ab0470be9ed324f1f"},
]

[package.dependencies]
cffi = {version = "*", markers = "implementation_name == \"pypy\""}

[[package]]
name = "referencing"
version = "0.35.1"
description = "JSON Referencing + Python"
optional = false
python-versions = ">=3.8"
files = [
    {file = "referencing-0.35.1-py3-none-any.whl", hash = "sha256:eda6d3234d62814d1c64e305c1331c9a3a6132da475ab6382eaa997b21ee75de"},
    {file = "referencing-0.35.1.tar.gz", hash = "sha256:25b42124a6c8b632a425174f24087783efb348a6f1e0008e63cd4466fedf703c"},
]

[package.dependencies]
attrs = ">=22.2.0"
rpds-py = ">=0.7.0"

[[package]]
name = "requests"
version = "2.32.3"
description = "Python HTTP for Humans."
optional = false
python-versions = ">=3.8"
files = [
    {file = "requests-2.32.3-py3-none-any.whl", hash = "sha256:70761cfe03c773ceb22aa2f671b4757976145175cdfca038c02654d061d6dcc6"},
    {file = "requests-2.32.3.tar.gz", hash = "sha256:55365417734eb18255590a9ff9eb97e9e1da868d4ccd6402399eaf68af20a760"},
]

[package.dependencies]
certifi = ">=2017.4.17"
charset-normalizer = ">=2,<4"
idna = ">=2.5,<4"
urllib3 = ">=1.21.1,<3"

[package.extras]
socks = ["PySocks (>=1.5.6,!=1.5.7)"]
use-chardet-on-py3 = ["chardet (>=3.0.2,<6)"]

[[package]]
name = "rfc3339-validator"
version = "0.1.4"
description = "A pure python RFC3339 validator"
optional = false
python-versions = ">=2.7, !=3.0.*, !=3.1.*, !=3.2.*, !=3.3.*, !=3.4.*"
files = [
    {file = "rfc3339_validator-0.1.4-py2.py3-none-any.whl", hash = "sha256:24f6ec1eda14ef823da9e36ec7113124b39c04d50a4d3d3a3c2859577e7791fa"},
    {file = "rfc3339_validator-0.1.4.tar.gz", hash = "sha256:138a2abdf93304ad60530167e51d2dfb9549521a836871b88d7f4695d0022f6b"},
]

[package.dependencies]
six = "*"

[[package]]
name = "rfc3986-validator"
version = "0.1.1"
description = "Pure python rfc3986 validator"
optional = false
python-versions = ">=2.7, !=3.0.*, !=3.1.*, !=3.2.*, !=3.3.*, !=3.4.*"
files = [
    {file = "rfc3986_validator-0.1.1-py2.py3-none-any.whl", hash = "sha256:2f235c432ef459970b4306369336b9d5dbdda31b510ca1e327636e01f528bfa9"},
    {file = "rfc3986_validator-0.1.1.tar.gz", hash = "sha256:3d44bde7921b3b9ec3ae4e3adca370438eccebc676456449b145d533b240d055"},
]

[[package]]
name = "rich"
version = "13.9.4"
description = "Render rich text, tables, progress bars, syntax highlighting, markdown and more to the terminal"
optional = false
python-versions = ">=3.8.0"
files = [
    {file = "rich-13.9.4-py3-none-any.whl", hash = "sha256:6049d5e6ec054bf2779ab3358186963bac2ea89175919d699e378b99738c2a90"},
    {file = "rich-13.9.4.tar.gz", hash = "sha256:439594978a49a09530cff7ebc4b5c7103ef57baf48d5ea3184f21d9a2befa098"},
]

[package.dependencies]
markdown-it-py = ">=2.2.0"
pygments = ">=2.13.0,<3.0.0"

[package.extras]
jupyter = ["ipywidgets (>=7.5.1,<9)"]

[[package]]
name = "rich-toolkit"
version = "0.12.0"
description = "Rich toolkit for building command-line applications"
optional = false
python-versions = ">=3.8"
files = [
    {file = "rich_toolkit-0.12.0-py3-none-any.whl", hash = "sha256:a2da4416384410ae871e890db7edf8623e1f5e983341dbbc8cc03603ce24f0ab"},
    {file = "rich_toolkit-0.12.0.tar.gz", hash = "sha256:facb0b40418010309f77abd44e2583b4936656f6ee5c8625da807564806a6c40"},
]

[package.dependencies]
click = ">=8.1.7"
rich = ">=13.7.1"
typing-extensions = ">=4.12.2"

[[package]]
name = "rpds-py"
version = "0.22.3"
description = "Python bindings to Rust's persistent data structures (rpds)"
optional = false
python-versions = ">=3.9"
files = [
    {file = "rpds_py-0.22.3-cp310-cp310-macosx_10_12_x86_64.whl", hash = "sha256:6c7b99ca52c2c1752b544e310101b98a659b720b21db00e65edca34483259967"},
    {file = "rpds_py-0.22.3-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:be2eb3f2495ba669d2a985f9b426c1797b7d48d6963899276d22f23e33d47e37"},
    {file = "rpds_py-0.22.3-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:70eb60b3ae9245ddea20f8a4190bd79c705a22f8028aaf8bbdebe4716c3fab24"},
    {file = "rpds_py-0.22.3-cp310-cp310-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:4041711832360a9b75cfb11b25a6a97c8fb49c07b8bd43d0d02b45d0b499a4ff"},
    {file = "rpds_py-0.22.3-cp310-cp310-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:64607d4cbf1b7e3c3c8a14948b99345eda0e161b852e122c6bb71aab6d1d798c"},
    {file = "rpds_py-0.22.3-cp310-cp310-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:81e69b0a0e2537f26d73b4e43ad7bc8c8efb39621639b4434b76a3de50c6966e"},
    {file = "rpds_py-0.22.3-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:bc27863442d388870c1809a87507727b799c8460573cfbb6dc0eeaef5a11b5ec"},
    {file = "rpds_py-0.22.3-cp310-cp310-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:e79dd39f1e8c3504be0607e5fc6e86bb60fe3584bec8b782578c3b0fde8d932c"},
    {file = "rpds_py-0.22.3-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:e0fa2d4ec53dc51cf7d3bb22e0aa0143966119f42a0c3e4998293a3dd2856b09"},
    {file = "rpds_py-0.22.3-cp310-cp310-musllinux_1_2_i686.whl", hash = "sha256:fda7cb070f442bf80b642cd56483b5548e43d366fe3f39b98e67cce780cded00"},
    {file = "rpds_py-0.22.3-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:cff63a0272fcd259dcc3be1657b07c929c466b067ceb1c20060e8d10af56f5bf"},
    {file = "rpds_py-0.22.3-cp310-cp310-win32.whl", hash = "sha256:9bd7228827ec7bb817089e2eb301d907c0d9827a9e558f22f762bb690b131652"},
    {file = "rpds_py-0.22.3-cp310-cp310-win_amd64.whl", hash = "sha256:9beeb01d8c190d7581a4d59522cd3d4b6887040dcfc744af99aa59fef3e041a8"},
    {file = "rpds_py-0.22.3-cp311-cp311-macosx_10_12_x86_64.whl", hash = "sha256:d20cfb4e099748ea39e6f7b16c91ab057989712d31761d3300d43134e26e165f"},
    {file = "rpds_py-0.22.3-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:68049202f67380ff9aa52f12e92b1c30115f32e6895cd7198fa2a7961621fc5a"},
    {file = "rpds_py-0.22.3-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:fb4f868f712b2dd4bcc538b0a0c1f63a2b1d584c925e69a224d759e7070a12d5"},
    {file = "rpds_py-0.22.3-cp311-cp311-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:bc51abd01f08117283c5ebf64844a35144a0843ff7b2983e0648e4d3d9f10dbb"},
    {file = "rpds_py-0.22.3-cp311-cp311-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:0f3cec041684de9a4684b1572fe28c7267410e02450f4561700ca5a3bc6695a2"},
    {file = "rpds_py-0.22.3-cp311-cp311-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:7ef9d9da710be50ff6809fed8f1963fecdfecc8b86656cadfca3bc24289414b0"},
    {file = "rpds_py-0.22.3-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:59f4a79c19232a5774aee369a0c296712ad0e77f24e62cad53160312b1c1eaa1"},
    {file = "rpds_py-0.22.3-cp311-cp311-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:1a60bce91f81ddaac922a40bbb571a12c1070cb20ebd6d49c48e0b101d87300d"},
    {file = "rpds_py-0.22.3-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:e89391e6d60251560f0a8f4bd32137b077a80d9b7dbe6d5cab1cd80d2746f648"},
    {file = "rpds_py-0.22.3-cp311-cp311-musllinux_1_2_i686.whl", hash = "sha256:e3fb866d9932a3d7d0c82da76d816996d1667c44891bd861a0f97ba27e84fc74"},
    {file = "rpds_py-0.22.3-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:1352ae4f7c717ae8cba93421a63373e582d19d55d2ee2cbb184344c82d2ae55a"},
    {file = "rpds_py-0.22.3-cp311-cp311-win32.whl", hash = "sha256:b0b4136a252cadfa1adb705bb81524eee47d9f6aab4f2ee4fa1e9d3cd4581f64"},
    {file = "rpds_py-0.22.3-cp311-cp311-win_amd64.whl", hash = "sha256:8bd7c8cfc0b8247c8799080fbff54e0b9619e17cdfeb0478ba7295d43f635d7c"},
    {file = "rpds_py-0.22.3-cp312-cp312-macosx_10_12_x86_64.whl", hash = "sha256:27e98004595899949bd7a7b34e91fa7c44d7a97c40fcaf1d874168bb652ec67e"},
    {file = "rpds_py-0.22.3-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:1978d0021e943aae58b9b0b196fb4895a25cc53d3956b8e35e0b7682eefb6d56"},
    {file = "rpds_py-0.22.3-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:655ca44a831ecb238d124e0402d98f6212ac527a0ba6c55ca26f616604e60a45"},
    {file = "rpds_py-0.22.3-cp312-cp312-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:feea821ee2a9273771bae61194004ee2fc33f8ec7db08117ef9147d4bbcbca8e"},
    {file = "rpds_py-0.22.3-cp312-cp312-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:22bebe05a9ffc70ebfa127efbc429bc26ec9e9b4ee4d15a740033efda515cf3d"},
    {file = "rpds_py-0.22.3-cp312-cp312-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:3af6e48651c4e0d2d166dc1b033b7042ea3f871504b6805ba5f4fe31581d8d38"},
    {file = "rpds_py-0.22.3-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:e67ba3c290821343c192f7eae1d8fd5999ca2dc99994114643e2f2d3e6138b15"},
    {file = "rpds_py-0.22.3-cp312-cp312-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:02fbb9c288ae08bcb34fb41d516d5eeb0455ac35b5512d03181d755d80810059"},
    {file = "rpds_py-0.22.3-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:f56a6b404f74ab372da986d240e2e002769a7d7102cc73eb238a4f72eec5284e"},
    {file = "rpds_py-0.22.3-cp312-cp312-musllinux_1_2_i686.whl", hash = "sha256:0a0461200769ab3b9ab7e513f6013b7a97fdeee41c29b9db343f3c5a8e2b9e61"},
    {file = "rpds_py-0.22.3-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:8633e471c6207a039eff6aa116e35f69f3156b3989ea3e2d755f7bc41754a4a7"},
    {file = "rpds_py-0.22.3-cp312-cp312-win32.whl", hash = "sha256:593eba61ba0c3baae5bc9be2f5232430453fb4432048de28399ca7376de9c627"},
    {file = "rpds_py-0.22.3-cp312-cp312-win_amd64.whl", hash = "sha256:d115bffdd417c6d806ea9069237a4ae02f513b778e3789a359bc5856e0404cc4"},
    {file = "rpds_py-0.22.3-cp313-cp313-macosx_10_12_x86_64.whl", hash = "sha256:ea7433ce7e4bfc3a85654aeb6747babe3f66eaf9a1d0c1e7a4435bbdf27fea84"},
    {file = "rpds_py-0.22.3-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:6dd9412824c4ce1aca56c47b0991e65bebb7ac3f4edccfd3f156150c96a7bf25"},
    {file = "rpds_py-0.22.3-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:20070c65396f7373f5df4005862fa162db5d25d56150bddd0b3e8214e8ef45b4"},
    {file = "rpds_py-0.22.3-cp313-cp313-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:0b09865a9abc0ddff4e50b5ef65467cd94176bf1e0004184eb915cbc10fc05c5"},
    {file = "rpds_py-0.22.3-cp313-cp313-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:3453e8d41fe5f17d1f8e9c383a7473cd46a63661628ec58e07777c2fff7196dc"},
    {file = "rpds_py-0.22.3-cp313-cp313-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:f5d36399a1b96e1a5fdc91e0522544580dbebeb1f77f27b2b0ab25559e103b8b"},
    {file = "rpds_py-0.22.3-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:009de23c9c9ee54bf11303a966edf4d9087cd43a6003672e6aa7def643d06518"},
    {file = "rpds_py-0.22.3-cp313-cp313-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:1aef18820ef3e4587ebe8b3bc9ba6e55892a6d7b93bac6d29d9f631a3b4befbd"},
    {file = "rpds_py-0.22.3-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:f60bd8423be1d9d833f230fdbccf8f57af322d96bcad6599e5a771b151398eb2"},
    {file = "rpds_py-0.22.3-cp313-cp313-musllinux_1_2_i686.whl", hash = "sha256:62d9cfcf4948683a18a9aff0ab7e1474d407b7bab2ca03116109f8464698ab16"},
    {file = "rpds_py-0.22.3-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:9253fc214112405f0afa7db88739294295f0e08466987f1d70e29930262b4c8f"},
    {file = "rpds_py-0.22.3-cp313-cp313-win32.whl", hash = "sha256:fb0ba113b4983beac1a2eb16faffd76cb41e176bf58c4afe3e14b9c681f702de"},
    {file = "rpds_py-0.22.3-cp313-cp313-win_amd64.whl", hash = "sha256:c58e2339def52ef6b71b8f36d13c3688ea23fa093353f3a4fee2556e62086ec9"},
    {file = "rpds_py-0.22.3-cp313-cp313t-macosx_10_12_x86_64.whl", hash = "sha256:f82a116a1d03628a8ace4859556fb39fd1424c933341a08ea3ed6de1edb0283b"},
    {file = "rpds_py-0.22.3-cp313-cp313t-macosx_11_0_arm64.whl", hash = "sha256:3dfcbc95bd7992b16f3f7ba05af8a64ca694331bd24f9157b49dadeeb287493b"},
    {file = "rpds_py-0.22.3-cp313-cp313t-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:59259dc58e57b10e7e18ce02c311804c10c5a793e6568f8af4dead03264584d1"},
    {file = "rpds_py-0.22.3-cp313-cp313t-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:5725dd9cc02068996d4438d397e255dcb1df776b7ceea3b9cb972bdb11260a83"},
    {file = "rpds_py-0.22.3-cp313-cp313t-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:99b37292234e61325e7a5bb9689e55e48c3f5f603af88b1642666277a81f1fbd"},
    {file = "rpds_py-0.22.3-cp313-cp313t-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:27b1d3b3915a99208fee9ab092b8184c420f2905b7d7feb4aeb5e4a9c509b8a1"},
    {file = "rpds_py-0.22.3-cp313-cp313t-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:f612463ac081803f243ff13cccc648578e2279295048f2a8d5eb430af2bae6e3"},
    {file = "rpds_py-0.22.3-cp313-cp313t-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:f73d3fef726b3243a811121de45193c0ca75f6407fe66f3f4e183c983573e130"},
    {file = "rpds_py-0.22.3-cp313-cp313t-musllinux_1_2_aarch64.whl", hash = "sha256:3f21f0495edea7fdbaaa87e633a8689cd285f8f4af5c869f27bc8074638ad69c"},
    {file = "rpds_py-0.22.3-cp313-cp313t-musllinux_1_2_i686.whl", hash = "sha256:1e9663daaf7a63ceccbbb8e3808fe90415b0757e2abddbfc2e06c857bf8c5e2b"},
    {file = "rpds_py-0.22.3-cp313-cp313t-musllinux_1_2_x86_64.whl", hash = "sha256:a76e42402542b1fae59798fab64432b2d015ab9d0c8c47ba7addddbaf7952333"},
    {file = "rpds_py-0.22.3-cp313-cp313t-win32.whl", hash = "sha256:69803198097467ee7282750acb507fba35ca22cc3b85f16cf45fb01cb9097730"},
    {file = "rpds_py-0.22.3-cp313-cp313t-win_amd64.whl", hash = "sha256:f5cf2a0c2bdadf3791b5c205d55a37a54025c6e18a71c71f82bb536cf9a454bf"},
    {file = "rpds_py-0.22.3-cp39-cp39-macosx_10_12_x86_64.whl", hash = "sha256:378753b4a4de2a7b34063d6f95ae81bfa7b15f2c1a04a9518e8644e81807ebea"},
    {file = "rpds_py-0.22.3-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:3445e07bf2e8ecfeef6ef67ac83de670358abf2996916039b16a218e3d95e97e"},
    {file = "rpds_py-0.22.3-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:7b2513ba235829860b13faa931f3b6846548021846ac808455301c23a101689d"},
    {file = "rpds_py-0.22.3-cp39-cp39-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:eaf16ae9ae519a0e237a0f528fd9f0197b9bb70f40263ee57ae53c2b8d48aeb3"},
    {file = "rpds_py-0.22.3-cp39-cp39-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:583f6a1993ca3369e0f80ba99d796d8e6b1a3a2a442dd4e1a79e652116413091"},
    {file = "rpds_py-0.22.3-cp39-cp39-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:4617e1915a539a0d9a9567795023de41a87106522ff83fbfaf1f6baf8e85437e"},
    {file = "rpds_py-0.22.3-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:0c150c7a61ed4a4f4955a96626574e9baf1adf772c2fb61ef6a5027e52803543"},
    {file = "rpds_py-0.22.3-cp39-cp39-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:2fa4331c200c2521512595253f5bb70858b90f750d39b8cbfd67465f8d1b596d"},
    {file = "rpds_py-0.22.3-cp39-cp39-musllinux_1_2_aarch64.whl", hash = "sha256:214b7a953d73b5e87f0ebece4a32a5bd83c60a3ecc9d4ec8f1dca968a2d91e99"},
    {file = "rpds_py-0.22.3-cp39-cp39-musllinux_1_2_i686.whl", hash = "sha256:f47ad3d5f3258bd7058d2d506852217865afefe6153a36eb4b6928758041d831"},
    {file = "rpds_py-0.22.3-cp39-cp39-musllinux_1_2_x86_64.whl", hash = "sha256:f276b245347e6e36526cbd4a266a417796fc531ddf391e43574cf6466c492520"},
    {file = "rpds_py-0.22.3-cp39-cp39-win32.whl", hash = "sha256:bbb232860e3d03d544bc03ac57855cd82ddf19c7a07651a7c0fdb95e9efea8b9"},
    {file = "rpds_py-0.22.3-cp39-cp39-win_amd64.whl", hash = "sha256:cfbc454a2880389dbb9b5b398e50d439e2e58669160f27b60e5eca11f68ae17c"},
    {file = "rpds_py-0.22.3-pp310-pypy310_pp73-macosx_10_12_x86_64.whl", hash = "sha256:d48424e39c2611ee1b84ad0f44fb3b2b53d473e65de061e3f460fc0be5f1939d"},
    {file = "rpds_py-0.22.3-pp310-pypy310_pp73-macosx_11_0_arm64.whl", hash = "sha256:24e8abb5878e250f2eb0d7859a8e561846f98910326d06c0d51381fed59357bd"},
    {file = "rpds_py-0.22.3-pp310-pypy310_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:4b232061ca880db21fa14defe219840ad9b74b6158adb52ddf0e87bead9e8493"},
    {file = "rpds_py-0.22.3-pp310-pypy310_pp73-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:ac0a03221cdb5058ce0167ecc92a8c89e8d0decdc9e99a2ec23380793c4dcb96"},
    {file = "rpds_py-0.22.3-pp310-pypy310_pp73-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:eb0c341fa71df5a4595f9501df4ac5abfb5a09580081dffbd1ddd4654e6e9123"},
    {file = "rpds_py-0.22.3-pp310-pypy310_pp73-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:bf9db5488121b596dbfc6718c76092fda77b703c1f7533a226a5a9f65248f8ad"},
    {file = "rpds_py-0.22.3-pp310-pypy310_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:0b8db6b5b2d4491ad5b6bdc2bc7c017eec108acbf4e6785f42a9eb0ba234f4c9"},
    {file = "rpds_py-0.22.3-pp310-pypy310_pp73-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:b3d504047aba448d70cf6fa22e06cb09f7cbd761939fdd47604f5e007675c24e"},
    {file = "rpds_py-0.22.3-pp310-pypy310_pp73-musllinux_1_2_aarch64.whl", hash = "sha256:e61b02c3f7a1e0b75e20c3978f7135fd13cb6cf551bf4a6d29b999a88830a338"},
    {file = "rpds_py-0.22.3-pp310-pypy310_pp73-musllinux_1_2_i686.whl", hash = "sha256:e35ba67d65d49080e8e5a1dd40101fccdd9798adb9b050ff670b7d74fa41c566"},
    {file = "rpds_py-0.22.3-pp310-pypy310_pp73-musllinux_1_2_x86_64.whl", hash = "sha256:26fd7cac7dd51011a245f29a2cc6489c4608b5a8ce8d75661bb4a1066c52dfbe"},
    {file = "rpds_py-0.22.3-pp310-pypy310_pp73-win_amd64.whl", hash = "sha256:177c7c0fce2855833819c98e43c262007f42ce86651ffbb84f37883308cb0e7d"},
    {file = "rpds_py-0.22.3-pp39-pypy39_pp73-macosx_10_12_x86_64.whl", hash = "sha256:bb47271f60660803ad11f4c61b42242b8c1312a31c98c578f79ef9387bbde21c"},
    {file = "rpds_py-0.22.3-pp39-pypy39_pp73-macosx_11_0_arm64.whl", hash = "sha256:70fb28128acbfd264eda9bf47015537ba3fe86e40d046eb2963d75024be4d055"},
    {file = "rpds_py-0.22.3-pp39-pypy39_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:44d61b4b7d0c2c9ac019c314e52d7cbda0ae31078aabd0f22e583af3e0d79723"},
    {file = "rpds_py-0.22.3-pp39-pypy39_pp73-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:5f0e260eaf54380380ac3808aa4ebe2d8ca28b9087cf411649f96bad6900c728"},
    {file = "rpds_py-0.22.3-pp39-pypy39_pp73-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:b25bc607423935079e05619d7de556c91fb6adeae9d5f80868dde3468657994b"},
    {file = "rpds_py-0.22.3-pp39-pypy39_pp73-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:fb6116dfb8d1925cbdb52595560584db42a7f664617a1f7d7f6e32f138cdf37d"},
    {file = "rpds_py-0.22.3-pp39-pypy39_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:a63cbdd98acef6570c62b92a1e43266f9e8b21e699c363c0fef13bd530799c11"},
    {file = "rpds_py-0.22.3-pp39-pypy39_pp73-manylinux_2_5_i686.manylinux1_i686.whl", hash = "sha256:2b8f60e1b739a74bab7e01fcbe3dddd4657ec685caa04681df9d562ef15b625f"},
    {file = "rpds_py-0.22.3-pp39-pypy39_pp73-musllinux_1_2_aarch64.whl", hash = "sha256:2e8b55d8517a2fda8d95cb45d62a5a8bbf9dd0ad39c5b25c8833efea07b880ca"},
    {file = "rpds_py-0.22.3-pp39-pypy39_pp73-musllinux_1_2_i686.whl", hash = "sha256:2de29005e11637e7a2361fa151f780ff8eb2543a0da1413bb951e9f14b699ef3"},
    {file = "rpds_py-0.22.3-pp39-pypy39_pp73-musllinux_1_2_x86_64.whl", hash = "sha256:666ecce376999bf619756a24ce15bb14c5bfaf04bf00abc7e663ce17c3f34fe7"},
    {file = "rpds_py-0.22.3-pp39-pypy39_pp73-win_amd64.whl", hash = "sha256:5246b14ca64a8675e0a7161f7af68fe3e910e6b90542b4bfb5439ba752191df6"},
    {file = "rpds_py-0.22.3.tar.gz", hash = "sha256:e32fee8ab45d3c2db6da19a5323bc3362237c8b653c70194414b892fd06a080d"},
]

[[package]]
name = "ruff"
version = "0.6.9"
description = "An extremely fast Python linter and code formatter, written in Rust."
optional = false
python-versions = ">=3.7"
files = [
    {file = "ruff-0.6.9-py3-none-linux_armv6l.whl", hash = "sha256:064df58d84ccc0ac0fcd63bc3090b251d90e2a372558c0f057c3f75ed73e1ccd"},
    {file = "ruff-0.6.9-py3-none-macosx_10_12_x86_64.whl", hash = "sha256:140d4b5c9f5fc7a7b074908a78ab8d384dd7f6510402267bc76c37195c02a7ec"},
    {file = "ruff-0.6.9-py3-none-macosx_11_0_arm64.whl", hash = "sha256:53fd8ca5e82bdee8da7f506d7b03a261f24cd43d090ea9db9a1dc59d9313914c"},
    {file = "ruff-0.6.9-py3-none-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:645d7d8761f915e48a00d4ecc3686969761df69fb561dd914a773c1a8266e14e"},
    {file = "ruff-0.6.9-py3-none-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:eae02b700763e3847595b9d2891488989cac00214da7f845f4bcf2989007d577"},
    {file = "ruff-0.6.9-py3-none-manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:7d5ccc9e58112441de8ad4b29dcb7a86dc25c5f770e3c06a9d57e0e5eba48829"},
    {file = "ruff-0.6.9-py3-none-manylinux_2_17_ppc64.manylinux2014_ppc64.whl", hash = "sha256:417b81aa1c9b60b2f8edc463c58363075412866ae4e2b9ab0f690dc1e87ac1b5"},
    {file = "ruff-0.6.9-py3-none-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:3c866b631f5fbce896a74a6e4383407ba7507b815ccc52bcedabb6810fdb3ef7"},
    {file = "ruff-0.6.9-py3-none-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:7b118afbb3202f5911486ad52da86d1d52305b59e7ef2031cea3425142b97d6f"},
    {file = "ruff-0.6.9-py3-none-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:a67267654edc23c97335586774790cde402fb6bbdb3c2314f1fc087dee320bfa"},
    {file = "ruff-0.6.9-py3-none-musllinux_1_2_aarch64.whl", hash = "sha256:3ef0cc774b00fec123f635ce5c547dac263f6ee9fb9cc83437c5904183b55ceb"},
    {file = "ruff-0.6.9-py3-none-musllinux_1_2_armv7l.whl", hash = "sha256:12edd2af0c60fa61ff31cefb90aef4288ac4d372b4962c2864aeea3a1a2460c0"},
    {file = "ruff-0.6.9-py3-none-musllinux_1_2_i686.whl", hash = "sha256:55bb01caeaf3a60b2b2bba07308a02fca6ab56233302406ed5245180a05c5625"},
    {file = "ruff-0.6.9-py3-none-musllinux_1_2_x86_64.whl", hash = "sha256:925d26471fa24b0ce5a6cdfab1bb526fb4159952385f386bdcc643813d472039"},
    {file = "ruff-0.6.9-py3-none-win32.whl", hash = "sha256:eb61ec9bdb2506cffd492e05ac40e5bc6284873aceb605503d8494180d6fc84d"},
    {file = "ruff-0.6.9-py3-none-win_amd64.whl", hash = "sha256:785d31851c1ae91f45b3d8fe23b8ae4b5170089021fbb42402d811135f0b7117"},
    {file = "ruff-0.6.9-py3-none-win_arm64.whl", hash = "sha256:a9641e31476d601f83cd602608739a0840e348bda93fec9f1ee816f8b6798b93"},
    {file = "ruff-0.6.9.tar.gz", hash = "sha256:b076ef717a8e5bc819514ee1d602bbdca5b4420ae13a9cf61a0c0a4f53a2baa2"},
]

[[package]]
name = "seaborn"
version = "0.13.2"
description = "Statistical data visualization"
optional = false
python-versions = ">=3.8"
files = [
    {file = "seaborn-0.13.2-py3-none-any.whl", hash = "sha256:636f8336facf092165e27924f223d3c62ca560b1f2bb5dff7ab7fad265361987"},
    {file = "seaborn-0.13.2.tar.gz", hash = "sha256:93e60a40988f4d65e9f4885df477e2fdaff6b73a9ded434c1ab356dd57eefff7"},
]

[package.dependencies]
matplotlib = ">=3.4,<3.6.1 || >3.6.1"
numpy = ">=1.20,<1.24.0 || >1.24.0"
pandas = ">=1.2"

[package.extras]
dev = ["flake8", "flit", "mypy", "pandas-stubs", "pre-commit", "pytest", "pytest-cov", "pytest-xdist"]
docs = ["ipykernel", "nbconvert", "numpydoc", "pydata_sphinx_theme (==0.10.0rc2)", "pyyaml", "sphinx (<6.0.0)", "sphinx-copybutton", "sphinx-design", "sphinx-issues"]
stats = ["scipy (>=1.7)", "statsmodels (>=0.12)"]

[[package]]
name = "send2trash"
version = "1.8.3"
description = "Send file to trash natively under Mac OS X, Windows and Linux"
optional = false
python-versions = "!=3.0.*,!=3.1.*,!=3.2.*,!=3.3.*,!=3.4.*,!=3.5.*,>=2.7"
files = [
    {file = "Send2Trash-1.8.3-py3-none-any.whl", hash = "sha256:0c31227e0bd08961c7665474a3d1ef7193929fedda4233843689baa056be46c9"},
    {file = "Send2Trash-1.8.3.tar.gz", hash = "sha256:b18e7a3966d99871aefeb00cfbcfdced55ce4871194810fc71f4aa484b953abf"},
]

[package.extras]
nativelib = ["pyobjc-framework-Cocoa", "pywin32"]
objc = ["pyobjc-framework-Cocoa"]
win32 = ["pywin32"]

[[package]]
name = "setuptools"
version = "75.6.0"
description = "Easily download, build, install, upgrade, and uninstall Python packages"
optional = false
python-versions = ">=3.9"
files = [
    {file = "setuptools-75.6.0-py3-none-any.whl", hash = "sha256:ce74b49e8f7110f9bf04883b730f4765b774ef3ef28f722cce7c273d253aaf7d"},
    {file = "setuptools-75.6.0.tar.gz", hash = "sha256:8199222558df7c86216af4f84c30e9b34a61d8ba19366cc914424cdbd28252f6"},
]

[package.extras]
check = ["pytest-checkdocs (>=2.4)", "pytest-ruff (>=0.2.1)", "ruff (>=0.7.0)"]
core = ["importlib_metadata (>=6)", "jaraco.collections", "jaraco.functools (>=4)", "jaraco.text (>=3.7)", "more_itertools", "more_itertools (>=8.8)", "packaging", "packaging (>=24.2)", "platformdirs (>=4.2.2)", "tomli (>=2.0.1)", "wheel (>=0.43.0)"]
cover = ["pytest-cov"]
doc = ["furo", "jaraco.packaging (>=9.3)", "jaraco.tidelift (>=1.4)", "pygments-github-lexers (==0.0.5)", "pyproject-hooks (!=1.1)", "rst.linker (>=1.9)", "sphinx (>=3.5)", "sphinx-favicon", "sphinx-inline-tabs", "sphinx-lint", "sphinx-notfound-page (>=1,<2)", "sphinx-reredirects", "sphinxcontrib-towncrier", "towncrier (<24.7)"]
enabler = ["pytest-enabler (>=2.2)"]
test = ["build[virtualenv] (>=1.0.3)", "filelock (>=3.4.0)", "ini2toml[lite] (>=0.14)", "jaraco.develop (>=7.21)", "jaraco.envs (>=2.2)", "jaraco.path (>=3.2.0)", "jaraco.test (>=5.5)", "packaging (>=24.2)", "pip (>=19.1)", "pyproject-hooks (!=1.1)", "pytest (>=6,!=8.1.*)", "pytest-home (>=0.5)", "pytest-perf", "pytest-subprocess", "pytest-timeout", "pytest-xdist (>=3)", "tomli-w (>=1.0.0)", "virtualenv (>=13.0.0)", "wheel (>=0.44.0)"]
type = ["importlib_metadata (>=7.0.2)", "jaraco.develop (>=7.21)", "mypy (>=1.12,<1.14)", "pytest-mypy"]

[[package]]
name = "shellingham"
version = "1.5.4"
description = "Tool to Detect Surrounding Shell"
optional = false
python-versions = ">=3.7"
files = [
    {file = "shellingham-1.5.4-py2.py3-none-any.whl", hash = "sha256:7ecfff8f2fd72616f7481040475a65b2bf8af90a56c89140852d1120324e8686"},
    {file = "shellingham-1.5.4.tar.gz", hash = "sha256:8dbca0739d487e5bd35ab3ca4b36e11c4078f3a234bfce294b0a0291363404de"},
]

[[package]]
name = "six"
version = "1.17.0"
description = "Python 2 and 3 compatibility utilities"
optional = false
python-versions = "!=3.0.*,!=3.1.*,!=3.2.*,>=2.7"
files = [
    {file = "six-1.17.0-py2.py3-none-any.whl", hash = "sha256:4721f391ed90541fddacab5acf947aa0d3dc7d27b2e1e8eda2be8970586c3274"},
    {file = "six-1.17.0.tar.gz", hash = "sha256:ff70335d468e7eb6ec65b95b99d3a2836546063f63acc5171de367e834932a81"},
]

[[package]]
name = "smmap"
version = "5.0.1"
description = "A pure Python implementation of a sliding window memory map manager"
optional = false
python-versions = ">=3.7"
files = [
    {file = "smmap-5.0.1-py3-none-any.whl", hash = "sha256:e6d8668fa5f93e706934a62d7b4db19c8d9eb8cf2adbb75ef1b675aa332b69da"},
    {file = "smmap-5.0.1.tar.gz", hash = "sha256:dceeb6c0028fdb6734471eb07c0cd2aae706ccaecab45965ee83f11c8d3b1f62"},
]

[[package]]
name = "sniffio"
version = "1.3.1"
description = "Sniff out which async library your code is running under"
optional = false
python-versions = ">=3.7"
files = [
    {file = "sniffio-1.3.1-py3-none-any.whl", hash = "sha256:2f6da418d1f1e0fddd844478f41680e794e6051915791a034ff65e5f100525a2"},
    {file = "sniffio-1.3.1.tar.gz", hash = "sha256:f4324edc670a0f49750a81b895f35c3adb843cca46f0530f79fc1babb23789dc"},
]

[[package]]
name = "sortedcontainers"
version = "2.4.0"
description = "Sorted Containers -- Sorted List, Sorted Dict, Sorted Set"
optional = false
python-versions = "*"
files = [
    {file = "sortedcontainers-2.4.0-py2.py3-none-any.whl", hash = "sha256:a163dcaede0f1c021485e957a39245190e74249897e2ae4b2aa38595db237ee0"},
    {file = "sortedcontainers-2.4.0.tar.gz", hash = "sha256:25caa5a06cc30b6b83d11423433f65d1f9d76c4c6a0c90e3379eaa43b9bfdb88"},
]

[[package]]
name = "soupsieve"
version = "2.6"
description = "A modern CSS selector implementation for Beautiful Soup."
optional = false
python-versions = ">=3.8"
files = [
    {file = "soupsieve-2.6-py3-none-any.whl", hash = "sha256:e72c4ff06e4fb6e4b5a9f0f55fe6e81514581fca1515028625d0f299c602ccc9"},
    {file = "soupsieve-2.6.tar.gz", hash = "sha256:e2e68417777af359ec65daac1057404a3c8a5455bb8abc36f1a9866ab1a51abb"},
]

[[package]]
name = "sqlalchemy"
version = "2.0.36"
description = "Database Abstraction Library"
optional = false
python-versions = ">=3.7"
files = [
    {file = "SQLAlchemy-2.0.36-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:59b8f3adb3971929a3e660337f5dacc5942c2cdb760afcabb2614ffbda9f9f72"},
    {file = "SQLAlchemy-2.0.36-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:37350015056a553e442ff672c2d20e6f4b6d0b2495691fa239d8aa18bb3bc908"},
    {file = "SQLAlchemy-2.0.36-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:8318f4776c85abc3f40ab185e388bee7a6ea99e7fa3a30686580b209eaa35c08"},
    {file = "SQLAlchemy-2.0.36-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:c245b1fbade9c35e5bd3b64270ab49ce990369018289ecfde3f9c318411aaa07"},
    {file = "SQLAlchemy-2.0.36-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:69f93723edbca7342624d09f6704e7126b152eaed3cdbb634cb657a54332a3c5"},
    {file = "SQLAlchemy-2.0.36-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:f9511d8dd4a6e9271d07d150fb2f81874a3c8c95e11ff9af3a2dfc35fe42ee44"},
    {file = "SQLAlchemy-2.0.36-cp310-cp310-win32.whl", hash = "sha256:c3f3631693003d8e585d4200730616b78fafd5a01ef8b698f6967da5c605b3fa"},
    {file = "SQLAlchemy-2.0.36-cp310-cp310-win_amd64.whl", hash = "sha256:a86bfab2ef46d63300c0f06936bd6e6c0105faa11d509083ba8f2f9d237fb5b5"},
    {file = "SQLAlchemy-2.0.36-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:fd3a55deef00f689ce931d4d1b23fa9f04c880a48ee97af488fd215cf24e2a6c"},
    {file = "SQLAlchemy-2.0.36-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:4f5e9cd989b45b73bd359f693b935364f7e1f79486e29015813c338450aa5a71"},
    {file = "SQLAlchemy-2.0.36-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:d0ddd9db6e59c44875211bc4c7953a9f6638b937b0a88ae6d09eb46cced54eff"},
    {file = "SQLAlchemy-2.0.36-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:2519f3a5d0517fc159afab1015e54bb81b4406c278749779be57a569d8d1bb0d"},
    {file = "SQLAlchemy-2.0.36-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:59b1ee96617135f6e1d6f275bbe988f419c5178016f3d41d3c0abb0c819f75bb"},
    {file = "SQLAlchemy-2.0.36-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:39769a115f730d683b0eb7b694db9789267bcd027326cccc3125e862eb03bfd8"},
    {file = "SQLAlchemy-2.0.36-cp311-cp311-win32.whl", hash = "sha256:66bffbad8d6271bb1cc2f9a4ea4f86f80fe5e2e3e501a5ae2a3dc6a76e604e6f"},
    {file = "SQLAlchemy-2.0.36-cp311-cp311-win_amd64.whl", hash = "sha256:23623166bfefe1487d81b698c423f8678e80df8b54614c2bf4b4cfcd7c711959"},
    {file = "SQLAlchemy-2.0.36-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:f7b64e6ec3f02c35647be6b4851008b26cff592a95ecb13b6788a54ef80bbdd4"},
    {file = "SQLAlchemy-2.0.36-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:46331b00096a6db1fdc052d55b101dbbfc99155a548e20a0e4a8e5e4d1362855"},
    {file = "SQLAlchemy-2.0.36-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:fdf3386a801ea5aba17c6410dd1dc8d39cf454ca2565541b5ac42a84e1e28f53"},
    {file = "SQLAlchemy-2.0.36-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:ac9dfa18ff2a67b09b372d5db8743c27966abf0e5344c555d86cc7199f7ad83a"},
    {file = "SQLAlchemy-2.0.36-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:90812a8933df713fdf748b355527e3af257a11e415b613dd794512461eb8a686"},
    {file = "SQLAlchemy-2.0.36-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:1bc330d9d29c7f06f003ab10e1eaced295e87940405afe1b110f2eb93a233588"},
    {file = "SQLAlchemy-2.0.36-cp312-cp312-win32.whl", hash = "sha256:79d2e78abc26d871875b419e1fd3c0bca31a1cb0043277d0d850014599626c2e"},
    {file = "SQLAlchemy-2.0.36-cp312-cp312-win_amd64.whl", hash = "sha256:b544ad1935a8541d177cb402948b94e871067656b3a0b9e91dbec136b06a2ff5"},
    {file = "SQLAlchemy-2.0.36-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:b5cc79df7f4bc3d11e4b542596c03826063092611e481fcf1c9dfee3c94355ef"},
    {file = "SQLAlchemy-2.0.36-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:3c01117dd36800f2ecaa238c65365b7b16497adc1522bf84906e5710ee9ba0e8"},
    {file = "SQLAlchemy-2.0.36-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:9bc633f4ee4b4c46e7adcb3a9b5ec083bf1d9a97c1d3854b92749d935de40b9b"},
    {file = "SQLAlchemy-2.0.36-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:9e46ed38affdfc95d2c958de328d037d87801cfcbea6d421000859e9789e61c2"},
    {file = "SQLAlchemy-2.0.36-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:b2985c0b06e989c043f1dc09d4fe89e1616aadd35392aea2844f0458a989eacf"},
    {file = "SQLAlchemy-2.0.36-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:4a121d62ebe7d26fec9155f83f8be5189ef1405f5973ea4874a26fab9f1e262c"},
    {file = "SQLAlchemy-2.0.36-cp313-cp313-win32.whl", hash = "sha256:0572f4bd6f94752167adfd7c1bed84f4b240ee6203a95e05d1e208d488d0d436"},
    {file = "SQLAlchemy-2.0.36-cp313-cp313-win_amd64.whl", hash = "sha256:8c78ac40bde930c60e0f78b3cd184c580f89456dd87fc08f9e3ee3ce8765ce88"},
    {file = "SQLAlchemy-2.0.36-cp37-cp37m-macosx_10_9_x86_64.whl", hash = "sha256:be9812b766cad94a25bc63bec11f88c4ad3629a0cec1cd5d4ba48dc23860486b"},
    {file = "SQLAlchemy-2.0.36-cp37-cp37m-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:50aae840ebbd6cdd41af1c14590e5741665e5272d2fee999306673a1bb1fdb4d"},
    {file = "SQLAlchemy-2.0.36-cp37-cp37m-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:4557e1f11c5f653ebfdd924f3f9d5ebfc718283b0b9beebaa5dd6b77ec290971"},
    {file = "SQLAlchemy-2.0.36-cp37-cp37m-musllinux_1_2_aarch64.whl", hash = "sha256:07b441f7d03b9a66299ce7ccf3ef2900abc81c0db434f42a5694a37bd73870f2"},
    {file = "SQLAlchemy-2.0.36-cp37-cp37m-musllinux_1_2_x86_64.whl", hash = "sha256:28120ef39c92c2dd60f2721af9328479516844c6b550b077ca450c7d7dc68575"},
    {file = "SQLAlchemy-2.0.36-cp37-cp37m-win32.whl", hash = "sha256:b81ee3d84803fd42d0b154cb6892ae57ea6b7c55d8359a02379965706c7efe6c"},
    {file = "SQLAlchemy-2.0.36-cp37-cp37m-win_amd64.whl", hash = "sha256:f942a799516184c855e1a32fbc7b29d7e571b52612647866d4ec1c3242578fcb"},
    {file = "SQLAlchemy-2.0.36-cp38-cp38-macosx_10_9_x86_64.whl", hash = "sha256:3d6718667da04294d7df1670d70eeddd414f313738d20a6f1d1f379e3139a545"},
    {file = "SQLAlchemy-2.0.36-cp38-cp38-macosx_11_0_arm64.whl", hash = "sha256:72c28b84b174ce8af8504ca28ae9347d317f9dba3999e5981a3cd441f3712e24"},
    {file = "SQLAlchemy-2.0.36-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:b11d0cfdd2b095e7b0686cf5fabeb9c67fae5b06d265d8180715b8cfa86522e3"},
    {file = "SQLAlchemy-2.0.36-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:e32092c47011d113dc01ab3e1d3ce9f006a47223b18422c5c0d150af13a00687"},
    {file = "SQLAlchemy-2.0.36-cp38-cp38-musllinux_1_2_aarch64.whl", hash = "sha256:6a440293d802d3011028e14e4226da1434b373cbaf4a4bbb63f845761a708346"},
    {file = "SQLAlchemy-2.0.36-cp38-cp38-musllinux_1_2_x86_64.whl", hash = "sha256:c54a1e53a0c308a8e8a7dffb59097bff7facda27c70c286f005327f21b2bd6b1"},
    {file = "SQLAlchemy-2.0.36-cp38-cp38-win32.whl", hash = "sha256:1e0d612a17581b6616ff03c8e3d5eff7452f34655c901f75d62bd86449d9750e"},
    {file = "SQLAlchemy-2.0.36-cp38-cp38-win_amd64.whl", hash = "sha256:8958b10490125124463095bbdadda5aa22ec799f91958e410438ad6c97a7b793"},
    {file = "SQLAlchemy-2.0.36-cp39-cp39-macosx_10_9_x86_64.whl", hash = "sha256:dc022184d3e5cacc9579e41805a681187650e170eb2fd70e28b86192a479dcaa"},
    {file = "SQLAlchemy-2.0.36-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:b817d41d692bf286abc181f8af476c4fbef3fd05e798777492618378448ee689"},
    {file = "SQLAlchemy-2.0.36-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:a4e46a888b54be23d03a89be510f24a7652fe6ff660787b96cd0e57a4ebcb46d"},
    {file = "SQLAlchemy-2.0.36-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:c4ae3005ed83f5967f961fd091f2f8c5329161f69ce8480aa8168b2d7fe37f06"},
    {file = "SQLAlchemy-2.0.36-cp39-cp39-musllinux_1_2_aarch64.whl", hash = "sha256:03e08af7a5f9386a43919eda9de33ffda16b44eb11f3b313e6822243770e9763"},
    {file = "SQLAlchemy-2.0.36-cp39-cp39-musllinux_1_2_x86_64.whl", hash = "sha256:3dbb986bad3ed5ceaf090200eba750b5245150bd97d3e67343a3cfed06feecf7"},
    {file = "SQLAlchemy-2.0.36-cp39-cp39-win32.whl", hash = "sha256:9fe53b404f24789b5ea9003fc25b9a3988feddebd7e7b369c8fac27ad6f52f28"},
    {file = "SQLAlchemy-2.0.36-cp39-cp39-win_amd64.whl", hash = "sha256:af148a33ff0349f53512a049c6406923e4e02bf2f26c5fb285f143faf4f0e46a"},
    {file = "SQLAlchemy-2.0.36-py3-none-any.whl", hash = "sha256:fddbe92b4760c6f5d48162aef14824add991aeda8ddadb3c31d56eb15ca69f8e"},
    {file = "sqlalchemy-2.0.36.tar.gz", hash = "sha256:7f2767680b6d2398aea7082e45a774b2b0767b5c8d8ffb9c8b683088ea9b29c5"},
]

[package.dependencies]
typing-extensions = ">=4.6.0"

[package.extras]
aiomysql = ["aiomysql (>=0.2.0)", "greenlet (!=0.4.17)"]
aioodbc = ["aioodbc", "greenlet (!=0.4.17)"]
aiosqlite = ["aiosqlite", "greenlet (!=0.4.17)", "typing_extensions (!=3.10.0.1)"]
asyncio = ["greenlet (!=0.4.17)"]
asyncmy = ["asyncmy (>=0.2.3,!=0.2.4,!=0.2.6)", "greenlet (!=0.4.17)"]
mariadb-connector = ["mariadb (>=1.0.1,!=1.1.2,!=1.1.5,!=1.1.10)"]
mssql = ["pyodbc"]
mssql-pymssql = ["pymssql"]
mssql-pyodbc = ["pyodbc"]
mypy = ["mypy (>=0.910)"]
mysql = ["mysqlclient (>=1.4.0)"]
mysql-connector = ["mysql-connector-python"]
oracle = ["cx_oracle (>=8)"]
oracle-oracledb = ["oracledb (>=1.0.1)"]
postgresql = ["psycopg2 (>=2.7)"]
postgresql-asyncpg = ["asyncpg", "greenlet (!=0.4.17)"]
postgresql-pg8000 = ["pg8000 (>=1.29.1)"]
postgresql-psycopg = ["psycopg (>=3.0.7)"]
postgresql-psycopg2binary = ["psycopg2-binary"]
postgresql-psycopg2cffi = ["psycopg2cffi"]
postgresql-psycopgbinary = ["psycopg[binary] (>=3.0.7)"]
pymysql = ["pymysql"]
sqlcipher = ["sqlcipher3_binary"]

[[package]]
name = "sqlparse"
version = "0.5.3"
description = "A non-validating SQL parser."
optional = false
python-versions = ">=3.8"
files = [
    {file = "sqlparse-0.5.3-py3-none-any.whl", hash = "sha256:cf2196ed3418f3ba5de6af7e82c694a9fbdbfecccdfc72e281548517081f16ca"},
    {file = "sqlparse-0.5.3.tar.gz", hash = "sha256:09f67787f56a0b16ecdbde1bfc7f5d9c3371ca683cfeaa8e6ff60b4807ec9272"},
]

[package.extras]
dev = ["build", "hatch"]
doc = ["sphinx"]

[[package]]
name = "stack-data"
version = "0.6.3"
description = "Extract data from python stack frames and tracebacks for informative displays"
optional = false
python-versions = "*"
files = [
    {file = "stack_data-0.6.3-py3-none-any.whl", hash = "sha256:d5558e0c25a4cb0853cddad3d77da9891a08cb85dd9f9f91b9f8cd66e511e695"},
    {file = "stack_data-0.6.3.tar.gz", hash = "sha256:836a778de4fec4dcd1dcd89ed8abff8a221f58308462e1c4aa2a3cf30148f0b9"},
]

[package.dependencies]
asttokens = ">=2.1.0"
executing = ">=1.2.0"
pure-eval = "*"

[package.extras]
tests = ["cython", "littleutils", "pygments", "pytest", "typeguard"]

[[package]]
name = "starlette"
version = "0.38.6"
description = "The little ASGI library that shines."
optional = false
python-versions = ">=3.8"
files = [
    {file = "starlette-0.38.6-py3-none-any.whl", hash = "sha256:4517a1409e2e73ee4951214ba012052b9e16f60e90d73cfb06192c19203bbb05"},
    {file = "starlette-0.38.6.tar.gz", hash = "sha256:863a1588f5574e70a821dadefb41e4881ea451a47a3cd1b4df359d4ffefe5ead"},
]

[package.dependencies]
anyio = ">=3.4.0,<5"

[package.extras]
full = ["httpx (>=0.22.0)", "itsdangerous", "jinja2", "python-multipart (>=0.0.7)", "pyyaml"]

[[package]]
name = "tenacity"
version = "9.0.0"
description = "Retry code until it succeeds"
optional = false
python-versions = ">=3.8"
files = [
    {file = "tenacity-9.0.0-py3-none-any.whl", hash = "sha256:93de0c98785b27fcf659856aa9f54bfbd399e29969b0621bc7f762bd441b4539"},
    {file = "tenacity-9.0.0.tar.gz", hash = "sha256:807f37ca97d62aa361264d497b0e31e92b8027044942bfa756160d908320d73b"},
]

[package.extras]
doc = ["reno", "sphinx"]
test = ["pytest", "tornado (>=4.5)", "typeguard"]

[[package]]
name = "termcolor"
version = "2.5.0"
description = "ANSI color formatting for output in terminal"
optional = false
python-versions = ">=3.9"
files = [
    {file = "termcolor-2.5.0-py3-none-any.whl", hash = "sha256:37b17b5fc1e604945c2642c872a3764b5d547a48009871aea3edd3afa180afb8"},
    {file = "termcolor-2.5.0.tar.gz", hash = "sha256:998d8d27da6d48442e8e1f016119076b690d962507531df4890fcd2db2ef8a6f"},
]

[package.extras]
tests = ["pytest", "pytest-cov"]

[[package]]
name = "terminado"
version = "0.18.1"
description = "Tornado websocket backend for the Xterm.js Javascript terminal emulator library."
optional = false
python-versions = ">=3.8"
files = [
    {file = "terminado-0.18.1-py3-none-any.whl", hash = "sha256:a4468e1b37bb318f8a86514f65814e1afc977cf29b3992a4500d9dd305dcceb0"},
    {file = "terminado-0.18.1.tar.gz", hash = "sha256:de09f2c4b85de4765f7714688fff57d3e75bad1f909b589fde880460c753fd2e"},
]

[package.dependencies]
ptyprocess = {version = "*", markers = "os_name != \"nt\""}
pywinpty = {version = ">=1.1.0", markers = "os_name == \"nt\""}
tornado = ">=6.1.0"

[package.extras]
docs = ["myst-parser", "pydata-sphinx-theme", "sphinx"]
test = ["pre-commit", "pytest (>=7.0)", "pytest-timeout"]
typing = ["mypy (>=1.6,<2.0)", "traitlets (>=5.11.1)"]

[[package]]
name = "tinycss2"
version = "1.4.0"
description = "A tiny CSS parser"
optional = false
python-versions = ">=3.8"
files = [
    {file = "tinycss2-1.4.0-py3-none-any.whl", hash = "sha256:3a49cf47b7675da0b15d0c6e1df8df4ebd96e9394bb905a5775adb0d884c5289"},
    {file = "tinycss2-1.4.0.tar.gz", hash = "sha256:10c0972f6fc0fbee87c3edb76549357415e94548c1ae10ebccdea16fb404a9b7"},
]

[package.dependencies]
webencodings = ">=0.4"

[package.extras]
doc = ["sphinx", "sphinx_rtd_theme"]
test = ["pytest", "ruff"]

[[package]]
name = "tomli"
version = "2.2.1"
description = "A lil' TOML parser"
optional = false
python-versions = ">=3.8"
files = [
    {file = "tomli-2.2.1-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:678e4fa69e4575eb77d103de3df8a895e1591b48e740211bd1067378c69e8249"},
    {file = "tomli-2.2.1-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:023aa114dd824ade0100497eb2318602af309e5a55595f76b626d6d9f3b7b0a6"},
    {file = "tomli-2.2.1-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:ece47d672db52ac607a3d9599a9d48dcb2f2f735c6c2d1f34130085bb12b112a"},
    {file = "tomli-2.2.1-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:6972ca9c9cc9f0acaa56a8ca1ff51e7af152a9f87fb64623e31d5c83700080ee"},
    {file = "tomli-2.2.1-cp311-cp311-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:c954d2250168d28797dd4e3ac5cf812a406cd5a92674ee4c8f123c889786aa8e"},
    {file = "tomli-2.2.1-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:8dd28b3e155b80f4d54beb40a441d366adcfe740969820caf156c019fb5c7ec4"},
    {file = "tomli-2.2.1-cp311-cp311-musllinux_1_2_i686.whl", hash = "sha256:e59e304978767a54663af13c07b3d1af22ddee3bb2fb0618ca1593e4f593a106"},
    {file = "tomli-2.2.1-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:33580bccab0338d00994d7f16f4c4ec25b776af3ffaac1ed74e0b3fc95e885a8"},
    {file = "tomli-2.2.1-cp311-cp311-win32.whl", hash = "sha256:465af0e0875402f1d226519c9904f37254b3045fc5084697cefb9bdde1ff99ff"},
    {file = "tomli-2.2.1-cp311-cp311-win_amd64.whl", hash = "sha256:2d0f2fdd22b02c6d81637a3c95f8cd77f995846af7414c5c4b8d0545afa1bc4b"},
    {file = "tomli-2.2.1-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:4a8f6e44de52d5e6c657c9fe83b562f5f4256d8ebbfe4ff922c495620a7f6cea"},
    {file = "tomli-2.2.1-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:8d57ca8095a641b8237d5b079147646153d22552f1c637fd3ba7f4b0b29167a8"},
    {file = "tomli-2.2.1-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:4e340144ad7ae1533cb897d406382b4b6fede8890a03738ff1683af800d54192"},
    {file = "tomli-2.2.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:db2b95f9de79181805df90bedc5a5ab4c165e6ec3fe99f970d0e302f384ad222"},
    {file = "tomli-2.2.1-cp312-cp312-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:40741994320b232529c802f8bc86da4e1aa9f413db394617b9a256ae0f9a7f77"},
    {file = "tomli-2.2.1-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:400e720fe168c0f8521520190686ef8ef033fb19fc493da09779e592861b78c6"},
    {file = "tomli-2.2.1-cp312-cp312-musllinux_1_2_i686.whl", hash = "sha256:02abe224de6ae62c19f090f68da4e27b10af2b93213d36cf44e6e1c5abd19fdd"},
    {file = "tomli-2.2.1-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:b82ebccc8c8a36f2094e969560a1b836758481f3dc360ce9a3277c65f374285e"},
    {file = "tomli-2.2.1-cp312-cp312-win32.whl", hash = "sha256:889f80ef92701b9dbb224e49ec87c645ce5df3fa2cc548664eb8a25e03127a98"},
    {file = "tomli-2.2.1-cp312-cp312-win_amd64.whl", hash = "sha256:7fc04e92e1d624a4a63c76474610238576942d6b8950a2d7f908a340494e67e4"},
    {file = "tomli-2.2.1-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:f4039b9cbc3048b2416cc57ab3bda989a6fcf9b36cf8937f01a6e731b64f80d7"},
    {file = "tomli-2.2.1-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:286f0ca2ffeeb5b9bd4fcc8d6c330534323ec51b2f52da063b11c502da16f30c"},
    {file = "tomli-2.2.1-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:a92ef1a44547e894e2a17d24e7557a5e85a9e1d0048b0b5e7541f76c5032cb13"},
    {file = "tomli-2.2.1-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:9316dc65bed1684c9a98ee68759ceaed29d229e985297003e494aa825ebb0281"},
    {file = "tomli-2.2.1-cp313-cp313-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:e85e99945e688e32d5a35c1ff38ed0b3f41f43fad8df0bdf79f72b2ba7bc5272"},
    {file = "tomli-2.2.1-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:ac065718db92ca818f8d6141b5f66369833d4a80a9d74435a268c52bdfa73140"},
    {file = "tomli-2.2.1-cp313-cp313-musllinux_1_2_i686.whl", hash = "sha256:d920f33822747519673ee656a4b6ac33e382eca9d331c87770faa3eef562aeb2"},
    {file = "tomli-2.2.1-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:a198f10c4d1b1375d7687bc25294306e551bf1abfa4eace6650070a5c1ae2744"},
    {file = "tomli-2.2.1-cp313-cp313-win32.whl", hash = "sha256:d3f5614314d758649ab2ab3a62d4f2004c825922f9e370b29416484086b264ec"},
    {file = "tomli-2.2.1-cp313-cp313-win_amd64.whl", hash = "sha256:a38aa0308e754b0e3c67e344754dff64999ff9b513e691d0e786265c93583c69"},
    {file = "tomli-2.2.1-py3-none-any.whl", hash = "sha256:cb55c73c5f4408779d0cf3eef9f762b9c9f147a77de7b258bef0a5628adc85cc"},
    {file = "tomli-2.2.1.tar.gz", hash = "sha256:cd45e1dc79c835ce60f7404ec8119f2eb06d38b1deba146f07ced3bbc44505ff"},
]

[[package]]
name = "tomlkit"
version = "0.13.2"
description = "Style preserving TOML library"
optional = false
python-versions = ">=3.8"
files = [
    {file = "tomlkit-0.13.2-py3-none-any.whl", hash = "sha256:7a974427f6e119197f670fbbbeae7bef749a6c14e793db934baefc1b5f03efde"},
    {file = "tomlkit-0.13.2.tar.gz", hash = "sha256:fff5fe59a87295b278abd31bec92c15d9bc4a06885ab12bcea52c71119392e79"},
]

[[package]]
name = "tornado"
version = "6.4.2"
description = "Tornado is a Python web framework and asynchronous networking library, originally developed at FriendFeed."
optional = false
python-versions = ">=3.8"
files = [
    {file = "tornado-6.4.2-cp38-abi3-macosx_10_9_universal2.whl", hash = "sha256:e828cce1123e9e44ae2a50a9de3055497ab1d0aeb440c5ac23064d9e44880da1"},
    {file = "tornado-6.4.2-cp38-abi3-macosx_10_9_x86_64.whl", hash = "sha256:072ce12ada169c5b00b7d92a99ba089447ccc993ea2143c9ede887e0937aa803"},
    {file = "tornado-6.4.2-cp38-abi3-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:1a017d239bd1bb0919f72af256a970624241f070496635784d9bf0db640d3fec"},
    {file = "tornado-6.4.2-cp38-abi3-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:c36e62ce8f63409301537222faffcef7dfc5284f27eec227389f2ad11b09d946"},
    {file = "tornado-6.4.2-cp38-abi3-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:bca9eb02196e789c9cb5c3c7c0f04fb447dc2adffd95265b2c7223a8a615ccbf"},
    {file = "tornado-6.4.2-cp38-abi3-musllinux_1_2_aarch64.whl", hash = "sha256:304463bd0772442ff4d0f5149c6f1c2135a1fae045adf070821c6cdc76980634"},
    {file = "tornado-6.4.2-cp38-abi3-musllinux_1_2_i686.whl", hash = "sha256:c82c46813ba483a385ab2a99caeaedf92585a1f90defb5693351fa7e4ea0bf73"},
    {file = "tornado-6.4.2-cp38-abi3-musllinux_1_2_x86_64.whl", hash = "sha256:932d195ca9015956fa502c6b56af9eb06106140d844a335590c1ec7f5277d10c"},
    {file = "tornado-6.4.2-cp38-abi3-win32.whl", hash = "sha256:2876cef82e6c5978fde1e0d5b1f919d756968d5b4282418f3146b79b58556482"},
    {file = "tornado-6.4.2-cp38-abi3-win_amd64.whl", hash = "sha256:908b71bf3ff37d81073356a5fadcc660eb10c1476ee6e2725588626ce7e5ca38"},
    {file = "tornado-6.4.2.tar.gz", hash = "sha256:92bad5b4746e9879fd7bf1eb21dce4e3fc5128d71601f80005afa39237ad620b"},
]

[[package]]
name = "traitlets"
version = "5.14.3"
description = "Traitlets Python configuration system"
optional = false
python-versions = ">=3.8"
files = [
    {file = "traitlets-5.14.3-py3-none-any.whl", hash = "sha256:b74e89e397b1ed28cc831db7aea759ba6640cb3de13090ca145426688ff1ac4f"},
    {file = "traitlets-5.14.3.tar.gz", hash = "sha256:9ed0579d3502c94b4b3732ac120375cda96f923114522847de4b3bb98b96b6b7"},
]

[package.extras]
docs = ["myst-parser", "pydata-sphinx-theme", "sphinx"]
test = ["argcomplete (>=3.0.3)", "mypy (>=1.7.0)", "pre-commit", "pytest (>=7.0,<8.2)", "pytest-mock", "pytest-mypy-testing"]

[[package]]
name = "typer"
version = "0.12.5"
description = "Typer, build great CLIs. Easy to code. Based on Python type hints."
optional = false
python-versions = ">=3.7"
files = [
    {file = "typer-0.12.5-py3-none-any.whl", hash = "sha256:62fe4e471711b147e3365034133904df3e235698399bc4de2b36c8579298d52b"},
    {file = "typer-0.12.5.tar.gz", hash = "sha256:f592f089bedcc8ec1b974125d64851029c3b1af145f04aca64d69410f0c9b722"},
]

[package.dependencies]
click = ">=8.0.0"
rich = ">=10.11.0"
shellingham = ">=1.3.0"
typing-extensions = ">=3.7.4.3"

[[package]]
name = "types-python-dateutil"
version = "2.9.0.20241206"
description = "Typing stubs for python-dateutil"
optional = false
python-versions = ">=3.8"
files = [
    {file = "types_python_dateutil-2.9.0.20241206-py3-none-any.whl", hash = "sha256:e248a4bc70a486d3e3ec84d0dc30eec3a5f979d6e7ee4123ae043eedbb987f53"},
    {file = "types_python_dateutil-2.9.0.20241206.tar.gz", hash = "sha256:18f493414c26ffba692a72369fea7a154c502646301ebfe3d56a04b3767284cb"},
]

[[package]]
name = "types-pyyaml"
version = "6.0.12.20241221"
description = "Typing stubs for PyYAML"
optional = false
python-versions = ">=3.8"
files = [
    {file = "types_PyYAML-6.0.12.20241221-py3-none-any.whl", hash = "sha256:0657a4ff8411a030a2116a196e8e008ea679696b5b1a8e1a6aa8ebb737b34688"},
    {file = "types_pyyaml-6.0.12.20241221.tar.gz", hash = "sha256:4f149aa893ff6a46889a30af4c794b23833014c469cc57cbc3ad77498a58996f"},
]

[[package]]
name = "typing-extensions"
version = "4.12.2"
description = "Backported and Experimental Type Hints for Python 3.8+"
optional = false
python-versions = ">=3.8"
files = [
    {file = "typing_extensions-4.12.2-py3-none-any.whl", hash = "sha256:04e5ca0351e0f3f85c6853954072df659d0d13fac324d0072316b67d7794700d"},
    {file = "typing_extensions-4.12.2.tar.gz", hash = "sha256:1a7ead55c7e559dd4dee8856e3a88b41225abfe1ce8df57b7c13915fe121ffb8"},
]

[[package]]
name = "tzdata"
version = "2024.2"
description = "Provider of IANA time zone data"
optional = false
python-versions = ">=2"
files = [
    {file = "tzdata-2024.2-py2.py3-none-any.whl", hash = "sha256:a48093786cdcde33cad18c2555e8532f34422074448fbc874186f0abd79565cd"},
    {file = "tzdata-2024.2.tar.gz", hash = "sha256:7d85cc416e9382e69095b7bdf4afd9e3880418a2413feec7069d533d6b4e31cc"},
]

[[package]]
name = "ujson"
version = "5.10.0"
description = "Ultra fast JSON encoder and decoder for Python"
optional = false
python-versions = ">=3.8"
files = [
    {file = "ujson-5.10.0-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:2601aa9ecdbee1118a1c2065323bda35e2c5a2cf0797ef4522d485f9d3ef65bd"},
    {file = "ujson-5.10.0-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:348898dd702fc1c4f1051bc3aacbf894caa0927fe2c53e68679c073375f732cf"},
    {file = "ujson-5.10.0-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:22cffecf73391e8abd65ef5f4e4dd523162a3399d5e84faa6aebbf9583df86d6"},
    {file = "ujson-5.10.0-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:26b0e2d2366543c1bb4fbd457446f00b0187a2bddf93148ac2da07a53fe51569"},
    {file = "ujson-5.10.0-cp310-cp310-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:caf270c6dba1be7a41125cd1e4fc7ba384bf564650beef0df2dd21a00b7f5770"},
    {file = "ujson-5.10.0-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:a245d59f2ffe750446292b0094244df163c3dc96b3ce152a2c837a44e7cda9d1"},
    {file = "ujson-5.10.0-cp310-cp310-musllinux_1_2_i686.whl", hash = "sha256:94a87f6e151c5f483d7d54ceef83b45d3a9cca7a9cb453dbdbb3f5a6f64033f5"},
    {file = "ujson-5.10.0-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:29b443c4c0a113bcbb792c88bea67b675c7ca3ca80c3474784e08bba01c18d51"},
    {file = "ujson-5.10.0-cp310-cp310-win32.whl", hash = "sha256:c18610b9ccd2874950faf474692deee4223a994251bc0a083c114671b64e6518"},
    {file = "ujson-5.10.0-cp310-cp310-win_amd64.whl", hash = "sha256:924f7318c31874d6bb44d9ee1900167ca32aa9b69389b98ecbde34c1698a250f"},
    {file = "ujson-5.10.0-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:a5b366812c90e69d0f379a53648be10a5db38f9d4ad212b60af00bd4048d0f00"},
    {file = "ujson-5.10.0-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:502bf475781e8167f0f9d0e41cd32879d120a524b22358e7f205294224c71126"},
    {file = "ujson-5.10.0-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:5b91b5d0d9d283e085e821651184a647699430705b15bf274c7896f23fe9c9d8"},
    {file = "ujson-5.10.0-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:129e39af3a6d85b9c26d5577169c21d53821d8cf68e079060602e861c6e5da1b"},
    {file = "ujson-5.10.0-cp311-cp311-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:f77b74475c462cb8b88680471193064d3e715c7c6074b1c8c412cb526466efe9"},
    {file = "ujson-5.10.0-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:7ec0ca8c415e81aa4123501fee7f761abf4b7f386aad348501a26940beb1860f"},
    {file = "ujson-5.10.0-cp311-cp311-musllinux_1_2_i686.whl", hash = "sha256:ab13a2a9e0b2865a6c6db9271f4b46af1c7476bfd51af1f64585e919b7c07fd4"},
    {file = "ujson-5.10.0-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:57aaf98b92d72fc70886b5a0e1a1ca52c2320377360341715dd3933a18e827b1"},
    {file = "ujson-5.10.0-cp311-cp311-win32.whl", hash = "sha256:2987713a490ceb27edff77fb184ed09acdc565db700ee852823c3dc3cffe455f"},
    {file = "ujson-5.10.0-cp311-cp311-win_amd64.whl", hash = "sha256:f00ea7e00447918ee0eff2422c4add4c5752b1b60e88fcb3c067d4a21049a720"},
    {file = "ujson-5.10.0-cp312-cp312-macosx_10_9_x86_64.whl", hash = "sha256:98ba15d8cbc481ce55695beee9f063189dce91a4b08bc1d03e7f0152cd4bbdd5"},
    {file = "ujson-5.10.0-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:a9d2edbf1556e4f56e50fab7d8ff993dbad7f54bac68eacdd27a8f55f433578e"},
    {file = "ujson-5.10.0-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:6627029ae4f52d0e1a2451768c2c37c0c814ffc04f796eb36244cf16b8e57043"},
    {file = "ujson-5.10.0-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:f8ccb77b3e40b151e20519c6ae6d89bfe3f4c14e8e210d910287f778368bb3d1"},
    {file = "ujson-5.10.0-cp312-cp312-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:f3caf9cd64abfeb11a3b661329085c5e167abbe15256b3b68cb5d914ba7396f3"},
    {file = "ujson-5.10.0-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:6e32abdce572e3a8c3d02c886c704a38a1b015a1fb858004e03d20ca7cecbb21"},
    {file = "ujson-5.10.0-cp312-cp312-musllinux_1_2_i686.whl", hash = "sha256:a65b6af4d903103ee7b6f4f5b85f1bfd0c90ba4eeac6421aae436c9988aa64a2"},
    {file = "ujson-5.10.0-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:604a046d966457b6cdcacc5aa2ec5314f0e8c42bae52842c1e6fa02ea4bda42e"},
    {file = "ujson-5.10.0-cp312-cp312-win32.whl", hash = "sha256:6dea1c8b4fc921bf78a8ff00bbd2bfe166345f5536c510671bccececb187c80e"},
    {file = "ujson-5.10.0-cp312-cp312-win_amd64.whl", hash = "sha256:38665e7d8290188b1e0d57d584eb8110951a9591363316dd41cf8686ab1d0abc"},
    {file = "ujson-5.10.0-cp313-cp313-macosx_10_9_x86_64.whl", hash = "sha256:618efd84dc1acbd6bff8eaa736bb6c074bfa8b8a98f55b61c38d4ca2c1f7f287"},
    {file = "ujson-5.10.0-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:38d5d36b4aedfe81dfe251f76c0467399d575d1395a1755de391e58985ab1c2e"},
    {file = "ujson-5.10.0-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:67079b1f9fb29ed9a2914acf4ef6c02844b3153913eb735d4bf287ee1db6e557"},
    {file = "ujson-5.10.0-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:d7d0e0ceeb8fe2468c70ec0c37b439dd554e2aa539a8a56365fd761edb418988"},
    {file = "ujson-5.10.0-cp313-cp313-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:59e02cd37bc7c44d587a0ba45347cc815fb7a5fe48de16bf05caa5f7d0d2e816"},
    {file = "ujson-5.10.0-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:2a890b706b64e0065f02577bf6d8ca3b66c11a5e81fb75d757233a38c07a1f20"},
    {file = "ujson-5.10.0-cp313-cp313-musllinux_1_2_i686.whl", hash = "sha256:621e34b4632c740ecb491efc7f1fcb4f74b48ddb55e65221995e74e2d00bbff0"},
    {file = "ujson-5.10.0-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:b9500e61fce0cfc86168b248104e954fead61f9be213087153d272e817ec7b4f"},
    {file = "ujson-5.10.0-cp313-cp313-win32.whl", hash = "sha256:4c4fc16f11ac1612f05b6f5781b384716719547e142cfd67b65d035bd85af165"},
    {file = "ujson-5.10.0-cp313-cp313-win_amd64.whl", hash = "sha256:4573fd1695932d4f619928fd09d5d03d917274381649ade4328091ceca175539"},
    {file = "ujson-5.10.0-cp38-cp38-macosx_10_9_x86_64.whl", hash = "sha256:a984a3131da7f07563057db1c3020b1350a3e27a8ec46ccbfbf21e5928a43050"},
    {file = "ujson-5.10.0-cp38-cp38-macosx_11_0_arm64.whl", hash = "sha256:73814cd1b9db6fc3270e9d8fe3b19f9f89e78ee9d71e8bd6c9a626aeaeaf16bd"},
    {file = "ujson-5.10.0-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:61e1591ed9376e5eddda202ec229eddc56c612b61ac6ad07f96b91460bb6c2fb"},
    {file = "ujson-5.10.0-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:d2c75269f8205b2690db4572a4a36fe47cd1338e4368bc73a7a0e48789e2e35a"},
    {file = "ujson-5.10.0-cp38-cp38-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:7223f41e5bf1f919cd8d073e35b229295aa8e0f7b5de07ed1c8fddac63a6bc5d"},
    {file = "ujson-5.10.0-cp38-cp38-musllinux_1_2_aarch64.whl", hash = "sha256:d4dc2fd6b3067c0782e7002ac3b38cf48608ee6366ff176bbd02cf969c9c20fe"},
    {file = "ujson-5.10.0-cp38-cp38-musllinux_1_2_i686.whl", hash = "sha256:232cc85f8ee3c454c115455195a205074a56ff42608fd6b942aa4c378ac14dd7"},
    {file = "ujson-5.10.0-cp38-cp38-musllinux_1_2_x86_64.whl", hash = "sha256:cc6139531f13148055d691e442e4bc6601f6dba1e6d521b1585d4788ab0bfad4"},
    {file = "ujson-5.10.0-cp38-cp38-win32.whl", hash = "sha256:e7ce306a42b6b93ca47ac4a3b96683ca554f6d35dd8adc5acfcd55096c8dfcb8"},
    {file = "ujson-5.10.0-cp38-cp38-win_amd64.whl", hash = "sha256:e82d4bb2138ab05e18f089a83b6564fee28048771eb63cdecf4b9b549de8a2cc"},
    {file = "ujson-5.10.0-cp39-cp39-macosx_10_9_x86_64.whl", hash = "sha256:dfef2814c6b3291c3c5f10065f745a1307d86019dbd7ea50e83504950136ed5b"},
    {file = "ujson-5.10.0-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:4734ee0745d5928d0ba3a213647f1c4a74a2a28edc6d27b2d6d5bd9fa4319e27"},
    {file = "ujson-5.10.0-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:d47ebb01bd865fdea43da56254a3930a413f0c5590372a1241514abae8aa7c76"},
    {file = "ujson-5.10.0-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:dee5e97c2496874acbf1d3e37b521dd1f307349ed955e62d1d2f05382bc36dd5"},
    {file = "ujson-5.10.0-cp39-cp39-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:7490655a2272a2d0b072ef16b0b58ee462f4973a8f6bbe64917ce5e0a256f9c0"},
    {file = "ujson-5.10.0-cp39-cp39-musllinux_1_2_aarch64.whl", hash = "sha256:ba17799fcddaddf5c1f75a4ba3fd6441f6a4f1e9173f8a786b42450851bd74f1"},
    {file = "ujson-5.10.0-cp39-cp39-musllinux_1_2_i686.whl", hash = "sha256:2aff2985cef314f21d0fecc56027505804bc78802c0121343874741650a4d3d1"},
    {file = "ujson-5.10.0-cp39-cp39-musllinux_1_2_x86_64.whl", hash = "sha256:ad88ac75c432674d05b61184178635d44901eb749786c8eb08c102330e6e8996"},
    {file = "ujson-5.10.0-cp39-cp39-win32.whl", hash = "sha256:2544912a71da4ff8c4f7ab5606f947d7299971bdd25a45e008e467ca638d13c9"},
    {file = "ujson-5.10.0-cp39-cp39-win_amd64.whl", hash = "sha256:3ff201d62b1b177a46f113bb43ad300b424b7847f9c5d38b1b4ad8f75d4a282a"},
    {file = "ujson-5.10.0-pp310-pypy310_pp73-macosx_10_9_x86_64.whl", hash = "sha256:5b6fee72fa77dc172a28f21693f64d93166534c263adb3f96c413ccc85ef6e64"},
    {file = "ujson-5.10.0-pp310-pypy310_pp73-macosx_11_0_arm64.whl", hash = "sha256:61d0af13a9af01d9f26d2331ce49bb5ac1fb9c814964018ac8df605b5422dcb3"},
    {file = "ujson-5.10.0-pp310-pypy310_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:ecb24f0bdd899d368b715c9e6664166cf694d1e57be73f17759573a6986dd95a"},
    {file = "ujson-5.10.0-pp310-pypy310_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:fbd8fd427f57a03cff3ad6574b5e299131585d9727c8c366da4624a9069ed746"},
    {file = "ujson-5.10.0-pp310-pypy310_pp73-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:beeaf1c48e32f07d8820c705ff8e645f8afa690cca1544adba4ebfa067efdc88"},
    {file = "ujson-5.10.0-pp310-pypy310_pp73-win_amd64.whl", hash = "sha256:baed37ea46d756aca2955e99525cc02d9181de67f25515c468856c38d52b5f3b"},
    {file = "ujson-5.10.0-pp38-pypy38_pp73-macosx_10_9_x86_64.whl", hash = "sha256:7663960f08cd5a2bb152f5ee3992e1af7690a64c0e26d31ba7b3ff5b2ee66337"},
    {file = "ujson-5.10.0-pp38-pypy38_pp73-macosx_11_0_arm64.whl", hash = "sha256:d8640fb4072d36b08e95a3a380ba65779d356b2fee8696afeb7794cf0902d0a1"},
    {file = "ujson-5.10.0-pp38-pypy38_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:78778a3aa7aafb11e7ddca4e29f46bc5139131037ad628cc10936764282d6753"},
    {file = "ujson-5.10.0-pp38-pypy38_pp73-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:b0111b27f2d5c820e7f2dbad7d48e3338c824e7ac4d2a12da3dc6061cc39c8e6"},
    {file = "ujson-5.10.0-pp38-pypy38_pp73-win_amd64.whl", hash = "sha256:c66962ca7565605b355a9ed478292da628b8f18c0f2793021ca4425abf8b01e5"},
    {file = "ujson-5.10.0-pp39-pypy39_pp73-macosx_10_9_x86_64.whl", hash = "sha256:ba43cc34cce49cf2d4bc76401a754a81202d8aa926d0e2b79f0ee258cb15d3a4"},
    {file = "ujson-5.10.0-pp39-pypy39_pp73-macosx_11_0_arm64.whl", hash = "sha256:ac56eb983edce27e7f51d05bc8dd820586c6e6be1c5216a6809b0c668bb312b8"},
    {file = "ujson-5.10.0-pp39-pypy39_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:f44bd4b23a0e723bf8b10628288c2c7c335161d6840013d4d5de20e48551773b"},
    {file = "ujson-5.10.0-pp39-pypy39_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:7c10f4654e5326ec14a46bcdeb2b685d4ada6911050aa8baaf3501e57024b804"},
    {file = "ujson-5.10.0-pp39-pypy39_pp73-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:0de4971a89a762398006e844ae394bd46991f7c385d7a6a3b93ba229e6dac17e"},
    {file = "ujson-5.10.0-pp39-pypy39_pp73-win_amd64.whl", hash = "sha256:e1402f0564a97d2a52310ae10a64d25bcef94f8dd643fcf5d310219d915484f7"},
    {file = "ujson-5.10.0.tar.gz", hash = "sha256:b3cd8f3c5d8c7738257f1018880444f7b7d9b66232c64649f562d7ba86ad4bc1"},
]

[[package]]
name = "uri-template"
version = "1.3.0"
description = "RFC 6570 URI Template Processor"
optional = false
python-versions = ">=3.7"
files = [
    {file = "uri-template-1.3.0.tar.gz", hash = "sha256:0e00f8eb65e18c7de20d595a14336e9f337ead580c70934141624b6d1ffdacc7"},
    {file = "uri_template-1.3.0-py3-none-any.whl", hash = "sha256:a44a133ea12d44a0c0f06d7d42a52d71282e77e2f937d8abd5655b8d56fc1363"},
]

[package.extras]
dev = ["flake8", "flake8-annotations", "flake8-bandit", "flake8-bugbear", "flake8-commas", "flake8-comprehensions", "flake8-continuation", "flake8-datetimez", "flake8-docstrings", "flake8-import-order", "flake8-literal", "flake8-modern-annotations", "flake8-noqa", "flake8-pyproject", "flake8-requirements", "flake8-typechecking-import", "flake8-use-fstring", "mypy", "pep8-naming", "types-PyYAML"]

[[package]]
name = "urllib3"
version = "2.3.0"
description = "HTTP library with thread-safe connection pooling, file post, and more."
optional = false
python-versions = ">=3.9"
files = [
    {file = "urllib3-2.3.0-py3-none-any.whl", hash = "sha256:1cee9ad369867bfdbbb48b7dd50374c0967a0bb7710050facf0dd6911440e3df"},
    {file = "urllib3-2.3.0.tar.gz", hash = "sha256:f8c5449b3cf0861679ce7e0503c7b44b5ec981bec0d1d3795a07f1ba96f0204d"},
]

[package.extras]
brotli = ["brotli (>=1.0.9)", "brotlicffi (>=0.8.0)"]
h2 = ["h2 (>=4,<5)"]
socks = ["pysocks (>=1.5.6,!=1.5.7,<2.0)"]
zstd = ["zstandard (>=0.18.0)"]

[[package]]
name = "uvicorn"
version = "0.34.0"
description = "The lightning-fast ASGI server."
optional = false
python-versions = ">=3.9"
files = [
    {file = "uvicorn-0.34.0-py3-none-any.whl", hash = "sha256:023dc038422502fa28a09c7a30bf2b6991512da7dcdb8fd35fe57cfc154126f4"},
    {file = "uvicorn-0.34.0.tar.gz", hash = "sha256:404051050cd7e905de2c9a7e61790943440b3416f49cb409f965d9dcd0fa73e9"},
]

[package.dependencies]
click = ">=7.0"
colorama = {version = ">=0.4", optional = true, markers = "sys_platform == \"win32\" and extra == \"standard\""}
h11 = ">=0.8"
httptools = {version = ">=0.6.3", optional = true, markers = "extra == \"standard\""}
python-dotenv = {version = ">=0.13", optional = true, markers = "extra == \"standard\""}
pyyaml = {version = ">=5.1", optional = true, markers = "extra == \"standard\""}
uvloop = {version = ">=0.14.0,<0.15.0 || >0.15.0,<0.15.1 || >0.15.1", optional = true, markers = "(sys_platform != \"win32\" and sys_platform != \"cygwin\") and platform_python_implementation != \"PyPy\" and extra == \"standard\""}
watchfiles = {version = ">=0.13", optional = true, markers = "extra == \"standard\""}
websockets = {version = ">=10.4", optional = true, markers = "extra == \"standard\""}

[package.extras]
standard = ["colorama (>=0.4)", "httptools (>=0.6.3)", "python-dotenv (>=0.13)", "pyyaml (>=5.1)", "uvloop (>=0.14.0,!=0.15.0,!=0.15.1)", "watchfiles (>=0.13)", "websockets (>=10.4)"]

[[package]]
name = "uvloop"
version = "0.21.0"
description = "Fast implementation of asyncio event loop on top of libuv"
optional = false
python-versions = ">=3.8.0"
files = [
    {file = "uvloop-0.21.0-cp310-cp310-macosx_10_9_universal2.whl", hash = "sha256:ec7e6b09a6fdded42403182ab6b832b71f4edaf7f37a9a0e371a01db5f0cb45f"},
    {file = "uvloop-0.21.0-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:196274f2adb9689a289ad7d65700d37df0c0930fd8e4e743fa4834e850d7719d"},
    {file = "uvloop-0.21.0-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:f38b2e090258d051d68a5b14d1da7203a3c3677321cf32a95a6f4db4dd8b6f26"},
    {file = "uvloop-0.21.0-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:87c43e0f13022b998eb9b973b5e97200c8b90823454d4bc06ab33829e09fb9bb"},
    {file = "uvloop-0.21.0-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:10d66943def5fcb6e7b37310eb6b5639fd2ccbc38df1177262b0640c3ca68c1f"},
    {file = "uvloop-0.21.0-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:67dd654b8ca23aed0a8e99010b4c34aca62f4b7fce88f39d452ed7622c94845c"},
    {file = "uvloop-0.21.0-cp311-cp311-macosx_10_9_universal2.whl", hash = "sha256:c0f3fa6200b3108919f8bdabb9a7f87f20e7097ea3c543754cabc7d717d95cf8"},
    {file = "uvloop-0.21.0-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:0878c2640cf341b269b7e128b1a5fed890adc4455513ca710d77d5e93aa6d6a0"},
    {file = "uvloop-0.21.0-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:b9fb766bb57b7388745d8bcc53a359b116b8a04c83a2288069809d2b3466c37e"},
    {file = "uvloop-0.21.0-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:8a375441696e2eda1c43c44ccb66e04d61ceeffcd76e4929e527b7fa401b90fb"},
    {file = "uvloop-0.21.0-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:baa0e6291d91649c6ba4ed4b2f982f9fa165b5bbd50a9e203c416a2797bab3c6"},
    {file = "uvloop-0.21.0-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:4509360fcc4c3bd2c70d87573ad472de40c13387f5fda8cb58350a1d7475e58d"},
    {file = "uvloop-0.21.0-cp312-cp312-macosx_10_13_universal2.whl", hash = "sha256:359ec2c888397b9e592a889c4d72ba3d6befba8b2bb01743f72fffbde663b59c"},
    {file = "uvloop-0.21.0-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:f7089d2dc73179ce5ac255bdf37c236a9f914b264825fdaacaded6990a7fb4c2"},
    {file = "uvloop-0.21.0-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:baa4dcdbd9ae0a372f2167a207cd98c9f9a1ea1188a8a526431eef2f8116cc8d"},
    {file = "uvloop-0.21.0-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:86975dca1c773a2c9864f4c52c5a55631038e387b47eaf56210f873887b6c8dc"},
    {file = "uvloop-0.21.0-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:461d9ae6660fbbafedd07559c6a2e57cd553b34b0065b6550685f6653a98c1cb"},
    {file = "uvloop-0.21.0-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:183aef7c8730e54c9a3ee3227464daed66e37ba13040bb3f350bc2ddc040f22f"},
    {file = "uvloop-0.21.0-cp313-cp313-macosx_10_13_universal2.whl", hash = "sha256:bfd55dfcc2a512316e65f16e503e9e450cab148ef11df4e4e679b5e8253a5281"},
    {file = "uvloop-0.21.0-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:787ae31ad8a2856fc4e7c095341cccc7209bd657d0e71ad0dc2ea83c4a6fa8af"},
    {file = "uvloop-0.21.0-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:5ee4d4ef48036ff6e5cfffb09dd192c7a5027153948d85b8da7ff705065bacc6"},
    {file = "uvloop-0.21.0-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:f3df876acd7ec037a3d005b3ab85a7e4110422e4d9c1571d4fc89b0fc41b6816"},
    {file = "uvloop-0.21.0-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:bd53ecc9a0f3d87ab847503c2e1552b690362e005ab54e8a48ba97da3924c0dc"},
    {file = "uvloop-0.21.0-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:a5c39f217ab3c663dc699c04cbd50c13813e31d917642d459fdcec07555cc553"},
    {file = "uvloop-0.21.0-cp38-cp38-macosx_10_9_universal2.whl", hash = "sha256:17df489689befc72c39a08359efac29bbee8eee5209650d4b9f34df73d22e414"},
    {file = "uvloop-0.21.0-cp38-cp38-macosx_10_9_x86_64.whl", hash = "sha256:bc09f0ff191e61c2d592a752423c767b4ebb2986daa9ed62908e2b1b9a9ae206"},
    {file = "uvloop-0.21.0-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:f0ce1b49560b1d2d8a2977e3ba4afb2414fb46b86a1b64056bc4ab929efdafbe"},
    {file = "uvloop-0.21.0-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:e678ad6fe52af2c58d2ae3c73dc85524ba8abe637f134bf3564ed07f555c5e79"},
    {file = "uvloop-0.21.0-cp38-cp38-musllinux_1_2_aarch64.whl", hash = "sha256:460def4412e473896ef179a1671b40c039c7012184b627898eea5072ef6f017a"},
    {file = "uvloop-0.21.0-cp38-cp38-musllinux_1_2_x86_64.whl", hash = "sha256:10da8046cc4a8f12c91a1c39d1dd1585c41162a15caaef165c2174db9ef18bdc"},
    {file = "uvloop-0.21.0-cp39-cp39-macosx_10_9_universal2.whl", hash = "sha256:c097078b8031190c934ed0ebfee8cc5f9ba9642e6eb88322b9958b649750f72b"},
    {file = "uvloop-0.21.0-cp39-cp39-macosx_10_9_x86_64.whl", hash = "sha256:46923b0b5ee7fc0020bef24afe7836cb068f5050ca04caf6b487c513dc1a20b2"},
    {file = "uvloop-0.21.0-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:53e420a3afe22cdcf2a0f4846e377d16e718bc70103d7088a4f7623567ba5fb0"},
    {file = "uvloop-0.21.0-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:88cb67cdbc0e483da00af0b2c3cdad4b7c61ceb1ee0f33fe00e09c81e3a6cb75"},
    {file = "uvloop-0.21.0-cp39-cp39-musllinux_1_2_aarch64.whl", hash = "sha256:221f4f2a1f46032b403bf3be628011caf75428ee3cc204a22addf96f586b19fd"},
    {file = "uvloop-0.21.0-cp39-cp39-musllinux_1_2_x86_64.whl", hash = "sha256:2d1f581393673ce119355d56da84fe1dd9d2bb8b3d13ce792524e1607139feff"},
    {file = "uvloop-0.21.0.tar.gz", hash = "sha256:3bf12b0fda68447806a7ad847bfa591613177275d35b6724b1ee573faa3704e3"},
]

[package.extras]
dev = ["Cython (>=3.0,<4.0)", "setuptools (>=60)"]
docs = ["Sphinx (>=4.1.2,<4.2.0)", "sphinx-rtd-theme (>=0.5.2,<0.6.0)", "sphinxcontrib-asyncio (>=0.3.0,<0.4.0)"]
test = ["aiohttp (>=3.10.5)", "flake8 (>=5.0,<6.0)", "mypy (>=0.800)", "psutil", "pyOpenSSL (>=23.0.0,<23.1.0)", "pycodestyle (>=2.9.0,<2.10.0)"]

[[package]]
name = "watchfiles"
version = "1.0.3"
description = "Simple, modern and high performance file watching and code reload in python."
optional = false
python-versions = ">=3.9"
files = [
    {file = "watchfiles-1.0.3-cp310-cp310-macosx_10_12_x86_64.whl", hash = "sha256:1da46bb1eefb5a37a8fb6fd52ad5d14822d67c498d99bda8754222396164ae42"},
    {file = "watchfiles-1.0.3-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:2b961b86cd3973f5822826017cad7f5a75795168cb645c3a6b30c349094e02e3"},
    {file = "watchfiles-1.0.3-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:34e87c7b3464d02af87f1059fedda5484e43b153ef519e4085fe1a03dd94801e"},
    {file = "watchfiles-1.0.3-cp310-cp310-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:d9dd2b89a16cf7ab9c1170b5863e68de6bf83db51544875b25a5f05a7269e678"},
    {file = "watchfiles-1.0.3-cp310-cp310-manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:2b4691234d31686dca133c920f94e478b548a8e7c750f28dbbc2e4333e0d3da9"},
    {file = "watchfiles-1.0.3-cp310-cp310-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:90b0fe1fcea9bd6e3084b44875e179b4adcc4057a3b81402658d0eb58c98edf8"},
    {file = "watchfiles-1.0.3-cp310-cp310-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:0b90651b4cf9e158d01faa0833b073e2e37719264bcee3eac49fc3c74e7d304b"},
    {file = "watchfiles-1.0.3-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:c2e9fe695ff151b42ab06501820f40d01310fbd58ba24da8923ace79cf6d702d"},
    {file = "watchfiles-1.0.3-cp310-cp310-musllinux_1_1_aarch64.whl", hash = "sha256:62691f1c0894b001c7cde1195c03b7801aaa794a837bd6eef24da87d1542838d"},
    {file = "watchfiles-1.0.3-cp310-cp310-musllinux_1_1_x86_64.whl", hash = "sha256:275c1b0e942d335fccb6014d79267d1b9fa45b5ac0639c297f1e856f2f532552"},
    {file = "watchfiles-1.0.3-cp310-cp310-win32.whl", hash = "sha256:06ce08549e49ba69ccc36fc5659a3d0ff4e3a07d542b895b8a9013fcab46c2dc"},
    {file = "watchfiles-1.0.3-cp310-cp310-win_amd64.whl", hash = "sha256:f280b02827adc9d87f764972fbeb701cf5611f80b619c20568e1982a277d6146"},
    {file = "watchfiles-1.0.3-cp311-cp311-macosx_10_12_x86_64.whl", hash = "sha256:ffe709b1d0bc2e9921257569675674cafb3a5f8af689ab9f3f2b3f88775b960f"},
    {file = "watchfiles-1.0.3-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:418c5ce332f74939ff60691e5293e27c206c8164ce2b8ce0d9abf013003fb7fe"},
    {file = "watchfiles-1.0.3-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:2f492d2907263d6d0d52f897a68647195bc093dafed14508a8d6817973586b6b"},
    {file = "watchfiles-1.0.3-cp311-cp311-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:48c9f3bc90c556a854f4cab6a79c16974099ccfa3e3e150673d82d47a4bc92c9"},
    {file = "watchfiles-1.0.3-cp311-cp311-manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:75d3bcfa90454dba8df12adc86b13b6d85fda97d90e708efc036c2760cc6ba44"},
    {file = "watchfiles-1.0.3-cp311-cp311-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:5691340f259b8f76b45fb31b98e594d46c36d1dc8285efa7975f7f50230c9093"},
    {file = "watchfiles-1.0.3-cp311-cp311-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:1e263cc718545b7f897baeac1f00299ab6fabe3e18caaacacb0edf6d5f35513c"},
    {file = "watchfiles-1.0.3-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:1c6cf7709ed3e55704cc06f6e835bf43c03bc8e3cb8ff946bf69a2e0a78d9d77"},
    {file = "watchfiles-1.0.3-cp311-cp311-musllinux_1_1_aarch64.whl", hash = "sha256:703aa5e50e465be901e0e0f9d5739add15e696d8c26c53bc6fc00eb65d7b9469"},
    {file = "watchfiles-1.0.3-cp311-cp311-musllinux_1_1_x86_64.whl", hash = "sha256:bfcae6aecd9e0cb425f5145afee871465b98b75862e038d42fe91fd753ddd780"},
    {file = "watchfiles-1.0.3-cp311-cp311-win32.whl", hash = "sha256:6a76494d2c5311584f22416c5a87c1e2cb954ff9b5f0988027bc4ef2a8a67181"},
    {file = "watchfiles-1.0.3-cp311-cp311-win_amd64.whl", hash = "sha256:cf745cbfad6389c0e331786e5fe9ae3f06e9d9c2ce2432378e1267954793975c"},
    {file = "watchfiles-1.0.3-cp311-cp311-win_arm64.whl", hash = "sha256:2dcc3f60c445f8ce14156854a072ceb36b83807ed803d37fdea2a50e898635d6"},
    {file = "watchfiles-1.0.3-cp312-cp312-macosx_10_12_x86_64.whl", hash = "sha256:93436ed550e429da007fbafb723e0769f25bae178fbb287a94cb4ccdf42d3af3"},
    {file = "watchfiles-1.0.3-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:c18f3502ad0737813c7dad70e3e1cc966cc147fbaeef47a09463bbffe70b0a00"},
    {file = "watchfiles-1.0.3-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:6a5bc3ca468bb58a2ef50441f953e1f77b9a61bd1b8c347c8223403dc9b4ac9a"},
    {file = "watchfiles-1.0.3-cp312-cp312-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:0d1ec043f02ca04bf21b1b32cab155ce90c651aaf5540db8eb8ad7f7e645cba8"},
    {file = "watchfiles-1.0.3-cp312-cp312-manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:f58d3bfafecf3d81c15d99fc0ecf4319e80ac712c77cf0ce2661c8cf8bf84066"},
    {file = "watchfiles-1.0.3-cp312-cp312-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:1df924ba82ae9e77340101c28d56cbaff2c991bd6fe8444a545d24075abb0a87"},
    {file = "watchfiles-1.0.3-cp312-cp312-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:632a52dcaee44792d0965c17bdfe5dc0edad5b86d6a29e53d6ad4bf92dc0ff49"},
    {file = "watchfiles-1.0.3-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:80bf4b459d94a0387617a1b499f314aa04d8a64b7a0747d15d425b8c8b151da0"},
    {file = "watchfiles-1.0.3-cp312-cp312-musllinux_1_1_aarch64.whl", hash = "sha256:ca94c85911601b097d53caeeec30201736ad69a93f30d15672b967558df02885"},
    {file = "watchfiles-1.0.3-cp312-cp312-musllinux_1_1_x86_64.whl", hash = "sha256:65ab1fb635476f6170b07e8e21db0424de94877e4b76b7feabfe11f9a5fc12b5"},
    {file = "watchfiles-1.0.3-cp312-cp312-win32.whl", hash = "sha256:49bc1bc26abf4f32e132652f4b3bfeec77d8f8f62f57652703ef127e85a3e38d"},
    {file = "watchfiles-1.0.3-cp312-cp312-win_amd64.whl", hash = "sha256:48681c86f2cb08348631fed788a116c89c787fdf1e6381c5febafd782f6c3b44"},
    {file = "watchfiles-1.0.3-cp312-cp312-win_arm64.whl", hash = "sha256:9e080cf917b35b20c889225a13f290f2716748362f6071b859b60b8847a6aa43"},
    {file = "watchfiles-1.0.3-cp313-cp313-macosx_10_12_x86_64.whl", hash = "sha256:e153a690b7255c5ced17895394b4f109d5dcc2a4f35cb809374da50f0e5c456a"},
    {file = "watchfiles-1.0.3-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:ac1be85fe43b4bf9a251978ce5c3bb30e1ada9784290441f5423a28633a958a7"},
    {file = "watchfiles-1.0.3-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:a2ec98e31e1844eac860e70d9247db9d75440fc8f5f679c37d01914568d18721"},
    {file = "watchfiles-1.0.3-cp313-cp313-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:0179252846be03fa97d4d5f8233d1c620ef004855f0717712ae1c558f1974a16"},
    {file = "watchfiles-1.0.3-cp313-cp313-manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:995c374e86fa82126c03c5b4630c4e312327ecfe27761accb25b5e1d7ab50ec8"},
    {file = "watchfiles-1.0.3-cp313-cp313-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:29b9cb35b7f290db1c31fb2fdf8fc6d3730cfa4bca4b49761083307f441cac5a"},
    {file = "watchfiles-1.0.3-cp313-cp313-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:6f8dc09ae69af50bead60783180f656ad96bd33ffbf6e7a6fce900f6d53b08f1"},
    {file = "watchfiles-1.0.3-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:489b80812f52a8d8c7b0d10f0d956db0efed25df2821c7a934f6143f76938bd6"},
    {file = "watchfiles-1.0.3-cp313-cp313-musllinux_1_1_aarch64.whl", hash = "sha256:228e2247de583475d4cebf6b9af5dc9918abb99d1ef5ee737155bb39fb33f3c0"},
    {file = "watchfiles-1.0.3-cp313-cp313-musllinux_1_1_x86_64.whl", hash = "sha256:1550be1a5cb3be08a3fb84636eaafa9b7119b70c71b0bed48726fd1d5aa9b868"},
    {file = "watchfiles-1.0.3-cp313-cp313-win32.whl", hash = "sha256:16db2d7e12f94818cbf16d4c8938e4d8aaecee23826344addfaaa671a1527b07"},
    {file = "watchfiles-1.0.3-cp313-cp313-win_amd64.whl", hash = "sha256:160eff7d1267d7b025e983ca8460e8cc67b328284967cbe29c05f3c3163711a3"},
    {file = "watchfiles-1.0.3-cp39-cp39-macosx_10_12_x86_64.whl", hash = "sha256:c05b021f7b5aa333124f2a64d56e4cb9963b6efdf44e8d819152237bbd93ba15"},
    {file = "watchfiles-1.0.3-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:310505ad305e30cb6c5f55945858cdbe0eb297fc57378f29bacceb534ac34199"},
    {file = "watchfiles-1.0.3-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:ddff3f8b9fa24a60527c137c852d0d9a7da2a02cf2151650029fdc97c852c974"},
    {file = "watchfiles-1.0.3-cp39-cp39-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:46e86ed457c3486080a72bc837300dd200e18d08183f12b6ca63475ab64ed651"},
    {file = "watchfiles-1.0.3-cp39-cp39-manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:f79fe7993e230a12172ce7d7c7db061f046f672f2b946431c81aff8f60b2758b"},
    {file = "watchfiles-1.0.3-cp39-cp39-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:ea2b51c5f38bad812da2ec0cd7eec09d25f521a8b6b6843cbccedd9a1d8a5c15"},
    {file = "watchfiles-1.0.3-cp39-cp39-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:0fe4e740ea94978b2b2ab308cbf9270a246bcbb44401f77cc8740348cbaeac3d"},
    {file = "watchfiles-1.0.3-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:9af037d3df7188ae21dc1c7624501f2f90d81be6550904e07869d8d0e6766655"},
    {file = "watchfiles-1.0.3-cp39-cp39-musllinux_1_1_aarch64.whl", hash = "sha256:52bb50a4c4ca2a689fdba84ba8ecc6a4e6210f03b6af93181bb61c4ec3abaf86"},
    {file = "watchfiles-1.0.3-cp39-cp39-musllinux_1_1_x86_64.whl", hash = "sha256:c14a07bdb475eb696f85c715dbd0f037918ccbb5248290448488a0b4ef201aad"},
    {file = "watchfiles-1.0.3-cp39-cp39-win32.whl", hash = "sha256:be37f9b1f8934cd9e7eccfcb5612af9fb728fecbe16248b082b709a9d1b348bf"},
    {file = "watchfiles-1.0.3-cp39-cp39-win_amd64.whl", hash = "sha256:ef9ec8068cf23458dbf36a08e0c16f0a2df04b42a8827619646637be1769300a"},
    {file = "watchfiles-1.0.3-pp310-pypy310_pp73-macosx_10_12_x86_64.whl", hash = "sha256:84fac88278f42d61c519a6c75fb5296fd56710b05bbdcc74bdf85db409a03780"},
    {file = "watchfiles-1.0.3-pp310-pypy310_pp73-macosx_11_0_arm64.whl", hash = "sha256:c68be72b1666d93b266714f2d4092d78dc53bd11cf91ed5a3c16527587a52e29"},
    {file = "watchfiles-1.0.3-pp310-pypy310_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:889a37e2acf43c377b5124166bece139b4c731b61492ab22e64d371cce0e6e80"},
    {file = "watchfiles-1.0.3-pp310-pypy310_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:7ca05cacf2e5c4a97d02a2878a24020daca21dbb8823b023b978210a75c79098"},
    {file = "watchfiles-1.0.3-pp39-pypy39_pp73-macosx_10_12_x86_64.whl", hash = "sha256:8af4b582d5fc1b8465d1d2483e5e7b880cc1a4e99f6ff65c23d64d070867ac58"},
    {file = "watchfiles-1.0.3-pp39-pypy39_pp73-macosx_11_0_arm64.whl", hash = "sha256:127de3883bdb29dbd3b21f63126bb8fa6e773b74eaef46521025a9ce390e1073"},
    {file = "watchfiles-1.0.3-pp39-pypy39_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:713f67132346bdcb4c12df185c30cf04bdf4bf6ea3acbc3ace0912cab6b7cb8c"},
    {file = "watchfiles-1.0.3-pp39-pypy39_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:abd85de513eb83f5ec153a802348e7a5baa4588b818043848247e3e8986094e8"},
    {file = "watchfiles-1.0.3.tar.gz", hash = "sha256:f3ff7da165c99a5412fe5dd2304dd2dbaaaa5da718aad942dcb3a178eaa70c56"},
]

[package.dependencies]
anyio = ">=3.0.0"

[[package]]
name = "wcwidth"
version = "0.2.13"
description = "Measures the displayed width of unicode strings in a terminal"
optional = false
python-versions = "*"
files = [
    {file = "wcwidth-0.2.13-py2.py3-none-any.whl", hash = "sha256:3da69048e4540d84af32131829ff948f1e022c1c6bdb8d6102117aac784f6859"},
    {file = "wcwidth-0.2.13.tar.gz", hash = "sha256:72ea0c06399eb286d978fdedb6923a9eb47e1c486ce63e9b4e64fc18303972b5"},
]

[[package]]
name = "webcolors"
version = "24.11.1"
description = "A library for working with the color formats defined by HTML and CSS."
optional = false
python-versions = ">=3.9"
files = [
    {file = "webcolors-24.11.1-py3-none-any.whl", hash = "sha256:515291393b4cdf0eb19c155749a096f779f7d909f7cceea072791cb9095b92e9"},
    {file = "webcolors-24.11.1.tar.gz", hash = "sha256:ecb3d768f32202af770477b8b65f318fa4f566c22948673a977b00d589dd80f6"},
]

[[package]]
name = "webencodings"
version = "0.5.1"
description = "Character encoding aliases for legacy web content"
optional = false
python-versions = "*"
files = [
    {file = "webencodings-0.5.1-py2.py3-none-any.whl", hash = "sha256:a0af1213f3c2226497a97e2b3aa01a7e4bee4f403f95be16fc9acd2947514a78"},
    {file = "webencodings-0.5.1.tar.gz", hash = "sha256:b36a1c245f2d304965eb4e0a82848379241dc04b865afcc4aab16748587e1923"},
]

[[package]]
name = "websocket-client"
version = "1.8.0"
description = "WebSocket client for Python with low level API options"
optional = false
python-versions = ">=3.8"
files = [
    {file = "websocket_client-1.8.0-py3-none-any.whl", hash = "sha256:17b44cc997f5c498e809b22cdf2d9c7a9e71c02c8cc2b6c56e7c2d1239bfa526"},
    {file = "websocket_client-1.8.0.tar.gz", hash = "sha256:3239df9f44da632f96012472805d40a23281a991027ce11d2f45a6f24ac4c3da"},
]

[package.extras]
docs = ["Sphinx (>=6.0)", "myst-parser (>=2.0.0)", "sphinx-rtd-theme (>=1.1.0)"]
optional = ["python-socks", "wsaccel"]
test = ["websockets"]

[[package]]
name = "websockets"
version = "14.1"
description = "An implementation of the WebSocket Protocol (RFC 6455 & 7692)"
optional = false
python-versions = ">=3.9"
files = [
    {file = "websockets-14.1-cp310-cp310-macosx_10_9_universal2.whl", hash = "sha256:a0adf84bc2e7c86e8a202537b4fd50e6f7f0e4a6b6bf64d7ccb96c4cd3330b29"},
    {file = "websockets-14.1-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:90b5d9dfbb6d07a84ed3e696012610b6da074d97453bd01e0e30744b472c8179"},
    {file = "websockets-14.1-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:2177ee3901075167f01c5e335a6685e71b162a54a89a56001f1c3e9e3d2ad250"},
    {file = "websockets-14.1-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:3f14a96a0034a27f9d47fd9788913924c89612225878f8078bb9d55f859272b0"},
    {file = "websockets-14.1-cp310-cp310-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:1f874ba705deea77bcf64a9da42c1f5fc2466d8f14daf410bc7d4ceae0a9fcb0"},
    {file = "websockets-14.1-cp310-cp310-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:9607b9a442392e690a57909c362811184ea429585a71061cd5d3c2b98065c199"},
    {file = "websockets-14.1-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:bea45f19b7ca000380fbd4e02552be86343080120d074b87f25593ce1700ad58"},
    {file = "websockets-14.1-cp310-cp310-musllinux_1_2_i686.whl", hash = "sha256:219c8187b3ceeadbf2afcf0f25a4918d02da7b944d703b97d12fb01510869078"},
    {file = "websockets-14.1-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:ad2ab2547761d79926effe63de21479dfaf29834c50f98c4bf5b5480b5838434"},
    {file = "websockets-14.1-cp310-cp310-win32.whl", hash = "sha256:1288369a6a84e81b90da5dbed48610cd7e5d60af62df9851ed1d1d23a9069f10"},
    {file = "websockets-14.1-cp310-cp310-win_amd64.whl", hash = "sha256:e0744623852f1497d825a49a99bfbec9bea4f3f946df6eb9d8a2f0c37a2fec2e"},
    {file = "websockets-14.1-cp311-cp311-macosx_10_9_universal2.whl", hash = "sha256:449d77d636f8d9c17952628cc7e3b8faf6e92a17ec581ec0c0256300717e1512"},
    {file = "websockets-14.1-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:a35f704be14768cea9790d921c2c1cc4fc52700410b1c10948511039be824aac"},
    {file = "websockets-14.1-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:b1f3628a0510bd58968c0f60447e7a692933589b791a6b572fcef374053ca280"},
    {file = "websockets-14.1-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:3c3deac3748ec73ef24fc7be0b68220d14d47d6647d2f85b2771cb35ea847aa1"},
    {file = "websockets-14.1-cp311-cp311-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:7048eb4415d46368ef29d32133134c513f507fff7d953c18c91104738a68c3b3"},
    {file = "websockets-14.1-cp311-cp311-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:f6cf0ad281c979306a6a34242b371e90e891bce504509fb6bb5246bbbf31e7b6"},
    {file = "websockets-14.1-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:cc1fc87428c1d18b643479caa7b15db7d544652e5bf610513d4a3478dbe823d0"},
    {file = "websockets-14.1-cp311-cp311-musllinux_1_2_i686.whl", hash = "sha256:f95ba34d71e2fa0c5d225bde3b3bdb152e957150100e75c86bc7f3964c450d89"},
    {file = "websockets-14.1-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:9481a6de29105d73cf4515f2bef8eb71e17ac184c19d0b9918a3701c6c9c4f23"},
    {file = "websockets-14.1-cp311-cp311-win32.whl", hash = "sha256:368a05465f49c5949e27afd6fbe0a77ce53082185bbb2ac096a3a8afaf4de52e"},
    {file = "websockets-14.1-cp311-cp311-win_amd64.whl", hash = "sha256:6d24fc337fc055c9e83414c94e1ee0dee902a486d19d2a7f0929e49d7d604b09"},
    {file = "websockets-14.1-cp312-cp312-macosx_10_13_universal2.whl", hash = "sha256:ed907449fe5e021933e46a3e65d651f641975a768d0649fee59f10c2985529ed"},
    {file = "websockets-14.1-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:87e31011b5c14a33b29f17eb48932e63e1dcd3fa31d72209848652310d3d1f0d"},
    {file = "websockets-14.1-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:bc6ccf7d54c02ae47a48ddf9414c54d48af9c01076a2e1023e3b486b6e72c707"},
    {file = "websockets-14.1-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:9777564c0a72a1d457f0848977a1cbe15cfa75fa2f67ce267441e465717dcf1a"},
    {file = "websockets-14.1-cp312-cp312-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:a655bde548ca98f55b43711b0ceefd2a88a71af6350b0c168aa77562104f3f45"},
    {file = "websockets-14.1-cp312-cp312-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:a3dfff83ca578cada2d19e665e9c8368e1598d4e787422a460ec70e531dbdd58"},
    {file = "websockets-14.1-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:6a6c9bcf7cdc0fd41cc7b7944447982e8acfd9f0d560ea6d6845428ed0562058"},
    {file = "websockets-14.1-cp312-cp312-musllinux_1_2_i686.whl", hash = "sha256:4b6caec8576e760f2c7dd878ba817653144d5f369200b6ddf9771d64385b84d4"},
    {file = "websockets-14.1-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:eb6d38971c800ff02e4a6afd791bbe3b923a9a57ca9aeab7314c21c84bf9ff05"},
    {file = "websockets-14.1-cp312-cp312-win32.whl", hash = "sha256:1d045cbe1358d76b24d5e20e7b1878efe578d9897a25c24e6006eef788c0fdf0"},
    {file = "websockets-14.1-cp312-cp312-win_amd64.whl", hash = "sha256:90f4c7a069c733d95c308380aae314f2cb45bd8a904fb03eb36d1a4983a4993f"},
    {file = "websockets-14.1-cp313-cp313-macosx_10_13_universal2.whl", hash = "sha256:3630b670d5057cd9e08b9c4dab6493670e8e762a24c2c94ef312783870736ab9"},
    {file = "websockets-14.1-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:36ebd71db3b89e1f7b1a5deaa341a654852c3518ea7a8ddfdf69cc66acc2db1b"},
    {file = "websockets-14.1-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:5b918d288958dc3fa1c5a0b9aa3256cb2b2b84c54407f4813c45d52267600cd3"},
    {file = "websockets-14.1-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:00fe5da3f037041da1ee0cf8e308374e236883f9842c7c465aa65098b1c9af59"},
    {file = "websockets-14.1-cp313-cp313-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:8149a0f5a72ca36720981418eeffeb5c2729ea55fa179091c81a0910a114a5d2"},
    {file = "websockets-14.1-cp313-cp313-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:77569d19a13015e840b81550922056acabc25e3f52782625bc6843cfa034e1da"},
    {file = "websockets-14.1-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:cf5201a04550136ef870aa60ad3d29d2a59e452a7f96b94193bee6d73b8ad9a9"},
    {file = "websockets-14.1-cp313-cp313-musllinux_1_2_i686.whl", hash = "sha256:88cf9163ef674b5be5736a584c999e98daf3aabac6e536e43286eb74c126b9c7"},
    {file = "websockets-14.1-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:836bef7ae338a072e9d1863502026f01b14027250a4545672673057997d5c05a"},
    {file = "websockets-14.1-cp313-cp313-win32.whl", hash = "sha256:0d4290d559d68288da9f444089fd82490c8d2744309113fc26e2da6e48b65da6"},
    {file = "websockets-14.1-cp313-cp313-win_amd64.whl", hash = "sha256:8621a07991add373c3c5c2cf89e1d277e49dc82ed72c75e3afc74bd0acc446f0"},
    {file = "websockets-14.1-cp39-cp39-macosx_10_9_universal2.whl", hash = "sha256:01bb2d4f0a6d04538d3c5dfd27c0643269656c28045a53439cbf1c004f90897a"},
    {file = "websockets-14.1-cp39-cp39-macosx_10_9_x86_64.whl", hash = "sha256:414ffe86f4d6f434a8c3b7913655a1a5383b617f9bf38720e7c0799fac3ab1c6"},
    {file = "websockets-14.1-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:8fda642151d5affdee8a430bd85496f2e2517be3a2b9d2484d633d5712b15c56"},
    {file = "websockets-14.1-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:cd7c11968bc3860d5c78577f0dbc535257ccec41750675d58d8dc66aa47fe52c"},
    {file = "websockets-14.1-cp39-cp39-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:a032855dc7db987dff813583d04f4950d14326665d7e714d584560b140ae6b8b"},
    {file = "websockets-14.1-cp39-cp39-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:b7e7ea2f782408c32d86b87a0d2c1fd8871b0399dd762364c731d86c86069a78"},
    {file = "websockets-14.1-cp39-cp39-musllinux_1_2_aarch64.whl", hash = "sha256:39450e6215f7d9f6f7bc2a6da21d79374729f5d052333da4d5825af8a97e6735"},
    {file = "websockets-14.1-cp39-cp39-musllinux_1_2_i686.whl", hash = "sha256:ceada5be22fa5a5a4cdeec74e761c2ee7db287208f54c718f2df4b7e200b8d4a"},
    {file = "websockets-14.1-cp39-cp39-musllinux_1_2_x86_64.whl", hash = "sha256:3fc753451d471cff90b8f467a1fc0ae64031cf2d81b7b34e1811b7e2691bc4bc"},
    {file = "websockets-14.1-cp39-cp39-win32.whl", hash = "sha256:14839f54786987ccd9d03ed7f334baec0f02272e7ec4f6e9d427ff584aeea8b4"},
    {file = "websockets-14.1-cp39-cp39-win_amd64.whl", hash = "sha256:d9fd19ecc3a4d5ae82ddbfb30962cf6d874ff943e56e0c81f5169be2fda62979"},
    {file = "websockets-14.1-pp310-pypy310_pp73-macosx_10_15_x86_64.whl", hash = "sha256:e5dc25a9dbd1a7f61eca4b7cb04e74ae4b963d658f9e4f9aad9cd00b688692c8"},
    {file = "websockets-14.1-pp310-pypy310_pp73-macosx_11_0_arm64.whl", hash = "sha256:04a97aca96ca2acedf0d1f332c861c5a4486fdcba7bcef35873820f940c4231e"},
    {file = "websockets-14.1-pp310-pypy310_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:df174ece723b228d3e8734a6f2a6febbd413ddec39b3dc592f5a4aa0aff28098"},
    {file = "websockets-14.1-pp310-pypy310_pp73-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:034feb9f4286476f273b9a245fb15f02c34d9586a5bc936aff108c3ba1b21beb"},
    {file = "websockets-14.1-pp310-pypy310_pp73-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:660c308dabd2b380807ab64b62985eaccf923a78ebc572bd485375b9ca2b7dc7"},
    {file = "websockets-14.1-pp310-pypy310_pp73-win_amd64.whl", hash = "sha256:5a42d3ecbb2db5080fc578314439b1d79eef71d323dc661aa616fb492436af5d"},
    {file = "websockets-14.1-pp39-pypy39_pp73-macosx_10_15_x86_64.whl", hash = "sha256:ddaa4a390af911da6f680be8be4ff5aaf31c4c834c1a9147bc21cbcbca2d4370"},
    {file = "websockets-14.1-pp39-pypy39_pp73-macosx_11_0_arm64.whl", hash = "sha256:a4c805c6034206143fbabd2d259ec5e757f8b29d0a2f0bf3d2fe5d1f60147a4a"},
    {file = "websockets-14.1-pp39-pypy39_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:205f672a6c2c671a86d33f6d47c9b35781a998728d2c7c2a3e1cf3333fcb62b7"},
    {file = "websockets-14.1-pp39-pypy39_pp73-manylinux_2_5_i686.manylinux1_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:5ef440054124728cc49b01c33469de06755e5a7a4e83ef61934ad95fc327fbb0"},
    {file = "websockets-14.1-pp39-pypy39_pp73-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:e7591d6f440af7f73c4bd9404f3772bfee064e639d2b6cc8c94076e71b2471c1"},
    {file = "websockets-14.1-pp39-pypy39_pp73-win_amd64.whl", hash = "sha256:25225cc79cfebc95ba1d24cd3ab86aaa35bcd315d12fa4358939bd55e9bd74a5"},
    {file = "websockets-14.1-py3-none-any.whl", hash = "sha256:4d4fc827a20abe6d544a119896f6b78ee13fe81cbfef416f3f2ddf09a03f0e2e"},
    {file = "websockets-14.1.tar.gz", hash = "sha256:398b10c77d471c0aab20a845e7a60076b6390bfdaac7a6d2edb0d2c59d75e8d8"},
]

[[package]]
name = "widgetsnbextension"
version = "4.0.13"
description = "Jupyter interactive widgets for Jupyter Notebook"
optional = false
python-versions = ">=3.7"
files = [
    {file = "widgetsnbextension-4.0.13-py3-none-any.whl", hash = "sha256:74b2692e8500525cc38c2b877236ba51d34541e6385eeed5aec15a70f88a6c71"},
    {file = "widgetsnbextension-4.0.13.tar.gz", hash = "sha256:ffcb67bc9febd10234a362795f643927f4e0c05d9342c727b65d2384f8feacb6"},
]

[[package]]
name = "win32-setctime"
version = "1.2.0"
description = "A small Python utility to set file creation time on Windows"
optional = false
python-versions = ">=3.5"
files = [
    {file = "win32_setctime-1.2.0-py3-none-any.whl", hash = "sha256:95d644c4e708aba81dc3704a116d8cbc974d70b3bdb8be1d150e36be6e9d1390"},
    {file = "win32_setctime-1.2.0.tar.gz", hash = "sha256:ae1fdf948f5640aae05c511ade119313fb6a30d7eabe25fef9764dca5873c4c0"},
]

[package.extras]
dev = ["black (>=19.3b0)", "pytest (>=4.6.2)"]

[[package]]
name = "yapf"
version = "0.40.2"
description = "A formatter for Python code"
optional = false
python-versions = ">=3.7"
files = [
    {file = "yapf-0.40.2-py3-none-any.whl", hash = "sha256:adc8b5dd02c0143108878c499284205adb258aad6db6634e5b869e7ee2bd548b"},
    {file = "yapf-0.40.2.tar.gz", hash = "sha256:4dab8a5ed7134e26d57c1647c7483afb3f136878b579062b786c9ba16b94637b"},
]

[package.dependencies]
importlib-metadata = ">=6.6.0"
platformdirs = ">=3.5.1"
tomli = ">=2.0.1"

[[package]]
name = "zipp"
version = "3.21.0"
description = "Backport of pathlib-compatible object wrapper for zip files"
optional = false
python-versions = ">=3.9"
files = [
    {file = "zipp-3.21.0-py3-none-any.whl", hash = "sha256:ac1bbe05fd2991f160ebce24ffbac5f6d11d83dc90891255885223d42b3cd931"},
    {file = "zipp-3.21.0.tar.gz", hash = "sha256:2c9958f6430a2040341a52eb608ed6dd93ef4392e02ffe219417c1b28b5dd1f4"},
]

[package.extras]
check = ["pytest-checkdocs (>=2.4)", "pytest-ruff (>=0.2.1)"]
cover = ["pytest-cov"]
doc = ["furo", "jaraco.packaging (>=9.3)", "jaraco.tidelift (>=1.4)", "rst.linker (>=1.9)", "sphinx (>=3.5)", "sphinx-lint"]
enabler = ["pytest-enabler (>=2.2)"]
test = ["big-O", "importlib-resources", "jaraco.functools", "jaraco.itertools", "jaraco.test", "more-itertools", "pytest (>=6,!=8.1.*)", "pytest-ignore-flaky"]
type = ["pytest-mypy"]

[metadata]
lock-version = "2.0"
python-versions = "^3.13"
content-hash = "633d749e4b8c2aa8f0ab1397cc1752bf3a58f752792929f369ca8f2fd133b3e4"

```

## pyproject.toml

```toml
[tool.poetry]
name = "python-check-updates"
version = "1.0.0"
description = "like npm-check-updates but for python"
authors = ["Wyatt Walsh <wyattowalsh@gmail.com>"]
readme = "README.md"

[tool.poetry.dependencies]
python = "^3.13"
pydantic = "^2.9.0"
pydantic-settings = "^2.4.0"
typer = {extras = ["all"], version = "^0.12.5"}
fastapi = {extras = ["all"], version = "^0.113.0"}
loguru = "^0.7.2"
rich = "^13.8.0"
pyyaml = "^6.0.2"
python-dotenv = "^1.0.1"
types-pyyaml = "^6.0.12.20240808"
nest-asyncio = "^1.6.0"
pandas = "^2.2.3"
plotly = {extras = ["all"], version = "^5.24.1"}
matplotlib = {extras = ["all"], version = "^3.9.2"}
seaborn = {extras = ["all"], version = "^0.13.2"}


[tool.poetry.group.notebook.dependencies]
nbdime = "^4.0.2"
jupyterlab-git = "^0.50.1"
jupyter-contrib-nbextensions = "^0.7.0"
jupyterlab = "^4.2.5"
jupyterlab-github = "^4.0.0"
jupyter-nbextensions-configurator = "^0.6.4"
nbconvert = "^7.16.4"
ipykernel = "^6.29.5"
ipywidgets = "^8.1.5"
ipython-sql = "^0.5.0"

[tool.poetry.group.notebook]
optional = true

[tool.poetry.group.test.dependencies]
pytest = "^8.3.2"
pytest-sugar = "^1.0.0"
pytest-emoji = "^0.2.0"
pytest-html = "^4.1.1"
pytest-icdiff = "^0.9"
pytest-instafail = "^0.5.0"
pytest-timeout = "^2.3.1"
pytest-benchmark = "^4.0.0"
pytest-cov = "^5.0.0"
hypothesis = "^6.112.0"
pytest-mock = "^3.14.0"
pytest-xdist = {extras = ["all"], version = "^3.6.1"}

[tool.poetry.group.format.dependencies]
isort = "^5.13.2"
black = "^24.8.0"
autoflake = "^2.3.1"
autopep8 = "^2.3.1"
yapf = "^0.40.2"

[tool.poetry.group.lint.dependencies]
ruff = "^0.6.4"
mypy = "^1.11.2"
pylint = "^3.2.7"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"

[tool.pytest.ini_options]
addopts = "-n auto --verbose --hypothesis-show-statistics --html=logs/report.html --self-contained-html --emoji --instafail --cov=python-check-updates --cov-append --cov-report html:logs/coverage"
testpaths = ["tests"]
console_output_style = "progress"
junit_logging = "all"
log_cli = true
log_cli_date_format = "%Y-%m-%d %H:%M:%S"
log_cli_format = "%(asctime)s - [%(levelname)s] - %(name)s - (%(filename)s).%(funcName)s(%(lineno)d) - %(message)s"
log_cli_level = "DEBUG"
log_file = "logs/pytest-logs.txt"
log_file_date_format = "%Y-%m-%d %H:%M:%S"
log_file_format = "%(asctime)s - [%(levelname)s] - %(name)s - (%(filename)s).%(funcName)s(%(lineno)d) - %(message)s"
log_file_level = "DEBUG"
log_format = "%(asctime)s - [%(levelname)s] - %(name)s - (%(filename)s).%(funcName)s(%(lineno)d) - %(message)s"
log_level = "DEBUG"
required_plugins = ["pytest-sugar", "pytest-html", "pytest-emoji", "pytest-icdiff", "pytest-instafail", "pytest-timeout", "pytest-benchmark", "pytest-cov"]
timeout = 500

[tool.coverage.run]
data_file = "logs/.coverage"

[tool.isort]
profile = "black"
src_paths = ["python-check-updates", "tests"]

[tool.autoflake]
remove-all-unused-imports = true
remove-unused-variables = true
in-place = true
ignore-init-module-imports = true

[tool.yapf]
based_on_style = "pep8"
space_inside_brackets = true
spaces_around_dict_delimiters = true
spaces_around_list_delimiters = true
spaces_around_power_operator = true
spaces_around_tuple_delimiters = true
spaces_before_comment = "15, 20"

[tool.ruff]
line-length = 100
lint.select = ["E", "F", "W", "C", "B", "A", "I"]
lint.ignore = [
  "E201", "E202", "E203", "E501", "B017", "I001"
]
exclude = ["python-check-updates/__init__.py", "tests/__init__.py"]

[tool.mypy]
python_version = "3.13"
strict = true
ignore_missing_imports = true
show_error_codes = true
warn_unused_ignores = true
warn_redundant_casts = true
warn_unused_configs = true
warn_unreachable = true
disable_error_code = ["no-untyped-def", "valid-type"]

[[tool.mypy.overrides]]
module = "python-check-updates.config"
ignore_errors = true

[[tool.mypy.overrides]]
module = "tests.test_logging_config"
ignore_errors = true

[[tool.mypy.overrides]]
module = "tests.test_config"
ignore_errors = true

[tool.pylint]
max-line-length = 100
disable = [
  "C0114", "C0115", "C0116",  # Disable missing docstring warnings for simplicity
  "R0903",                     # Too few public methods
  "C0301",                     # Line too long
  "E0213",                     # No-self-argument error (for static methods, Pydantic)
  "W0621",                     # Redefining name
  "W0613",                     # Unused arguments in tests
  "R0801",                     # Duplicate code
  "R1705",                     # Unnecessary "else" after "return"
]
```

## README.md

```md
---
# python check updates

like npm-check-updates but for python

![Python Version](https:
![License](https:
![Stars](https:
![Forks](https:
![Issues](https:
![Contributors](https:
![Last Commit](https:
![Commit Activity](https:

<p align="center">
  <img src="https:
</p>

## Table of Contents

- [Technologies Used](#technologies-used)
- [Project Demo](#project-demo)
- [GitHub Actions](#github-actions)
- [Screenshots](#screenshots)
- [Star History](#star-history)
- [Installation](#installation)
- [Usage](#usage)
- [Development](#development)
- [Documentation](#documentation)
- [Contributing](#contributing)
- [Issue Templates](#issue-templates)
- [License](#license)
- [Feedback](#feedback)

## Technologies Used

![Pyenv](https:

## Project Demo

![Demo GIF](https:

## GitHub Actions

![Build Status](https:

## Screenshots

<p align="center">
  <img src="https:
  <img src="https:
</p>

## Star History

[![Star History Chart](https:

## Installation

### Prerequisites

- [Python 3.13](https:
- [Poetry](https:
- [Pyenv](https:

```bash
# Clone the repository
git clone https:

# Navigate into the project directory
cd python-check-updates

# Install dependencies using Poetry
poetry install
```

## Usage

```bash
poetry run python -m python-check-updates
```

## Development

For details on contributing to development and setting up the development environment, please refer to [DEVELOPMENT.md](DEVELOPMENT.md).

### Issue Templates

To help facilitate contributions and track issues effectively, please refer to our [Issue Templates](https:

- Bug reports
- Feature requests
- Documentation issues

## Documentation

Documentation can be found at [Documentation Link](https:

## Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## Feedback

If you have any feedback or suggestions, feel free to open an issue or reach out directly at [email@example.com](mailto:email@example.com).

## License

Distributed under the MIT License (MIT) License. See `LICENSE` for more information.

---

<p align="center">💻 Made with ❤️ and Python by Wyatt Walsh</p>
```

