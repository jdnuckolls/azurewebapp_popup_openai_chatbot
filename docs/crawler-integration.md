# Crawler integration

This popup widget is designed to consume the same Azure AI Search chunk index as the standalone chatbot.

## Expected pipeline

```text
Website
  -> azurefunction_webcrawler_to_blobstorage
  -> normalized JSON in Azure Blob Storage
  -> Azure AI Search Blob indexer + skillset
  -> website-knowledge-chunks
  -> this popup chatbot
```

## Expected Azure AI Search fields

The default configuration expects the index shape from the crawler repository's `infra/azure-search` templates:

| Field | Purpose |
| --- | --- |
| `content` | Chunk text used as grounding context. |
| `content_vector` | Vector field populated by Azure AI Search integrated vectorization. |
| `title` | Citation label and semantic title field. |
| `url` | Citation URL. |
| `canonical_url` | Normalized source URL fallback. |
| `site`, `path`, `content_type`, `source_engine` | Filters and citation context. |
| `crawl_timestamp`, `last_modified` | Freshness metadata. |

## Recommended settings

```dotenv
AZURE_SEARCH_INDEX_NAME="website-knowledge-chunks"
AZURE_SEMANTIC_CONFIGURATION="semantic-config"
AZURE_SEARCH_API_VERSION="2024-07-01"
USE_VECTOR_SEARCH="true"
USE_SEARCH_INTEGRATED_VECTORIZATION="true"
AZURE_VECTOR_FIELD="content_vector"
AZURE_CONTENT_FIELD="content"
AZURE_TITLE_FIELD="title"
AZURE_URL_FIELD="url"
```

With `USE_SEARCH_INTEGRATED_VECTORIZATION=true`, the app sends text vector queries to Azure AI Search and does not compute query embeddings itself. Set it to `false` only for older indexes that require the app to call Azure OpenAI embeddings directly.

Use `AZURE_SEARCH_FILTER` to scope the popup to the current host or a specific site section, for example:

```text
site eq 'example.com' and startswith(path, '/services')
```

Do not place secrets in files under `public/`. Browser-visible configuration should contain only non-secret UI settings.
