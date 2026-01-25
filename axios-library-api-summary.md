# Axios Library API Summary for Roo Code Integration

## Table of Contents

1. [Overview](#overview)
2. [Connection Setup](#connection-setup)
3. [Timeout Configuration](#timeout-configuration)
4. [Retry Mechanisms](#retry-mechanisms)
5. [Logging and Interceptors](#logging-and-interceptors)
6. [Error Handling](#error-handling)
7. [Implementation Recommendations for Roo Code](#implementation-recommendations-for-roo-code)
8. [Code References](#code-references)
9. [Related Documentation](#related-documentation)

---

## Overview

This document provides a comprehensive guide to configuring Axios for use in Roo Code's Ollama integration. It covers connection setup, timeout configuration, retry mechanisms, and logging capabilities based on the Axios source code.

**Axios Repository**: [https://github.com/axios/axios](https://github.com/axios/axios)

**Related Documentation**: See [OLLAMA_INTEGRATION_DOCUMENTATION.md](./OLLAMA_INTEGRATION_DOCUMENTATION.md) for details on how Roo Code currently uses Axios for Ollama integration.

---

## Connection Setup

### Creating Axios Instances

Axios supports creating custom instances with default configuration. This is the recommended approach for Roo Code to centralize Ollama connection settings.

**File**: `lib/axios.js` (main entry point)
**File**: `lib/core/Axios.js` (core implementation)

**Constructor**: `Axios(instanceConfig)`

- **Location**: `lib/core/Axios.js:21-28`
- **Purpose**: Creates a new Axios instance with default configuration

```typescript:21:28:lib/core/Axios.js
class Axios {
  constructor(instanceConfig) {
    this.defaults = instanceConfig || {};
    this.interceptors = {
      request: new InterceptorManager(),
      response: new InterceptorManager()
    };
  }
```

**Factory Function**: `axios.create([config])`

- **Location**: Defined in `lib/axios.js` (exported from main module)
- **Purpose**: Creates a new Axios instance with custom defaults

### Connection Configuration Options

**File**: `index.d.ts` (TypeScript definitions)
**Location**: Lines 320-378

Key connection-related configuration options:

```typescript:320:378:index.d.ts
export interface AxiosRequestConfig<D = any> {
  url?: string;
  method?: StringLiteralsOrString<Method>;
  baseURL?: string;
  allowAbsoluteUrls?: boolean;
  timeout?: Milliseconds;
  timeoutErrorMessage?: string;
  headers?: (RawAxiosRequestHeaders & MethodsHeaders) | AxiosHeaders;
  httpAgent?: any;
  httpsAgent?: any;
  proxy?: AxiosProxyConfig | false;
  socketPath?: string | null;
  transport?: any;
  // ... other options
}
```

### Default Configuration

**File**: `lib/defaults/index.js`
**Location**: Lines 36-161

Default values are defined in the defaults object:

```typescript:36:155:lib/defaults/index.js
const defaults = {
  transitional: transitionalDefaults,
  adapter: ['xhr', 'http', 'fetch'],
  timeout: 0,  // Line 132: Default is 0 (no timeout)
  // ... other defaults
  headers: {
    common: {
      'Accept': 'application/json, text/plain, */*',
      'Content-Type': undefined
    }
  }
};
```

**Key Defaults**:

- **timeout**: `0` (no timeout by default) - **Line 132**
- **adapter**: `['xhr', 'http', 'fetch']` - **Line 40** (tries adapters in order)
- **baseURL**: `undefined` (must be set explicitly)

### Recommended Instance Creation for Roo Code

```typescript
import axios from "axios"

const ollamaAxiosInstance = axios.create({
	baseURL: "http://localhost:11434",
	timeout: 30000, // 30 seconds
	headers: {
		"Content-Type": "application/json",
	},
	// Connection-specific options
	httpAgent: new http.Agent({
		keepAlive: true,
		keepAliveMsecs: 1000,
		maxSockets: 50,
		maxFreeSockets: 10,
		timeout: 30000,
	}),
	httpsAgent: new https.Agent({
		keepAlive: true,
		keepAliveMsecs: 1000,
		maxSockets: 50,
		maxFreeSockets: 10,
		timeout: 30000,
	}),
})
```

---

## Timeout Configuration

### Timeout Property

**File**: `index.d.ts`
**Location**: Line 331

```typescript:331:331:index.d.ts
timeout?: Milliseconds;
```

**File**: `lib/defaults/index.js`
**Location**: Lines 129-132

```typescript:129:132:lib/defaults/index.js
/**
 * A timeout in milliseconds to abort a request. If set to 0 (default) a
 * timeout is not created.
 */
timeout: 0,
```

### Timeout Implementation - HTTP Adapter (Node.js)

**File**: `lib/adapters/http.js`
**Location**: Lines 829-867

The HTTP adapter implements timeout using Node.js `ClientRequest.setTimeout()`:

```typescript:829:867:lib/adapters/http.js
// Handle request timeout
if (config.timeout) {
  // This is forcing a int timeout to avoid problems if the `req` interface doesn't handle other types.
  const timeout = parseInt(config.timeout, 10);

  if (Number.isNaN(timeout)) {
    abort(new AxiosError(
      'error trying to parse `config.timeout` to int',
      AxiosError.ERR_BAD_OPTION_VALUE,
      config,
      req
    ));
    return;
  }

  // ClientRequest.setTimeout will be fired on the specify milliseconds, and can make sure that abort() will be fired after connect.
  req.setTimeout(timeout, function handleRequestTimeout() {
    if (isDone) return;
    let timeoutErrorMessage = config.timeout ? 'timeout of ' + config.timeout + 'ms exceeded' : 'timeout exceeded';
    const transitional = config.transitional || transitionalDefaults;
    if (config.timeoutErrorMessage) {
      timeoutErrorMessage = config.timeoutErrorMessage;
    }
    abort(new AxiosError(
      timeoutErrorMessage,
      transitional.clarifyTimeoutError ? AxiosError.ETIMEDOUT : AxiosError.ECONNABORTED,
      config,
      req
    ));
  });
} else {
  // explicitly reset the socket timeout value for a possible `keep-alive` request
  req.setTimeout(0);
}
```

### Timeout Implementation - XHR Adapter (Browser)

**File**: `lib/adapters/xhr.js`
**Location**: Lines 37-38, 119-134

The XHR adapter sets the timeout directly on the XMLHttpRequest object:

```typescript:37:38:lib/adapters/xhr.js
// Set the request timeout in MS
request.timeout = _config.timeout;
```

```typescript:119:134:lib/adapters/xhr.js
// Handle timeout
request.ontimeout = function handleTimeout() {
  let timeoutErrorMessage = _config.timeout ? 'timeout of ' + _config.timeout + 'ms exceeded' : 'timeout exceeded';
  const transitional = _config.transitional || transitionalDefaults;
  if (_config.timeoutErrorMessage) {
    timeoutErrorMessage = _config.timeoutErrorMessage;
  }
  reject(new AxiosError(
    timeoutErrorMessage,
    transitional.clarifyTimeoutError ? AxiosError.ETIMEDOUT : AxiosError.ECONNABORTED,
    config,
    request));

  // Clean up request
  request = null;
};
```

### Timeout Error Messages

**File**: `index.d.ts`
**Location**: Line 332

```typescript:332:332:index.d.ts
timeoutErrorMessage?: string;
```

Custom timeout error messages can be set via `timeoutErrorMessage` property.

### Timeout Error Codes

**File**: `lib/core/AxiosError.js`
**Location**: Lines 62-63

```typescript:62:63:lib/core/AxiosError.js
AxiosError.ECONNABORTED = 'ECONNABORTED';
AxiosError.ETIMEDOUT = 'ETIMEDOUT';
```

The error code depends on the `transitional.clarifyTimeoutError` setting:

- `true`: Uses `ETIMEDOUT` (more specific)
- `false`: Uses `ECONNABORTED` (legacy behavior)

**File**: `lib/defaults/transitional.js`
**Location**: Line 6

```typescript:6:6:lib/defaults/transitional.js
clarifyTimeoutError: false
```

### Timeout with AbortController

**File**: `lib/helpers/composeSignals.js`
**Location**: Lines 5-48

Axios can compose multiple abort signals including timeout:

```typescript:22:25:lib/helpers/composeSignals.js
let timer = timeout && setTimeout(() => {
  timer = null;
  onabort(new AxiosError(`timeout of ${timeout}ms exceeded`, AxiosError.ETIMEDOUT))
}, timeout)
```

### Recommended Timeout Configuration for Roo Code

```typescript
const ollamaAxiosInstance = axios.create({
	baseURL: "http://localhost:11434",
	timeout: 30000, // 30 seconds for model discovery
	timeoutErrorMessage: "Ollama request timed out after 30 seconds",
	transitional: {
		clarifyTimeoutError: true, // Use ETIMEDOUT instead of ECONNABORTED
	},
})
```

---

## Retry Mechanisms

### Built-in Retry Support

**Status**: Axios does **NOT** have built-in retry functionality.

Axios does not include automatic retry logic. Retry mechanisms must be implemented using interceptors or external libraries.

### Retry Implementation Options

#### Option 1: Response Interceptor with Retry Logic

Implement retry logic in a response error interceptor:

**File**: `lib/core/InterceptorManager.js`
**Location**: Lines 5-72

```typescript:19:27:lib/core/InterceptorManager.js
use(fulfilled, rejected, options) {
  this.handlers.push({
    fulfilled,
    rejected,
    synchronous: options ? options.synchronous : false,
    runWhen: options ? options.runWhen : null
  });
  return this.handlers.length - 1;
}
```

#### Option 2: External Libraries

The Axios ecosystem includes retry libraries (see `ECOSYSTEM.md`), but they are not part of the core library.

**File**: `ECOSYSTEM.md`
**Location**: Lines 1-50

### Recommended Retry Implementation for Roo Code

```typescript
import axios, { AxiosError } from "axios"

interface RetryConfig {
	retries?: number
	retryDelay?: number
	retryCondition?: (error: AxiosError) => boolean
}

function setupRetryInterceptor(instance: AxiosInstance, config: RetryConfig = {}) {
	const {
		retries = 3,
		retryDelay = 1000,
		retryCondition = (error: AxiosError) => {
			// Retry on network errors or 5xx errors
			return !error.response || (error.response.status >= 500 && error.response.status < 600)
		},
	} = config

	instance.interceptors.response.use(
		(response) => response,
		async (error: AxiosError) => {
			const config = error.config as any

			// Don't retry if retry count exceeded
			config.__retryCount = config.__retryCount || 0
			if (config.__retryCount >= retries) {
				return Promise.reject(error)
			}

			// Check if we should retry
			if (!retryCondition(error)) {
				return Promise.reject(error)
			}

			// Increment retry count
			config.__retryCount += 1

			// Calculate delay (exponential backoff)
			const delay = retryDelay * Math.pow(2, config.__retryCount - 1)

			// Wait before retrying
			await new Promise((resolve) => setTimeout(resolve, delay))

			// Retry the request
			return instance(config)
		},
	)
}

// Usage
const ollamaAxiosInstance = axios.create({
	baseURL: "http://localhost:11434",
	timeout: 30000,
})

setupRetryInterceptor(ollamaAxiosInstance, {
	retries: 3,
	retryDelay: 1000,
	retryCondition: (error) => {
		// Retry on network errors, timeouts, or 5xx errors
		if (error.code === "ECONNREFUSED" || error.code === "ETIMEDOUT" || error.code === "ERR_NETWORK") {
			return true
		}
		if (error.response && error.response.status >= 500) {
			return true
		}
		return false
	},
})
```

---

## Logging and Interceptors

### Interceptor System

Axios provides a powerful interceptor system for logging and request/response transformation.

**File**: `lib/core/InterceptorManager.js`
**Location**: Lines 5-72

**File**: `lib/core/Axios.js`
**Location**: Lines 24-27, 132-148

```typescript:24:27:lib/core/Axios.js
this.interceptors = {
  request: new InterceptorManager(),
  response: new InterceptorManager()
};
```

### Request Interceptors

**File**: `lib/core/Axios.js`
**Location**: Lines 132-143

Request interceptors are executed before the request is sent:

```typescript:132:143:lib/core/Axios.js
// filter out skipped interceptors
const requestInterceptorChain = [];
let synchronousRequestInterceptors = true;
this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
  if (typeof interceptor.runWhen === 'function' && interceptor.runWhen(config) === false) {
    return;
  }

  synchronousRequestInterceptors = synchronousRequestInterceptors && interceptor.synchronous;

  requestInterceptorChain.unshift(interceptor.fulfilled, interceptor.rejected);
});
```

### Response Interceptors

**File**: `lib/core/Axios.js`
**Location**: Lines 145-148

Response interceptors are executed after the response is received:

```typescript:145:148:lib/core/Axios.js
const responseInterceptorChain = [];
this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
  responseInterceptorChain.push(interceptor.fulfilled, interceptor.rejected);
});
```

### Logging Implementation for Roo Code

```typescript
import axios, { AxiosRequestConfig, AxiosResponse, AxiosError } from "axios"

interface LogConfig {
	logRequests?: boolean
	logResponses?: boolean
	logErrors?: boolean
	logLevel?: "debug" | "info" | "warn" | "error"
}

function setupLoggingInterceptor(
	instance: AxiosInstance,
	logger: (level: string, message: string, data?: any) => void,
	config: LogConfig = {},
) {
	const { logRequests = true, logResponses = true, logErrors = true, logLevel = "debug" } = config

	// Request interceptor
	instance.interceptors.request.use(
		(config: InternalAxiosRequestConfig) => {
			if (logRequests) {
				const logData = {
					method: config.method?.toUpperCase(),
					url: config.url,
					baseURL: config.baseURL,
					headers: config.headers?.toJSON ? config.headers.toJSON() : config.headers,
					timeout: config.timeout,
					timestamp: new Date().toISOString(),
				}
				logger(logLevel, `[Axios Request] ${logData.method} ${logData.url}`, logData)
			}
			return config
		},
		(error: AxiosError) => {
			if (logErrors) {
				logger("error", "[Axios Request Error]", {
					message: error.message,
					code: error.code,
					config: error.config,
					timestamp: new Date().toISOString(),
				})
			}
			return Promise.reject(error)
		},
	)

	// Response interceptor
	instance.interceptors.response.use(
		(response: AxiosResponse) => {
			if (logResponses) {
				const logData = {
					status: response.status,
					statusText: response.statusText,
					url: response.config.url,
					method: response.config.method?.toUpperCase(),
					headers: response.headers?.toJSON ? response.headers.toJSON() : response.headers,
					dataSize: JSON.stringify(response.data).length,
					timestamp: new Date().toISOString(),
				}
				logger(logLevel, `[Axios Response] ${logData.status} ${logData.method} ${logData.url}`, logData)
			}
			return response
		},
		(error: AxiosError) => {
			if (logErrors) {
				const logData = {
					message: error.message,
					code: error.code,
					status: error.response?.status,
					statusText: error.response?.statusText,
					url: error.config?.url,
					method: error.config?.method?.toUpperCase(),
					responseData: error.response?.data,
					timestamp: new Date().toISOString(),
				}
				logger(
					"error",
					`[Axios Error] ${logData.code || logData.status} ${logData.method} ${logData.url}`,
					logData,
				)
			}
			return Promise.reject(error)
		},
	)
}

// Usage with VSCode output channel or console
const ollamaAxiosInstance = axios.create({
	baseURL: "http://localhost:11434",
	timeout: 30000,
})

setupLoggingInterceptor(
	ollamaAxiosInstance,
	(level, message, data) => {
		// Use VSCode's output channel or console
		console[level](message, data)
		// Or use VSCode output channel:
		// outputChannel.appendLine(`[${level.toUpperCase()}] ${message}`);
		// if (data) outputChannel.appendLine(JSON.stringify(data, null, 2));
	},
	{
		logRequests: true,
		logResponses: true,
		logErrors: true,
		logLevel: "debug",
	},
)
```

### Connection Event Logging

For connection-specific events, you can log:

1. **Connection Start**: Logged in request interceptor
2. **Connection Success**: Logged in response interceptor
3. **Connection Failure**: Logged in error interceptor
4. **Timeout Events**: Detected via error code `ETIMEDOUT` or `ECONNABORTED`
5. **Retry Attempts**: Logged within retry interceptor

```typescript
// Enhanced logging with connection details
instance.interceptors.request.use((config) => {
	logger("debug", "[Ollama Connection] Starting request", {
		url: `${config.baseURL}${config.url}`,
		method: config.method,
		timeout: config.timeout,
		connectionStart: new Date().toISOString(),
	})
	return config
})

instance.interceptors.response.use(
	(response) => {
		const duration = Date.now() - new Date(response.config.metadata?.startTime || 0).getTime()
		logger("debug", "[Ollama Connection] Request successful", {
			url: response.config.url,
			status: response.status,
			duration: `${duration}ms`,
		})
		return response
	},
	(error: AxiosError) => {
		const duration = error.config?.metadata?.startTime
			? Date.now() - new Date(error.config.metadata.startTime).getTime()
			: undefined

		logger("error", "[Ollama Connection] Request failed", {
			url: error.config?.url,
			code: error.code,
			status: error.response?.status,
			message: error.message,
			duration: duration ? `${duration}ms` : undefined,
			isTimeout: error.code === "ETIMEDOUT" || error.code === "ECONNABORTED",
			isConnectionRefused: error.code === "ECONNREFUSED",
		})
		return Promise.reject(error)
	},
)
```

---

## Error Handling

### AxiosError Class

**File**: `lib/core/AxiosError.js`
**Location**: Lines 5-73

```typescript:5:36:lib/core/AxiosError.js
class AxiosError extends Error {
    static from(error, code, config, request, response, customProps) {
        const axiosError = new AxiosError(error.message, code || error.code, config, request, response);
        axiosError.cause = error;
        axiosError.name = error.name;
        customProps && Object.assign(axiosError, customProps);
        return axiosError;
    }

    constructor(message, code, config, request, response) {
        super(message);
        this.name = 'AxiosError';
        this.isAxiosError = true;
        code && (this.code = code);
        config && (this.config = config);
        request && (this.request = request);
        if (response) {
            this.response = response;
            this.status = response.status;
        }
    }
```

### Error Codes

**File**: `lib/core/AxiosError.js`
**Location**: Lines 59-71

```typescript:59:71:lib/core/AxiosError.js
AxiosError.ERR_BAD_OPTION_VALUE = 'ERR_BAD_OPTION_VALUE';
AxiosError.ERR_BAD_OPTION = 'ERR_BAD_OPTION';
AxiosError.ECONNABORTED = 'ECONNABORTED';
AxiosError.ETIMEDOUT = 'ETIMEDOUT';
AxiosError.ERR_NETWORK = 'ERR_NETWORK';
AxiosError.ERR_FR_TOO_MANY_REDIRECTS = 'ERR_FR_TOO_MANY_REDIRECTS';
AxiosError.ERR_BAD_RESPONSE = 'ERR_BAD_RESPONSE';
AxiosError.ERR_BAD_REQUEST = 'ERR_BAD_REQUEST';
AxiosError.ERR_CANCELED = 'ERR_CANCELED';
AxiosError.ERR_NOT_SUPPORT = 'ERR_NOT_SUPPORT';
AxiosError.ERR_INVALID_URL = 'ERR_INVALID_URL';
```

### Error Detection for Roo Code

```typescript
import { AxiosError } from "axios"

function handleOllamaError(error: unknown) {
	if (AxiosError.isAxiosError(error)) {
		const axiosError = error as AxiosError

		// Connection errors
		if (axiosError.code === "ECONNREFUSED") {
			console.error("Ollama service is not running or not accessible")
			return
		}

		// Timeout errors
		if (axiosError.code === "ETIMEDOUT" || axiosError.code === "ECONNABORTED") {
			console.error("Ollama request timed out")
			return
		}

		// Network errors
		if (axiosError.code === "ERR_NETWORK") {
			console.error("Network error connecting to Ollama")
			return
		}

		// HTTP errors
		if (axiosError.response) {
			console.error(`Ollama API error: ${axiosError.response.status} ${axiosError.response.statusText}`)
			return
		}
	}

	// Unknown error
	console.error("Unknown error:", error)
}
```

---

## Implementation Recommendations for Roo Code

### Recommended Configuration

Based on the Axios source code analysis, here's a recommended implementation for Roo Code's Ollama integration:

```typescript
// File: src/api/providers/fetchers/ollama-axios-config.ts

import axios, { AxiosInstance, AxiosError, InternalAxiosRequestConfig, AxiosResponse } from "axios"
import * as vscode from "vscode"

interface OllamaAxiosConfig {
	baseUrl?: string
	apiKey?: string
	timeout?: number
	retries?: number
	retryDelay?: number
	enableLogging?: boolean
}

export function createOllamaAxiosInstance(config: OllamaAxiosConfig = {}): AxiosInstance {
	const {
		baseUrl = "http://localhost:11434",
		apiKey,
		timeout = 30000, // 30 seconds default
		retries = 3,
		retryDelay = 1000,
		enableLogging = false,
	} = config

	// Create axios instance
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
			clarifyTimeoutError: true, // Use ETIMEDOUT for better error handling
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

			// Don't retry if already retried too many times
			axiosConfig.__retryCount = axiosConfig.__retryCount || 0
			if (axiosConfig.__retryCount >= config.retries) {
				return Promise.reject(error)
			}

			// Only retry on specific errors
			const shouldRetry =
				error.code === "ECONNREFUSED" ||
				error.code === "ETIMEDOUT" ||
				error.code === "ECONNABORTED" ||
				error.code === "ERR_NETWORK" ||
				(error.response && error.response.status >= 500)

			if (!shouldRetry) {
				return Promise.reject(error)
			}

			// Increment retry count
			axiosConfig.__retryCount += 1

			// Exponential backoff
			const delay = config.retryDelay * Math.pow(2, axiosConfig.__retryCount - 1)

			// Wait before retrying
			await new Promise((resolve) => setTimeout(resolve, delay))

			// Retry the request
			return instance(axiosConfig)
		},
	)
}

function setupLoggingInterceptor(instance: AxiosInstance) {
	// Request logging
	instance.interceptors.request.use((config: InternalAxiosRequestConfig) => {
		;(config as any).metadata = { startTime: new Date() }
		console.debug("[Ollama] Request:", {
			method: config.method?.toUpperCase(),
			url: `${config.baseURL}${config.url}`,
			timeout: config.timeout,
		})
		return config
	})

	// Response logging
	instance.interceptors.response.use(
		(response: AxiosResponse) => {
			const duration = Date.now() - new Date((response.config as any).metadata?.startTime || 0).getTime()
			console.debug("[Ollama] Response:", {
				status: response.status,
				url: response.config.url,
				duration: `${duration}ms`,
			})
			return response
		},
		(error: AxiosError) => {
			const duration = error.config?.metadata?.startTime
				? Date.now() - new Date(error.config.metadata.startTime).getTime()
				: undefined
			console.error("[Ollama] Error:", {
				code: error.code,
				message: error.message,
				status: error.response?.status,
				url: error.config?.url,
				duration: duration ? `${duration}ms` : undefined,
			})
			return Promise.reject(error)
		},
	)
}
```

### Updated getOllamaModels Function

```typescript
// File: src/api/providers/fetchers/ollama.ts

import { createOllamaAxiosInstance } from "./ollama-axios-config"
import { ModelInfo, ollamaDefaultModelInfo } from "@roo-code/types"
import { z } from "zod"

// ... schema definitions ...

export async function getOllamaModels(
	baseUrl = "http://localhost:11434",
	apiKey?: string,
): Promise<Record<string, ModelInfo>> {
	const models: Record<string, ModelInfo> = {}

	baseUrl = baseUrl === "" ? "http://localhost:11434" : baseUrl

	try {
		if (!URL.canParse(baseUrl)) {
			return models
		}

		// Create configured axios instance
		const axiosInstance = createOllamaAxiosInstance({
			baseUrl,
			apiKey,
			timeout: 30000, // 30 seconds for model discovery
			retries: 2, // Retry twice for model discovery
			retryDelay: 1000,
			enableLogging: true, // Enable logging for debugging
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

---

## Code References

### Key Files and Locations

1. **Core Axios Class**

    - `lib/core/Axios.js:21-28` - Constructor
    - `lib/core/Axios.js:38-203` - Request handling
    - `lib/core/Axios.js:132-148` - Interceptor execution

2. **Interceptor Manager**

    - `lib/core/InterceptorManager.js:5-72` - Interceptor management

3. **Default Configuration**

    - `lib/defaults/index.js:36-161` - Default values
    - `lib/defaults/index.js:132` - Default timeout (0)

4. **HTTP Adapter (Node.js)**

    - `lib/adapters/http.js:829-867` - Timeout implementation
    - `lib/adapters/http.js:824-827` - Keep-alive configuration

5. **XHR Adapter (Browser)**

    - `lib/adapters/xhr.js:37-38` - Timeout setting
    - `lib/adapters/xhr.js:119-134` - Timeout handling

6. **Error Handling**

    - `lib/core/AxiosError.js:5-73` - Error class definition
    - `lib/core/AxiosError.js:59-71` - Error codes

7. **Configuration Merging**

    - `lib/core/mergeConfig.js:74-75` - Timeout merging logic

8. **Signal Composition**

    - `lib/helpers/composeSignals.js:5-48` - Timeout with AbortController

9. **TypeScript Definitions**
    - `index.d.ts:320-378` - Request config interface
    - `index.d.ts:331-332` - Timeout properties

---

## Related Documentation

- **Roo Code Ollama Integration**: See [OLLAMA_INTEGRATION_DOCUMENTATION.md](./OLLAMA_INTEGRATION_DOCUMENTATION.md) for current implementation details
- **Axios GitHub Repository**: [https://github.com/axios/axios](https://github.com/axios/axios)
- **Axios Official Documentation**: [https://axios-http.com/docs/intro](https://axios-http.com/docs/intro)
- **Axios Migration Guide**: See `MIGRATION_GUIDE.md` in the axios repository for version migration details
- **Axios Ecosystem**: See `ECOSYSTEM.md` in the axios repository for related libraries

---

## Summary

### Key Findings

1. **Timeout Configuration**:

    - Default is `0` (no timeout)
    - Can be set per request or instance
    - Supports custom timeout error messages
    - Uses `ETIMEDOUT` or `ECONNABORTED` error codes

2. **Retry Mechanisms**:

    - Not built into Axios core
    - Must be implemented via interceptors
    - Recommended: Response error interceptor with exponential backoff

3. **Logging**:

    - Implemented via request/response interceptors
    - Can log connection events, timing, errors
    - Supports conditional logging via `runWhen` option

4. **Connection Setup**:
    - Use `axios.create()` for instance-based configuration
    - Supports `httpAgent`/`httpsAgent` for connection pooling
    - Supports proxy configuration
    - Supports custom headers including authentication

### Recommendations for Roo Code

1. **Create a dedicated Axios instance** for Ollama with:

    - Configurable timeout (default 30 seconds)
    - Retry logic (2-3 retries with exponential backoff)
    - Comprehensive logging
    - Proper error handling

2. **Make timeout configurable** via VSCode settings, similar to `apiRequestTimeout` but specific to Ollama

3. **Implement retry logic** for transient errors (network errors, timeouts, 5xx responses)

4. **Add logging** for debugging connection issues and API call tracking

5. **Update `getOllamaModels`** to use the configured instance instead of the default axios

This will provide better control, observability, and reliability for Roo Code's Ollama integration.
