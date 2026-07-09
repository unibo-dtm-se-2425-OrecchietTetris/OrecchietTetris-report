---
title: Deployment
has_children: false
nav_order: 8
---

# Deployment

## User installation

> You need Python from version 3.11 to 3.14 to install and use the application.

To install OrecchietTetris, run the following commands in your terminal. You can either download it from PyPI or install it from source.

### From PyPI

```bash
pip install OrecchietTetris
OrecchietTetris
```
Once the installation has been completed successfully, launching the command:
```bash
# Windows (cmd)
python -m OrecchietTetris 

# macOS / Linux
OrecchietTetris
```
it will bring up the main menu of the game.

### From source

1. Clone the repository:

    ```bash
    git clone https://github.com/unibo-dtm-se-2425-OrecchietTetris/OrecchietTetris-artifact
    cd OrecchietTetris-artifact
    ```

2. Install Poetry if you don't have it yet:

    ```bash
    pip install -r requirements.txt
    ```

3. Install the project's dependencies (creates `.venv/` inside the project):

    ```bash
    poetry install
    ```
4. Run the game with the command:
    ```bash
    poetry run OrecchietTetris
    ```

#### Virtual environment

All project commands run inside the `.venv/` virtual environment created by Poetry.
Activate it once per terminal session, then use tools directly without any prefix:

```bash
python3 -m venv .venv

# macOS / Linux
source .venv/bin/activate

# Windows (PowerShell)
.venv\Scripts\Activate.ps1

# Windows (cmd)
.venv\Scripts\activate.bat
```
