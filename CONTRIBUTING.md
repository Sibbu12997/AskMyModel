# Contributing to AskMyModel

Thanks for helping improve AskMyModel. Issues and pull requests are welcome.

## Before opening an issue

- Search existing issues for the same problem.
- Include Power BI Desktop, Node.js, PowerShell, bridge, visual, and Ollama versions where relevant.
- Include the exact question, sanitized error text, and steps to reproduce.
- Never attach a PBIX, exported model metadata, credentials, customer data, or private Ollama prompts. Remove business names and values from logs/screenshots.

## Pull requests

1. Fork the repository and create a focused branch.
2. Keep changes small and explain the user-visible effect.
3. Do not commit `node_modules`, generated `bridge/model-metadata`, PBIX files, secrets, or local logs.
4. Run `node --check bridge/server.js`. For the visual, run `npm ci` and `npm run lint` in `visual/`, then `pbiviz package` when you have the Power BI visual tooling installed.
5. Describe the tests you ran and any environment limits.
6. Open a pull request and link any related issue.

By submitting a contribution, you agree that it may be distributed under this repository's MIT License.
