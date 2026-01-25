# Ollama Model Filtering UI Implementation Plan

## Overview

This plan outlines the implementation of a two-section model display in the Ollama settings UI:

1. **Models with Tools Support** - Selectable models that support native tool calling
2. **Models without Tools Support** - Non-selectable list showing all other discovered models

This provides transparency that all models were discovered while clearly indicating which models are recommended for use with Roo Code.

---

## Table of Contents

1. [Requirements](#requirements)
2. [Design Overview](#design-overview)
3. [Backend Changes](#backend-changes)
4. [Frontend Changes](#frontend-changes)
5. [Type Definitions](#type-definitions)
6. [Translation Keys](#translation-keys)
7. [Testing Requirements](#testing-requirements)
8. [Implementation Phases](#implementation-phases)

---

## Requirements

### Functional Requirements

1. **Model Discovery**: Continue to discover all models from Ollama API
2. **Model Filtering**: Separate models into two groups:
    - Models with `supportsNativeTools: true` (selectable)
    - Models without `supportsNativeTools: true` (non-selectable, informational only)
3. **UI Display**: Show two distinct sections:
    - "Tools Support" section with selectable radio buttons
    - "No Tools Support" section with non-selectable list items
4. **Selection Restriction**: Only models with tools support can be selected
5. **Model Count Display**: Show count of models in each section

### Non-Functional Requirements

1. **Performance**: No degradation in model discovery speed
2. **User Experience**: Clear visual distinction between selectable and non-selectable models
3. **Accessibility**: Proper ARIA labels for screen readers
4. **Responsiveness**: UI should work on different screen sizes

---

## Design Overview

### UI Layout

```
[Model ID Input Field] [Refresh Models Button]

[Success/Error Messages]

┌─ Tools Support (4 models) ─────────────────────────────────────────────────────────────┐
│                                                                                          │
│ ┌────────────────────────────────────────────────────────────────────────────────────┐ │
│ │ Model Name          │ Context │ Size    │ Quantization │ Family │ Images          │ │
│ ├─────────────────────┼─────────┼─────────┼──────────────┼────────┼─────────────────┤ │
│ │ ○ codellama:13b     │ 4,096   │ 7.4 GB  │ Q4_K_M       │ llama  │ No              │ │
│ │ ○ codellama:34b     │ 4,096   │ 19 GB   │ Q4_K_M       │ llama  │ No              │ │
│ │ ○ codellama:7b      │ 4,096   │ 3.8 GB  │ Q4_K_M       │ llama  │ No              │ │
│ │ ● nomic-embed-text  │ 8,192   │ 274 MB  │ -            │ nomic  │ No              │ │
│ └────────────────────────────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────────────────────────┘

┌─ No Tools Support (10 models) ─────────────────────────────────────────────────────────┐
│ These models do not support native tool calling and cannot be used with Roo Code.      │
│                                                                                          │
│ • gpt-oss:latest                                                                         │
│ • llama3-groq-tool-use:70b-q2_K                                                         │
│ • gpt-oss:20b                                                                           │
│ • nemotron-3-nano:latest                                                              │
│ • qwen2.5-coder:32b                                                                     │
│ • qwen3-coder:30b                                                                       │
│ • gemini-3-flash-preview:latest                                                         │
│ • llama3.2:3b                                                                           │
│ • deepseek-r1:32b                                                                       │
│ • llama3.2:latest                                                                       │
└──────────────────────────────────────────────────────────────────────────────────────────┘
```

### Visual Design

- **Tools Support Section**:

    - Section header: "Tools Support (X models)"
    - Radio buttons (○) for selection
    - Models are selectable
    - Selected model has filled radio button (●)

- **No Tools Support Section**:
    - Section header: "No Tools Support (X models)"
    - Bullet points (•) or simple list items (no radio buttons)
    - Models are NOT selectable
    - Grayed out or muted text color
    - Tooltip or help text explaining why they're not selectable

---

## Backend Changes

### 1. Available Model Metadata from Ollama API

Based on the Ollama API response structure, the following metadata is available for each model:

**From `/api/tags` endpoint**:

- `name` - Model name (e.g., "codellama:13b")
- `size` - Model size in bytes (number)
- `modified_at` - Last modification timestamp
- `digest` - Model digest/hash

**From `/api/show` endpoint**:

- `details.family` - Model family (e.g., "llama", "qwen3", "nomic")
- `details.families` - Array of families (can be null)
- `details.parameter_size` - Parameter size as string (e.g., "32.8B", "23.6B")
- `details.quantization_level` - Quantization level (e.g., "Q4_K_M", "Q6_K", or empty)
- `details.format` - Model format (e.g., "gguf")
- `details.parent_model` - Parent model path
- `model_info` - Object containing architecture-specific info:
    - `context_length` - Found by searching for keys containing "context_length" (e.g., "qwen3.context_length", "ollama.context_length")
- `capabilities` - Array of capabilities:
    - `"tools"` - Native tool calling support
    - `"vision"` - Image support
    - `"completion"` - Text completion
- `modelfile` - Modelfile content
- `parameters` - Model parameters
- `template` - Template string

**Metadata to Display in Table**:

1. **Model Name** - From `name` field
2. **Context Window** - From `model_info` (search for "context_length" key)
3. **Model Size** - From `size` field (format as GB/MB)
4. **Quantization Level** - From `details.quantization_level` (or "-" if not available)
5. **Family** - From `details.family`
6. **Supports Images** - From `capabilities.includes("vision")`

### 2. Update `getOllamaModels` Function

**File**: `src/api/providers/fetchers/ollama.ts`

**Changes**:

- Keep the current filtering logic (return `null` for models without tools support)
- Ensure all metadata is preserved in the returned `ModelInfo` objects
- The existing code already extracts context window and other metadata

**Rationale**: The backend already filters models correctly and extracts most metadata. We may need to enhance the `ModelInfo` type or create an extended type for table display.

### 2. Add New Function: `getOllamaModelsWithFiltering`

**File**: `src/api/providers/fetchers/ollama.ts`

**Purpose**: Return both filtered (tools support) and unfiltered (all models) results

**Signature**:

```typescript
export async function getOllamaModelsWithFiltering(
	baseUrl = "http://localhost:11434",
	apiKey?: string,
	config?: {
		timeout?: number
		modelDiscoveryTimeout?: number
		maxRetries?: number
		retryDelay?: number
		enableLogging?: boolean
	},
): Promise<{
	modelsWithTools: Record<string, ModelInfo>
	modelsWithoutTools: string[] // Just model names, no ModelInfo needed
	totalCount: number
}>
```

**Implementation**:

1. Call existing `getOllamaModels` to get models with tools support
2. Make additional API call to `/api/tags` to get ALL model names
3. Compare lists to identify models without tools support
4. Return structured result with both lists

**Alternative Approach** (Simpler):

- Modify `getOllamaModels` to return both filtered and unfiltered results
- Add optional parameter to control return format

**Recommended**: Modify existing function to return both lists when requested.

### 3. Update Message Handler

**File**: `src/core/webview/webviewMessageHandler.ts`

**Changes**:

- Update `case "refreshOllamaModels"` to call new function or modified function
- Return both `ollamaModels` (with tools) and `ollamaModelsWithoutTools` (names only) in message

---

## Frontend Changes

### 1. Update Type Definitions

**File**: `packages/types/src/vscode-extension-host.ts`

**Changes**:

- Update `ollamaModelsRefreshResult` message type to include:

    ```typescript
    {
      type: "ollamaModelsRefreshResult"
      success: boolean
      message: string
      durationMs?: number
      modelsWithTools?: ModelRecord
      modelsWithoutTools?: string[] // Array of model names
      totalCount?: number
    }
    ```

- Update `ollamaModels` message type (if needed):
    ```typescript
    {
      type: "ollamaModels"
      ollamaModels: ModelRecord
      modelsWithoutTools?: string[] // Optional array of model names
    }
    ```

### 2. Update Ollama Component State

**File**: `webview-ui/src/components/settings/providers/Ollama.tsx`

**Changes**:

- Add new state variable:

    ```typescript
    const [modelsWithoutTools, setModelsWithoutTools] = useState<string[]>([])
    ```

- Update `onMessage` handler to process `modelsWithoutTools` from messages

### 3. Update UI Rendering

**File**: `webview-ui/src/components/settings/providers/Ollama.tsx`

**Current Code** (lines ~222-234):

```typescript
{Object.keys(ollamaModels).length > 0 && (
	<VSCodeRadioGroup
		value={
			(apiConfiguration?.ollamaModelId || "") in ollamaModels ? apiConfiguration?.ollamaModelId : ""
		}
		onChange={handleInputChange("ollamaModelId")}>
		{Object.keys(ollamaModels).map((model) => (
			<VSCodeRadio key={model} value={model} checked={apiConfiguration?.ollamaModelId === model}>
				{model}
			</VSCodeRadio>
		))}
	</VSCodeRadioGroup>
)}
```

**New Code**:

```typescript
{/* Tools Support Section */}
{Object.keys(ollamaModels).length > 0 && (
	<div className="flex flex-col gap-2 mt-4">
		<div className="text-sm font-medium text-vscode-foreground">
			{t("settings:providers.ollama.toolsSupport")} ({Object.keys(ollamaModels).length} {t("settings:providers.ollama.models")})
		</div>
		<VSCodeRadioGroup
			value={
				(apiConfiguration?.ollamaModelId || "") in ollamaModels ? apiConfiguration?.ollamaModelId : ""
			}
			onChange={handleInputChange("ollamaModelId")}>
			{Object.keys(ollamaModels).map((model) => (
				<VSCodeRadio key={model} value={model} checked={apiConfiguration?.ollamaModelId === model}>
					{model}
				</VSCodeRadio>
			))}
		</VSCodeRadioGroup>
	</div>
)}

{/* No Tools Support Section */}
{modelsWithoutTools.length > 0 && (
	<div className="flex flex-col gap-2 mt-4">
		<div className="text-sm font-medium text-vscode-descriptionForeground">
			{t("settings:providers.ollama.noToolsSupport")} ({modelsWithoutTools.length} {t("settings:providers.ollama.models")})
		</div>
		<div className="text-xs text-vscode-descriptionForeground mb-2">
			{t("settings:providers.ollama.noToolsSupportHelp")}
		</div>
		<div className="flex flex-col gap-1 pl-4">
			{modelsWithoutTools.map((model) => (
				<div
					key={model}
					className="text-sm text-vscode-descriptionForeground flex items-center gap-2">
					<span className="codicon codicon-circle-small" />
					<span>{model}</span>
				</div>
			))}
		</div>
	</div>
)}
```

### 4. Update Message Handlers

**File**: `webview-ui/src/components/settings/providers/Ollama.tsx`

**Changes**:

- Update `case "ollamaModels"` to also set `modelsWithoutTools`:

    ```typescript
    case "ollamaModels":
      {
        const newModels = message.ollamaModels ?? {}
        setOllamaModels(newModels)
        setModelsWithoutTools(message.modelsWithoutTools ?? [])
      }
      break
    ```

- Update `case "ollamaModelsRefreshResult"` to also set `modelsWithoutTools`:
    ```typescript
    case "ollamaModelsRefreshResult":
      setRefreshResult({
        success: message.success ?? false,
        message: message.message ?? "Unknown error",
        durationMs: message.durationMs,
      })
      setRefreshingModels(false)
      if (message.modelsWithoutTools) {
        setModelsWithoutTools(message.modelsWithoutTools)
      }
      // ... rest of handler
      break
    ```

---

## Type Definitions

### Backend Types

**File**: `src/api/providers/fetchers/ollama.ts`

Add return type for filtered results:

```typescript
export interface OllamaModelsResult {
	modelsWithTools: Record<string, ModelInfo>
	modelsWithoutTools: string[]
	totalCount: number
}
```

### Frontend Types

**File**: `packages/types/src/vscode-extension-host.ts`

Update message types as described in Frontend Changes section.

---

## Translation Keys

**File**: `webview-ui/src/i18n/locales/en/settings.json`

**Add new keys under `providers.ollama`**:

```json
{
	"toolsSupport": "Tools Support",
	"noToolsSupport": "No Tools Support",
	"models": "models",
	"noToolsSupportHelp": "These models do not support native tool calling and cannot be used with Roo Code. They are shown for reference only."
}
```

**Translation Notes**:

- Keep "Tools Support" and "No Tools Support" concise
- "models" is a simple plural form (may need localization for other languages)
- Help text should be clear but concise

---

## Testing Requirements

### Unit Tests

**File**: `src/api/providers/fetchers/__tests__/ollama.test.ts`

**Test Cases**:

1. `getOllamaModelsWithFiltering` returns correct structure
2. Models with tools support are in `modelsWithTools`
3. Models without tools support are in `modelsWithoutTools`
4. `totalCount` matches sum of both lists
5. Empty results handled correctly
6. All models have tools support (edge case)
7. No models have tools support (edge case)

### Integration Tests

**Test Scenarios**:

1. Refresh models shows both sections correctly
2. Only tools support models are selectable
3. Non-tools models are displayed but not selectable
4. Model counts are accurate
5. UI updates correctly when models are refreshed
6. Error handling when API fails

### Manual Testing

**Test Checklist**:

- [ ] Refresh models button works
- [ ] Tools support section shows correct models
- [ ] No tools support section shows correct models
- [ ] Model counts are accurate
- [ ] Only tools support models can be selected
- [ ] Non-tools models are visually distinct (grayed out)
- [ ] Help text is clear and helpful
- [ ] UI is responsive on different screen sizes
- [ ] Accessibility: screen readers can distinguish sections

---

## Implementation Phases

### Phase 1: Backend Changes

**Duration**: 2-3 hours

**Tasks**:

1. Create `OllamaExtendedModelInfo` type to include size, quantization, family
2. Modify `getOllamaModels` to:
    - Preserve `size` from `/api/tags` response
    - Preserve `quantization_level` and `family` from `/api/show` response
    - Return extended model info with all metadata
3. Modify function to optionally return both filtered and unfiltered results
4. Add new return type `OllamaModelsResult` with both lists
5. Update function to fetch all model names and compare with filtered results
6. Add unit tests for new functionality
7. Update message handler to return both lists

**Definition of Done**:

- [ ] `OllamaExtendedModelInfo` type created
- [ ] Function preserves size, quantization, and family metadata
- [ ] Function returns both `modelsWithTools` and `modelsWithoutTools`
- [ ] All unit tests pass
- [ ] Message handler includes both lists in response

### Phase 2: Type Definitions

**Duration**: 30 minutes

**Tasks**:

1. Update `ExtensionMessage` types to include `modelsWithoutTools`
2. Update `WebviewMessage` types if needed
3. Verify TypeScript compilation

**Definition of Done**:

- [ ] Types are correctly defined
- [ ] No TypeScript errors
- [ ] Types are exported correctly

### Phase 3: Frontend State Management

**Duration**: 1 hour

**Tasks**:

1. Add `modelsWithoutTools` state variable
2. Update `onMessage` handlers to process new data
3. Verify state updates correctly

**Definition of Done**:

- [ ] State variable added
- [ ] Message handlers updated
- [ ] State updates correctly on refresh

### Phase 4: UI Implementation

**Duration**: 2-3 hours

**Tasks**:

1. Add "Tools Support" section with header and count
2. Add "No Tools Support" section with header, count, and help text
3. Style non-selectable models appropriately
4. Ensure proper spacing and layout
5. Add accessibility attributes

**Definition of Done**:

- [ ] Both sections render correctly
- [ ] Model counts are accurate
- [ ] Visual distinction is clear
- [ ] Only tools support models are selectable
- [ ] UI is accessible

### Phase 5: Translation Keys

**Duration**: 30 minutes

**Tasks**:

1. Add translation keys to `en/settings.json`
2. Verify keys are used correctly in component
3. Test with English locale

**Definition of Done**:

- [ ] All translation keys added
- [ ] Keys are used correctly
- [ ] English translations are clear

### Phase 6: Testing and Refinement

**Duration**: 1-2 hours

**Tasks**:

1. Run unit tests
2. Manual testing with various model configurations
3. Test edge cases (all models have tools, no models have tools)
4. Verify accessibility
5. Fix any issues found

**Definition of Done**:

- [ ] All tests pass
- [ ] Manual testing complete
- [ ] Edge cases handled
- [ ] No accessibility issues
- [ ] Code review ready

---

## Performance Considerations

1. **API Calls**: Only one additional API call needed (`/api/tags` to get all model names)
2. **Data Processing**: Minimal - just comparing two arrays
3. **UI Rendering**: Both sections render independently, no performance impact
4. **State Management**: Simple array state, no complex updates

---

## Accessibility Considerations

1. **Section Headers**: Use proper heading levels or `aria-label`
2. **Radio Buttons**: Properly labeled and keyboard navigable
3. **Non-selectable Models**: Use `aria-disabled="true"` or `role="listitem"`
4. **Screen Readers**: Clear distinction between sections announced
5. **Keyboard Navigation**: Tab order should skip non-selectable items

---

## Future Enhancements (Out of Scope)

1. Search/filter within each section
2. Sort models alphabetically or by size
3. Show model details (size, context window) in non-tools section
4. Warning when selecting non-tools model (if filtering is removed)
5. Configurable filtering (allow users to show/hide non-tools models)

---

## Notes

- This implementation maintains backward compatibility (existing `ollamaModels` still works)
- The filtering logic remains the same (models without tools are filtered out for selection)
- The UI provides transparency without allowing problematic selections
- All discovered models are shown, making it clear that filtering is intentional
