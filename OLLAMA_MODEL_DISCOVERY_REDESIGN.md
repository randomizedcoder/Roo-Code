# Ollama Model Discovery Redesign

## Table of Contents

1. [Overview](#overview)
2. [Current Issues](#current-issues)
3. [Design Goals](#design-goals)
4. [API Call Strategy](#api-call-strategy)
5. [Data Structure](#data-structure)
6. [Implementation Plan](#implementation-plan)
7. [Code Changes](#code-changes)
8. [Testing Strategy](#testing-strategy)

---

## Overview

This document outlines the redesign of the Ollama model discovery process to:

- Minimize API calls
- Efficiently sort models into "Tools Support" and "No Tools Support" groups
- Display tools models in a table format with detailed metadata
- Display non-tools models as a simple list

**Related Documentation**:

- [OLLAMA_MODEL_FILTERING_UI_PLAN.md](./OLLAMA_MODEL_FILTERING_UI_PLAN.md) - Original UI design plan
- [OLLAMA_INTEGRATION_DOCUMENTATION.md](./OLLAMA_INTEGRATION_DOCUMENTATION.md) - Current Ollama integration details
- [OLLAMA_API_ENHANCEMENTS.md](./OLLAMA_API_ENHANCEMENTS.md) - Ollama API capabilities

---

## Current Issues

1. **Multiple API Calls**: Currently making one `/api/tags` call plus one `/api/show` call per model
2. **Inefficient Filtering**: Models are being filtered during discovery, making it hard to separate into two groups
3. **Missing Models**: Models with tools support are not being detected correctly
4. **Complex Logic**: The filtering logic in `getOllamaModels` and `getOllamaModelsWithFiltering` is confusing

---

## Design Goals

1. **Single Discovery Pass**: Make one `/api/tags` call, then batch `/api/show` calls for all models in parallel
2. **Clear Separation**: Sort all discovered models into two distinct groups:
    - **Tools Support**: Models with `capabilities.includes("tools")`
    - **No Tools Support**: All other models
3. **Efficient Data Collection**: Collect all necessary metadata in one pass
4. **Simple Logic**: Clear, straightforward sorting logic without complex filtering

---

## API Call Strategy

### Step 1: Get All Models

**Endpoint**: `GET /api/tags`
**Purpose**: Get list of all available models
**Response**: Array of model objects with basic info (name, size, modified_at)

### Step 2: Get Detailed Info for All Models (Parallel)

**Endpoint**: `POST /api/show` (called once per model, in parallel)
**Purpose**: Get detailed information including capabilities
**Response**: Model details including:

- `capabilities`: Array of strings (e.g., `["completion", "tools"]`)
- `details`: Family, quantization, parameter size
- `model_info`: Context window and other metadata

### Implementation

```typescript
// Pseudo-code
const allModels = await axios.get("/api/tags")
const detailPromises = allModels.data.models.map((model) => axios.post("/api/show", { model: model.name }))
const allDetails = await Promise.all(detailPromises)

// Now sort into two groups
const modelsWithTools = []
const modelsWithoutTools = []

for (const detail of allDetails) {
	if (detail.data.capabilities?.includes("tools")) {
		modelsWithTools.push({
			name: detail.data.name,
			...extractTableData(detail.data),
		})
	} else {
		modelsWithoutTools.push(detail.data.name)
	}
}
```

---

## Data Structure

### Backend Return Type

```typescript
export interface OllamaModelsDiscoveryResult {
	modelsWithTools: OllamaModelWithTools[]
	modelsWithoutTools: string[] // Just names, no detailed info needed
	totalCount: number
}

export interface OllamaModelWithTools {
	name: string
	contextWindow: number
	size?: number // In bytes, from /api/tags response
	quantizationLevel?: string
	family?: string
	supportsImages: boolean
	modelInfo: ModelInfo // Full ModelInfo for compatibility
}
```

### Frontend State

```typescript
const [modelsWithTools, setModelsWithTools] = useState<OllamaModelWithTools[]>([])
const [modelsWithoutTools, setModelsWithoutTools] = useState<string[]>([])
```

---

## Implementation Plan

### Phase 1: Backend - Redesign Discovery Function

**File**: `src/api/providers/fetchers/ollama.ts`

**New Function**: `discoverOllamaModelsWithSorting`

**Steps**:

1. Call `/api/tags` to get all models
2. Create parallel promises for `/api/show` for each model
3. Wait for all details to be fetched
4. Sort results into two arrays:
    - Models with `capabilities.includes("tools")` → `modelsWithTools`
    - All other models → `modelsWithoutTools` (just names)
5. Extract table metadata for tools models
6. Return `OllamaModelsDiscoveryResult`

**Key Changes**:

- Remove filtering logic from `getOllamaModels`
- Create new dedicated function for discovery with sorting
- Keep `getOllamaModels` for backward compatibility (but simplify it)

### Phase 2: Backend - Update Message Handler

**File**: `src/core/webview/webviewMessageHandler.ts`

**Changes**:

- Update `refreshOllamaModels` to use `discoverOllamaModelsWithSorting`
- Update `requestOllamaModels` to use `discoverOllamaModelsWithSorting`
- Pass both arrays to frontend

### Phase 3: Types - Update Message Types

**File**: `packages/types/src/vscode-extension-host.ts`

**Changes**:

- Update `ExtensionMessage` to include new structure
- Add `OllamaModelWithTools` type (or import from ollama.ts)

### Phase 4: Frontend - Update State and Handlers

**File**: `webview-ui/src/components/settings/providers/Ollama.tsx`

**Changes**:

- Update state to use `OllamaModelWithTools[]` instead of `ModelRecord`
- Update message handlers to process new data structure
- Table rendering already implemented (just needs data structure update)

### Phase 5: Testing

- Unit tests for sorting logic
- Integration tests for API calls
- Manual testing with various model configurations

---

## Code Changes

### Backend Function Signature

```typescript
export async function discoverOllamaModelsWithSorting(
	baseUrl = "http://localhost:11434",
	apiKey?: string,
	config?: {
		timeout?: number
		modelDiscoveryTimeout?: number
		maxRetries?: number
		retryDelay?: number
		enableLogging?: boolean
	},
): Promise<OllamaModelsDiscoveryResult> {
	// Implementation
}
```

### Sorting Logic

```typescript
const modelsWithTools: OllamaModelWithTools[] = []
const modelsWithoutTools: string[] = []

for (let i = 0; i < allModels.data.models.length; i++) {
	const model = allModels.data.models[i]
	const details = allDetails[i].data

	if (details.capabilities?.includes("tools")) {
		// Extract table data
		const contextKey = Object.keys(details.model_info).find((k) => k.includes("context_length"))
		const contextWindow =
			contextKey && typeof details.model_info[contextKey] === "number"
				? details.model_info[contextKey]
				: undefined

		modelsWithTools.push({
			name: model.name,
			contextWindow: contextWindow || ollamaDefaultModelInfo.contextWindow,
			size: model.size,
			quantizationLevel: details.details.quantization_level,
			family: details.details.family,
			supportsImages: details.capabilities?.includes("vision") ?? false,
			modelInfo: parseOllamaModel(details, model.size), // For compatibility
		})
	} else {
		modelsWithoutTools.push(model.name)
	}
}
```

### Frontend Table Rendering

```typescript
{modelsWithTools.map((model) => (
  <tr key={model.name} ...>
    <td>
      <VSCodeRadio ... />
      <span>{model.name}</span>
    </td>
    <td>{model.contextWindow.toLocaleString()}</td>
    <td>{formatSize(model.size)}</td>
    <td>{model.quantizationLevel || "-"}</td>
    <td>{model.family || "-"}</td>
    <td>{model.supportsImages ? "Yes" : "No"}</td>
  </tr>
))}
```

---

## Testing Strategy

### Unit Tests

1. **Sorting Logic**:

    - Test with models that have tools capability
    - Test with models that don't have tools capability
    - Test with models that have undefined capabilities
    - Test with mixed scenarios

2. **Data Extraction**:
    - Test context window extraction
    - Test size formatting
    - Test quantization level extraction
    - Test family extraction

### Integration Tests

1. **API Calls**:

    - Mock `/api/tags` response
    - Mock multiple `/api/show` responses
    - Verify parallel execution
    - Verify error handling

2. **End-to-End**:
    - Test full discovery flow
    - Verify both groups are populated correctly
    - Verify UI displays correctly

### Manual Testing Checklist

- [ ] Refresh models button works
- [ ] Models with tools appear in table
- [ ] Models without tools appear in list
- [ ] Table columns display correct data
- [ ] Radio button selection works
- [ ] Error handling works for API failures
- [ ] Performance is acceptable (no noticeable delay)

---

## Performance Considerations

1. **Parallel API Calls**: All `/api/show` calls should be made in parallel using `Promise.all()`
2. **Timeout Handling**: Use `modelDiscoveryTimeout` for the entire discovery process
3. **Error Handling**: If one model fails, continue with others
4. **Caching**: Consider caching results to avoid repeated API calls

---

## Migration Notes

- `getOllamaModels` can remain for backward compatibility but should be simplified
- `getOllamaModelsWithFiltering` can be deprecated in favor of `discoverOllamaModelsWithSorting`
- Update all callers to use the new function
- Remove debug logging after verification

---

## Related Files

- `src/api/providers/fetchers/ollama.ts` - Main implementation
- `src/core/webview/webviewMessageHandler.ts` - Message handlers
- `packages/types/src/vscode-extension-host.ts` - Type definitions
- `webview-ui/src/components/settings/providers/Ollama.tsx` - Frontend UI
- `webview-ui/src/i18n/locales/en/settings.json` - Translation keys
