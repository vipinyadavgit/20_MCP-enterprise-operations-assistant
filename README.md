# IntelliBI Enterprise Operations Assistant

A local learning project demonstrating an enterprise operations assistant built
with Python, `uv`, the Model Context Protocol (MCP), Groq, and Streamlit. It
uses synthetic JSON data for customer, employee, ticket, and policy information.

## Verify the project

Run these commands from the repository root (the directory containing
`pyproject.toml`). They work in Git Bash as well as Windows terminals:

```bash
# Install the project and its declared development dependencies.
uv sync

# Install the runtime dependencies currently listed in requirements.txt.
uv pip install -r requirements.txt

# Check Python syntax, build the package, and run the tests.
uv run python -m compileall -q src/20_mcp_enterprise_operations_assistant
uv build
uv run pytest -q
```

**Current status:** the syntax check and package build have succeeded. The MCP
client, customer data filename references, and root `.env` lookup are
implemented for the current layout. Run the checks above to verify the latest
working tree. The Groq-backed UI additionally requires a valid API key and
network access.

To start the browser UI after the runtime issues below are resolved:

```bash
uv run streamlit run src/20_mcp_enterprise_operations_assistant/app/streamlit_app.py
```

The UI requires a valid Groq API key in the repository-root `.env`. Stop it
with `Ctrl+C`.

## Project structure

```text
.
├── .env                         # Local secrets; ignored by Git
├── .gitignore
├── .python-version              # Python 3.13
├── pyproject.toml               # Project/build configuration and pytest
├── requirements.txt             # Current runtime dependency list
├── uv.lock                      # Resolved uv dependencies
├── README.md
└── src/
    └── 20_mcp_enterprise_operations_assistant/
        ├── app/
        │   ├── __init__.py
        │   ├── config.py         # Environment variables and data paths
        │   ├── groq_agent.py     # Groq tool-calling loop
        │   ├── mcp_client.py     # MCP client helpers
        │   ├── mcp_server.py     # MCP tools and resource
        │   └── streamlit_app.py  # Browser UI
        ├── data/
        │   ├── customer.json
        │   ├── employees.json
        │   ├── policies.json
        │   └── tickets.json
        └── tests/
            ├── test_data.py
            ├── test_mcp_connection.py
            └── test_mcp_tool.py
```

The `app` Python package is under `src/20_mcp_enterprise_operations_assistant/`.
The data and tests are alongside it, not at the repository root. The MCP test
files are standalone scripts with `main()` functions; pytest runs the
pytest-style JSON data test, while the MCP scripts can be run directly as shown
below.

## Setup

Install [uv](https://docs.astral.sh/uv/) and Python 3.13 or later. From the
repository root, synchronize the project and runtime dependencies:

```bash
uv sync
uv pip install -r requirements.txt
```

The runtime packages are listed in `requirements.txt`; `pyproject.toml`
currently declares pytest as a development dependency. Keeping runtime
dependencies in `pyproject.toml` as well would let `uv sync` install the whole
application from the project metadata alone.

Create a `.env` file beside `pyproject.toml` and add your Groq settings:

```dotenv
GROQ_API_KEY=your_groq_api_key
GROQ_MODEL=openai/gpt-oss-120b
```

Use a model ID available to your Groq account that supports tool calling. Do
not commit or share `.env`.

## Application components

- `app/mcp_server.py` exposes read-only MCP tools for customers, employees,
  tickets, and policies, plus the `enterprise://overview` resource.
- `app/mcp_client.py` is intended to launch the MCP server over stdio, discover
  its tools, and invoke them. Its discovery and invocation functions are
  currently missing.
- `app/groq_agent.py` uses Groq tool calls to answer questions using MCP results.
- `app/streamlit_app.py` provides the browser interface and displays tool traces.
- `data/*.json` contains synthetic records for local use.

The intended request flow is:

```text
Streamlit UI -> Groq agent -> MCP client -> MCP server -> JSON data
```

## Dependency note

Runtime packages are listed in `requirements.txt`; `pyproject.toml` currently
declares pytest as a development dependency. Install both with the setup
commands above. Declaring runtime packages in `pyproject.toml` as well would
allow `uv sync` to install the whole application from project metadata alone.

## Test files

- `tests/test_data.py` is the pytest-style test for JSON data files. Run it
  alone with:

  ```bash
  uv run pytest -q src/20_mcp_enterprise_operations_assistant/tests/test_data.py
  ```

- `tests/test_mcp_connection.py` is intended to exercise MCP tool discovery.
- `tests/test_mcp_tool.py` is intended to invoke an example MCP tool.

Run the MCP scripts separately from the repository root:

```bash
uv run python src/20_mcp_enterprise_operations_assistant/tests/test_mcp_connection.py
uv run python src/20_mcp_enterprise_operations_assistant/tests/test_mcp_tool.py
```

## Sample questions

- Who manages ABC Industries?
- Find employees with Databricks skills.
- Show open high-priority tickets for customer C001.
- What is the policy for requesting Databricks access?
- Who manages ABC Industries, and does it have open incidents?

The project is a local learning demo using synthetic data, not a production
enterprise integration. Do not enter real confidential or personal
information.
