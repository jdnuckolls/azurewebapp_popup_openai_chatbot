# Azure Web App Popup OpenAI Chatbot

An embeddable popup chatbot widget for existing websites. It uses Azure AI Search for grounded retrieval, Azure OpenAI for response generation, and optional Azure Speech for microphone input.

This repo is intended to pair with [`azurefunction_webcrawler_to_blobstorage`](https://github.com/jdnuckolls/azurefunction_webcrawler_to_blobstorage):

```text
Website(s)
  -> azurefunction_webcrawler_to_blobstorage
  -> Azure Blob Storage normalized JSON
  -> Azure AI Search integrated vectorization
  -> this popup chatbot widget
```

## Features

- Popup chatbot UI with avatars and website-friendly styling
- Express + Node.js backend API
- Azure AI Search semantic + vector retrieval
- Modern `vectorQueries` support
- Azure AI Search integrated query-time vectorization by default
- Optional legacy app-side query embeddings
- Crawler metadata-aware citations
- Optional managed identity authentication
- Optional Azure Speech microphone input

## Related repositories

| Repository | Role |
| --- | --- |
| `azurefunction_webcrawler_to_blobstorage` | Crawls websites and writes normalized JSON source documents to Blob Storage. |
| `simple-azure-openai-gov-chatbot` | Standalone RAG reference app. |
| `azurewebapp_popup_openai_chatbot` | Embeddable popup chatbot widget for existing websites. |

## Expected Azure AI Search index

The app defaults to the chunk index created by the crawler repo's `infra/azure-search` templates:

| Field | Purpose |
| --- | --- |
| `content` | Chunk text used as grounding context. |
| `content_vector` | Vector field populated by Azure AI Search integrated vectorization. |
| `title` | Citation label and semantic title. |
| `url` | Citation link. |
| `canonical_url` | Normalized source URL fallback. |
| `site`, `path`, `content_type`, `source_engine` | Metadata for filters and context. |
| `crawl_timestamp`, `last_modified` | Freshness metadata. |

See [Crawler integration](docs/crawler-integration.md) for index details and [Three-repo architecture](docs/architecture.md) for the end-to-end portfolio story.

## Local setup

```bash
git clone https://github.com/jdnuckolls/azurewebapp_popup_openai_chatbot.git
cd azurewebapp_popup_openai_chatbot
npm install
cp .env.example .env
npm start
```

Open `http://localhost:3000` and launch the popup chatbot.

## Required settings

```dotenv
AZURE_OPENAI_DEPLOYMENT_NAME="gpt-4o"
AZURE_OPENAI_ENDPOINT="https://your-openai-endpoint.openai.azure.com"
AZURE_OPENAI_API_KEY="your-azure-openai-api-key"
AZURE_OPENAI_API_VERSION="2024-02-15-preview"

AZURE_SEARCH_ENDPOINT="https://your-search-endpoint.search.windows.net"
AZURE_SEARCH_KEY="your-azure-search-key"
AZURE_SEARCH_INDEX_NAME="website-knowledge-chunks"
AZURE_SEARCH_API_VERSION="2024-07-01"
AZURE_SEMANTIC_CONFIGURATION="semantic-config"

USE_VECTOR_SEARCH="true"
USE_SEARCH_INTEGRATED_VECTORIZATION="true"
AZURE_VECTOR_FIELD="content_vector"
AZURE_CONTENT_FIELD="content"
AZURE_TITLE_FIELD="title"
AZURE_URL_FIELD="url"
```

Use `AZURE_SEARCH_FILTER` to scope a widget instance to a specific site or path:

```dotenv
AZURE_SEARCH_FILTER="site eq 'example.com' and startswith(path, '/services')"
```

## Authentication modes

For local demos:

```dotenv
AUTH_MODE="api_key"
```

For Azure App Service, prefer managed identity:

```dotenv
AUTH_MODE="managed_identity"
```

When using managed identity, assign the web app identity appropriate Azure AI Search and Azure OpenAI RBAC permissions.

## Security

- Never commit `.env`, real app settings, keys, tokens, or connection strings.
- Do not put secrets in files under `public/`; everything in `public/` is browser-accessible.
- Prefer managed identity and RBAC for production or government deployments.
- Use Key Vault references for App Service settings when possible.

## Customizing

- UI: edit `public/styles.css` and `public/index.html`.
- Client behavior: edit `public/script.js`.
- Backend retrieval and answer generation: edit `server.js`.
