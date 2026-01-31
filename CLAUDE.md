
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Python 3.14.2 project named "liancosmetic" currently in its initial setup phase.

## Environment Setup

The project uses a virtual environment located at `.venv/`.

**Activate the virtual environment:**
```bash
source .venv/bin/activate
```

**Run the main script:**
```bash
python main.py
```

## Project Structure

Currently minimal structure with:
- `main.py` - Entry point with sample code
- `.venv/` - Python 3.14.2 virtual environment
- `.idea/` - PyCharm IDE configuration

## Development Workflow

**Install dependencies:**
```bash
source .venv/bin/activate
pip install -r requirements.txt  # When requirements.txt is added
```

## Development Rules

**When adding new components, ALWAYS use the "superpower" plugin.**

## Code Quality

**Type checking and linting:**
```bash
pyright
```

**Run pyright on a specific file:**
```bash
pyright path/to/file.py
```
