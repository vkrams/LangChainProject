# LangChain Project

A LangChain application built and run inside **Google Antigravity**, Google's agent-first IDE (a fork of VS Code). This README covers setting up the Python environment from scratch on Windows, installing dependencies, wiring up API keys, and running your first agent.

---

## Prerequisites

- **Python 3.13** — [python.org/downloads](https://www.python.org/downloads/). During install, tick **"Add Python to PATH"**.
- **Google Antigravity IDE** — [antigravity.google](https://antigravity.google/). Install the desktop app for Windows and sign in.
- **Git** (optional but recommended) — for version control.
- An **API key** for your model provider put that in `.env` file at root folder (OPENAI_API_KEY, GOOGLE_API_KEY, GROQ_API_KEY).

Verify Python is available by opening a terminal and running:

```powershell
python --version
```

---

## 1. Open the project in Antigravity

1. Launch Antigravity and choose **Open Folder** / **Select Project**, then point it at `LangChainProject`. Antigravity is project-centric — the folder you select defines the scope your agents and the Python extension can see.
2. Install the **Python** and **Jupyter** extensions from the marketplace if they aren't already present. Antigravity inherits the VS Code extension ecosystem, so these are the same extensions you'd use in VS Code.

### (Optional) Add an `AGENTS.md`

Antigravity reads a project-root `AGENTS.md` before any agent starts work — it's the equivalent of `.cursorrules` or `CLAUDE.md`. A minimal example for this project:

```markdown
# AGENTS.md

## Environment
- Python 3.13, virtual environment at `.venv`
- Manage dependencies via `requirements.txt`

## Code Style
- Follow PEP 8
- Type hints on all function signatures
- Docstrings required on every tool function (LangChain uses these as tool descriptions)
```

That last line matters — see the troubleshooting section below.

---

## 2. Create and activate a virtual environment

From the integrated terminal (**Ctrl+`**), in the project root:

```powershell
# Create the virtual environment
python -m venv .venv

# Activate it (PowerShell)
.\.venv\Scripts\Activate.ps1
```

If PowerShell blocks the activation script with an execution-policy error, run this once:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

> **Command Prompt instead of PowerShell?** Use `.\.venv\Scripts\activate.bat`.

Your prompt should now be prefixed with `(.venv)`.

### Select the interpreter in Antigravity

Open the command palette (**Ctrl+Shift+P**) → **Python: Select Interpreter** → choose the one inside `.\.venv\Scripts\python.exe`. This ensures notebooks, the Run button, and the terminal all use the same environment.

---

## 3. Install dependencies

Upgrade `pip`, then install the packages:

```powershell
python -m pip install --upgrade pip
pip install -r requirements.txt
```

If you don't have a `requirements.txt` yet, create one with:

```text
langchain
langchain-openai
langgraph
python-dotenv
```

Then install and pin your exact versions afterward with:

```powershell
pip freeze > requirements.txt
```

> Swap `langchain-openai` for the provider package you actually use (e.g. `langchain-anthropic`, `langchain-google-genai`).

---

## 4. Configure API keys

Never hard-code secrets. Create a `.env` file in the project root:

```text
OPENAI_API_KEY=sk-your-key-here
```

Add `.env` and `.venv` to your `.gitignore` so they're never committed:

```text
.env
.venv/
__pycache__/
*.pyc
```

Load the keys at runtime:

```python
from dotenv import load_dotenv
load_dotenv()  # reads .env into environment variables
```

---

## 5. Run your first agent

`main.py`:

```python
from dotenv import load_dotenv
from langchain.agents import create_agent

load_dotenv()


def get_weather(city: str) -> str:
    """Get the current weather for a given city.

    Args:
        city: The name of the city to get weather for.
    """
    return f"The weather in {city} is sunny."


agent = create_agent(
    model="gpt-5",
    tools=[get_weather],
    system_prompt="You are a helpful assistant.",
)

result = agent.invoke(
    {"messages": [{"role": "user", "content": "What's the weather in Sydney?"}]}
)
print(result["messages"][-1].content)
```

Run it from the terminal:

```powershell
python main.py
```

...or press the **Run** button in the Antigravity editor.

---

## Project structure

```
LangChainProject/
├── .venv/              # virtual environment (not committed)
├── .env                # API keys (not committed)
├── .gitignore
├── AGENTS.md           # standing instructions for Antigravity agents
├── requirements.txt    # pinned dependencies
├── main.py             # entry point
└── README.md
```

---

## Troubleshooting

**`ValueError: Function must have a docstring if description not provided.`**
LangChain turns each function in `tools=[...]` into a tool and uses its **docstring** as the description the model sees. Every tool function must have a docstring — or you must pass an explicit `description`. Add a triple-quoted docstring describing what the tool does (and its arguments).

**`ModuleNotFoundError` after installing a package.**
You're almost certainly running a different interpreter than the one you installed into. Confirm `(.venv)` is showing in the terminal, then re-run **Python: Select Interpreter** and pick the `.venv` one.

**PowerShell won't activate the venv.**
Run `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser` once, then re-activate.

**Antigravity agent hangs on a terminal command.**
On Windows, prefer flat, single-line commands (e.g. `pwsh -c .venv\Scripts\python.exe main.py`) and avoid inline `python -c "..."` scripts, which the agent's terminal parser can stall on. You can also tune the **Terminal Execution policy** under Agent settings to *Agent Decides* or *Request Review*.

---

## Useful commands

```powershell
# Reactivate the environment in a new terminal
.\.venv\Scripts\Activate.ps1

# Add a new dependency and re-pin
pip install <package>
pip freeze > requirements.txt

# Deactivate
deactivate
```