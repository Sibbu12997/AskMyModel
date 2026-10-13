# AskMyModel

AskMyModel is a Power BI custom visual that lets a report user ask questions about the semantic model in plain language. The visual sends the question to a local bridge. The bridge reads model structure, asks a local Ollama model to plan a query, validates and runs DAX against the open Power BI Desktop model, and returns the result.

This project is designed for **Power BI Desktop on Windows with a local model and a local Ollama installation**. It is not a hosted service and does not make a Desktop report work for viewers in Power BI Service. See [Deployment and limitations](#deployment-and-limitations).

## Repository contents

- `visual/` — editable Power BI visual source and build configuration.
- `release/AskMyModel-1.0.6.0.pbiviz` — installable Power BI visual package.
- `bridge/` — local HTTP bridge and PowerShell DAX/model metadata scripts.
- `LICENSE`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md` — licensing and community guidance.

## Requirements

- Windows 10 or later.
- Power BI Desktop with the report/model you want to query open.
- Node.js 20 or later and npm.
- PowerShell 5.1 or later.
- [Ollama for Windows](https://ollama.com/download).
- DAX Studio installed so its ADOMD.NET library is available. The bridge looks in the standard DAX Studio install folders; set `ASKMYMODEL_ADOMD_DLL` if it is installed elsewhere.

## Quick setup

### 1. Start the local language model

Install Ollama, then in PowerShell run:

```powershell
ollama pull qwen2.5:3b-instruct
ollama serve
```

Leave Ollama running. If you prefer another model, set `ASKMYMODEL_OLLAMA_MODEL` before starting the bridge.

### 2. Export the open model's structure

Open the target PBIX in Power BI Desktop and keep it open. In PowerShell, from the repository root:

```powershell
Set-Location .\bridge
powershell -ExecutionPolicy Bypass -File .\scripts\Export-ModelMetadata.ps1
```

The exporter discovers the active local Power BI model. It writes table, column, measure, and relationship definitions to `bridge/model-metadata/`. It does not export report rows. The generated metadata is local, ignored by Git, and should be regenerated for each model you use.

### 3. Start the bridge

In the same `bridge` folder:

```powershell
npm ci
npm start
```

Check that the bridge responds at [http://127.0.0.1:3210/health](http://127.0.0.1:3210/health). The bridge binds to loopback only. Keep the Power BI Desktop model and Ollama running while asking questions.

Optional settings can be provided through environment variables before `npm start`:

```powershell
$env:ASKMYMODEL_PORT = '3210'
$env:ASKMYMODEL_OLLAMA_URL = 'http://127.0.0.1:11434/api/chat'
$env:ASKMYMODEL_OLLAMA_MODEL = 'qwen2.5:3b-instruct'
$env:ASKMYMODEL_ADOMD_DLL = 'C:\Path\To\Microsoft.AnalysisServices.AdomdClient.dll'
npm start
```

### 4. Add the visual to Power BI Desktop

In Power BI Desktop, choose **Insert → More visuals → Import a visual from a file**, select `release/AskMyModel-1.0.6.0.pbiviz`, and add AskMyModel to the report. The visual connects to the local bridge at `127.0.0.1:3210`.

To build the visual from source instead of using the included package:

```powershell
Set-Location .\visual
npm ci
npm install --global powerbi-visuals-tools
pbiviz package
```

The resulting `.pbiviz` is written to `visual/dist/`. For the latest Power BI import steps, see Microsoft's [custom visual import guide](https://learn.microsoft.com/en-us/power-bi/developer/visuals/import-visual).

## Asking questions

Ask about fields and measures present in the loaded model, for example:

- “What is total sales by year?”
- “Show profit by month and region.”
- “Compare profit margin for 2023 and 2024.”
- “Show the top 10 customers by Sales Amount.”

Use measure names and dimension names shown in the Power BI Fields pane. Questions unrelated to the loaded model can receive a general local-model response; they do not query Power BI. Query results remain subject to the model's relationships, calculation logic, filters, and row-level security.

## Deployment and limitations

The included visual is configured to call `http://127.0.0.1:3210`, and the bridge listens only on the local machine. This works for a local Power BI Desktop setup. A report published to Power BI Service runs in a different environment, so it cannot call the report viewer's `127.0.0.1`. Supporting shared/service viewers requires a separately hosted API, authentication, network and CORS configuration, and an appropriately reachable semantic model. That hosted deployment is not included in this repository.

Power BI may require enabling custom visuals or local web access according to your organization’s policy. Do not expose the local bridge to the network without adding appropriate authentication and security controls.

## Troubleshooting

- **Bridge connection failed:** confirm `npm start` is still running and `/health` opens at `127.0.0.1:3210/health`.
- **Ollama error:** confirm `ollama serve` is running and the configured model is installed (`ollama list`).
- **Model metadata not found:** keep the correct PBIX open and rerun the exporter. Confirm DAX Studio/ADOMD.NET is installed.
- **Power BI model connection failed:** keep the PBIX open; the local model port changes when Power BI restarts. Re-export the metadata and retry.
- **Unknown field/measure:** use the exact name from the Fields pane or rerun the exporter after model changes.
- **Slow query:** try a narrower question, fewer grouping dimensions, or a smaller local Ollama model.

## Privacy

The bridge is intended to run locally. Model metadata is generated on the user’s computer and ignored by Git. The visual sends questions and model context to the local bridge, which sends its prompt to the locally configured Ollama endpoint. The bridge executes DAX against the open local model. This project does not include telemetry or a hosted inference endpoint.

## Contributing and support

See [CONTRIBUTING.md](CONTRIBUTING.md), [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md), and [SECURITY.md](SECURITY.md). Report bugs or request features through [GitHub Issues](https://github.com/Sibbu12997/AskMyModel/issues).

## License

AskMyModel is available under the MIT License. See [LICENSE](LICENSE).

