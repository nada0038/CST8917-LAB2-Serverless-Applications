# CST8917 Lab 2 – Smart Image Analyzer with Durable Functions
## Akash Nadackanal Vinod
## 041156265

## Demo Link: 

This project builds on the Week 4 exercise by using the **Fan-Out/Fan-In pattern** instead of simple function chaining. When an image gets uploaded to Blob Storage, four analyses run at the same time (colors, objects, OCR, and metadata), the results get combined into one report, and that report is saved to Table Storage.

## How it works

```
Image uploaded to Blob Storage
        ↓
   blob_trigger fires
        ↓
  image_analyzer_orchestrator starts
        ↓  (fan-out - all 4 run at the same time)
  analyze_colors  |  analyze_objects  |  analyze_text  |  analyze_metadata
        ↓  (fan-in - wait for all to finish)
   generate_report
        ↓
   store_results → Table Storage

GET /api/results → retrieves stored results
```

## Functions overview

| Function | Type | What it does |
|----------|------|-------------|
| `blob_trigger` | Client | Detects image upload, kicks off the orchestrator |
| `image_analyzer_orchestrator` | Orchestrator | Runs the fan-out/fan-in then chains the last two steps |
| `analyze_colors` | Activity | Pulls dominant colors from the image (real – uses Pillow) |
| `analyze_objects` | Activity | Object detection (mock) |
| `analyze_text` | Activity | OCR text extraction (mock) |
| `analyze_metadata` | Activity | Gets image dimensions, format, EXIF data (real – uses Pillow) |
| `generate_report` | Activity | Merges all 4 results into one report |
| `store_results` | Activity | Saves the report to Azure Table Storage |
| `get_results` | HTTP | Query stored results by ID or get all |

## Running locally

### What you need
- Python 3.11 or 3.12
- Azure Functions Core Tools v4
- VS Code with the Azure Functions extension
- Azurite (the Azure storage emulator)

### Steps

1. Clone the repo and open the folder in VS Code

2. Create and activate a virtual environment
```powershell
python -m venv .venv
.venv\Scripts\activate
```

3. Install the dependencies
```powershell
pip install -r requirements.txt
```

4. Copy the settings template
```powershell
copy local.settings.example.json local.settings.json
```
The default settings already point to Azurite so you don't need to change anything for local testing.

5. Start Azurite — press `F1` in VS Code and select **Azurite: Start**

6. Create the `images` blob container using the VS Code Azure extension or Azure Storage Explorer
   - Navigate to Local Emulator → Blob Containers → right-click → Create Blob Container → name it `images`

7. Start the function app
```powershell
func start
```

8. Upload any image to the `images` container in Azure Storage Explorer — this triggers the orchestration automatically

9. Check the results
```
http://localhost:7071/api/results
```

## Testing with the .http file

Open `test-function.http` in VS Code (requires the REST Client extension). It has 5 tests:
- Get all results
- Get results with a limit
- Get a specific result by ID
- Get the most recent result
- Try a bad ID (expects a 404)

## Deploying to Azure

1. Press `F1` → **Azure Functions: Create Function App in Azure... (Advanced)** and fill in the prompts (Python 3.12, Linux, Consumption plan)
2. In the Azure Portal, go to your Function App → **Environment variables** and add `ImageStorageConnection` with your storage account connection string
3. Create an `images` container in your Azure Storage Account
4. Press `F1` → **Azure Functions: Deploy to Function App**
5. Test it at `https://<your-app-name>.azurewebsites.net/api/results`

## Note on security

`local.settings.json` is in `.gitignore` so it won't get committed. Use `local.settings.example.json` as the template if someone else needs to run the project.

## AI Disclosure Statement

Used ChatGPT to draft and organize the summary/analysis and to help identify relevant official documentation sources for Azure Durable Functions. I reviewed and edited the text for accuracy and ensured all claims are supported by citations.
