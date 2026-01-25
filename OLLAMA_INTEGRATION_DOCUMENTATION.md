# Ollama Integration in Roo Code

## Table of Contents

1. [Overview](#overview)
2. [About Axios](#about-axios)
3. [API Verification](#api-verification)
4. [Model Discovery (`ollama list` equivalent)](#model-discovery-ollama-list-equivalent)
5. [Connection Setup](#connection-setup)
6. [API Request Timeouts](#api-request-timeouts)
7. [Code References](#code-references)
8. [Related Documentation](#related-documentation)

---

## Overview

Roo Code integrates with Ollama to support local and cloud-based large language models. The integration consists of three main components:

1. **Model Discovery**: Fetches available models from Ollama instances
2. **Native API Handler**: Handles chat completions and tool calling using Ollama's native API
3. **Embedding Service**: Provides code indexing capabilities using Ollama embedding models

---

## About Axios

### What is Axios?

**Axios** is a popular, promise-based HTTP client library for JavaScript and TypeScript. It provides a simple and intuitive API for making HTTP requests from both browser and Node.js environments.

**For detailed information on configuring Axios for connection setup, timeouts, retries, and logging, see [axios-library-api-summary.md](./axios-library-api-summary.md).**

### Key Features

- **Promise-based**: Uses JavaScript Promises for asynchronous operations
- **Request/Response Interceptors**: Allows modification of requests and responses
- **Automatic JSON Data Transformation**: Automatically serializes/deserializes JSON
- **Request Timeout Support**: Configurable timeout settings for requests
- **Request Cancellation**: Support for canceling requests
- **Error Handling**: Built-in error handling with detailed error information

### Usage in Roo Code's Ollama Integration

In the Ollama integration, Axios is used specifically for **model discovery** operations. It makes HTTP requests to Ollama's REST API endpoints:

- `GET /api/tags` - To fetch the list of available models
- `POST /api/show` - To get detailed information about each model

Axios is used here instead of the native `fetch` API or the `ollama` npm package because:

1. It provides simpler error handling for HTTP requests
2. It automatically handles JSON parsing
3. It's already a dependency in the project

### API Verification

The following section verifies that Roo Code's Ollama integration uses the correct API endpoints and parses responses correctly.

#### 1. List Models Endpoint (`GET /api/tags`)

**Command**:

```bash
curl -s http://localhost:11434/api/tags | jq .
```

**Response Structure**:
The endpoint returns a JSON object with a `models` array. Each model object contains:

- `name`: Model name (e.g., "llama3-groq-tool-use:70b-q2_K")
- `model`: Model identifier (same as name)
- `modified_at`: ISO 8601 timestamp
- `size`: Model size in bytes
- `digest`: SHA256 digest
- `details`: Object containing:
    - `family`: Model family (e.g., "llama", "nomic-bert")
    - `families`: Array of family strings (can be `null`)
    - `format`: Format type (e.g., "gguf")
    - `parameter_size`: Human-readable size (e.g., "70.6B")
    - `quantization_level`: Quantization level (e.g., "Q2_K")
    - `parent_model`: Parent model identifier (empty string if none)

**Example Response** (truncated):

```json
{
	"models": [
		{
			"name": "llama3-groq-tool-use:70b-q2_K",
			"model": "llama3-groq-tool-use:70b-q2_K",
			"modified_at": "2026-01-24T09:54:44.219006531-08:00",
			"size": 26375176825,
			"digest": "dab8a158f092acc40d86d59fecf529f502791d528c24c79586757db2e90302f7",
			"details": {
				"parent_model": "",
				"format": "gguf",
				"family": "llama",
				"families": ["llama"],
				"parameter_size": "70.6B",
				"quantization_level": "Q2_K"
			}
		}
	]
}
```

**Verification**: ✅ The response structure matches `OllamaModelsResponseSchema` defined in `src/api/providers/fetchers/ollama.ts:32-34`.

#### 2. Model Details Endpoint (`POST /api/show`)

**Command**:

```bash
curl -s -X POST http://localhost:11434/api/show -d '{"model": "llama3-groq-tool-use:70b-q2_K"}' | jq '{details, model_info: (.model_info | with_entries(select(.key | contains("context")))), capabilities}'
```

**Response Structure**:
The endpoint returns detailed information about a specific model:

- `details`: Same structure as in `/api/tags` response
- `model_info`: Object with model-specific metadata keys (e.g., `llama.context_length`, `general.architecture`)
- `capabilities`: Array of capability strings (e.g., `["completion", "tools"]` or `["embedding"]`)
- `modelfile`: Modelfile content (string)
- `parameters`: Parameters string
- `template`: Template string

**Example Response** (key fields):

```json
{
	"details": {
		"parent_model": "",
		"format": "gguf",
		"family": "llama",
		"families": ["llama"],
		"parameter_size": "70.6B",
		"quantization_level": "Q2_K"
	},
	"model_info": {
		"llama.context_length": 8192
	},
	"capabilities": ["completion", "tools"]
}
```

**Verification**: ✅ The response structure matches `OllamaModelInfoResponseSchema` defined in `src/api/providers/fetchers/ollama.ts:23-30`.

#### 3. Response Parsing Verification

**Context Length Extraction**:

- **Location**: `src/api/providers/fetchers/ollama.ts:41-43`
- **Logic**: Searches for a key in `model_info` that includes "context_length"
- **Example**: For llama models, finds `llama.context_length` (e.g., 8192 or 16384)
- **Verification**: ✅ The curl response shows `"llama.context_length": 8192`, which will be correctly extracted

**Tool Support Detection**:

- **Location**: `src/api/providers/fetchers/ollama.ts:45-47`
- **Logic**: Checks if `capabilities` array includes `"tools"`
- **Example**:
    - `llama3-groq-tool-use:70b-q2_K` has `["completion", "tools"]` → ✅ Included
    - `codellama:7b` has `["completion"]` → ❌ Filtered out (no tools)
- **Verification**: ✅ Models with `"tools"` in capabilities are correctly identified

**Model Filtering**:

- **Location**: `src/api/providers/fetchers/ollama.ts:49-53`
- **Logic**: Only models with `supportsNativeTools === true` are included
- **Verification**: ✅ Models without tool support are correctly filtered out

#### 4. Complete API Call Flow Verification

**Step 1: List all models**

```bash
curl -s http://localhost:11434/api/tags
```

Returns list of all available models.

**Step 2: Get details for each model**

```bash
curl -s -X POST http://localhost:11434/api/show -d '{"model": "MODEL_NAME"}'
```

Returns detailed information including capabilities.

**Step 3: Filter by tool support**
Only models with `capabilities` containing `"tools"` are included in the final result.

**Verification Summary**:

- ✅ `/api/tags` endpoint returns expected structure
- ✅ `/api/show` endpoint returns expected structure
- ✅ Context length extraction works correctly (finds `llama.context_length` in `model_info`)
- ✅ Tool support detection works correctly (checks `capabilities` array)
- ✅ Model filtering works correctly (only includes models with tools capability)

### Default Timeout Behavior

By default, Axios has **no timeout** (timeout value of 0), which means requests can hang indefinitely if the server doesn't respond. In Roo Code's implementation, no explicit timeout is configured for Axios requests to Ollama, so they rely on Axios's default behavior.

### Learn More

For more information about Axios, including its API documentation, configuration options, and examples, visit:

**GitHub Repository**: [https://github.com/axios/axios](https://github.com/axios/axios)

The repository contains:

- Complete API documentation
- Configuration options (including timeout settings)
- Examples and usage patterns
- Issue tracking and community discussions

**For Roo Code-specific Axios configuration recommendations, see [axios-library-api-summary.md](./axios-library-api-summary.md).**

---

## Model Discovery (`ollama list` equivalent)

### How It Works

Roo Code performs the equivalent of `ollama list` by making HTTP requests to the Ollama API. The discovery process is implemented in the `getOllamaModels` function.

### Implementation Details

**File**: `src/api/providers/fetchers/ollama.ts`

**Function**: `getOllamaModels(baseUrl, apiKey)`

- **Lines**: 67-127
- **Default base URL**: `http://localhost:11434`
- **API Endpoints Used**:
    1. `GET ${baseUrl}/api/tags` - Lists all available models
    2. `POST ${baseUrl}/api/show` - Gets detailed information for each model

### Code Flow

```typescript:67:127:src/api/providers/fetchers/ollama.ts
export async function getOllamaModels(
	baseUrl = "http://localhost:11434",
	apiKey?: string,
): Promise<Record<string, ModelInfo>> {
	const models: Record<string, ModelInfo> = {}

	// clearing the input can leave an empty string; use the default in that case
	baseUrl = baseUrl === "" ? "http://localhost:11434" : baseUrl

	try {
		if (!URL.canParse(baseUrl)) {
			return models
		}

		// Prepare headers with optional API key
		const headers: Record<string, string> = {}
		if (apiKey) {
			headers["Authorization"] = `Bearer ${apiKey}`
		}

		const response = await axios.get<OllamaModelsResponse>(`${baseUrl}/api/tags`, { headers })
		const parsedResponse = OllamaModelsResponseSchema.safeParse(response.data)
		let modelInfoPromises = []

		if (parsedResponse.success) {
			for (const ollamaModel of parsedResponse.data.models) {
				modelInfoPromises.push(
					axios
						.post<OllamaModelInfoResponse>(
							`${baseUrl}/api/show`,
							{
								model: ollamaModel.model,
							},
							{ headers },
						)
						.then((ollamaModelInfo) => {
							const modelInfo = parseOllamaModel(ollamaModelInfo.data)
							// Only include models that support native tools
							if (modelInfo) {
								models[ollamaModel.name] = modelInfo
							}
						}),
				)
			}

			await Promise.all(modelInfoPromises)
		} else {
			console.error(`Error parsing Ollama models response: ${JSON.stringify(parsedResponse.error, null, 2)}`)
		}
	} catch (error) {
		if (error.code === "ECONNREFUSED") {
			console.warn(`Failed connecting to Ollama at ${baseUrl}`)
		} else {
			console.error(
				`Error fetching Ollama models: ${JSON.stringify(error, Object.getOwnPropertyNames(error), 2)}`,
			)
		}
	}

	return models
}
```

### Key Points

1. **Two-Step Process**:

    - First, calls `/api/tags` to get a list of all models (line 87)
    - Then, for each model, calls `/api/show` to get detailed information including capabilities (lines 94-109)

2. **Model Filtering**:

    - Only models that support native tools are included (lines 46-52, 104-106)
    - The `parseOllamaModel` function checks for `capabilities.includes("tools")` (line 47)

3. **Authentication**:

    - Supports optional API key via Bearer token authentication (lines 82-85)
    - API key is included in headers for both requests

4. **Error Handling**:

    - Handles connection refused errors gracefully (lines 117-118)
    - Returns empty models object on failure

5. **Webview Integration**:
    - Models are requested from the webview via `requestOllamaModels` message
    - **File**: `src/core/webview/webviewMessageHandler.ts`
    - **Lines**: 988-1009

```typescript:988:1009:src/core/webview/webviewMessageHandler.ts
case "requestOllamaModels": {
	// Specific handler for Ollama models only.
	const { apiConfiguration: ollamaApiConfig } = await provider.getState()
	try {
		const ollamaOptions = {
			provider: "ollama" as const,
			baseUrl: ollamaApiConfig.ollamaBaseUrl,
			apiKey: ollamaApiConfig.ollamaApiKey,
		}
		// Flush cache and refresh to ensure fresh models.
		await flushModels(ollamaOptions, true)

		const ollamaModels = await getModels(ollamaOptions)

		if (Object.keys(ollamaModels).length > 0) {
			provider.postMessageToWebview({ type: "ollamaModels", ollamaModels: ollamaModels })
		}
	} catch (error) {
		// Silently fail - user hasn't configured Ollama yet
		console.debug("Ollama models fetch failed:", error)
	}
	break
```

---

## Connection Setup

### Native Ollama Handler

The main connection to Ollama for chat completions is handled by the `NativeOllamaHandler` class.

**File**: `src/api/providers/native-ollama.ts`

**Class**: `NativeOllamaHandler`

- **Lines**: 148-384

### Client Initialization

The Ollama client is lazily initialized when first needed:

```typescript:158:179:src/api/providers/native-ollama.ts
private ensureClient(): Ollama {
	if (!this.client) {
		try {
			const clientOptions: OllamaOptions = {
				host: this.options.ollamaBaseUrl || "http://localhost:11434",
				// Note: The ollama npm package handles timeouts internally
			}

			// Add API key if provided (for Ollama cloud or authenticated instances)
			if (this.options.ollamaApiKey) {
				clientOptions.headers = {
					Authorization: `Bearer ${this.options.ollamaApiKey}`,
				}
			}

			this.client = new Ollama(clientOptions)
		} catch (error: any) {
			throw new Error(`Error creating Ollama client: ${error.message}`)
		}
	}
	return this.client
}
```

### Configuration Options

The handler accepts the following options (defined in `packages/types/src/provider-settings.ts`, lines 257-262):

```typescript:257:262:packages/types/src/provider-settings.ts
const ollamaSchema = baseProviderSettingsSchema.extend({
	ollamaModelId: z.string().optional(),
	ollamaBaseUrl: z.string().optional(),
	ollamaApiKey: z.string().optional(),
	ollamaNumCtx: z.number().int().min(128).optional(),
})
```

**Configuration Parameters**:

- `ollamaBaseUrl`: Base URL for the Ollama instance (default: `http://localhost:11434`)
- `ollamaApiKey`: Optional API key for authenticated instances or cloud services
- `ollamaModelId`: The model identifier to use
- `ollamaNumCtx`: Optional context window size override

### Connection Details

1. **Default Host**: `http://localhost:11434` (line 162)
2. **Authentication**: Bearer token via `Authorization` header (lines 167-170)
3. **Client Library**: Uses the official `ollama` npm package
4. **Lazy Initialization**: Client is created on first use, not at construction time

### API Request Flow

When making chat completion requests:

```typescript:203:250:src/api/providers/native-ollama.ts
override async *createMessage(
	systemPrompt: string,
	messages: Anthropic.Messages.MessageParam[],
	metadata?: ApiHandlerCreateMessageMetadata,
): ApiStream {
	const client = this.ensureClient()
	const { id: modelId, info: modelInfo } = await this.fetchModel()
	const useR1Format = modelId.toLowerCase().includes("deepseek-r1")

	const ollamaMessages: Message[] = [
		{ role: "system", content: systemPrompt },
		...convertToOllamaMessages(messages),
	]

	// ... message conversion logic ...

	// Create the actual API request promise
	const stream = await client.chat({
		model: modelId,
		messages: ollamaMessages,
		stream: true,
		options: chatOptions,
		// Native tool calling support
		...(useNativeTools && { tools: this.convertToolsToOllama(metadata.tools) }),
	})
```

---

## API Request Timeouts

### Timeout Configuration Overview

Roo Code handles timeouts differently for different parts of the Ollama integration:

1. **Model Discovery (Fetcher)**: No explicit timeout configured
2. **Native Chat Handler**: Relies on the `ollama` npm package's internal timeout handling
3. **Embedding Service**: Hardcoded timeouts (60s for embeddings, 30s for validation)

### 1. Model Discovery Timeouts

**File**: `src/api/providers/fetchers/ollama.ts`

**Status**: **No explicit timeout configured**

The `getOllamaModels` function uses `axios` for HTTP requests but does not specify a timeout option:

```typescript:87:87:src/api/providers/fetchers/ollama.ts
const response = await axios.get<OllamaModelsResponse>(`${baseUrl}/api/tags`, { headers })
```

This means:

- Axios will use its default timeout (typically 0, meaning no timeout)
- Requests could hang indefinitely if Ollama is unresponsive
- **This timeout is NOT adjustable** - it's hardcoded to use axios defaults

### 2. Native Chat Handler Timeouts

**File**: `src/api/providers/native-ollama.ts`

**Status**: **Handled by the `ollama` npm package internally**

The comment on line 163 explicitly states:

```typescript:163:163:src/api/providers/native-ollama.ts
// Note: The ollama npm package handles timeouts internally
```

**Implications**:

- The timeout behavior is controlled by the `ollama` npm package
- Roo Code does not configure or override these timeouts
- **This timeout is NOT adjustable** from Roo Code's configuration

### 3. Embedding Service Timeouts

**File**: `src/services/code-index/embedders/ollama.ts`

**Status**: **Hardcoded timeouts**

The embedding service uses explicit timeout constants:

```typescript:10:12:src/services/code-index/embedders/ollama.ts
// Timeout constants for Ollama API requests
const OLLAMA_EMBEDDING_TIMEOUT_MS = 60000 // 60 seconds for embedding requests
const OLLAMA_VALIDATION_TIMEOUT_MS = 30000 // 30 seconds for validation requests
```

**Embedding Requests** (60 seconds):

```typescript:71:86:src/services/code-index/embedders/ollama.ts
// Add timeout to prevent indefinite hanging
const controller = new AbortController()
const timeoutId = setTimeout(() => controller.abort(), OLLAMA_EMBEDDING_TIMEOUT_MS)

const response = await fetch(url, {
	method: "POST",
	headers: {
		"Content-Type": "application/json",
	},
	body: JSON.stringify({
		model: modelToUse,
		input: processedTexts, // Using 'input' as requested
	}),
	signal: controller.signal,
})
clearTimeout(timeoutId)
```

**Validation Requests** (30 seconds):

```typescript:150:161:src/services/code-index/embedders/ollama.ts
// Add timeout to prevent indefinite hanging
const controller = new AbortController()
const timeoutId = setTimeout(() => controller.abort(), OLLAMA_VALIDATION_TIMEOUT_MS)

const modelsResponse = await fetch(modelsUrl, {
	method: "GET",
	headers: {
		"Content-Type": "application/json",
	},
	signal: controller.signal,
})
clearTimeout(timeoutId)
```

**Status**: **These timeouts are NOT adjustable** - they are hardcoded constants

### 4. General API Timeout Configuration

**File**: `src/api/providers/utils/timeout-config.ts`

Roo Code has a general timeout configuration system, but **it is NOT used for Ollama's native handler**:

```typescript:10:26:src/api/providers/utils/timeout-config.ts
export function getApiRequestTimeout(): number | undefined {
	// Get timeout with validation to ensure it's a valid non-negative number
	const configTimeout = vscode.workspace.getConfiguration(Package.name).get<number>("apiRequestTimeout", 600)

	// Validate that it's actually a number and not NaN
	if (typeof configTimeout !== "number" || isNaN(configTimeout)) {
		return 600 * 1000 // Default to 600 seconds
	}

	// 0 or negative means "no timeout" - return undefined to let SDK use its default
	// (OpenAI SDK interprets 0 as "abort immediately", so we return undefined instead)
	if (configTimeout <= 0) {
		return undefined
	}

	return configTimeout * 1000 // Convert to milliseconds
}
```

**Important Notes**:

- This function is used by OpenAI-compatible providers (see `src/api/providers/base-openai-compatible-provider.ts`, line 66)
- **It is NOT used by `NativeOllamaHandler`**
- The VSCode setting `apiRequestTimeout` (default: 600 seconds) does not affect Ollama requests
- This timeout setting is adjustable via VSCode settings, but only affects OpenAI-compatible providers

### Timeout Summary Table

| Component                     | Timeout Value                   | Adjustable?             | Location                                         |
| ----------------------------- | ------------------------------- | ----------------------- | ------------------------------------------------ |
| Model Discovery (`/api/tags`) | None (axios default)            | ❌ No                   | `src/api/providers/fetchers/ollama.ts:87`        |
| Model Discovery (`/api/show`) | None (axios default)            | ❌ No                   | `src/api/providers/fetchers/ollama.ts:95`        |
| Native Chat Handler           | Handled by `ollama` npm package | ❌ No                   | `src/api/providers/native-ollama.ts:163`         |
| Embedding Requests            | 60 seconds                      | ❌ No (hardcoded)       | `src/services/code-index/embedders/ollama.ts:11` |
| Validation Requests           | 30 seconds                      | ❌ No (hardcoded)       | `src/services/code-index/embedders/ollama.ts:12` |
| General API Timeout Setting   | 600 seconds (default)           | ✅ Yes (VSCode setting) | `src/api/providers/utils/timeout-config.ts:12`   |
|                               |                                 |                         | **Note: Not used for Ollama**                    |

---

## Code References

### Key Files

1. **Model Discovery**:

    - `src/api/providers/fetchers/ollama.ts` - Main model fetching logic
    - `src/core/webview/webviewMessageHandler.ts:988-1009` - Webview message handler

2. **Connection Setup**:

    - `src/api/providers/native-ollama.ts` - Native Ollama API handler
    - `packages/types/src/provider-settings.ts:257-262` - Configuration schema

3. **Timeouts**:
    - `src/api/providers/fetchers/ollama.ts:87,95` - Model discovery (no timeout)
    - `src/api/providers/native-ollama.ts:163` - Native handler (package-managed)
    - `src/services/code-index/embedders/ollama.ts:10-12` - Embedding timeouts
    - `src/api/providers/utils/timeout-config.ts` - General timeout config (not used for Ollama)

### Key Functions

1. **`getOllamaModels(baseUrl, apiKey)`**

    - **File**: `src/api/providers/fetchers/ollama.ts`
    - **Lines**: 67-127
    - **Purpose**: Discovers available Ollama models

2. **`NativeOllamaHandler.ensureClient()`**

    - **File**: `src/api/providers/native-ollama.ts`
    - **Lines**: 158-179
    - **Purpose**: Initializes the Ollama client connection

3. **`NativeOllamaHandler.createMessage()`**

    - **File**: `src/api/providers/native-ollama.ts`
    - **Lines**: 203-338
    - **Purpose**: Handles chat completion requests

4. **`CodeIndexOllamaEmbedder.createEmbeddings()`**
    - **File**: `src/services/code-index/embedders/ollama.ts`
    - **Lines**: 38-138
    - **Purpose**: Creates embeddings with 60-second timeout

---

## Summary

### Model Discovery

- Uses `GET /api/tags` followed by `POST /api/show` for each model
- No explicit timeout configured (uses axios defaults)
- Filters models to only include those with native tool support

### Connection Setup

- Defaults to `http://localhost:11434`
- Supports optional API key authentication
- Uses the official `ollama` npm package
- Client is lazily initialized

### Timeouts

- **Model Discovery**: No timeout configured (could hang indefinitely)
- **Chat Handler**: Managed by `ollama` npm package (not adjustable)
- **Embeddings**: Hardcoded 60s timeout (not adjustable)
- **Validation**: Hardcoded 30s timeout (not adjustable)
- **General Setting**: `apiRequestTimeout` exists but does NOT affect Ollama

**Conclusion**: Currently, Ollama timeouts are **not adjustable** through Roo Code configuration. The model discovery could benefit from explicit timeout configuration, and the embedding timeouts are hardcoded constants that cannot be changed without modifying the source code.

---

## Related Documentation

- [OLLAMA_SETTINGS_ENHANCEMENT_DESIGN.md](./OLLAMA_SETTINGS_ENHANCEMENT_DESIGN.md) - Design for making Ollama API settings configurable
- [OLLAMA_API_ENHANCEMENTS.md](./OLLAMA_API_ENHANCEMENTS.md) - Enhancements for detailed logging and metrics capture
- [axios-library-api-summary.md](./axios-library-api-summary.md) - Axios configuration options for connection setup, timeouts, retries, and logging
