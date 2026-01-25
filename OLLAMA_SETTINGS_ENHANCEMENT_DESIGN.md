# Ollama Provider Settings Enhancement Design

## Table of Contents

1. [Overview](#overview)
2. [Current State](#current-state)
3. [Proposed Changes](#proposed-changes)
4. [Implementation Details](#implementation-details)
5. [UI/UX Design](#uiux-design)
6. [Data Model Changes](#data-model-changes)
7. [Backend Implementation](#backend-implementation)
8. [Frontend Implementation](#frontend-implementation)
9. [Testing Requirements](#testing-requirements)
10. [Migration Strategy](#migration-strategy)
11. [Related Documentation](#related-documentation)

---

## Overview

This document outlines the design for enhancing the Ollama provider settings in Roo Code to:

1. **Make API call settings configurable** (timeout, retries, etc.)
2. **Add a "Test" button** next to "Base URL" to verify Ollama connectivity
3. **Add a "Refresh Models" button** near "Model ID" to manually trigger model discovery
4. **Document streaming support** with a read-only checkbox to indicate that streaming is always enabled

These enhancements will improve user experience by providing better control over API behavior and easier troubleshooting of connection issues.

**Note on Timeout Configuration**: The design includes two separate timeout settings:

- **Request Timeout** (default: 60 minutes) - For LLM API requests that may take many minutes for complex coding tasks
- **Model Discovery Timeout** (default: 10 seconds) - For model discovery requests which should be fast, but user-configurable if needed

**Note on Streaming**: Streaming is always enabled for Ollama API requests. A read-only checkbox in the Advanced Settings section documents this behavior. The checkbox is checked and disabled (greyed out) to indicate that streaming cannot be disabled.

**Related Documentation**:

- [OLLAMA_INTEGRATION_DOCUMENTATION.md](./OLLAMA_INTEGRATION_DOCUMENTATION.md) - Current Ollama integration details
- [OLLAMA_API_ENHANCEMENTS.md](./OLLAMA_API_ENHANCEMENTS.md) - Enhancements for detailed logging and metrics capture
- [axios-library-api-summary.md](./axios-library-api-summary.md) - Axios configuration options

---

## Current State

### Existing Settings

The current Ollama provider settings (`webview-ui/src/components/settings/providers/Ollama.tsx`) include:

1. **Base URL** (optional) - `ollamaBaseUrl`
2. **Ollama API Key** (optional) - `ollamaApiKey`
3. **Model ID** - `ollamaModelId`
4. **Context Window Size** (num_ctx) - `ollamaNumCtx`

### Current Limitations

1. **No configurable timeout**: Uses axios defaults (no timeout)
2. **No retry configuration**: No retry mechanism implemented
3. **No connection testing**: Users cannot verify if Ollama is reachable
4. **No manual model refresh**: Models are only discovered on mount or when base URL changes
5. **Limited observability**: No logging or connection status feedback

### Current Code Patterns

**File**: `webview-ui/src/components/settings/providers/Ollama.tsx`

- Uses `VSCodeTextField` for input fields
- Uses `VSCodeRadioGroup` for model selection
- Listens for `ollamaModels` messages from backend
- Sends `requestOllamaModels` message on mount

**File**: `src/core/webview/webviewMessageHandler.ts`

- Handles `requestOllamaModels` message (lines 988-1009)
- Calls `getOllamaModels()` from `src/api/providers/fetchers/ollama.ts`

**File**: `src/api/providers/fetchers/ollama.ts`

- `getOllamaModels()` function (lines 67-127)
- Uses axios without timeout configuration
- No retry logic

---

## Proposed Changes

### 1. New Configuration Settings

Add the following configurable settings with sensible defaults:

| Setting                 | Key                           | Type      | Default           | Description                                                                     |
| ----------------------- | ----------------------------- | --------- | ----------------- | ------------------------------------------------------------------------------- |
| Streaming               | `ollamaStreaming`             | `boolean` | `true`            | Enable streaming responses (read-only, always enabled)                          |
| Request Timeout         | `ollamaRequestTimeout`        | `number`  | `3600000` (60min) | Timeout in milliseconds for LLM API requests (chat completions, thinking work)  |
| Model Discovery Timeout | `ollamaModelDiscoveryTimeout` | `number`  | `10000` (10s)     | Timeout in milliseconds for model discovery requests (`/api/tags`, `/api/show`) |
| Max Retries             | `ollamaMaxRetries`            | `number`  | `0`               | Maximum number of retry attempts                                                |
| Retry Delay             | `ollamaRetryDelay`            | `number`  | `1000` (1s)       | Initial delay between retries in milliseconds                                   |
| Enable Logging          | `ollamaEnableLogging`         | `boolean` | `false`           | Enable detailed request/response logging                                        |

### 2. UI Enhancements

1. **Test Connection Button**: Next to "Base URL" field

    - Verifies Ollama is reachable at the specified URL
    - Shows success/error feedback
    - Disabled while testing

2. **Refresh Models Button**: Near "Model ID" field

    - Manually triggers model discovery
    - Shows loading state
    - Updates model list when complete

3. **Advanced Settings Section**: Collapsible section for API configuration
    - Streaming checkbox (read-only, always enabled, greyed out)
    - Request Timeout input (for LLM requests)
    - Model Discovery Timeout input (for model list requests)
    - Max Retries input
    - Retry Delay input
    - Enable Logging checkbox

---

## Implementation Details

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Webview (React)                          │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Ollama.tsx Component                                │  │
│  │  - Test Connection Button                            │  │
│  │  - Refresh Models Button                             │  │
│  │  - Advanced Settings Section                         │  │
│  └──────────────────────────────────────────────────────┘  │
│                        │                                    │
│                        │ vscode.postMessage()               │
│                        ▼                                    │
└─────────────────────────────────────────────────────────────┘
                        │
                        │ Message Types:
                        │ - testOllamaConnection
                        │ - refreshOllamaModels
                        │
┌─────────────────────────────────────────────────────────────┐
│              Extension Host (TypeScript)                     │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  webviewMessageHandler.ts                            │  │
│  │  - Handles testOllamaConnection                      │  │
│  │  - Handles refreshOllamaModels                       │  │
│  └──────────────────────────────────────────────────────┘  │
│                        │                                    │
│                        │ Uses configured axios instance    │
│                        ▼                                    │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  ollama-axios-config.ts (NEW)                         │  │
│  │  - createOllamaAxiosInstance()                        │  │
│  │  - Configures timeout, retries, logging              │  │
│  └──────────────────────────────────────────────────────┘  │
│                        │                                    │
│                        │ Uses configured instance          │
│                        ▼                                    │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  fetchers/ollama.ts                                   │  │
│  │  - getOllamaModels() (UPDATED)                        │  │
│  │  - testOllamaConnection() (NEW)                       │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## UI/UX Design

### Layout Structure

```
┌─────────────────────────────────────────────────────────────┐
│ Base URL (optional)                    [Test]                │
│ http://localhost:11434                                       │
│                                                              │
│ Ollama API Key                                               │
│ •••••••••••••••••                                            │
│ Optional API key for authenticated Ollama instances...       │
│                                                              │
│ Model ID                                    [Refresh Models] │
│ llama3-groq-tool-use:70b-q2_K                               │
│                                                              │
│ Context Window Size (num_ctx)                                │
│ e.g., 4096                                                   │
│                                                              │
│ ▼ Advanced Settings                                          │
│   ☑ Streaming (always enabled)                               │
│                                                              │
│   Request Timeout (ms)                                       │
│   3600000                                                    │
│                                                              │
│   Model Discovery Timeout (ms)                               │
│   10000                                                      │
│                                                              │
│   Max Retries                                                │
│   0                                                          │
│                                                              │
│   Retry Delay (ms)                                           │
│   1000                                                       │
│                                                              │
│   ☑ Enable Request Logging                                   │
└─────────────────────────────────────────────────────────────┘
```

### Component States

#### Test Connection Button

- **Default**: "Test" (enabled)
- **Testing**: "Testing..." (disabled, spinner)
- **Success**: "✓ Connected" (green, temporary)
- **Error**: "✗ Connection Failed" (red, temporary)

#### Refresh Models Button

- **Default**: "Refresh Models" (enabled)
- **Loading**: "Refreshing..." (disabled, spinner)
- **Success**: "✓ Models Updated" (green, temporary)
- **Error**: "✗ Refresh Failed" (red, temporary)

---

## Data Model Changes

### Provider Settings Schema

**File**: `packages/types/src/provider-settings.ts`

**Current Schema** (lines 257-262):

```typescript
const ollamaSchema = baseProviderSettingsSchema.extend({
	ollamaModelId: z.string().optional(),
	ollamaBaseUrl: z.string().optional(),
	ollamaApiKey: z.string().optional(),
	ollamaNumCtx: z.number().int().min(128).optional(),
})
```

**Updated Schema**:

```typescript
const ollamaSchema = baseProviderSettingsSchema.extend({
	ollamaModelId: z.string().optional(),
	ollamaBaseUrl: z.string().optional(),
	ollamaApiKey: z.string().optional(),
	ollamaNumCtx: z.number().int().min(128).optional(),
	// New fields
	// Note: ollamaStreaming is not stored in schema as it's always true (read-only UI element)
	ollamaRequestTimeout: z.number().int().min(1000).max(7200000).optional(), // 1s to 120min
	ollamaModelDiscoveryTimeout: z.number().int().min(1000).max(600000).optional(), // 1s to 10min
	ollamaMaxRetries: z.number().int().min(0).max(10).optional(),
	ollamaRetryDelay: z.number().int().min(100).max(10000).optional(), // 100ms to 10s
	ollamaEnableLogging: z.boolean().optional(),
})
```

### Default Values

**File**: `packages/types/src/provider-settings.ts` (add to defaults object)

```typescript
const defaultOllamaSettings = {
	// Note: ollamaStreaming is always true and not stored (read-only UI element)
	ollamaRequestTimeout: 3600000, // 60 minutes (similar to unlimited) - for LLM requests
	ollamaModelDiscoveryTimeout: 10000, // 10 seconds - for model discovery requests
	ollamaMaxRetries: 0, // No retries by default
	ollamaRetryDelay: 1000, // 1 second
	ollamaEnableLogging: false,
}
```

---

## Backend Implementation

### 1. Create Ollama Axios Configuration Module

**New File**: `src/api/providers/fetchers/ollama-axios-config.ts`

**Note on Comprehensive Logging**: For detailed logging of LLM API responses (including token rates, timing metrics, etc.), see [OLLAMA_API_ENHANCEMENTS.md](./OLLAMA_API_ENHANCEMENTS.md). The logging implemented in this module focuses on:

- Connection-level events (requests, responses, errors)
- Timing information for test connection and model refresh operations
- Request/response metadata

```typescript
import axios, { AxiosInstance, AxiosError, InternalAxiosRequestConfig, AxiosResponse } from "axios"
import type { ApiHandlerOptions } from "../../../shared/api"

interface OllamaAxiosConfig {
	baseUrl?: string
	apiKey?: string
	timeout?: number // Timeout in milliseconds (use modelDiscoveryTimeout for model discovery, requestTimeout for LLM requests)
	retries?: number
	retryDelay?: number
	enableLogging?: boolean
}

/**
 * Creates a configured Axios instance for Ollama API requests.
 *
 * Note: This function is used for both LLM requests and model discovery.
 * - For LLM requests: Use `ollamaRequestTimeout` (default: 60 minutes)
 * - For model discovery: Use `ollamaModelDiscoveryTimeout` (default: 10 seconds)
 *
 * @param config - Configuration options
 * @returns Configured Axios instance
 */
export function createOllamaAxiosInstance(config: OllamaAxiosConfig = {}): AxiosInstance {
	const {
		baseUrl = "http://localhost:11434",
		apiKey,
		timeout = 3600000, // 60 minutes default (for LLM requests)
		retries = 0, // No retries by default
		retryDelay = 1000,
		enableLogging = false,
	} = config

	const instance = axios.create({
		baseURL: baseUrl,
		timeout: timeout,
		timeoutErrorMessage: `Ollama request timed out after ${timeout}ms`,
		headers: apiKey
			? {
					Authorization: `Bearer ${apiKey}`,
				}
			: {},
		transitional: {
			clarifyTimeoutError: true,
		},
	})

	// Setup retry interceptor
	setupRetryInterceptor(instance, { retries, retryDelay })

	// Setup logging interceptor
	if (enableLogging) {
		setupLoggingInterceptor(instance)
	}

	return instance
}

function setupRetryInterceptor(instance: AxiosInstance, config: { retries: number; retryDelay: number }) {
	instance.interceptors.response.use(
		(response) => response,
		async (error: AxiosError) => {
			const axiosConfig = error.config as any

			axiosConfig.__retryCount = axiosConfig.__retryCount || 0
			if (axiosConfig.__retryCount >= config.retries) {
				return Promise.reject(error)
			}

			const shouldRetry =
				error.code === "ECONNREFUSED" ||
				error.code === "ETIMEDOUT" ||
				error.code === "ECONNABORTED" ||
				error.code === "ERR_NETWORK" ||
				(error.response && error.response.status >= 500)

			if (!shouldRetry) {
				return Promise.reject(error)
			}

			axiosConfig.__retryCount += 1
			const delay = config.retryDelay * Math.pow(2, axiosConfig.__retryCount - 1)

			await new Promise((resolve) => setTimeout(resolve, delay))

			return instance(axiosConfig)
		},
	)
}

function setupLoggingInterceptor(instance: AxiosInstance) {
	instance.interceptors.request.use((config: InternalAxiosRequestConfig) => {
		;(config as any).metadata = { startTime: Date.now() }
		console.debug("[Ollama] Request:", {
			method: config.method?.toUpperCase(),
			url: `${config.baseURL}${config.url}`,
			timeout: config.timeout,
			timestamp: new Date().toISOString(),
		})
		return config
	})

	instance.interceptors.response.use(
		(response: AxiosResponse) => {
			const startTime = (response.config as any).metadata?.startTime
			const duration = startTime ? Date.now() - startTime : undefined
			console.debug("[Ollama] Response:", {
				status: response.status,
				url: response.config.url,
				durationMs: duration,
				duration: duration ? `${duration}ms` : undefined,
				timestamp: new Date().toISOString(),
			})
			return response
		},
		(error: AxiosError) => {
			const startTime = (error.config as any)?.metadata?.startTime
			const duration = startTime ? Date.now() - startTime : undefined
			console.error("[Ollama] Error:", {
				code: error.code,
				message: error.message,
				status: error.response?.status,
				url: error.config?.url,
				durationMs: duration,
				duration: duration ? `${duration}ms` : undefined,
				timestamp: new Date().toISOString(),
			})
			return Promise.reject(error)
		},
	)
}
```

### 2. Update getOllamaModels Function

**File**: `src/api/providers/fetchers/ollama.ts`

**Changes**:

- Import `createOllamaAxiosInstance`
- Accept configuration options
- Use configured instance instead of default axios

```typescript
import { createOllamaAxiosInstance } from "./ollama-axios-config"

export async function getOllamaModels(
	baseUrl = "http://localhost:11434",
	apiKey?: string,
	config?: {
		timeout?: number // Deprecated: use modelDiscoveryTimeout instead
		modelDiscoveryTimeout?: number // Timeout for model discovery requests
		maxRetries?: number
		retryDelay?: number
		enableLogging?: boolean
	},
): Promise<Record<string, ModelInfo>> {
	const models: Record<string, ModelInfo> = {}

	baseUrl = baseUrl === "" ? "http://localhost:11434" : baseUrl

	try {
		if (!URL.canParse(baseUrl)) {
			return models
		}

		// Create configured axios instance for model discovery
		// Use shorter timeout for model discovery (should be fast)
		const axiosInstance = createOllamaAxiosInstance({
			baseUrl,
			apiKey,
			timeout: config?.modelDiscoveryTimeout ?? config?.timeout ?? 10000, // 10 seconds default for model discovery
			retries: config?.maxRetries ?? 0, // No retries by default
			retryDelay: config?.retryDelay ?? 1000,
			enableLogging: config?.enableLogging ?? false,
		})

		// Get list of models
		const response = await axiosInstance.get<OllamaModelsResponse>("/api/tags")
		const parsedResponse = OllamaModelsResponseSchema.safeParse(response.data)
		let modelInfoPromises = []

		if (parsedResponse.success) {
			for (const ollamaModel of parsedResponse.data.models) {
				modelInfoPromises.push(
					axiosInstance
						.post<OllamaModelInfoResponse>("/api/show", {
							model: ollamaModel.model,
						})
						.then((ollamaModelInfo) => {
							const modelInfo = parseOllamaModel(ollamaModelInfo.data)
							if (modelInfo) {
								models[ollamaModel.name] = modelInfo
							}
						})
						.catch((error) => {
							console.warn(`Failed to get info for model ${ollamaModel.name}:`, error.message)
						}),
				)
			}

			await Promise.all(modelInfoPromises)
		} else {
			console.error(`Error parsing Ollama models response: ${JSON.stringify(parsedResponse.error, null, 2)}`)
		}
	} catch (error) {
		if (AxiosError.isAxiosError(error)) {
			const axiosError = error as AxiosError
			if (axiosError.code === "ECONNREFUSED") {
				console.warn(`Failed connecting to Ollama at ${baseUrl}`)
			} else if (axiosError.code === "ETIMEDOUT" || axiosError.code === "ECONNABORTED") {
				console.warn(`Ollama request timed out at ${baseUrl}`)
			} else {
				console.error(`Error fetching Ollama models: ${axiosError.message}`)
			}
		} else {
			console.error(
				`Error fetching Ollama models: ${JSON.stringify(error, Object.getOwnPropertyNames(error), 2)}`,
			)
		}
	}

	return models
}
```

### 3. Add Test Connection Function

**File**: `src/api/providers/fetchers/ollama.ts`

```typescript
/**
 * Tests connection to Ollama instance.
 * Uses model discovery timeout (default: 10 seconds) since this is a quick connectivity check.
 *
 * @param baseUrl - Base URL of Ollama instance
 * @param apiKey - Optional API key
 * @param config - Optional configuration
 * @returns Promise resolving to connection test result
 */
export async function testOllamaConnection(
	baseUrl = "http://localhost:11434",
	apiKey?: string,
	config?: {
		timeout?: number // Uses model discovery timeout (default: 10 seconds)
		enableLogging?: boolean
	},
): Promise<{ success: boolean; message: string; durationMs?: number }> {
	baseUrl = baseUrl === "" ? "http://localhost:11434" : baseUrl
	const startTime = Date.now()

	try {
		if (!URL.canParse(baseUrl)) {
			return {
				success: false,
				message: `Invalid URL: ${baseUrl}`,
				durationMs: Date.now() - startTime,
			}
		}

		const axiosInstance = createOllamaAxiosInstance({
			baseUrl,
			apiKey,
			timeout: config?.timeout ?? 10000, // 10 seconds for connection test (shorter than default)
			retries: 0, // No retries for connection test
			enableLogging: config?.enableLogging ?? false,
		})

		// Try to reach the /api/tags endpoint
		await axiosInstance.get("/api/tags", { timeout: config?.timeout ?? 10000 })

		const durationMs = Date.now() - startTime

		// Log comprehensive metrics if logging is enabled
		if (config?.enableLogging) {
			console.debug("[Ollama Connection Test]", {
				baseUrl,
				success: true,
				durationMs,
				timestamp: new Date().toISOString(),
			})
		}

		return {
			success: true,
			message: `Successfully connected to Ollama at ${baseUrl}`,
			durationMs,
		}
	} catch (error) {
		const durationMs = Date.now() - startTime

		// Log comprehensive error metrics if logging is enabled
		if (config?.enableLogging) {
			console.debug("[Ollama Connection Test]", {
				baseUrl,
				success: false,
				durationMs,
				error: error instanceof Error ? error.message : String(error),
				errorCode: AxiosError.isAxiosError(error) ? error.code : undefined,
				timestamp: new Date().toISOString(),
			})
		}

		if (AxiosError.isAxiosError(error)) {
			const axiosError = error as AxiosError
			if (axiosError.code === "ECONNREFUSED") {
				return {
					success: false,
					message: `Cannot connect to Ollama at ${baseUrl}. Make sure Ollama is running.`,
					durationMs,
				}
			} else if (axiosError.code === "ETIMEDOUT" || axiosError.code === "ECONNABORTED") {
				return {
					success: false,
					message: `Connection to Ollama timed out. Check if the URL is correct and Ollama is accessible.`,
					durationMs,
				}
			} else if (axiosError.code === "ERR_NETWORK") {
				return {
					success: false,
					message: `Network error connecting to Ollama. Check your network connection.`,
					durationMs,
				}
			} else if (axiosError.response) {
				return {
					success: false,
					message: `Ollama returned error: ${axiosError.response.status} ${axiosError.response.statusText}`,
					durationMs,
				}
			}
		}

		return {
			success: false,
			message: `Failed to connect: ${error instanceof Error ? error.message : String(error)}`,
			durationMs,
		}
	}
}
```

### 4. Update Webview Message Handler

**File**: `src/core/webview/webviewMessageHandler.ts`

**Add new message handlers** (after line 1009):

```typescript
case "testOllamaConnection": {
	const { apiConfiguration: ollamaApiConfig } = await provider.getState()
	try {
		const result = await testOllamaConnection(
			ollamaApiConfig.ollamaBaseUrl,
			ollamaApiConfig.ollamaApiKey,
			{
				timeout: ollamaApiConfig.ollamaModelDiscoveryTimeout ?? 10000, // 10 seconds for connection test
				enableLogging: ollamaApiConfig.ollamaEnableLogging ?? false,
			}
		);

		provider.postMessageToWebview({
			type: "ollamaConnectionTestResult",
			success: result.success,
			message: result.message,
			durationMs: result.durationMs, // Include timing information
		});
	} catch (error) {
		provider.postMessageToWebview({
			type: "ollamaConnectionTestResult",
			success: false,
			message: `Error testing connection: ${error instanceof Error ? error.message : String(error)}`,
		});
	}
	break
}

case "refreshOllamaModels": {
	const { apiConfiguration: ollamaApiConfig } = await provider.getState()
	const startTime = Date.now();

	try {
		const ollamaOptions = {
			provider: "ollama" as const,
			baseUrl: ollamaApiConfig.ollamaBaseUrl,
			apiKey: ollamaApiConfig.ollamaApiKey,
		};

		// Flush cache and refresh
		await flushModels(ollamaOptions, true);

		const ollamaModels = await getModels({
			...ollamaOptions,
			// Pass configuration for axios instance
			// Use model discovery timeout for model fetching
			ollamaModelDiscoveryTimeout: ollamaApiConfig.ollamaModelDiscoveryTimeout,
			ollamaMaxRetries: ollamaApiConfig.ollamaMaxRetries,
			ollamaRetryDelay: ollamaApiConfig.ollamaRetryDelay,
			ollamaEnableLogging: ollamaApiConfig.ollamaEnableLogging,
		});

		const durationMs = Date.now() - startTime;

		// Log comprehensive metrics if logging is enabled
		if (ollamaApiConfig.ollamaEnableLogging) {
			console.debug('[Ollama Model Refresh]', {
				baseUrl: ollamaApiConfig.ollamaBaseUrl,
				modelCount: Object.keys(ollamaModels).length,
				durationMs,
				models: Object.keys(ollamaModels),
				timestamp: new Date().toISOString(),
			});
		}

		if (Object.keys(ollamaModels).length > 0) {
			provider.postMessageToWebview({
				type: "ollamaModels",
				ollamaModels: ollamaModels,
			});
			provider.postMessageToWebview({
				type: "ollamaModelsRefreshResult",
				success: true,
				message: `Found ${Object.keys(ollamaModels).length} model(s)`,
				durationMs, // Include timing information
			});
		} else {
			provider.postMessageToWebview({
				type: "ollamaModelsRefreshResult",
				success: false,
				message: "No models found. Make sure Ollama is running and has models installed.",
				durationMs, // Include timing information even on failure
			});
		}
	} catch (error) {
		const durationMs = Date.now() - startTime;

		// Log error metrics if logging is enabled
		if (ollamaApiConfig.ollamaEnableLogging) {
			console.debug('[Ollama Model Refresh]', {
				baseUrl: ollamaApiConfig.ollamaBaseUrl,
				success: false,
				durationMs,
				error: error instanceof Error ? error.message : String(error),
				timestamp: new Date().toISOString(),
			});
		}

		provider.postMessageToWebview({
			type: "ollamaModelsRefreshResult",
			success: false,
			message: `Failed to refresh models: ${error instanceof Error ? error.message : String(error)}`,
			durationMs, // Include timing information
		});
	}
	break
}
```

**Note**: The `getModels` function in `modelCache.ts` will need to be updated to accept and pass through Ollama-specific configuration options.

### 5. Update Model Cache Functions

**File**: `src/api/providers/fetchers/modelCache.ts`

**Update `fetchModelsFromProvider`** to pass configuration:

```typescript
		case "ollama":
			models = await getOllamaModels(
				options.baseUrl,
				options.apiKey,
				{
					modelDiscoveryTimeout: (options as any).ollamaModelDiscoveryTimeout,
					maxRetries: (options as any).ollamaMaxRetries,
					retryDelay: (options as any).ollamaRetryDelay,
					enableLogging: (options as any).ollamaEnableLogging,
				}
			);
			break
```

**Update `GetModelsOptions` type** to include Ollama-specific options (if needed, or use type assertion as shown above).

---

## Frontend Implementation

### 1. Update Ollama Component

**File**: `webview-ui/src/components/settings/providers/Ollama.tsx`

**Key Changes**:

1. Add state for test connection and refresh models
2. Add Test button next to Base URL
3. Add Refresh Models button near Model ID
4. Add Advanced Settings collapsible section
5. Handle new message types

**Implementation**:

```typescript
import { useState, useCallback, useMemo, useEffect, useRef } from "react"
import { useEvent } from "react-use"
import { VSCodeTextField, VSCodeRadioGroup, VSCodeRadio, VSCodeCheckbox } from "@vscode/webview-ui-toolkit/react"

import type { ProviderSettings, ExtensionMessage, ModelRecord } from "@roo-code/types"

import { useAppTranslation } from "@src/i18n/TranslationContext"
import { useRouterModels } from "@src/components/ui/hooks/useRouterModels"
import { vscode } from "@src/utils/vscode"
import { Button } from "@src/components/ui"

import { inputEventTransform } from "../transforms"

type OllamaProps = {
	apiConfiguration: ProviderSettings
	setApiConfigurationField: (field: keyof ProviderSettings, value: ProviderSettings[keyof ProviderSettings]) => void
}

export const Ollama = ({ apiConfiguration, setApiConfigurationField }: OllamaProps) => {
	const { t } = useAppTranslation()

	const [ollamaModels, setOllamaModels] = useState<ModelRecord>({})
	const [testingConnection, setTestingConnection] = useState(false)
	const [testResult, setTestResult] = useState<{ success: boolean; message: string; durationMs?: number } | null>(null)
	const [refreshingModels, setRefreshingModels] = useState(false)
	const [refreshResult, setRefreshResult] = useState<{ success: boolean; message: string; durationMs?: number } | null>(null)
	const [showAdvanced, setShowAdvanced] = useState(false)

	const routerModels = useRouterModels()

	const testResultTimerRef = useRef<NodeJS.Timeout>()
	const refreshResultTimerRef = useRef<NodeJS.Timeout>()

	const handleInputChange = useCallback(
		<K extends keyof ProviderSettings, E>(
			field: K,
			transform: (event: E) => ProviderSettings[K] = inputEventTransform,
		) =>
			(event: E | Event) => {
				setApiConfigurationField(field, transform(event as E))
			},
		[setApiConfigurationField],
	)

	const onMessage = useCallback((event: MessageEvent) => {
		const message: ExtensionMessage = event.data

		switch (message.type) {
			case "ollamaModels":
				{
					const newModels = message.ollamaModels ?? {}
					setOllamaModels(newModels)
				}
				break
			case "ollamaConnectionTestResult":
				setTestResult({
					success: message.success ?? false,
					message: message.message ?? "Unknown error",
					durationMs: message.durationMs, // Include timing information
				})
				setTestingConnection(false)
				// Clear result after 5 seconds
				if (testResultTimerRef.current) {
					clearTimeout(testResultTimerRef.current)
				}
				testResultTimerRef.current = setTimeout(() => setTestResult(null), 5000)
				break
			case "ollamaModelsRefreshResult":
				setRefreshResult({
					success: message.success ?? false,
					message: message.message ?? "Unknown error",
					durationMs: message.durationMs, // Include timing information
				})
				setRefreshingModels(false)
				// Clear result after 5 seconds
				if (refreshResultTimerRef.current) {
					clearTimeout(refreshResultTimerRef.current)
				}
				refreshResultTimerRef.current = setTimeout(() => setRefreshResult(null), 5000)
				break
		}
	}, [])

	useEvent("message", onMessage)

	// Cleanup timers on unmount
	useEffect(() => {
		return () => {
			if (testResultTimerRef.current) {
				clearTimeout(testResultTimerRef.current)
			}
			if (refreshResultTimerRef.current) {
				clearTimeout(refreshResultTimerRef.current)
			}
		}
	}, [])

	// Refresh models on mount
	useEffect(() => {
		vscode.postMessage({ type: "requestOllamaModels" })
	}, [])

	const handleTestConnection = useCallback(() => {
		setTestingConnection(true)
		setTestResult(null)
		vscode.postMessage({ type: "testOllamaConnection" })
	}, [])

	const handleRefreshModels = useCallback(() => {
		setRefreshingModels(true)
		setRefreshResult(null)
		vscode.postMessage({ type: "refreshOllamaModels" })
	}, [])

	// Check if the selected model exists in the fetched models
	const modelNotAvailable = useMemo(() => {
		const selectedModel = apiConfiguration?.ollamaModelId
		if (!selectedModel) return false

		if (Object.keys(ollamaModels).length > 0 && selectedModel in ollamaModels) {
			return false
		}

		if (routerModels.data?.ollama) {
			const availableModels = Object.keys(routerModels.data.ollama)
			return !availableModels.includes(selectedModel)
		}

		return false
	}, [apiConfiguration?.ollamaModelId, routerModels.data, ollamaModels])

	return (
		<>
			{/* Base URL with Test Button */}
			<div className="flex items-center gap-2">
				<VSCodeTextField
					value={apiConfiguration?.ollamaBaseUrl || ""}
					type="url"
					onInput={handleInputChange("ollamaBaseUrl")}
					placeholder={t("settings:defaults.ollamaUrl")}
					className="flex-1">
					<label className="block font-medium mb-1">{t("settings:providers.ollama.baseUrl")}</label>
				</VSCodeTextField>
				<Button
					onClick={handleTestConnection}
					disabled={testingConnection || !apiConfiguration?.ollamaBaseUrl}
					variant="outline"
					className="mt-6">
					{testingConnection ? t("settings:providers.ollama.testing") : t("settings:providers.ollama.test")}
				</Button>
			</div>
			{testResult && (
				<div
					className={`p-2 rounded-xs text-sm ${
						testResult.success
							? "bg-green-800/20 text-green-400"
							: "bg-red-800/20 text-red-400"
					}`}>
					<div>{testResult.message}</div>
					{testResult.durationMs !== undefined && (
						<div className="text-xs mt-1 opacity-80">
							Completed in {testResult.durationMs}ms
						</div>
					)}
				</div>
			)}

			{/* API Key */}
			{apiConfiguration?.ollamaBaseUrl && (
				<VSCodeTextField
					value={apiConfiguration?.ollamaApiKey || ""}
					type="password"
					onInput={handleInputChange("ollamaApiKey")}
					placeholder={t("settings:placeholders.apiKey")}
					className="w-full">
					<label className="block font-medium mb-1">{t("settings:providers.ollama.apiKey")}</label>
					<div className="text-xs text-vscode-descriptionForeground mt-1">
						{t("settings:providers.ollama.apiKeyHelp")}
					</div>
				</VSCodeTextField>
			)}

			{/* Model ID with Refresh Button */}
			<div className="flex items-center gap-2">
				<VSCodeTextField
					value={apiConfiguration?.ollamaModelId || ""}
					onInput={handleInputChange("ollamaModelId")}
					placeholder={t("settings:placeholders.modelId.ollama")}
					className="flex-1">
					<label className="block font-medium mb-1">{t("settings:providers.ollama.modelId")}</label>
				</VSCodeTextField>
				<Button
					onClick={handleRefreshModels}
					disabled={refreshingModels}
					variant="outline"
					className="mt-6">
					{refreshingModels ? t("settings:providers.ollama.refreshing") : t("settings:providers.ollama.refreshModels")}
				</Button>
			</div>
			{refreshResult && (
				<div
					className={`p-2 rounded-xs text-sm ${
						refreshResult.success
							? "bg-green-800/20 text-green-400"
							: "bg-red-800/20 text-red-400"
					}`}>
					<div>{refreshResult.message}</div>
					{refreshResult.durationMs !== undefined && (
						<div className="text-xs mt-1 opacity-80">
							Completed in {refreshResult.durationMs}ms
						</div>
					)}
				</div>
			)}
			{modelNotAvailable && (
				<div className="flex flex-col gap-2 text-vscode-errorForeground text-sm">
					<div className="flex flex-row items-center gap-1">
						<div className="codicon codicon-close" />
						<div>
							{t("settings:validation.modelAvailability", { modelId: apiConfiguration?.ollamaModelId })}
						</div>
					</div>
				</div>
			)}
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

			{/* Context Window Size */}
			<VSCodeTextField
				value={apiConfiguration?.ollamaNumCtx?.toString() || ""}
				onInput={(e: any) => {
					const value = e.target?.value
					if (value === "") {
						setApiConfigurationField("ollamaNumCtx", undefined)
					} else {
						const numValue = parseInt(value, 10)
						if (!isNaN(numValue) && numValue >= 128) {
							setApiConfigurationField("ollamaNumCtx", numValue)
						}
					}
				}}
				placeholder="e.g., 4096"
				className="w-full">
				<label className="block font-medium mb-1">{t("settings:providers.ollama.numCtx")}</label>
				<div className="text-xs text-vscode-descriptionForeground mt-1">
					{t("settings:providers.ollama.numCtxHelp")}
				</div>
			</VSCodeTextField>

			{/* Advanced Settings */}
			<div className="flex flex-col gap-2">
				<button
					type="button"
					onClick={() => setShowAdvanced(!showAdvanced)}
					className="flex items-center gap-2 text-sm font-medium text-vscode-foreground hover:text-vscode-foreground/80">
					<span className={`codicon ${showAdvanced ? "codicon-chevron-down" : "codicon-chevron-right"}`} />
					{t("settings:providers.ollama.advancedSettings")}
				</button>
				{showAdvanced && (
					<div className="flex flex-col gap-3 pl-4 border-l-2 border-vscode-foreground/10">
						{/* Streaming - Read-only checkbox to document that streaming is always enabled */}
						<VSCodeCheckbox
							checked={true}
							disabled={true}
							onChange={() => {}}>
							<label className="block font-medium mb-1">{t("settings:providers.ollama.streaming")}</label>
							<div className="text-xs text-vscode-descriptionForeground mt-1">
								{t("settings:providers.ollama.streamingHelp")}
							</div>
						</VSCodeCheckbox>

						<VSCodeTextField
							value={apiConfiguration?.ollamaRequestTimeout?.toString() || "3600000"}
							type="number"
							onInput={(e: any) => {
								const value = e.target?.value
								if (value === "") {
									setApiConfigurationField("ollamaRequestTimeout", undefined)
								} else {
									const numValue = parseInt(value, 10)
									if (!isNaN(numValue) && numValue >= 1000 && numValue <= 7200000) {
										setApiConfigurationField("ollamaRequestTimeout", numValue)
									}
								}
							}}
							className="w-full">
							<label className="block font-medium mb-1">{t("settings:providers.ollama.requestTimeout")}</label>
							<div className="text-xs text-vscode-descriptionForeground mt-1">
								{t("settings:providers.ollama.requestTimeoutHelp")}
							</div>
						</VSCodeTextField>

						<VSCodeTextField
							value={apiConfiguration?.ollamaModelDiscoveryTimeout?.toString() || "10000"}
							type="number"
							onInput={(e: any) => {
								const value = e.target?.value
								if (value === "") {
									setApiConfigurationField("ollamaModelDiscoveryTimeout", undefined)
								} else {
									const numValue = parseInt(value, 10)
									if (!isNaN(numValue) && numValue >= 1000 && numValue <= 600000) {
										setApiConfigurationField("ollamaModelDiscoveryTimeout", numValue)
									}
								}
							}}
							className="w-full">
							<label className="block font-medium mb-1">{t("settings:providers.ollama.modelDiscoveryTimeout")}</label>
							<div className="text-xs text-vscode-descriptionForeground mt-1">
								{t("settings:providers.ollama.modelDiscoveryTimeoutHelp")}
							</div>
						</VSCodeTextField>

						<VSCodeTextField
							value={apiConfiguration?.ollamaMaxRetries?.toString() || "0"}
							type="number"
							onInput={(e: any) => {
								const value = e.target?.value
								if (value === "") {
									setApiConfigurationField("ollamaMaxRetries", undefined)
								} else {
									const numValue = parseInt(value, 10)
									if (!isNaN(numValue) && numValue >= 0 && numValue <= 10) {
										setApiConfigurationField("ollamaMaxRetries", numValue)
									}
								}
							}}
							className="w-full">
							<label className="block font-medium mb-1">{t("settings:providers.ollama.maxRetries")}</label>
							<div className="text-xs text-vscode-descriptionForeground mt-1">
								{t("settings:providers.ollama.maxRetriesHelp")}
							</div>
						</VSCodeTextField>

						<VSCodeTextField
							value={apiConfiguration?.ollamaRetryDelay?.toString() || "1000"}
							type="number"
							onInput={(e: any) => {
								const value = e.target?.value
								if (value === "") {
									setApiConfigurationField("ollamaRetryDelay", undefined)
								} else {
									const numValue = parseInt(value, 10)
									if (!isNaN(numValue) && numValue >= 100 && numValue <= 10000) {
										setApiConfigurationField("ollamaRetryDelay", numValue)
									}
								}
							}}
							className="w-full">
							<label className="block font-medium mb-1">{t("settings:providers.ollama.retryDelay")}</label>
							<div className="text-xs text-vscode-descriptionForeground mt-1">
								{t("settings:providers.ollama.retryDelayHelp")}
							</div>
						</VSCodeTextField>

						<VSCodeCheckbox
							checked={apiConfiguration?.ollamaEnableLogging ?? false}
							onChange={(e: any) => {
								setApiConfigurationField("ollamaEnableLogging", e.target.checked)
							}}>
							<label className="block font-medium mb-1">{t("settings:providers.ollama.enableLogging")}</label>
							<div className="text-xs text-vscode-descriptionForeground mt-1">
								{t("settings:providers.ollama.enableLoggingHelp")}
							</div>
						</VSCodeCheckbox>
					</div>
				)}
			</div>

			{/* Description */}
			<div className="text-sm text-vscode-descriptionForeground">
				{t("settings:providers.ollama.description")}
				<span className="text-vscode-errorForeground ml-1">{t("settings:providers.ollama.warning")}</span>
			</div>
		</>
	)
}
```

### 2. Add Translation Keys

**File**: `webview-ui/src/i18n/locales/en/settings.json`

Add the following keys to the `providers.ollama` section:

```json
{
	"providers": {
		"ollama": {
			"test": "Test",
			"testing": "Testing...",
			"refreshModels": "Refresh Models",
			"refreshing": "Refreshing...",
			"advancedSettings": "Advanced Settings",
			"streaming": "Streaming",
			"streamingHelp": "Streaming is always enabled for Ollama API requests. Responses are streamed in real-time as they are generated.",
			"requestTimeout": "Request Timeout (ms)",
			"requestTimeoutHelp": "Timeout in milliseconds for LLM API requests (chat completions, thinking work). Default: 3600000 (60 minutes). Range: 1000-7200000 (120 minutes).",
			"modelDiscoveryTimeout": "Model Discovery Timeout (ms)",
			"modelDiscoveryTimeoutHelp": "Timeout in milliseconds for model discovery requests (listing and fetching model details). Default: 10000 (10 seconds). Range: 1000-600000 (10 minutes).",
			"maxRetries": "Max Retries",
			"maxRetriesHelp": "Maximum number of retry attempts for failed requests. Default: 0 (no retries). Range: 0-10.",
			"retryDelay": "Retry Delay (ms)",
			"retryDelayHelp": "Initial delay between retries in milliseconds. Uses exponential backoff. Default: 1000 (1 second). Range: 100-10000.",
			"enableLogging": "Enable Request Logging",
			"enableLoggingHelp": "Enable detailed logging of API requests and responses for debugging purposes."
		}
	}
}
```

### 3. Update Extension Message Types

**File**: `packages/types/src/extension-message.ts`

Add new message types:

```typescript
export type ExtensionMessage =
	// ... existing types ...
	| { type: "testOllamaConnection" }
	| { type: "ollamaConnectionTestResult"; success: boolean; message: string; durationMs?: number }
	| { type: "refreshOllamaModels" }
	| { type: "ollamaModelsRefreshResult"; success: boolean; message: string; durationMs?: number }
```

---

## Testing Requirements

### Unit Tests

#### Backend Tests

**New File**: `src/api/providers/fetchers/__tests__/ollama-axios-config.spec.ts`

```typescript
import { describe, it, expect, vi, beforeEach } from "vitest"
import axios from "axios"
import { createOllamaAxiosInstance } from "../ollama-axios-config"

vi.mock("axios")

describe("createOllamaAxiosInstance", () => {
	beforeEach(() => {
		vi.clearAllMocks()
	})

	it("should create instance with default configuration", () => {
		const instance = createOllamaAxiosInstance()
		expect(instance).toBeDefined()
		expect(axios.create).toHaveBeenCalledWith(
			expect.objectContaining({
				baseURL: "http://localhost:11434",
				timeout: 30000,
			}),
		)
	})

	it("should create instance with custom configuration", () => {
		const instance = createOllamaAxiosInstance({
			baseUrl: "http://custom:11434",
			apiKey: "test-key",
			timeout: 60000,
			retries: 3,
			retryDelay: 2000,
			enableLogging: true,
		})
		expect(instance).toBeDefined()
		expect(axios.create).toHaveBeenCalledWith(
			expect.objectContaining({
				baseURL: "http://custom:11434",
				timeout: 60000,
				headers: {
					Authorization: "Bearer test-key",
				},
			}),
		)
	})

	it("should use model discovery timeout for model fetching", () => {
		const instance = createOllamaAxiosInstance({
			baseUrl: "http://localhost:11434",
			timeout: 10000, // Model discovery timeout
		})
		expect(instance).toBeDefined()
		expect(axios.create).toHaveBeenCalledWith(
			expect.objectContaining({
				timeout: 10000,
			}),
		)
	})
})
```

**Update File**: `src/api/providers/fetchers/__tests__/ollama.spec.ts`

Add tests for:

- `testOllamaConnection` function
- Updated `getOllamaModels` with configuration
- Error handling scenarios

#### Frontend Tests

**Update File**: `webview-ui/src/components/settings/providers/__tests__/Ollama.spec.tsx`

Add tests for:

- Test connection button functionality
- Refresh models button functionality
- Advanced settings toggle
- Configuration field updates
- Message handling

### Integration Tests

**New File**: `src/core/webview/__tests__/ollama-settings.spec.ts`

Test:

- `testOllamaConnection` message handler
- `refreshOllamaModels` message handler
- Error scenarios
- Success scenarios

### Manual Testing Checklist

- [ ] Test connection button works with valid URL
- [ ] Test connection button shows error for invalid URL
- [ ] Test connection button shows error for unreachable URL
- [ ] Test connection button displays timing information (duration in milliseconds)
- [ ] Refresh models button updates model list
- [ ] Refresh models button shows error when Ollama is unreachable
- [ ] Refresh models button displays timing information (duration in milliseconds)
- [ ] Advanced settings section toggles correctly
- [ ] Streaming checkbox is checked and disabled (read-only)
- [ ] Request timeout setting is applied to LLM requests (60 min default)
- [ ] Model discovery timeout setting is applied to model discovery requests (10 sec default)
- [ ] Retry configuration works
- [ ] Logging setting enables/disables logging
- [ ] Settings persist across sessions
- [ ] Default values are used when settings are not configured

---

## Migration Strategy

### Backward Compatibility

1. **Default Values**: All new settings have defaults that match current behavior

    - `ollamaStreaming`: true - always enabled (read-only UI element to document that streaming is always on)
    - `ollamaRequestTimeout`: 3600000 (60 minutes) - very long timeout similar to unlimited, suitable for complex coding tasks and LLM requests
    - `ollamaModelDiscoveryTimeout`: 10000 (10 seconds) - shorter timeout for model discovery requests which should be fast, but user can increase if needed
    - `ollamaMaxRetries`: 0 - no retries by default (user can enable if needed)
    - `ollamaRetryDelay`: 1000 (1s) - reasonable default (only used if retries enabled)
    - `ollamaEnableLogging`: false - no logging by default

2. **Optional Fields**: All new fields are optional in the schema

    - Existing configurations will continue to work
    - Missing fields will use defaults

3. **Gradual Migration**:
    - Existing users will get defaults automatically
    - No breaking changes to existing functionality
    - Settings can be configured incrementally

### Data Migration

No data migration needed - all fields are optional with sensible defaults.

---

## Related Documentation

- [OLLAMA_INTEGRATION_DOCUMENTATION.md](./OLLAMA_INTEGRATION_DOCUMENTATION.md) - Current Ollama integration implementation
- [axios-library-api-summary.md](./axios-library-api-summary.md) - Axios configuration options and recommendations
- [BrowserSettings.tsx](../webview-ui/src/components/settings/BrowserSettings.tsx) - Reference implementation for test connection button
- [Unbound.tsx](../webview-ui/src/components/settings/providers/Unbound.tsx) - Reference implementation for refresh models button

---

## Summary

This design document outlines a comprehensive enhancement to the Ollama provider settings that will:

1. **Improve user experience** with connection testing and manual model refresh
2. **Provide better control** over API behavior with configurable timeouts and retries
3. **Enhance debugging** with optional request/response logging
4. **Maintain backward compatibility** with sensible defaults
5. **Follow existing patterns** in the codebase for consistency

The implementation follows idiomatic TypeScript/React patterns used throughout Roo Code and includes comprehensive testing requirements to ensure reliability.
