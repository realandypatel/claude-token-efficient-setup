# Attributions

This repository is a documentation project. It does not contain, modify, or redistribute code from any of the tools below. It describes how to combine them.

All credit for the heavy lifting belongs to the authors and maintainers of these tools.

---

## `code-review-graph`

The Python package that parses your codebase into a queryable graph and exposes it via the Model Context Protocol (MCP).

- **PyPI**: https://pypi.org/project/code-review-graph/
- **Install**: `pipx install code-review-graph`
- **License**: See the project's own LICENSE — install and check `~/.local/share/pipx/venvs/code-review-graph/...` or the project's PyPI page.

If you use this guide and find `code-review-graph` valuable, please:
- Check the project's GitHub/homepage and star it
- Consider contributing upstream if you hit improvements
- Respect the project's own license terms

---

## `@modelcontextprotocol/server-filesystem`

The official Anthropic MCP server for filesystem access. Distributed as an npm package and also available as a native Claude Desktop Extension.

- **npm**: https://www.npmjs.com/package/@modelcontextprotocol/server-filesystem
- **GitHub (MCP org)**: https://github.com/modelcontextprotocol
- **License**: MIT (as of last check — verify in the package itself)

---

## Anthropic — Claude Desktop and MCP

- **Claude Desktop**: https://claude.com/download
- **Model Context Protocol (MCP) specification**: https://modelcontextprotocol.io
- **MCP is an open protocol** published by Anthropic, with a growing ecosystem of server implementations by Anthropic and the community.

---

## pipx

Used for installing `code-review-graph` in an isolated Python environment.

- **GitHub**: https://github.com/pypa/pipx
- **License**: MIT

---

## Homebrew

Used for installing pipx and other prerequisites on macOS.

- **Site**: https://brew.sh
- **License**: BSD 2-Clause

---

## What this repo contributes

Original work in this repository — released under MIT (see [LICENSE](LICENSE)):

- The step-by-step setup guide ([SETUP.md](SETUP.md))
- The troubleshooting notes ([docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)) — written from real failures during a production setup
- The prompt templates ([prompts/](prompts/)) — session starters, closers, task frames
- The config templates ([templates/](templates/)) — placeholder-filled examples, not copies of any tool's own config
- The honest-caveats writeup on token savings ([docs/TOKEN_SAVINGS.md](docs/TOKEN_SAVINGS.md))
- The workflow writeup ([docs/WORKFLOW.md](docs/WORKFLOW.md))

These are documentation of a workflow, not new software. They describe how to use the tools above effectively — nothing more, nothing less.

---

## Noticing something missing or wrong?

If I've attributed a tool incorrectly, missed a license term, or described a project's behavior in a misleading way — open an issue or PR. Getting attribution right matters.
