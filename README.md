# Local Business AI, SEO, and Web Development Prospecting

This project contains a self-hosted n8n workflow for discovering local businesses in **Jaipur, India**, auditing public website signals, estimating AI-automation/SEO/web-development opportunities, and writing a CSV review queue.

The workflow is designed for human review. It does **not** automatically send email, LinkedIn messages, SMS, or other outreach.

## Project files

```text
New folder/
├── .env
├── .gitignore
├── README.md
├── docs/
│   └── local-business-prospecting-plan.md
├── leads/
│   └── local-business-prospects.csv       # created after a successful run
└── n8n/
    └── local-business-ai-seo-prospecting.json
```

## What the workflow does

1. Geocodes Jaipur, India with OpenStreetMap Nominatim.
2. Discovers businesses around the city using the OpenStreetMap Overpass API.
3. Audits each listed public website when available.
4. Calculates separate opportunity scores for:
   - local SEO;
   - web development;
   - AI automation.
5. Assigns a conservative `low`, `mid`, `proxy-low`, or `proxy-mid` tier.
6. Optionally requests a recommendation from an OpenAI-compatible model endpoint.
7. Writes the results to:

```text
C:\Users\choud\OneDrive\Desktop\main\New folder\leads\local-business-prospects.csv
```

## Requirements

- n8n installed locally on Windows;
- internet access for Nominatim, Overpass, and public website audits;
- an OpenAI-compatible endpoint if AI summaries are enabled;
- a model ID supported by that endpoint.

The workflow can still produce deterministic audit results when the AI endpoint is unavailable, but set `useAI` to `false` if you want to test discovery and scoring first.

## Configure the OpenAI-compatible endpoint

The root `.env` file contains placeholders only. Replace these values locally:

```env
OPENAI_COMPATIBLE_BASE_URL=https://your-compatible-provider.example.com/v1
OPENAI_COMPATIBLE_MODEL=your-model-id
OPENAI_COMPATIBLE_API_KEY=replace_with_your_compatible_endpoint_key
```

The base URL must end at `/v1`. Do not add `/chat/completions`; the workflow adds that path.

Examples:

```env
# Native local server
OPENAI_COMPATIBLE_BASE_URL=http://localhost:1234/v1

# Docker n8n calling a server on the Windows host
OPENAI_COMPATIBLE_BASE_URL=http://host.docker.internal:1234/v1
```

If the compatible server does not require authentication, leave the key empty:

```env
OPENAI_COMPATIBLE_API_KEY=
```

The workflow reads the variables through:

```text
{{$env.OPENAI_COMPATIBLE_BASE_URL}}
{{$env.OPENAI_COMPATIBLE_MODEL}}
{{$env.OPENAI_COMPATIBLE_API_KEY}}
```

## Start n8n on Windows

Open PowerShell and run n8n from the project folder so it receives the environment variables:

```powershell
cd "C:\Users\choud\OneDrive\Desktop\main\New folder"

$env:OPENAI_COMPATIBLE_BASE_URL="https://your-compatible-provider.example.com/v1"
$env:OPENAI_COMPATIBLE_MODEL="your-model-id"
$env:OPENAI_COMPATIBLE_API_KEY="your-key"
$env:N8N_BLOCK_ENV_ACCESS_IN_NODE="false"
$env:N8N_RESTRICT_FILE_ACCESS_TO="C:\Users\choud\OneDrive\Desktop\main\New folder\leads"

n8n start
```

If you use a process manager or Docker, configure the same variables in that service instead. Restart n8n after changing `.env`; n8n normally reads environment variables at startup.

## Import the workflow

1. Open the local n8n interface, normally `http://localhost:5678`.
2. Choose **Import from File**.
3. Select:

```text
C:\Users\choud\OneDrive\Desktop\main\New folder\n8n\local-business-ai-seo-prospecting.json
```

4. Save the imported workflow.
5. Open the **Configuration** node and verify:
   - `city` is `Jaipur`;
   - `country` is `India`;
   - `outputFilePath` points to the `leads` folder;
   - `useAI` is enabled only if the compatible endpoint is ready.

The JSON file on disk does not update a workflow already imported into n8n. Reimport the file after making workflow-file changes.

## First test

Use a small test run:

```text
maxLeads: 5
radiusMeters: 5000
useAI: false
```

Then click **Execute Workflow**. A successful run should create:

```text
C:\Users\choud\OneDrive\Desktop\main\New folder\leads\local-business-prospects.csv
```

After discovery and CSV writing work, set `useAI` to `true` and test the compatible endpoint.

## Troubleshooting

### City could not be geocoded

Verify that the Configuration node contains plain text:

```text
city: Jaipur
country: India
```

You can test the geocoder in a browser:

```text
https://nominatim.openstreetmap.org/search?q=Jaipur%2C%20India&format=jsonv2&limit=1
```

### Access to the file is not allowed

Verify that:

```text
outputFilePath = C:\Users\choud\OneDrive\Desktop\main\New folder\leads\local-business-prospects.csv
```

Also start n8n with:

```text
N8N_RESTRICT_FILE_ACCESS_TO=C:\Users\choud\OneDrive\Desktop\main\New folder\leads
```

The `leads` folder must exist before the workflow runs.

### 401 or 403 from the model endpoint

Check the compatible API key and make sure it is available to the n8n process. Do not put the real key into the workflow JSON.

### 404 from the model endpoint

Check that the base URL ends with `/v1` and that the provider supports:

```text
POST /v1/chat/completions
```

Do not set the base URL to `/v1/chat/completions` because the workflow appends that route.

### 502 from the model endpoint

Check whether n8n runs directly on Windows or in Docker. Use `localhost` for a native n8n process and `host.docker.internal` when Docker n8n calls a service running on the Windows host.

## Data and compliance notes

- OpenStreetMap does not provide dependable review counts for every business. Missing review information remains unknown.
- The workflow does not claim an exact Google ranking. Optional SearXNG results are only a visibility proxy.
- Respect Nominatim, Overpass, website, and model-provider usage policies and rate limits.
- Review evidence manually before contacting any business.
- Keep `.env` private. It is excluded by `.gitignore`.

## More detail

See:

```text
docs/local-business-prospecting-plan.md
```

for the scoring model, workflow stages, limitations, and suggested review process.
