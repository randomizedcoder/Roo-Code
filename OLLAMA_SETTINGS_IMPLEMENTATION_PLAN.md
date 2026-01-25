# Ollama Settings Enhancement Implementation Plan

## Table of Contents

1. [Overview](#overview)
2. [Implementation Phases](#implementation-phases)
    - [Phase 1: Data Model and Schema](#phase-1-data-model-and-schema)
    - [Phase 2: Backend Axios Configuration](#phase-2-backend-axios-configuration)
    - [Phase 3: Backend Model Discovery Updates](#phase-3-backend-model-discovery-updates)
    - [Phase 4: Backend Connection Testing](#phase-4-backend-connection-testing)
    - [Phase 5: Backend Message Handlers](#phase-5-backend-message-handlers)
    - [Phase 6: Frontend Component Updates](#phase-6-frontend-component-updates)
    - [Phase 7: Translation Keys](#phase-7-translation-keys)
    - [Phase 8: Integration Testing](#phase-8-integration-testing)
3. [Performance Optimizations](#performance-optimizations)
4. [Code Quality Standards](#code-quality-standards)
5. [Related Documentation](#related-documentation)

---

## Overview

This implementation plan breaks down the Ollama settings enhancement into 8 distinct phases, each with clear deliverables, tests, and definition of done. The implementation follows idiomatic TypeScript/React patterns with minimal comments, focusing on self-documenting code.

**Key Principles**:

- **No fallback code**: Direct implementation without backward compatibility shims
- **Idiomatic code**: Follow existing Roo Code patterns and conventions
- **Minimal comments**: Code should be self-documenting through clear naming
- **Performance-first**: Each phase considers performance optimizations
- **Test-driven**: Tests defined before implementation

**Related Documentation**:

- [OLLAMA_SETTINGS_ENHANCEMENT_DESIGN.md](./OLLAMA_SETTINGS_ENHANCEMENT_DESIGN.md) - Complete design specification
- [OLLAMA_INTEGRATION_DOCUMENTATION.md](./OLLAMA_INTEGRATION_DOCUMENTATION.md) - Current integration details
- [OLLAMA_API_ENHANCEMENTS.md](./OLLAMA_API_ENHANCEMENTS.md) - API enhancements for logging

---

## Implementation Phases

### Phase 1: Data Model and Schema

**Objective**: Extend the provider settings schema to include new Ollama configuration fields.

**Files to Modify**:

- `packages/types/src/provider-settings.ts`

**Implementation Tasks**:

1. **Update Zod Schema**:

    ```typescript
    const ollamaSchema = baseProviderSettingsSchema.extend({
    	ollamaModelId: z.string().optional(),
    	ollamaBaseUrl: z.string().optional(),
    	ollamaApiKey: z.string().optional(),
    	ollamaNumCtx: z.number().int().min(128).optional(),
    	ollamaRequestTimeout: z.number().int().min(1000).max(7200000).optional(),
    	ollamaModelDiscoveryTimeout: z.number().int().min(1000).max(600000).optional(),
    	ollamaMaxRetries: z.number().int().min(0).max(10).optional(),
    	ollamaRetryDelay: z.number().int().min(100).max(10000).optional(),
    	ollamaEnableLogging: z.boolean().optional(),
    })
    ```

2. **Add Default Values** (if defaults object exists):
    ```typescript
    const defaultOllamaSettings = {
    	ollamaRequestTimeout: 3600000,
    	ollamaModelDiscoveryTimeout: 10000,
    	ollamaMaxRetries: 0,
    	ollamaRetryDelay: 1000,
    	ollamaEnableLogging: false,
    }
    ```

**Tests**:

**File**: `packages/types/src/__tests__/provider-settings.spec.ts`

```typescript
import { describe, it, expect } from "vitest"
import { providerSettingsSchema } from "../provider-settings"

describe("Ollama Settings Schema", () => {
	it("should accept valid ollamaRequestTimeout", () => {
		const result = providerSettingsSchema.safeParse({
			ollamaRequestTimeout: 3600000,
		})
		expect(result.success).toBe(true)
	})

	it("should reject ollamaRequestTimeout below minimum", () => {
		const result = providerSettingsSchema.safeParse({
			ollamaRequestTimeout: 500,
		})
		expect(result.success).toBe(false)
	})

	it("should reject ollamaRequestTimeout above maximum", () => {
		const result = providerSettingsSchema.safeParse({
			ollamaRequestTimeout: 8000000,
		})
		expect(result.success).toBe(false)
	})

	it("should accept valid ollamaModelDiscoveryTimeout", () => {
		const result = providerSettingsSchema.safeParse({
			ollamaModelDiscoveryTimeout: 10000,
		})
		expect(result.success).toBe(true)
	})

	it("should reject ollamaModelDiscoveryTimeout above maximum", () => {
		const result = providerSettingsSchema.safeParse({
			ollamaModelDiscoveryTimeout: 700000,
		})
		expect(result.success).toBe(false)
	})

	it("should accept valid ollamaMaxRetries", () => {
		const result = providerSettingsSchema.safeParse({
			ollamaMaxRetries: 3,
		})
		expect(result.success).toBe(true)
	})

	it("should reject ollamaMaxRetries above maximum", () => {
		const result = providerSettingsSchema.safeParse({
			ollamaMaxRetries: 15,
		})
		expect(result.success).toBe(false)
	})

	it("should accept valid ollamaRetryDelay", () => {
		const result = providerSettingsSchema.safeParse({
			ollamaRetryDelay: 2000,
		})
		expect(result.success).toBe(true)
	})

	it("should reject ollamaRetryDelay below minimum", () => {
		const result = providerSettingsSchema.safeParse({
			ollamaRetryDelay: 50,
		})
		expect(result.success).toBe(false)
	})

	it("should accept ollamaEnableLogging boolean", () => {
		const result = providerSettingsSchema.safeParse({
			ollamaEnableLogging: true,
		})
		expect(result.success).toBe(true)
	})

	it("should accept all optional fields together", () => {
		const result = providerSettingsSchema.safeParse({
			ollamaRequestTimeout: 3600000,
			ollamaModelDiscoveryTimeout: 10000,
			ollamaMaxRetries: 2,
			ollamaRetryDelay: 1000,
			ollamaEnableLogging: true,
		})
		expect(result.success).toBe(true)
	})
})
```

**Performance Optimizations**:

- Use Zod's `.optional()` to avoid unnecessary validation overhead for undefined values
- Schema validation is cached by Zod internally

**Definition of Done**:

- ✅ Schema validates all new fields with correct min/max constraints
- ✅ All tests pass
- ✅ TypeScript types are correctly inferred from schema
- ✅ No breaking changes to existing provider settings
- ✅ Default values are properly typed

---

### Phase 2: Backend Axios Configuration

**Objective**: Create reusable Axios instance factory with retry and logging capabilities.

**Files to Create**:

- `src/api/providers/fetchers/ollama-axios-config.ts`

**Implementation Tasks**:

1. **Create `createOllamaAxiosInstance` function**:

    - Accepts configuration (baseUrl, apiKey, timeout, retries, retryDelay, enableLogging)
    - Creates axios instance with proper defaults
    - Sets up retry interceptor conditionally
    - Sets up logging interceptor conditionally
    - Returns configured instance

2. **Implement `setupRetryInterceptor`**:

    - Exponential backoff: `delay * 2^(retryCount - 1)`
    - Retries only on network errors and 5xx status codes
    - Tracks retry count in request config metadata

3. **Implement `setupLoggingInterceptor`**:
    - Logs request method, URL, timeout, timestamp
    - Logs response status, duration, timestamp
    - Logs errors with code, message, duration, timestamp
    - Uses `Date.now()` for accurate timing (not `new Date()`)

**Tests**:

**File**: `src/api/providers/fetchers/__tests__/ollama-axios-config.spec.ts`

```typescript
import { describe, it, expect, vi, beforeEach } from "vitest"
import axios from "axios"
import { createOllamaAxiosInstance } from "../ollama-axios-config"

vi.mock("axios")

describe("createOllamaAxiosInstance", () => {
	beforeEach(() => {
		vi.clearAllMocks()
		vi.mocked(axios.create).mockReturnValue({
			interceptors: {
				request: { use: vi.fn() },
				response: { use: vi.fn() },
			},
		} as any)
	})

	it("should create instance with default configuration", () => {
		const instance = createOllamaAxiosInstance()
		expect(instance).toBeDefined()
		expect(axios.create).toHaveBeenCalledWith(
			expect.objectContaining({
				baseURL: "http://localhost:11434",
				timeout: 3600000,
			}),
		)
	})

	it("should create instance with custom baseUrl", () => {
		createOllamaAxiosInstance({ baseUrl: "http://custom:11434" })
		expect(axios.create).toHaveBeenCalledWith(
			expect.objectContaining({
				baseURL: "http://custom:11434",
			}),
		)
	})

	it("should include Authorization header when apiKey provided", () => {
		createOllamaAxiosInstance({ apiKey: "test-key" })
		expect(axios.create).toHaveBeenCalledWith(
			expect.objectContaining({
				headers: {
					Authorization: "Bearer test-key",
				},
			}),
		)
	})

	it("should set up retry interceptor when retries > 0", () => {
		const instance = createOllamaAxiosInstance({ retries: 2, retryDelay: 1000 })
		expect(instance.interceptors.response.use).toHaveBeenCalled()
	})

	it("should set up logging interceptor when enableLogging is true", () => {
		const instance = createOllamaAxiosInstance({ enableLogging: true })
		expect(instance.interceptors.request.use).toHaveBeenCalled()
		expect(instance.interceptors.response.use).toHaveBeenCalledTimes(2)
	})

	it("should not set up logging interceptor when enableLogging is false", () => {
		const instance = createOllamaAxiosInstance({ enableLogging: false })
		expect(instance.interceptors.request.use).not.toHaveBeenCalled()
	})

	it("should use custom timeout", () => {
		createOllamaAxiosInstance({ timeout: 5000 })
		expect(axios.create).toHaveBeenCalledWith(
			expect.objectContaining({
				timeout: 5000,
			}),
		)
	})
})

describe("Retry Interceptor", () => {
	it("should retry on ECONNREFUSED", async () => {
		// Test retry logic with mocked axios error
	})

	it("should retry on ETIMEDOUT", async () => {
		// Test retry logic
	})

	it("should retry on 5xx status codes", async () => {
		// Test retry logic
	})

	it("should not retry on 4xx status codes", async () => {
		// Test no retry
	})

	it("should use exponential backoff", async () => {
		// Test delay calculation: delay * 2^(retryCount - 1)
	})

	it("should respect max retries", async () => {
		// Test that retries stop after max count
	})
})

describe("Logging Interceptor", () => {
	it("should log request with metadata", () => {
		const consoleSpy = vi.spyOn(console, "debug").mockImplementation(() => {})
		// Test request logging
		consoleSpy.mockRestore()
	})

	it("should log response with duration", () => {
		// Test response logging with timing
	})

	it("should log errors with duration", () => {
		// Test error logging with timing
	})
})
```

**Performance Optimizations**:

- Only set up interceptors when needed (conditional setup)
- Use `Date.now()` instead of `new Date()` for better performance
- Store startTime in request metadata to avoid object creation overhead
- Retry interceptor only processes errors that match retry criteria (early return)

**Definition of Done**:

- ✅ Axios instance created with correct configuration
- ✅ Retry interceptor implements exponential backoff correctly
- ✅ Logging interceptor captures timing accurately
- ✅ All unit tests pass
- ✅ No memory leaks (interceptors properly configured)
- ✅ Code follows existing patterns (no comments, idiomatic TypeScript)

---

### Phase 3: Backend Model Discovery Updates

**Objective**: Update `getOllamaModels` to use configured Axios instance with timeout and retry support.

**Files to Modify**:

- `src/api/providers/fetchers/ollama.ts`

**Implementation Tasks**:

1. **Update function signature**:

    - Add optional `config` parameter with timeout, retries, etc.
    - Maintain backward compatibility (all config fields optional)

2. **Replace axios calls**:

    - Use `createOllamaAxiosInstance` instead of default axios
    - Pass model discovery timeout (default: 10000ms)
    - Pass retry configuration
    - Pass logging configuration

3. **Maintain existing behavior**:
    - Same error handling
    - Same model filtering logic
    - Same return type

**Tests**:

**File**: `src/api/providers/fetchers/__tests__/ollama.spec.ts`

```typescript
import { describe, it, expect, vi, beforeEach } from "vitest"
import { getOllamaModels } from "../ollama"
import { createOllamaAxiosInstance } from "../ollama-axios-config"

vi.mock("../ollama-axios-config")

describe("getOllamaModels", () => {
	const mockAxiosInstance = {
		get: vi.fn(),
		post: vi.fn(),
	}

	beforeEach(() => {
		vi.clearAllMocks()
		vi.mocked(createOllamaAxiosInstance).mockReturnValue(mockAxiosInstance as any)
	})

	it("should use default timeout when no config provided", async () => {
		mockAxiosInstance.get.mockResolvedValue({
			data: { models: [] },
		})

		await getOllamaModels()

		expect(createOllamaAxiosInstance).toHaveBeenCalledWith(
			expect.objectContaining({
				timeout: 10000,
			}),
		)
	})

	it("should use modelDiscoveryTimeout from config", async () => {
		mockAxiosInstance.get.mockResolvedValue({
			data: { models: [] },
		})

		await getOllamaModels("http://localhost:11434", undefined, {
			modelDiscoveryTimeout: 5000,
		})

		expect(createOllamaAxiosInstance).toHaveBeenCalledWith(
			expect.objectContaining({
				timeout: 5000,
			}),
		)
	})

	it("should pass retry configuration", async () => {
		mockAxiosInstance.get.mockResolvedValue({
			data: { models: [] },
		})

		await getOllamaModels("http://localhost:11434", undefined, {
			maxRetries: 2,
			retryDelay: 500,
		})

		expect(createOllamaAxiosInstance).toHaveBeenCalledWith(
			expect.objectContaining({
				retries: 2,
				retryDelay: 500,
			}),
		)
	})

	it("should pass logging configuration", async () => {
		mockAxiosInstance.get.mockResolvedValue({
			data: { models: [] },
		})

		await getOllamaModels("http://localhost:11434", undefined, {
			enableLogging: true,
		})

		expect(createOllamaAxiosInstance).toHaveBeenCalledWith(
			expect.objectContaining({
				enableLogging: true,
			}),
		)
	})

	it("should handle API key in headers", async () => {
		mockAxiosInstance.get.mockResolvedValue({
			data: { models: [] },
		})

		await getOllamaModels("http://localhost:11434", "test-key")

		expect(createOllamaAxiosInstance).toHaveBeenCalledWith(
			expect.objectContaining({
				apiKey: "test-key",
			}),
		)
	})

	it("should maintain existing model filtering behavior", async () => {
		mockAxiosInstance.get.mockResolvedValue({
			data: {
				models: [
					{ name: "model1", model: "model1" },
					{ name: "model2", model: "model2" },
				],
			},
		})

		mockAxiosInstance.post
			.mockResolvedValueOnce({
				data: {
					capabilities: ["tools"],
					details: { family: "llama" },
					model_info: { "llama.context_length": 8192 },
				},
			})
			.mockResolvedValueOnce({
				data: {
					capabilities: ["completion"],
					details: { family: "llama" },
					model_info: { "llama.context_length": 4096 },
				},
			})

		const models = await getOllamaModels()

		expect(Object.keys(models).length).toBe(1)
		expect(models["model1"]).toBeDefined()
	})

	it("should handle connection errors gracefully", async () => {
		mockAxiosInstance.get.mockRejectedValue({
			code: "ECONNREFUSED",
		})

		const models = await getOllamaModels()

		expect(models).toEqual({})
	})
})
```

**Performance Optimizations**:

- Reuse single Axios instance for all requests in the function
- Parallel model detail fetching (existing `Promise.all` pattern)
- Early return on invalid URL (before creating axios instance)

**Definition of Done**:

- ✅ Function uses configured Axios instance
- ✅ Timeout configuration applied correctly
- ✅ Retry configuration applied correctly
- ✅ Logging configuration applied correctly
- ✅ All existing tests still pass
- ✅ New tests for configuration pass
- ✅ No performance regression

---

### Phase 4: Backend Connection Testing

**Objective**: Implement `testOllamaConnection` function with timing information.

**Files to Modify**:

- `src/api/providers/fetchers/ollama.ts`

**Implementation Tasks**:

1. **Create `testOllamaConnection` function**:

    - Accepts baseUrl, apiKey, optional config
    - Tracks start time with `Date.now()`
    - Uses `createOllamaAxiosInstance` with model discovery timeout
    - Makes GET request to `/api/tags`
    - Calculates duration
    - Returns `{ success: boolean, message: string, durationMs: number }`
    - Logs comprehensive metrics when logging enabled

2. **Error Handling**:
    - Invalid URL: immediate return with duration
    - ECONNREFUSED: specific error message
    - ETIMEDOUT/ECONNABORTED: timeout message
    - ERR_NETWORK: network error message
    - HTTP errors: status code message
    - Generic errors: error message

**Tests**:

**File**: `src/api/providers/fetchers/__tests__/ollama.spec.ts` (add to existing file)

```typescript
describe("testOllamaConnection", () => {
	it("should return success for valid connection", async () => {
		mockAxiosInstance.get.mockResolvedValue({
			status: 200,
			data: { models: [] },
		})

		const result = await testOllamaConnection("http://localhost:11434")

		expect(result.success).toBe(true)
		expect(result.message).toContain("Successfully connected")
		expect(result.durationMs).toBeGreaterThanOrEqual(0)
	})

	it("should return failure for invalid URL", async () => {
		const result = await testOllamaConnection("invalid-url")

		expect(result.success).toBe(false)
		expect(result.message).toContain("Invalid URL")
		expect(result.durationMs).toBeGreaterThanOrEqual(0)
	})

	it("should return failure for ECONNREFUSED", async () => {
		mockAxiosInstance.get.mockRejectedValue({
			code: "ECONNREFUSED",
		})

		const result = await testOllamaConnection("http://localhost:11434")

		expect(result.success).toBe(false)
		expect(result.message).toContain("Cannot connect")
		expect(result.durationMs).toBeGreaterThanOrEqual(0)
	})

	it("should return failure for timeout", async () => {
		mockAxiosInstance.get.mockRejectedValue({
			code: "ETIMEDOUT",
		})

		const result = await testOllamaConnection("http://localhost:11434")

		expect(result.success).toBe(false)
		expect(result.message).toContain("timed out")
		expect(result.durationMs).toBeGreaterThanOrEqual(0)
	})

	it("should use model discovery timeout", async () => {
		mockAxiosInstance.get.mockResolvedValue({
			status: 200,
			data: { models: [] },
		})

		await testOllamaConnection("http://localhost:11434", undefined, {
			timeout: 5000,
		})

		expect(createOllamaAxiosInstance).toHaveBeenCalledWith(
			expect.objectContaining({
				timeout: 5000,
			}),
		)
	})

	it("should log metrics when logging enabled", async () => {
		const consoleSpy = vi.spyOn(console, "debug").mockImplementation(() => {})
		mockAxiosInstance.get.mockResolvedValue({
			status: 200,
			data: { models: [] },
		})

		await testOllamaConnection("http://localhost:11434", undefined, {
			enableLogging: true,
		})

		expect(consoleSpy).toHaveBeenCalledWith(
			expect.stringContaining("Ollama Connection Test"),
			expect.objectContaining({
				success: true,
				durationMs: expect.any(Number),
			}),
		)

		consoleSpy.mockRestore()
	})

	it("should include API key in request", async () => {
		mockAxiosInstance.get.mockResolvedValue({
			status: 200,
			data: { models: [] },
		})

		await testOllamaConnection("http://localhost:11434", "test-key")

		expect(createOllamaAxiosInstance).toHaveBeenCalledWith(
			expect.objectContaining({
				apiKey: "test-key",
			}),
		)
	})
})
```

**Performance Optimizations**:

- Use `Date.now()` for timing (faster than `new Date()`)
- Early return on invalid URL (no axios instance creation)
- No retries for connection test (faster failure)
- Conditional logging (only when enabled)

**Definition of Done**:

- ✅ Function returns success/failure with timing
- ✅ All error cases handled correctly
- ✅ Timing information accurate
- ✅ Logging works when enabled
- ✅ All tests pass
- ✅ Function completes quickly (< 100ms for local connection)

---

### Phase 5: Backend Message Handlers

**Objective**: Add message handlers for test connection and refresh models with timing.

**Files to Modify**:

- `src/core/webview/webviewMessageHandler.ts`
- `src/api/providers/fetchers/modelCache.ts`

**Implementation Tasks**:

1. **Add `testOllamaConnection` handler**:

    - Extract Ollama config from provider state
    - Call `testOllamaConnection` with config
    - Post result with `durationMs` to webview

2. **Update `refreshOllamaModels` handler**:

    - Track start time
    - Extract Ollama config
    - Pass config to `getModels` via options
    - Calculate duration
    - Post result with `durationMs` to webview
    - Log metrics when logging enabled

3. **Update `modelCache.ts`**:

    - Update `fetchModelsFromProvider` to pass Ollama config
    - Use type assertion for Ollama-specific options

4. **Update message types**:
    - Add `durationMs?: number` to result message types

**Tests**:

**File**: `src/core/webview/__tests__/ollama-settings.spec.ts`

```typescript
import { describe, it, expect, vi, beforeEach } from "vitest"
import { testOllamaConnection } from "../../../api/providers/fetchers/ollama"

vi.mock("../../../api/providers/fetchers/ollama")
vi.mock("../../../api/providers/fetchers/modelCache")

describe("Ollama Message Handlers", () => {
	const mockProvider = {
		getState: vi.fn(),
		postMessageToWebview: vi.fn(),
	}

	beforeEach(() => {
		vi.clearAllMocks()
		mockProvider.getState.mockResolvedValue({
			apiConfiguration: {
				ollamaBaseUrl: "http://localhost:11434",
				ollamaApiKey: "test-key",
				ollamaModelDiscoveryTimeout: 10000,
				ollamaEnableLogging: false,
			},
		})
	})

	describe("testOllamaConnection handler", () => {
		it("should call testOllamaConnection with correct config", async () => {
			vi.mocked(testOllamaConnection).mockResolvedValue({
				success: true,
				message: "Connected",
				durationMs: 50,
			})

			// Simulate message handler
			const result = await testOllamaConnection("http://localhost:11434", "test-key", {
				timeout: 10000,
				enableLogging: false,
			})

			expect(testOllamaConnection).toHaveBeenCalledWith(
				"http://localhost:11434",
				"test-key",
				expect.objectContaining({
					timeout: 10000,
					enableLogging: false,
				}),
			)

			expect(result.durationMs).toBe(50)
		})

		it("should post message with durationMs", async () => {
			vi.mocked(testOllamaConnection).mockResolvedValue({
				success: true,
				message: "Connected",
				durationMs: 75,
			})

			// Handler should post message
			// Verify postMessageToWebview called with durationMs
		})

		it("should handle errors and still include durationMs", async () => {
			vi.mocked(testOllamaConnection).mockResolvedValue({
				success: false,
				message: "Connection failed",
				durationMs: 100,
			})

			// Verify error handling includes duration
		})
	})

	describe("refreshOllamaModels handler", () => {
		it("should track duration for model refresh", async () => {
			// Test that duration is calculated and included
		})

		it("should pass Ollama config to getModels", async () => {
			// Test config propagation
		})

		it("should log metrics when logging enabled", async () => {
			// Test logging
		})
	})
})
```

**Performance Optimizations**:

- Use `Date.now()` for timing (not `new Date()`)
- Avoid unnecessary object creation in hot path
- Cache provider state if accessed multiple times
- Parallel model fetching (existing pattern)

**Definition of Done**:

- ✅ Test connection handler works with timing
- ✅ Refresh models handler works with timing
- ✅ Config properly passed through to `getOllamaModels`
- ✅ Message types updated correctly
- ✅ All tests pass
- ✅ No performance regression in message handling

---

### Phase 6: Frontend Component Updates

**Objective**: Update Ollama component with new UI elements and message handling.

**Files to Modify**:

- `webview-ui/src/components/settings/providers/Ollama.tsx`

**Implementation Tasks**:

1. **Add State Management**:

    - `testingConnection`, `testResult` (with `durationMs`)
    - `refreshingModels`, `refreshResult` (with `durationMs`)
    - `showAdvanced` for collapsible section
    - Timer refs for auto-clearing results

2. **Add Test Connection Button**:

    - Next to Base URL field
    - Disabled while testing
    - Shows loading state
    - Displays result with timing

3. **Add Refresh Models Button**:

    - Near Model ID field
    - Disabled while refreshing
    - Shows loading state
    - Displays result with timing

4. **Add Advanced Settings Section**:

    - Collapsible button
    - Streaming checkbox (checked, disabled)
    - Request Timeout input
    - Model Discovery Timeout input
    - Max Retries input
    - Retry Delay input
    - Enable Logging checkbox

5. **Update Message Handlers**:
    - Handle `ollamaConnectionTestResult` with `durationMs`
    - Handle `ollamaModelsRefreshResult` with `durationMs`
    - Display timing information in results

**Tests**:

**File**: `webview-ui/src/components/settings/providers/__tests__/Ollama.spec.tsx`

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { render, screen, waitFor } from '@testing-library/react'
import { Ollama } from '../Ollama'

vi.mock('@src/utils/vscode', () => ({
	vscode: {
		postMessage: vi.fn(),
	},
}))

describe('Ollama Component', () => {
	const mockSetApiConfigurationField = vi.fn()
	const defaultProps = {
		apiConfiguration: {},
		setApiConfigurationField: mockSetApiConfigurationField,
	}

	beforeEach(() => {
		vi.clearAllMocks()
	})

	it('should render Base URL field', () => {
		render(<Ollama {...defaultProps} />)
		expect(screen.getByLabelText(/base url/i)).toBeInTheDocument()
	})

	it('should render Test Connection button', () => {
		render(<Ollama {...defaultProps} />)
		expect(screen.getByRole('button', { name: /test/i })).toBeInTheDocument()
	})

	it('should render Refresh Models button', () => {
		render(<Ollama {...defaultProps} />)
		expect(screen.getByRole('button', { name: /refresh/i })).toBeInTheDocument()
	})

	it('should display test result with timing', async () => {
		render(<Ollama {...defaultProps} />)

		// Simulate message with duration
		window.postMessage({
			type: 'ollamaConnectionTestResult',
			success: true,
			message: 'Connected',
			durationMs: 125,
		}, '*')

		await waitFor(() => {
			expect(screen.getByText(/connected/i)).toBeInTheDocument()
			expect(screen.getByText(/125ms/i)).toBeInTheDocument()
		})
	})

	it('should display refresh result with timing', async () => {
		render(<Ollama {...defaultProps} />)

		window.postMessage({
			type: 'ollamaModelsRefreshResult',
			success: true,
			message: 'Found 3 models',
			durationMs: 250,
		}, '*')

		await waitFor(() => {
			expect(screen.getByText(/found 3 models/i)).toBeInTheDocument()
			expect(screen.getByText(/250ms/i)).toBeInTheDocument()
		})
	})

	it('should toggle advanced settings', () => {
		render(<Ollama {...defaultProps} />)

		const toggleButton = screen.getByRole('button', { name: /advanced/i })
		expect(screen.queryByLabelText(/request timeout/i)).not.toBeInTheDocument()

		toggleButton.click()

		expect(screen.getByLabelText(/request timeout/i)).toBeInTheDocument()
	})

	it('should render streaming checkbox as checked and disabled', () => {
		render(<Ollama {...defaultProps} />)

		const toggleButton = screen.getByRole('button', { name: /advanced/i })
		toggleButton.click()

		const streamingCheckbox = screen.getByLabelText(/streaming/i)
		expect(streamingCheckbox).toBeChecked()
		expect(streamingCheckbox).toBeDisabled()
	})

	it('should update request timeout value', () => {
		render(<Ollama {...defaultProps} />)

		const toggleButton = screen.getByRole('button', { name: /advanced/i })
		toggleButton.click()

		const timeoutInput = screen.getByLabelText(/request timeout/i)
		timeoutInput.value = '7200000'
		timeoutInput.dispatchEvent(new Event('input'))

		expect(mockSetApiConfigurationField).toHaveBeenCalledWith('ollamaRequestTimeout', 7200000)
	})

	it('should validate timeout ranges', () => {
		render(<Ollama {...defaultProps} />)

		const toggleButton = screen.getByRole('button', { name: /advanced/i })
		toggleButton.click()

		const timeoutInput = screen.getByLabelText(/request timeout/i)
		timeoutInput.value = '500'
		timeoutInput.dispatchEvent(new Event('input'))

		expect(mockSetApiConfigurationField).not.toHaveBeenCalledWith('ollamaRequestTimeout', 500)
	})

	it('should disable test button while testing', () => {
		render(<Ollama {...defaultProps} />)

		const testButton = screen.getByRole('button', { name: /test/i })
		testButton.click()

		expect(testButton).toBeDisabled()
	})

	it('should disable refresh button while refreshing', () => {
		render(<Ollama {...defaultProps} />)

		const refreshButton = screen.getByRole('button', { name: /refresh/i })
		refreshButton.click()

		expect(refreshButton).toBeDisabled()
	})
})
```

**Performance Optimizations**:

- Use `useCallback` for event handlers to prevent re-renders
- Use `useMemo` for derived state
- Debounce input changes if needed (for number inputs)
- Clear timers on unmount to prevent memory leaks
- Minimize re-renders with proper state management

**Definition of Done**:

- ✅ All UI elements render correctly
- ✅ Test connection button works with timing display
- ✅ Refresh models button works with timing display
- ✅ Advanced settings toggle works
- ✅ All input fields update configuration correctly
- ✅ Streaming checkbox is read-only
- ✅ All tests pass
- ✅ No console errors or warnings
- ✅ Component performance is acceptable (no unnecessary re-renders)

---

### Phase 7: Translation Keys

**Objective**: Add all required translation keys for new UI elements.

**Files to Modify**:

- Translation files (e.g., `webview-ui/src/i18n/locales/en/settings.json`)

**Implementation Tasks**:

1. **Add Translation Keys**:
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
    			"retryDelayHelp": "Initial delay between retry attempts in milliseconds. Uses exponential backoff. Default: 1000 (1 second). Range: 100-10000.",
    			"enableLogging": "Enable Request Logging",
    			"enableLoggingHelp": "Enable detailed logging of Ollama API requests, responses, and errors. Logs include timing information and connection details."
    		}
    	}
    }
    ```

**Tests**:

**File**: `webview-ui/src/i18n/__tests__/translations.spec.ts`

```typescript
import { describe, it, expect } from "vitest"
import enTranslations from "../locales/en/settings.json"

describe("Ollama Translation Keys", () => {
	it("should have all required keys", () => {
		const ollamaKeys = enTranslations.providers?.ollama
		expect(ollamaKeys).toBeDefined()
		expect(ollamaKeys.test).toBeDefined()
		expect(ollamaKeys.testing).toBeDefined()
		expect(ollamaKeys.refreshModels).toBeDefined()
		expect(ollamaKeys.refreshing).toBeDefined()
		expect(ollamaKeys.advancedSettings).toBeDefined()
		expect(ollamaKeys.streaming).toBeDefined()
		expect(ollamaKeys.streamingHelp).toBeDefined()
		expect(ollamaKeys.requestTimeout).toBeDefined()
		expect(ollamaKeys.requestTimeoutHelp).toBeDefined()
		expect(ollamaKeys.modelDiscoveryTimeout).toBeDefined()
		expect(ollamaKeys.modelDiscoveryTimeoutHelp).toBeDefined()
		expect(ollamaKeys.maxRetries).toBeDefined()
		expect(ollamaKeys.maxRetriesHelp).toBeDefined()
		expect(ollamaKeys.retryDelay).toBeDefined()
		expect(ollamaKeys.retryDelayHelp).toBeDefined()
		expect(ollamaKeys.enableLogging).toBeDefined()
		expect(ollamaKeys.enableLoggingHelp).toBeDefined()
	})

	it("should have non-empty values", () => {
		const ollamaKeys = enTranslations.providers?.ollama
		Object.values(ollamaKeys).forEach((value) => {
			expect(typeof value).toBe("string")
			expect(value.length).toBeGreaterThan(0)
		})
	})
})
```

**Performance Optimizations**:

- Translation keys are loaded once and cached
- No runtime performance impact

**Definition of Done**:

- ✅ All translation keys added
- ✅ Keys are properly nested
- ✅ Help text is clear and informative
- ✅ All tests pass
- ✅ Translations work in UI

---

### Phase 8: Integration Testing

**Objective**: End-to-end testing of complete feature with real Ollama instance.

**Files to Create**:

- `src/api/providers/fetchers/__tests__/ollama-integration.spec.ts`
- `src/core/webview/__tests__/ollama-settings-integration.spec.ts`

**Implementation Tasks**:

1. **Create Integration Test Suite**:

    - Test with real Ollama instance (if available)
    - Test with mocked Ollama instance
    - Test complete flow: config → request → response → UI update

2. **Test Scenarios**:
    - Successful connection test with timing
    - Failed connection test with timing
    - Successful model refresh with timing
    - Failed model refresh with timing
    - Configuration persistence
    - Timeout behavior
    - Retry behavior
    - Logging behavior

**Tests**:

**File**: `src/api/providers/fetchers/__tests__/ollama-integration.spec.ts`

```typescript
import { describe, it, expect, beforeAll, afterAll } from "vitest"
import { testOllamaConnection, getOllamaModels } from "../ollama"

describe("Ollama Integration Tests", () => {
	const OLLAMA_URL = process.env.OLLAMA_URL || "http://localhost:11434"
	const hasOllama = process.env.RUN_INTEGRATION_TESTS === "true"

	beforeAll(() => {
		if (!hasOllama) {
			console.log("Skipping integration tests (set RUN_INTEGRATION_TESTS=true)")
		}
	})

	describe.skipIf(!hasOllama)("Real Ollama Instance", () => {
		it("should test connection successfully", async () => {
			const result = await testOllamaConnection(OLLAMA_URL)

			expect(result.success).toBe(true)
			expect(result.durationMs).toBeGreaterThan(0)
			expect(result.durationMs).toBeLessThan(5000)
		})

		it("should fetch models with timing", async () => {
			const startTime = Date.now()
			const models = await getOllamaModels(OLLAMA_URL, undefined, {
				modelDiscoveryTimeout: 10000,
			})
			const duration = Date.now() - startTime

			expect(Object.keys(models).length).toBeGreaterThan(0)
			expect(duration).toBeLessThan(10000)
		})

		it("should respect timeout configuration", async () => {
			const result = await testOllamaConnection(OLLAMA_URL, undefined, {
				timeout: 100,
			})

			expect(result.durationMs).toBeLessThan(5000)
		})

		it("should log when logging enabled", async () => {
			const consoleSpy = vi.spyOn(console, "debug").mockImplementation(() => {})

			await testOllamaConnection(OLLAMA_URL, undefined, {
				enableLogging: true,
			})

			expect(consoleSpy).toHaveBeenCalled()

			consoleSpy.mockRestore()
		})
	})

	describe("Configuration Flow", () => {
		it("should pass config through getOllamaModels", async () => {
			const models = await getOllamaModels("http://localhost:11434", undefined, {
				modelDiscoveryTimeout: 5000,
				maxRetries: 1,
				retryDelay: 500,
				enableLogging: false,
			})

			expect(models).toBeDefined()
		})
	})
})
```

**Performance Optimizations**:

- Integration tests can be skipped if Ollama not available
- Use environment variables to control test execution
- Mock external dependencies for faster test runs

**Definition of Done**:

- ✅ All integration tests pass (when Ollama available)
- ✅ Tests can run with mocked Ollama
- ✅ Complete flow works end-to-end
- ✅ Timing information accurate in real scenarios
- ✅ Configuration properly applied
- ✅ No performance regressions observed

---

## Performance Optimizations

### Cross-Phase Optimizations

1. **Timing Measurement**:

    - Always use `Date.now()` instead of `new Date()` (faster, no object creation)
    - Store start time in request metadata (single property access)
    - Calculate duration only when needed (lazy evaluation)

2. **Axios Instance Reuse**:

    - Create instance once per request flow
    - Reuse interceptors (they're shared across requests)
    - Avoid creating multiple instances unnecessarily

3. **Conditional Feature Loading**:

    - Only set up interceptors when needed (retries > 0, logging enabled)
    - Early returns in error handlers
    - Skip unnecessary operations

4. **Frontend Optimizations**:

    - Use `useCallback` for event handlers
    - Use `useMemo` for derived values
    - Debounce number input changes (optional, if needed)
    - Clear timers on unmount

5. **Memory Management**:

    - Clear timeout refs on component unmount
    - Remove event listeners properly
    - Avoid memory leaks in interceptors

6. **Network Optimizations**:
    - Use appropriate timeouts (shorter for discovery, longer for LLM)
    - No retries for connection tests (faster feedback)
    - Parallel model detail fetching (existing pattern)

---

## Code Quality Standards

### TypeScript

- **Strict typing**: No `any` types except where necessary (axios internals)
- **Type inference**: Let TypeScript infer types where possible
- **Interface definitions**: Clear, descriptive interface names
- **Optional chaining**: Use `?.` for safe property access

### React

- **Hooks**: Follow rules of hooks
- **Memoization**: Use `useCallback` and `useMemo` appropriately
- **State management**: Minimal state, derive when possible
- **Event handlers**: Properly typed with `useCallback`

### Error Handling

- **Specific errors**: Handle different error types appropriately
- **User-friendly messages**: Clear error messages for users
- **Logging**: Log errors with context when logging enabled
- **No silent failures**: Always return meaningful results

### Testing

- **Unit tests**: Test individual functions in isolation
- **Integration tests**: Test complete flows
- **Mocking**: Mock external dependencies appropriately
- **Coverage**: Aim for >80% coverage on new code

### Code Style

- **Idiomatic code**: Follow existing Roo Code patterns
- **Minimal comments**: Code should be self-documenting
- **Consistent naming**: Use clear, descriptive names
- **Formatting**: Follow project formatting rules (Prettier, ESLint)

---

## Related Documentation

- [OLLAMA_SETTINGS_ENHANCEMENT_DESIGN.md](./OLLAMA_SETTINGS_ENHANCEMENT_DESIGN.md) - Complete design specification
- [OLLAMA_INTEGRATION_DOCUMENTATION.md](./OLLAMA_INTEGRATION_DOCUMENTATION.md) - Current Ollama integration
- [OLLAMA_API_ENHANCEMENTS.md](./OLLAMA_API_ENHANCEMENTS.md) - API enhancements for logging
- [axios-library-api-summary.md](./axios-library-api-summary.md) - Axios configuration reference

---

## Summary

This implementation plan breaks down the Ollama settings enhancement into 8 manageable phases:

1. **Data Model**: Schema and type definitions
2. **Axios Configuration**: Reusable instance factory
3. **Model Discovery**: Update existing function
4. **Connection Testing**: New function with timing
5. **Message Handlers**: Backend message processing
6. **Frontend**: UI components and state management
7. **Translations**: Internationalization support
8. **Integration**: End-to-end testing

Each phase is independently testable and has clear success criteria. The implementation follows idiomatic TypeScript/React patterns with minimal comments and focuses on performance optimizations throughout.
