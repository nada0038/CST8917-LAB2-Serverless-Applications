# CST8917 Lab 2 – Smart Image Analyzer with Durable Functions
## Akash Nadackanal Vinod
## 041156265

## Live App: https://akashnadackanalvinod-image-analyzer-auehfng4ffeveda2.canadacentral-01.azurewebsites.net/api/results

## DEMO VIDEO: 
https://youtu.be/t-sGSfZ92Z8

---

## What this does

When you upload an image to Blob Storage, this app automatically analyzes it. Instead of doing things one at a time, it runs four analyses **at the same time** (colors, objects, text, and image info), waits for all of them to finish, then saves one combined report to Table Storage. You can then fetch those results through a simple HTTP endpoint.

This is the Fan-Out/Fan-In pattern using Azure Durable Functions.

```
Image uploaded → blob_trigger fires → orchestrator starts
      ↓ (all 4 run at the same time)
  analyze_colors | analyze_objects | analyze_text | analyze_metadata
      ↓ (wait for all to finish)
  generate_report → store_results → saved to Table Storage

  GET /api/results → get your results back
```

---

## Functions

| Function | Type | What it does |
|----------|------|--------------|
| `blob_trigger` | Client | Watches for image uploads and starts the orchestrator |
| `image_analyzer_orchestrator` | Orchestrator | Runs the fan-out/fan-in pattern |
| `analyze_colors` | Activity | Finds dominant colors using Pillow (real) |
| `analyze_objects` | Activity | Object detection (mock) |
| `analyze_text` | Activity | OCR text extraction (mock) |
| `analyze_metadata` | Activity | Gets image size, format, EXIF data using Pillow (real) |
| `generate_report` | Activity | Combines all 4 results into one report |
| `store_results` | Activity | Saves the report to Azure Table Storage |
| `get_results` | HTTP | Lets you query results by ID or get all of them |

---

## Running locally

You'll need Python 3.11+, Azure Functions Core Tools v4, VS Code with the Azure Functions extension, and Azurite.

```powershell
# 1. Create and activate virtual environment
python -m venv .venv
.venv\Scripts\activate

# 2. Install dependencies
pip install -r requirements.txt

# 3. Copy local settings
copy local.settings.example.json local.settings.json
```

Then:
- Press `F1` → **Azurite: Start** to start the local storage emulator
- Create an `images` blob container in the local emulator (via Azure Storage Explorer)
- Run `func start`
- Upload any image to the `images` container
- Check results at `http://localhost:7071/api/results`

---

## Testing

Open `test-function.http` in VS Code (you need the REST Client extension). There are 5 tests:
1. Get all results
2. Get results with a custom limit
3. Get one result by ID
4. Get only the most recent result
5. Try a fake ID — should return a 404

---

## Deploying to Azure

1. `F1` → **Azure Functions: Create Function App in Azure... (Advanced)** — use Python 3.11, Linux, Consumption plan
2. In Azure Portal → your Function App → **Environment variables** → add `ImageStorageConnection` with your storage account connection string
3. Create an `images` container in your Azure Storage Account
4. `F1` → **Azure Functions: Deploy to Function App**
5. Test at `https://akashnadackanalvinod-image-analyzer-auehfng4ffeveda2.canadacentral-01.azurewebsites.net/api/results`

---

## Security note

`local.settings.json` is gitignored so your keys won't get pushed. Use `local.settings.example.json` as a template for anyone else who needs to run this locally.

---

## AI Disclosure Statement

Used ChatGPT to draft and organize the summary/analysis and to help identify relevant official documentation sources for Azure Durable Functions. I reviewed and edited the text for accuracy and ensured all claims are supported by citations.
