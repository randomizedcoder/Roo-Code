# Ollama API Enhancements for Detailed Logging and Metrics

## Table of Contents

1. [Overview](#overview)
2. [Current Streaming Implementation](#current-streaming-implementation)
3. [Available Ollama API Metrics](#available-ollama-api-metrics)
4. [Enhanced Logging Implementation](#enhanced-logging-implementation)
5. [Streaming Response Details](#streaming-response-details)
6. [Code References](#code-references)
7. [Related Documentation](#related-documentation)

---

## Overview

This document outlines enhancements to Roo Code's Ollama integration to leverage the full capabilities of the Ollama API, particularly:

1. **Detailed Performance Metrics**: Capture and log timing information, token rates, and performance statistics
2. **Enhanced Logging**: Log comprehensive request/response data including all available Ollama API metrics
3. **Streaming Support**: Document and verify that streaming is properly implemented (it already is)

**Related Documentation**:

- [OLLAMA_INTEGRATION_DOCUMENTATION.md](./OLLAMA_INTEGRATION_DOCUMENTATION.md) - Current Ollama integration details
- [OLLAMA_SETTINGS_ENHANCEMENT_DESIGN.md](./OLLAMA_SETTINGS_ENHANCEMENT_DESIGN.md) - Design for configurable settings including logging
- [axios-library-api-summary.md](./axios-library-api-summary.md) - Axios configuration options

---

## Current Streaming Implementation

### ✅ Streaming is Already Implemented

Roo Code **already uses streaming** for Ollama API requests. The implementation is in `src/api/providers/native-ollama.ts`.

**File**: `src/api/providers/native-ollama.ts`
**Lines**: 243-250

```typescript:243:250:src/api/providers/native-ollama.ts
// Create the actual API request promise
const stream = await client.chat({
	model: modelId,
	messages: ollamaMessages,
	stream: true,  // ✅ Streaming is enabled
	options: chatOptions,
	// Native tool calling support
	...(useNativeTools && { tools: this.convertToolsToOllama(metadata.tools) }),
})
```

### Current Metrics Captured

The current implementation tracks:

- `prompt_eval_count` (input tokens) - **Lines 286-289**
- `eval_count` (output tokens) - **Lines 290-293**

**File**: `src/api/providers/native-ollama.ts`
**Lines**: 285-315

```typescript:285:315:src/api/providers/native-ollama.ts
// Handle token usage if available
if (chunk.eval_count !== undefined || chunk.prompt_eval_count !== undefined) {
	if (chunk.prompt_eval_count) {
		totalInputTokens = chunk.prompt_eval_count
	}
	if (chunk.eval_count) {
		totalOutputTokens = chunk.eval_count
	}
}
// ... later ...
// Yield usage information if available
if (totalInputTokens > 0 || totalOutputTokens > 0) {
	yield {
		type: "usage",
		inputTokens: totalInputTokens,
		outputTokens: totalOutputTokens,
	}
}
```

### Missing Metrics

The following detailed metrics are **available from Ollama but not currently captured**:

1. **Timing Metrics**:

    - `total_duration` - Total time spent generating the response (nanoseconds)
    - `load_duration` - Time spent loading the model into memory (nanoseconds)
    - `prompt_eval_duration` - Time spent evaluating the prompt (nanoseconds)
    - `eval_duration` - Time spent generating the response (nanoseconds)

2. **Calculated Metrics**:
    - **Tokens per second** (output): `eval_count / eval_duration * 10^9`
    - **Prompt evaluation rate**: `prompt_eval_count / prompt_eval_duration * 10^9`
    - **Model load time**: `load_duration / 10^9` (convert to seconds)

---

## Available Ollama API Metrics

Based on the [Ollama API documentation](https://github.com/ollama/ollama/blob/main/docs/api.md), the final chunk in a streaming response includes comprehensive statistics.

### Response Structure

**From Ollama API Documentation** (lines 108-134 of `api.md`):

The final response in the stream includes:

```json
{
	"model": "llama3.2",
	"created_at": "2023-08-04T19:22:45.499127Z",
	"message": {
		"role": "assistant",
		"content": ""
	},
	"done": true,
	"total_duration": 4883583458, // Total time in nanoseconds
	"load_duration": 1334875, // Model load time in nanoseconds
	"prompt_eval_count": 26, // Input tokens
	"prompt_eval_duration": 342546000, // Prompt evaluation time in nanoseconds
	"eval_count": 282, // Output tokens
	"eval_duration": 4535599000 // Response generation time in nanoseconds
}
```

### Metric Descriptions

| Metric                 | Type     | Description                              | Unit        |
| ---------------------- | -------- | ---------------------------------------- | ----------- |
| `total_duration`       | `number` | Total time spent generating the response | nanoseconds |
| `load_duration`        | `number` | Time spent loading the model into memory | nanoseconds |
| `prompt_eval_count`    | `number` | Number of tokens in the prompt           | tokens      |
| `prompt_eval_duration` | `number` | Time spent evaluating the prompt         | nanoseconds |
| `eval_count`           | `number` | Number of tokens in the response         | tokens      |
| `eval_duration`        | `number` | Time spent generating the response       | nanoseconds |

### Calculated Metrics

From these raw metrics, we can calculate:

1. **Output Token Rate** (tokens/second):

    ```
    tokens_per_second = eval_count / (eval_duration / 10^9)
    ```

2. **Prompt Evaluation Rate** (tokens/second):

    ```
    prompt_tokens_per_second = prompt_eval_count / (prompt_eval_duration / 10^9)
    ```

3. **Model Load Time** (seconds):

    ```
    load_time_seconds = load_duration / 10^9
    ```

4. **Total Request Time** (seconds):

    ```
    total_time_seconds = total_duration / 10^9
    ```

5. **Time to First Token** (seconds):
    ```
    time_to_first_token = (load_duration + prompt_eval_duration) / 10^9
    ```

---

## Enhanced Logging Implementation

### 1. Capture Additional Metrics in Stream Handler

**File**: `src/api/providers/native-ollama.ts`

**Current Implementation** (lines 252-315):

- Only tracks `eval_count` and `prompt_eval_count`
- Does not capture timing metrics

**Enhanced Implementation**:

```typescript
let totalInputTokens = 0
let totalOutputTokens = 0
// Add tracking for timing metrics
let totalDuration: number | undefined
let loadDuration: number | undefined
let promptEvalDuration: number | undefined
let evalDuration: number | undefined

try {
	for await (const chunk of stream) {
		// ... existing content and tool call handling ...

		// Enhanced: Capture all available metrics from final chunk
		if (chunk.done === true) {
			// Final chunk contains all statistics
			if (chunk.total_duration !== undefined) {
				totalDuration = chunk.total_duration
			}
			if (chunk.load_duration !== undefined) {
				loadDuration = chunk.load_duration
			}
			if (chunk.prompt_eval_duration !== undefined) {
				promptEvalDuration = chunk.prompt_eval_duration
			}
			if (chunk.eval_duration !== undefined) {
				evalDuration = chunk.eval_duration
			}
		}

		// Handle token usage if available (existing code)
		if (chunk.eval_count !== undefined || chunk.prompt_eval_count !== undefined) {
			if (chunk.prompt_eval_count) {
				totalInputTokens = chunk.prompt_eval_count
			}
			if (chunk.eval_count) {
				totalOutputTokens = chunk.eval_count
			}
		}
	}

	// ... existing matcher and tool call end handling ...

	// Enhanced: Log detailed metrics if logging is enabled
	if (this.options.ollamaEnableLogging) {
		this.logOllamaMetrics({
			modelId,
			inputTokens: totalInputTokens,
			outputTokens: totalOutputTokens,
			totalDuration,
			loadDuration,
			promptEvalDuration,
			evalDuration,
		})
	}

	// Yield usage information if available
	if (totalInputTokens > 0 || totalOutputTokens > 0) {
		yield {
			type: "usage",
			inputTokens: totalInputTokens,
			outputTokens: totalOutputTokens,
		}
	}
}
```

### 2. Add Logging Method

**File**: `src/api/providers/native-ollama.ts`

Add a new private method to log detailed metrics:

```typescript
/**
 * Logs detailed Ollama API metrics when logging is enabled.
 *
 * @param metrics - Metrics object containing all available Ollama statistics
 */
private logOllamaMetrics(metrics: {
	modelId: string
	inputTokens: number
	outputTokens: number
	totalDuration?: number
	loadDuration?: number
	promptEvalDuration?: number
	evalDuration?: number
}): void {
	const {
		modelId,
		inputTokens,
		outputTokens,
		totalDuration,
		loadDuration,
		promptEvalDuration,
		evalDuration,
	} = metrics

	// Calculate derived metrics
	const calculatedMetrics: Record<string, number | string> = {}

	if (evalDuration && outputTokens > 0) {
		// Tokens per second (output)
		calculatedMetrics.outputTokensPerSecond = (outputTokens / (evalDuration / 1e9)).toFixed(2)
	}

	if (promptEvalDuration && inputTokens > 0) {
		// Prompt evaluation rate
		calculatedMetrics.promptTokensPerSecond = (inputTokens / (promptEvalDuration / 1e9)).toFixed(2)
	}

	if (loadDuration !== undefined) {
		calculatedMetrics.modelLoadTimeSeconds = (loadDuration / 1e9).toFixed(3)
	}

	if (totalDuration !== undefined) {
		calculatedMetrics.totalTimeSeconds = (totalDuration / 1e9).toFixed(3)
	}

	if (loadDuration !== undefined && promptEvalDuration !== undefined) {
		calculatedMetrics.timeToFirstTokenSeconds = ((loadDuration + promptEvalDuration) / 1e9).toFixed(3)
	}

	// Log comprehensive metrics
	console.debug('[Ollama API Metrics]', {
		model: modelId,
		tokens: {
			input: inputTokens,
			output: outputTokens,
			total: inputTokens + outputTokens,
		},
		timing: {
			totalDurationNs: totalDuration,
			loadDurationNs: loadDuration,
			promptEvalDurationNs: promptEvalDuration,
			evalDurationNs: evalDuration,
		},
		calculated: calculatedMetrics,
		timestamp: new Date().toISOString(),
	})
}
```

### 3. Integration with Axios Logging

When the `ollamaEnableLogging` setting is enabled (as designed in `OLLAMA_SETTINGS_ENHANCEMENT_DESIGN.md`), the Axios interceptors will log request/response details, and the enhanced stream handler will log detailed metrics.

**File**: `src/api/providers/fetchers/ollama-axios-config.ts` (from design document)

The Axios logging interceptor (lines 359-396 of design document) will capture:

- Request method, URL, timeout
- Response status, duration
- Error details

The enhanced stream handler will capture:

- Token counts
- All timing metrics
- Calculated performance metrics

---

## Streaming Response Details

### How Streaming Works in Ollama

According to the Ollama API documentation:

1. **Streaming is the default**: Most endpoints stream responses as JSON objects
2. **Final chunk contains statistics**: The last chunk in the stream has `done: true` and includes all metrics
3. **Intermediate chunks**: Contain partial content with `done: false`

### Example Stream Flow

```typescript
// Chunk 1: Partial content
{
  "model": "llama3.2",
  "created_at": "2023-08-04T08:52:19.385406455-07:00",
  "message": {
    "role": "assistant",
    "content": "The"
  },
  "done": false
}

// Chunk 2: More partial content
{
  "model": "llama3.2",
  "created_at": "2023-08-04T08:52:19.385406455-07:00",
  "message": {
    "role": "assistant",
    "content": " sky"
  },
  "done": false
}

// Final chunk: Complete with statistics
{
  "model": "llama3.2",
  "created_at": "2023-08-04T19:22:45.499127Z",
  "message": {
    "role": "assistant",
    "content": ""
  },
  "done": true,
  "total_duration": 4883583458,
  "load_duration": 1334875,
  "prompt_eval_count": 26,
  "prompt_eval_duration": 342546000,
  "eval_count": 282,
  "eval_duration": 4535599000
}
```

### Current Implementation Status

✅ **Streaming is properly implemented**:

- `stream: true` is set in the API call (line 246)
- Stream is processed with `for await (const chunk of stream)` (line 260)
- Content is yielded incrementally (lines 261-265)
- Tool calls are handled in stream (lines 268-283)

⚠️ **Metrics capture can be enhanced**:

- Currently only captures token counts
- Does not capture timing metrics
- Does not calculate performance rates

---

## Code References

### Key Files

1. **Native Ollama Handler**:

    - `src/api/providers/native-ollama.ts` - Main handler with streaming implementation
    - **Lines 203-338**: `createMessage` method with streaming
    - **Lines 285-315**: Current metrics capture (token counts only)

2. **Ollama Axios Configuration** (from design document):

    - `src/api/providers/fetchers/ollama-axios-config.ts` - Axios instance configuration
    - **Lines 359-396**: Logging interceptor setup

3. **Provider Settings**:
    - `packages/types/src/provider-settings.ts` - Schema definitions
    - Includes `ollamaEnableLogging` boolean flag

### Comparison with Other Providers

Roo Code uses streaming for multiple providers:

1. **Claude Code** (`src/api/providers/claude-code.ts`):

    - Uses streaming with detailed metrics
    - Captures cache read/write tokens
    - **Lines 203-268**: Stream processing

2. **Anthropic** (`src/api/providers/anthropic.ts`):

    - Streaming implementation
    - Captures usage metrics

3. **OpenAI** (`src/api/providers/openai-native.ts`):
    - Streaming with usage tracking
    - Token counting

**Ollama should match this level of detail** by capturing all available metrics.

---

## Implementation Recommendations

### Priority 1: Capture All Metrics

1. **Update stream handler** to capture all timing metrics from final chunk
2. **Store metrics** in variables during stream processing
3. **Log metrics** when `ollamaEnableLogging` is enabled

### Priority 2: Calculate Derived Metrics

1. **Calculate tokens/second** for output generation
2. **Calculate prompt evaluation rate**
3. **Calculate time to first token**
4. **Log all calculated metrics** in a structured format

### Priority 3: Integration with Settings

1. **Use `ollamaEnableLogging` setting** to control detailed logging
2. **Integrate with Axios logging** from design document
3. **Provide consistent logging format** across all Ollama API calls

### Example Enhanced Log Output

When `ollamaEnableLogging` is enabled, the log output should look like:

```
[Ollama API Metrics] {
  model: "llama3-groq-tool-use:70b-q2_K",
  tokens: {
    input: 26,
    output: 282,
    total: 308
  },
  timing: {
    totalDurationNs: 4883583458,
    loadDurationNs: 1334875,
    promptEvalDurationNs: 342546000,
    evalDurationNs: 4535599000
  },
  calculated: {
    outputTokensPerSecond: "62.15",
    promptTokensPerSecond: "75.92",
    modelLoadTimeSeconds: "0.001",
    totalTimeSeconds: "4.884",
    timeToFirstTokenSeconds: "0.344"
  },
  timestamp: "2026-01-24T19:52:45.499Z"
}
```

---

## Related Documentation

- [OLLAMA_INTEGRATION_DOCUMENTATION.md](./OLLAMA_INTEGRATION_DOCUMENTATION.md) - Current Ollama integration details
- [OLLAMA_SETTINGS_ENHANCEMENT_DESIGN.md](./OLLAMA_SETTINGS_ENHANCEMENT_DESIGN.md) - Design for configurable settings including logging
- [axios-library-api-summary.md](./axios-library-api-summary.md) - Axios configuration options
- [Ollama API Documentation](https://github.com/ollama/ollama/blob/main/docs/api.md) - Official Ollama API reference

---

## Summary

### Current State

✅ **Streaming is implemented**: Roo Code already uses streaming for Ollama API requests
✅ **Basic metrics captured**: Token counts (input/output) are tracked
⚠️ **Detailed metrics missing**: Timing metrics and performance rates are not captured

### Recommended Enhancements

1. **Capture all timing metrics** from the final stream chunk:

    - `total_duration`
    - `load_duration`
    - `prompt_eval_duration`
    - `eval_duration`

2. **Calculate performance metrics**:

    - Output tokens per second
    - Prompt evaluation rate
    - Time to first token
    - Model load time

3. **Integrate with logging settings**:

    - Use `ollamaEnableLogging` flag to control detailed logging
    - Provide structured log output with all metrics
    - Integrate with Axios logging interceptors

4. **Maintain consistency**:
    - Match the level of detail provided by other providers (Claude, Anthropic, OpenAI)
    - Use consistent logging format across all Ollama API calls

These enhancements will provide users with comprehensive visibility into Ollama API performance, making it easier to:

- Debug slow responses
- Optimize model selection
- Monitor token generation rates
- Track model load times
- Analyze overall API performance
