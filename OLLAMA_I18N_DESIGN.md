# Ollama Settings Page Internationalization (i18n) Design Document

## Executive Summary

This document outlines the design and implementation plan for comprehensive internationalization (i18n) support for the Ollama provider settings page. The goal is to ensure all UI text, labels, descriptions, and messages are properly translated across all 18 locales (17 non-English + English).

## Current State Analysis

### Supported Languages

The application supports 18 locales (17 non-English + English):

1. English (en) - **Complete** ✅
2. Catalan (ca)
3. German (de)
4. Spanish (es)
5. French (fr)
6. Hindi (hi)
7. Indonesian (id)
8. Italian (it)
9. Japanese (ja)
10. Korean (ko)
11. Dutch (nl)
12. Polish (pl)
13. Portuguese (Brazil) (pt-BR)
14. Russian (ru)
15. Turkish (tr)
16. Vietnamese (vi)
17. Chinese (Simplified) (zh-CN)
18. Chinese (Traditional) (zh-TW)

### Translation File Structure

All translations are stored in JSON files at:

```
webview-ui/src/i18n/locales/{locale}/settings.json
```

The Ollama provider settings are nested under:

```json
{
	"providers": {
		"ollama": {
			// Ollama-specific translations
		}
	}
}
```

### Current Translation Status

#### English (en) - Complete ✅

All 29 new translation keys are present and properly defined.

#### All Other Languages - Incomplete ❌

Only 8 basic keys exist:

- `baseUrl`
- `modelId`
- `apiKey`
- `apiKeyHelp`
- `numCtx`
- `numCtxHelp`
- `description`
- `warning`

**Missing:** 29 new keys added for the enhanced Ollama UI features.

## Missing Translation Keys

### 1. Button Actions (4 keys)

| Key             | English Value    | Usage                                |
| --------------- | ---------------- | ------------------------------------ |
| `test`          | "Test"           | Test connection button label         |
| `testing`       | "Testing..."     | Test connection button loading state |
| `refreshModels` | "Refresh Models" | Refresh models button label          |
| `refreshing`    | "Refreshing..."  | Refresh models button loading state  |

### 2. Section Headers (3 keys)

| Key                  | English Value         | Usage                                        |
| -------------------- | --------------------- | -------------------------------------------- |
| `connectionSettings` | "Connection Settings" | Advanced settings collapsible section header |
| `toolsSupport`       | "Tools Support"       | Table section header for models with tools   |
| `noToolsSupport`     | "No Tools Support"    | Section header for models without tools      |

### 3. Model Information (2 keys)

| Key                  | English Value                                                                                                          | Usage                                              |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| `models`             | "models"                                                                                                               | Plural form used in counts (e.g., "9 models")      |
| `noToolsSupportHelp` | "These models do not support native tool calling and cannot be used with Roo Code. They are shown for reference only." | Help text explaining why certain models are listed |

### 4. Table Headers (8 keys)

| Key                  | English Value  | Usage                                     |
| -------------------- | -------------- | ----------------------------------------- |
| `table.modelName`    | "Model Name"   | Table column header                       |
| `table.context`      | "Context"      | Table column header (context window size) |
| `table.size`         | "Size"         | Table column header (model size)          |
| `table.quantization` | "Quantization" | Table column header (quantization level)  |
| `table.family`       | "Family"       | Table column header (model family)        |
| `table.images`       | "Images"       | Table column header (image support)       |
| `table.yes`          | "Yes"          | Boolean value in Images column            |
| `table.no`           | "No"           | Boolean value in Images column            |

### 5. Connection Settings (10 keys)

| Key                         | English Value                                                                                                                                              | Usage                                 |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------- |
| `streaming`                 | "Streaming"                                                                                                                                                | Streaming checkbox label              |
| `streamingHelp`             | "Streaming is always enabled for Ollama API requests. Responses are streamed in real-time as they are generated."                                          | Help text for streaming               |
| `requestTimeout`            | "Request Timeout (ms)"                                                                                                                                     | Request timeout field label           |
| `requestTimeoutHelp`        | "Timeout in milliseconds for LLM API requests (chat completions, thinking work). Default: 3600000 (60 minutes). Range: 1000-7200000 (120 minutes)."        | Help text for request timeout         |
| `modelDiscoveryTimeout`     | "Model Discovery Timeout (ms)"                                                                                                                             | Model discovery timeout field label   |
| `modelDiscoveryTimeoutHelp` | "Timeout in milliseconds for model discovery requests (listing and fetching model details). Default: 10000 (10 seconds). Range: 1000-600000 (10 minutes)." | Help text for model discovery timeout |
| `maxRetries`                | "Max Retries"                                                                                                                                              | Max retries field label               |
| `maxRetriesHelp`            | "Maximum number of retry attempts for failed requests. Default: 0 (no retries). Range: 0-10."                                                              | Help text for max retries             |
| `retryDelay`                | "Retry Delay (ms)"                                                                                                                                         | Retry delay field label               |
| `retryDelayHelp`            | "Initial delay between retry attempts in milliseconds. Uses exponential backoff. Default: 1000 (1 second). Range: 100-10000."                              | Help text for retry delay             |
| `enableLogging`             | "Enable Request Logging"                                                                                                                                   | Enable logging checkbox label         |
| `enableLoggingHelp`         | "Enable detailed logging of Ollama API requests, responses, and errors. Logs include timing information and connection details."                           | Help text for enable logging          |

**Note:** Some keys have both a label and a help text (e.g., `requestTimeout` and `requestTimeoutHelp`).

## Design Principles

### 1. Consistency

- Maintain consistent terminology across all languages
- Use the same translation patterns as existing provider settings (e.g., LM Studio, OpenAI)
- Follow existing i18n key naming conventions

### 2. Context Preservation

- Technical terms (e.g., "ms", "GB", "MB") should remain in English or use standard abbreviations
- Model-related terms should be consistent with how they appear in other parts of the application
- Preserve technical accuracy while making text natural in each language

### 3. Completeness

- Every UI string must have a translation key
- No hardcoded strings in the component
- Fallback to English if a key is missing (graceful degradation)

### 4. Maintainability

- Clear key naming that indicates purpose
- Grouped logically (buttons, headers, help text, etc.)
- Documented in this design doc for future reference

## Implementation Strategy

### Phase 1: Add Missing Keys to All Language Files

#### Step 1.1: Create Translation Template

Create a comprehensive template with all 29 missing keys, using English as the base. This template will serve as:

- A reference for translators
- A placeholder to ensure the UI works (shows English until translated)
- A checklist for completion

#### Step 1.2: Add Keys to All Language Files

For each of the 17 non-English locales:

1. Open `webview-ui/src/i18n/locales/{locale}/settings.json`
2. Locate the `providers.ollama` section
3. Add all 29 missing keys with English placeholder values initially
4. Ensure proper JSON formatting and indentation

**File Locations:**

```
webview-ui/src/i18n/locales/ca/settings.json
webview-ui/src/i18n/locales/de/settings.json
webview-ui/src/i18n/locales/es/settings.json
webview-ui/src/i18n/locales/fr/settings.json
webview-ui/src/i18n/locales/hi/settings.json
webview-ui/src/i18n/locales/id/settings.json
webview-ui/src/i18n/locales/it/settings.json
webview-ui/src/i18n/locales/ja/settings.json
webview-ui/src/i18n/locales/ko/settings.json
webview-ui/src/i18n/locales/nl/settings.json
webview-ui/src/i18n/locales/pl/settings.json
webview-ui/src/i18n/locales/pt-BR/settings.json
webview-ui/src/i18n/locales/ru/settings.json
webview-ui/src/i18n/locales/tr/settings.json
webview-ui/src/i18n/locales/vi/settings.json
webview-ui/src/i18n/locales/zh-CN/settings.json
webview-ui/src/i18n/locales/zh-TW/settings.json
```

### Phase 2: Translation Process

#### Step 2.1: Professional Translation

For production-quality translations:

1. Use professional translation services or native speakers
2. Provide context for technical terms
3. Review translations for technical accuracy
4. Ensure consistency with existing translations in the codebase

#### Step 2.2: Community Translation (Alternative)

If using community contributions:

1. Create a GitHub issue or PR template
2. Provide clear instructions for translators
3. Review translations before merging
4. Credit contributors

### Phase 3: Quality Assurance

#### Step 3.1: Automated Testing

- Verify all keys exist in all language files
- Check JSON syntax validity
- Ensure no missing keys cause UI breakage

#### Step 3.2: Manual Testing

- Test UI in each language
- Verify text fits in UI components (no overflow)
- Check that technical terms are appropriate
- Verify help text is readable and accurate

#### Step 3.3: Consistency Checks

- Compare terminology with other provider settings
- Ensure button labels match application-wide patterns
- Verify help text formatting is consistent

## Translation Template

### Complete Key Structure

```json
{
	"providers": {
		"ollama": {
			// Existing keys (8 keys) - already present in all languages
			"baseUrl": "...",
			"modelId": "...",
			"apiKey": "...",
			"apiKeyHelp": "...",
			"numCtx": "...",
			"numCtxHelp": "...",
			"description": "...",
			"warning": "...",

			// NEW KEYS TO ADD (29 keys)

			// Button Actions
			"test": "Test",
			"testing": "Testing...",
			"refreshModels": "Refresh Models",
			"refreshing": "Refreshing...",

			// Section Headers
			"connectionSettings": "Connection Settings",
			"toolsSupport": "Tools Support",
			"noToolsSupport": "No Tools Support",

			// Model Information
			"models": "models",
			"noToolsSupportHelp": "These models do not support native tool calling and cannot be used with Roo Code. They are shown for reference only.",

			// Table Headers
			"table": {
				"modelName": "Model Name",
				"context": "Context",
				"size": "Size",
				"quantization": "Quantization",
				"family": "Family",
				"images": "Images",
				"yes": "Yes",
				"no": "No",
				"sizeFormatting": {
					"gb": "GB",
					"mb": "MB"
				}
			},

			// Connection Settings
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

## Proposed Translations

This section contains the proposed translations for all 17 non-English locales. These translations are provided for review before implementation.

### Translation Notes

- Technical terms like "ms" (milliseconds), "GB", "MB" are kept in English as they are internationally recognized
- Technical terms like "API", "LLM", "Ollama" are kept in English
- Numbers and ranges are preserved as-is
- HTML-like tags in descriptions (e.g., `<quickstartLink>`) should not be translated
- **French size formatting**: "Go" and "Mo" (Gigaoctets, Mégaoctets) are correct and reflect French technical standards
- **German testing**: Uses "Verbindung wird geprüft..." (Connection is being checked) for a more professional IDE feel
- **Spanish requestTimeout**: Includes the article "la" for natural Spanish phrasing
- **Russian refreshing**: Uses verb form "Обновляем..." (Updating) which is common in Russian UIs for active processes
- **Russian pluralization note**: The "models" key is used with counts (e.g., "9 models"). Russian requires three plural forms (1, 2-4, 5+). However, since the count is displayed separately and the word "models" is used as a label, the current simple string approach is acceptable. If proper pluralization is desired, i18next pluralization would need to be implemented with `_one`, `_few`, `_many` suffixes.

### Catalan (ca)

```json
"test": "Provar",
"testing": "Provant...",
"refreshModels": "Actualitzar models",
"refreshing": "Actualitzant...",
"connectionSettings": "Configuració de connexió",
"toolsSupport": "Suport d'eines",
"noToolsSupport": "Sense suport d'eines",
"models": "models",
"noToolsSupportHelp": "Aquests models no admeten crides a eines natives i no es poden utilitzar amb Roo Code. Es mostren només com a referència.",
"table": {
  "modelName": "Nom del model",
  "context": "Context",
  "size": "Mida",
  "quantization": "Quantització",
  "family": "Família",
  "images": "Imatges",
  "yes": "Sí",
  "no": "No",
  "sizeFormatting": { "gb": "GB", "mb": "MB" }
},
"streaming": "Transmissió",
"streamingHelp": "La transmissió sempre està habilitada per a les sol·licituds de l'API d'Ollama. Les respostes es transmeten en temps real mentre es generen.",
"requestTimeout": "Temps d'espera de sol·licitud (ms)",
"requestTimeoutHelp": "Temps d'espera en mil·lisegons per a sol·licituds de l'API LLM (completat de xat, treball de pensament). Per defecte: 3600000 (60 minuts). Rang: 1000-7200000 (120 minuts).",
"modelDiscoveryTimeout": "Temps d'espera de descobriment de models (ms)",
"modelDiscoveryTimeoutHelp": "Temps d'espera en mil·lisegons per a sol·licituds de descobriment de models (llistat i obtenció de detalls del model). Per defecte: 10000 (10 segons). Rang: 1000-600000 (10 minuts).",
"maxRetries": "Màxim de reintents",
"maxRetriesHelp": "Nombre màxim d'intents de reintent per a sol·licituds fallides. Per defecte: 0 (sense reintents). Rang: 0-10.",
"retryDelay": "Retard de reintent (ms)",
"retryDelayHelp": "Retard inicial entre intents de reintent en mil·lisegons. Utilitza retrocés exponencial. Per defecte: 1000 (1 segon). Rang: 100-10000.",
"enableLogging": "Habilitar registre de sol·licituds",
"enableLoggingHelp": "Habilitar registre detallat de sol·licituds, respostes i errors de l'API d'Ollama. Els registres inclouen informació de temps i detalls de connexió."
```

### German (de)

```json
"test": "Testen",
"testing": "Verbindung wird geprüft...",
"refreshModels": "Modelle aktualisieren",
"refreshing": "Aktualisiere...",
"connectionSettings": "Verbindungseinstellungen",
"toolsSupport": "Tools-Unterstützung",
"noToolsSupport": "Keine Tools-Unterstützung",
"models": "Modelle",
"noToolsSupportHelp": "Diese Modelle unterstützen keine nativen Tool-Aufrufe und können nicht mit Roo Code verwendet werden. Sie werden nur zur Referenz angezeigt.",
"table": {
  "modelName": "Modellname",
  "context": "Kontext",
  "size": "Größe",
  "quantization": "Quantisierung",
  "family": "Familie",
  "images": "Bilder",
  "yes": "Ja",
  "no": "Nein",
  "sizeFormatting": { "gb": "GB", "mb": "MB" }
},
"streaming": "Streaming",
"streamingHelp": "Streaming ist für Ollama API-Anfragen immer aktiviert. Antworten werden in Echtzeit gestreamt, während sie generiert werden.",
"requestTimeout": "Anfrage-Timeout (ms)",
"requestTimeoutHelp": "Timeout in Millisekunden für LLM API-Anfragen (Chat-Vervollständigungen, Denkarbeit). Standard: 3600000 (60 Minuten). Bereich: 1000-7200000 (120 Minuten).",
"modelDiscoveryTimeout": "Modell-Erkennungs-Timeout (ms)",
"modelDiscoveryTimeoutHelp": "Timeout in Millisekunden für Modell-Erkennungsanfragen (Auflisten und Abrufen von Modelldetails). Standard: 10000 (10 Sekunden). Bereich: 1000-600000 (10 Minuten).",
"maxRetries": "Max. Wiederholungen",
"maxRetriesHelp": "Maximale Anzahl von Wiederholungsversuchen bei fehlgeschlagenen Anfragen. Standard: 0 (keine Wiederholungen). Bereich: 0-10.",
"retryDelay": "Wiederholungsverzögerung (ms)",
"retryDelayHelp": "Anfängliche Verzögerung zwischen Wiederholungsversuchen in Millisekunden. Verwendet exponentielles Backoff. Standard: 1000 (1 Sekunde). Bereich: 100-10000.",
"enableLogging": "Anfrage-Protokollierung aktivieren",
"enableLoggingHelp": "Detaillierte Protokollierung von Ollama API-Anfragen, Antworten und Fehlern aktivieren. Protokolle enthalten Zeitinformationen und Verbindungsdetails."
```

### Spanish (es)

```json
"test": "Probar",
"testing": "Probando...",
"refreshModels": "Actualizar modelos",
"refreshing": "Actualizando...",
"connectionSettings": "Configuración de conexión",
"toolsSupport": "Soporte de herramientas",
"noToolsSupport": "Sin soporte de herramientas",
"models": "modelos",
"noToolsSupportHelp": "Estos modelos no admiten llamadas a herramientas nativas y no se pueden usar con Roo Code. Se muestran solo como referencia.",
"table": {
  "modelName": "Nombre del modelo",
  "context": "Contexto",
  "size": "Tamaño",
  "quantization": "Cuantización",
  "family": "Familia",
  "images": "Imágenes",
  "yes": "Sí",
  "no": "No",
  "sizeFormatting": { "gb": "GB", "mb": "MB" }
},
"streaming": "Transmisión",
"streamingHelp": "La transmisión siempre está habilitada para las solicitudes de API de Ollama. Las respuestas se transmiten en tiempo real a medida que se generan.",
"requestTimeout": "Tiempo de espera de la solicitud (ms)",
"requestTimeoutHelp": "Tiempo de espera en milisegundos para solicitudes de API LLM (completado de chat, trabajo de pensamiento). Predeterminado: 3600000 (60 minutos). Rango: 1000-7200000 (120 minutos).",
"modelDiscoveryTimeout": "Tiempo de espera de descubrimiento de modelos (ms)",
"modelDiscoveryTimeoutHelp": "Tiempo de espera en milisegundos para solicitudes de descubrimiento de modelos (listado y obtención de detalles del modelo). Predeterminado: 10000 (10 segundos). Rango: 1000-600000 (10 minutos).",
"maxRetries": "Máximo de reintentos",
"maxRetriesHelp": "Número máximo de intentos de reintento para solicitudes fallidas. Predeterminado: 0 (sin reintentos). Rango: 0-10.",
"retryDelay": "Retraso de reintento (ms)",
"retryDelayHelp": "Retraso inicial entre intentos de reintento en milisegundos. Utiliza retroceso exponencial. Predeterminado: 1000 (1 segundo). Rango: 100-10000.",
"enableLogging": "Habilitar registro de solicitudes",
"enableLoggingHelp": "Habilitar registro detallado de solicitudes, respuestas y errores de la API de Ollama. Los registros incluyen información de tiempo y detalles de conexión."
```

### French (fr)

```json
"test": "Tester",
"testing": "Test en cours...",
"refreshModels": "Actualiser les modèles",
"refreshing": "Actualisation...",
"connectionSettings": "Paramètres de connexion",
"toolsSupport": "Support des outils",
"noToolsSupport": "Pas de support des outils",
"models": "modèles",
"noToolsSupportHelp": "Ces modèles ne prennent pas en charge les appels d'outils natifs et ne peuvent pas être utilisés avec Roo Code. Ils sont affichés uniquement à titre de référence.",
"table": {
  "modelName": "Nom du modèle",
  "context": "Contexte",
  "size": "Taille",
  "quantization": "Quantification",
  "family": "Famille",
  "images": "Images",
  "yes": "Oui",
  "no": "Non",
  "sizeFormatting": { "gb": "Go", "mb": "Mo" }
},
"streaming": "Diffusion en continu",
"streamingHelp": "La diffusion en continu est toujours activée pour les requêtes API Ollama. Les réponses sont diffusées en temps réel au fur et à mesure de leur génération.",
"requestTimeout": "Délai d'expiration de la requête (ms)",
"requestTimeoutHelp": "Délai d'expiration en millisecondes pour les requêtes API LLM (complétions de chat, travail de réflexion). Par défaut : 3600000 (60 minutes). Plage : 1000-7200000 (120 minutes).",
"modelDiscoveryTimeout": "Délai d'expiration de la découverte de modèles (ms)",
"modelDiscoveryTimeoutHelp": "Délai d'expiration en millisecondes pour les requêtes de découverte de modèles (liste et récupération des détails des modèles). Par défaut : 10000 (10 secondes). Plage : 1000-600000 (10 minutes).",
"maxRetries": "Nombre maximum de tentatives",
"maxRetriesHelp": "Nombre maximum de tentatives de nouvelle tentative pour les requêtes échouées. Par défaut : 0 (aucune nouvelle tentative). Plage : 0-10.",
"retryDelay": "Délai entre les tentatives (ms)",
"retryDelayHelp": "Délai initial entre les tentatives de nouvelle tentative en millisecondes. Utilise un backoff exponentiel. Par défaut : 1000 (1 seconde). Plage : 100-10000.",
"enableLogging": "Activer l'enregistrement des requêtes",
"enableLoggingHelp": "Activer l'enregistrement détaillé des requêtes, réponses et erreurs de l'API Ollama. Les journaux incluent les informations de timing et les détails de connexion."
```

### Hindi (hi)

```json
"test": "परीक्षण",
"testing": "परीक्षण कर रहे हैं...",
"refreshModels": "मॉडल ताज़ा करें",
"refreshing": "ताज़ा कर रहे हैं...",
"connectionSettings": "कनेक्शन सेटिंग्स",
"toolsSupport": "टूल्स सपोर्ट",
"noToolsSupport": "कोई टूल्स सपोर्ट नहीं",
"models": "मॉडल",
"noToolsSupportHelp": "ये मॉडल नेटिव टूल कॉलिंग का समर्थन नहीं करते हैं और Roo Code के साथ उपयोग नहीं किए जा सकते हैं। वे केवल संदर्भ के लिए दिखाए गए हैं।",
"table": {
  "modelName": "मॉडल नाम",
  "context": "संदर्भ",
  "size": "आकार",
  "quantization": "क्वांटाइज़ेशन",
  "family": "परिवार",
  "images": "छवियां",
  "yes": "हाँ",
  "no": "नहीं",
  "sizeFormatting": { "gb": "GB", "mb": "MB" }
},
"streaming": "स्ट्रीमिंग",
"streamingHelp": "Ollama API अनुरोधों के लिए स्ट्रीमिंग हमेशा सक्षम होती है। प्रतिक्रियाएं वास्तविक समय में स्ट्रीम की जाती हैं जैसे ही वे उत्पन्न होती हैं।",
"requestTimeout": "अनुरोध समय सीमा (ms)",
"requestTimeoutHelp": "LLM API अनुरोधों (चैट पूर्णता, सोच कार्य) के लिए मिलीसेकंड में समय सीमा। डिफ़ॉल्ट: 3600000 (60 मिनट)। सीमा: 1000-7200000 (120 मिनट)।",
"modelDiscoveryTimeout": "मॉडल खोज समय सीमा (ms)",
"modelDiscoveryTimeoutHelp": "मॉडल खोज अनुरोधों (सूची और मॉडल विवरण प्राप्त करना) के लिए मिलीसेकंड में समय सीमा। डिफ़ॉल्ट: 10000 (10 सेकंड)। सीमा: 1000-600000 (10 मिनट)।",
"maxRetries": "अधिकतम पुनः प्रयास",
"maxRetriesHelp": "विफल अनुरोधों के लिए पुनः प्रयास प्रयासों की अधिकतम संख्या। डिफ़ॉल्ट: 0 (कोई पुनः प्रयास नहीं)। सीमा: 0-10।",
"retryDelay": "पुनः प्रयास देरी (ms)",
"retryDelayHelp": "मिलीसेकंड में पुनः प्रयास प्रयासों के बीच प्रारंभिक देरी। घातीय बैकऑफ़ का उपयोग करता है। डिफ़ॉल्ट: 1000 (1 सेकंड)। सीमा: 100-10000।",
"enableLogging": "अनुरोध लॉगिंग सक्षम करें",
"enableLoggingHelp": "Ollama API अनुरोधों, प्रतिक्रियाओं और त्रुटियों का विस्तृत लॉगिंग सक्षम करें। लॉग में समय जानकारी और कनेक्शन विवरण शामिल हैं।"
```

### Indonesian (id)

```json
"test": "Uji",
"testing": "Menguji...",
"refreshModels": "Muat ulang model",
"refreshing": "Memuat ulang...",
"connectionSettings": "Pengaturan koneksi",
"toolsSupport": "Dukungan alat",
"noToolsSupport": "Tidak ada dukungan alat",
"models": "model",
"noToolsSupportHelp": "Model-model ini tidak mendukung pemanggilan alat native dan tidak dapat digunakan dengan Roo Code. Mereka ditampilkan hanya sebagai referensi.",
"table": {
  "modelName": "Nama model",
  "context": "Konteks",
  "size": "Ukuran",
  "quantization": "Kuantisasi",
  "family": "Keluarga",
  "images": "Gambar",
  "yes": "Ya",
  "no": "Tidak",
  "sizeFormatting": { "gb": "GB", "mb": "MB" }
},
"streaming": "Streaming",
"streamingHelp": "Streaming selalu diaktifkan untuk permintaan API Ollama. Respons di-stream secara real-time saat dibuat.",
"requestTimeout": "Batas waktu permintaan (ms)",
"requestTimeoutHelp": "Batas waktu dalam milidetik untuk permintaan API LLM (penyelesaian chat, pekerjaan berpikir). Default: 3600000 (60 menit). Rentang: 1000-7200000 (120 menit).",
"modelDiscoveryTimeout": "Batas waktu penemuan model (ms)",
"modelDiscoveryTimeoutHelp": "Batas waktu dalam milidetik untuk permintaan penemuan model (daftar dan mengambil detail model). Default: 10000 (10 detik). Rentang: 1000-600000 (10 menit).",
"maxRetries": "Maks. percobaan ulang",
"maxRetriesHelp": "Jumlah maksimum upaya percobaan ulang untuk permintaan yang gagal. Default: 0 (tidak ada percobaan ulang). Rentang: 0-10.",
"retryDelay": "Penundaan percobaan ulang (ms)",
"retryDelayHelp": "Penundaan awal antara upaya percobaan ulang dalam milidetik. Menggunakan exponential backoff. Default: 1000 (1 detik). Rentang: 100-10000.",
"enableLogging": "Aktifkan pencatatan permintaan",
"enableLoggingHelp": "Aktifkan pencatatan detail permintaan, respons, dan kesalahan API Ollama. Log mencakup informasi waktu dan detail koneksi."
```

### Italian (it)

```json
"test": "Testa",
"testing": "Test in corso...",
"refreshModels": "Aggiorna modelli",
"refreshing": "Aggiornamento...",
"connectionSettings": "Impostazioni di connessione",
"toolsSupport": "Supporto strumenti",
"noToolsSupport": "Nessun supporto strumenti",
"models": "modelli",
"noToolsSupportHelp": "Questi modelli non supportano le chiamate a strumenti native e non possono essere utilizzati con Roo Code. Sono mostrati solo come riferimento.",
"table": {
  "modelName": "Nome modello",
  "context": "Contesto",
  "size": "Dimensione",
  "quantization": "Quantizzazione",
  "family": "Famiglia",
  "images": "Immagini",
  "yes": "Sì",
  "no": "No",
  "sizeFormatting": { "gb": "GB", "mb": "MB" }
},
"streaming": "Streaming",
"streamingHelp": "Lo streaming è sempre abilitato per le richieste API Ollama. Le risposte vengono trasmesse in tempo reale mentre vengono generate.",
"requestTimeout": "Timeout richiesta (ms)",
"requestTimeoutHelp": "Timeout in millisecondi per le richieste API LLM (completamenti chat, lavoro di pensiero). Predefinito: 3600000 (60 minuti). Intervallo: 1000-7200000 (120 minuti).",
"modelDiscoveryTimeout": "Timeout scoperta modelli (ms)",
"modelDiscoveryTimeoutHelp": "Timeout in millisecondi per le richieste di scoperta modelli (elenco e recupero dettagli modello). Predefinito: 10000 (10 secondi). Intervallo: 1000-600000 (10 minuti).",
"maxRetries": "Tentativi massimi",
"maxRetriesHelp": "Numero massimo di tentativi di ripetizione per richieste fallite. Predefinito: 0 (nessun tentativo). Intervallo: 0-10.",
"retryDelay": "Ritardo tentativo (ms)",
"retryDelayHelp": "Ritardo iniziale tra i tentativi di ripetizione in millisecondi. Utilizza backoff esponenziale. Predefinito: 1000 (1 secondo). Intervallo: 100-10000.",
"enableLogging": "Abilita registrazione richieste",
"enableLoggingHelp": "Abilita la registrazione dettagliata di richieste, risposte ed errori API Ollama. I log includono informazioni temporali e dettagli di connessione."
```

### Japanese (ja)

```json
"test": "テスト",
"testing": "テスト中...",
"refreshModels": "モデルを更新",
"refreshing": "更新中...",
"connectionSettings": "接続設定",
"toolsSupport": "ツールサポート",
"noToolsSupport": "ツールサポートなし",
"models": "モデル",
"noToolsSupportHelp": "これらのモデルはネイティブツール呼び出しをサポートしておらず、Roo Codeでは使用できません。参考としてのみ表示されます。",
"table": {
  "modelName": "モデル名",
  "context": "コンテキスト",
  "size": "サイズ",
  "quantization": "量子化",
  "family": "ファミリー",
  "images": "画像",
  "yes": "はい",
  "no": "いいえ",
  "sizeFormatting": { "gb": "GB", "mb": "MB" }
},
"streaming": "ストリーミング",
"streamingHelp": "Ollama APIリクエストではストリーミングが常に有効です。応答は生成されると同時にリアルタイムでストリーミングされます。",
"requestTimeout": "リクエストタイムアウト (ms)",
"requestTimeoutHelp": "LLM APIリクエスト（チャット完了、思考作業）のタイムアウト（ミリ秒）。デフォルト: 3600000 (60分)。範囲: 1000-7200000 (120分)。",
"modelDiscoveryTimeout": "モデル検出タイムアウト (ms)",
"modelDiscoveryTimeoutHelp": "モデル検出リクエスト（モデル一覧と詳細の取得）のタイムアウト（ミリ秒）。デフォルト: 10000 (10秒)。範囲: 1000-600000 (10分)。",
"maxRetries": "最大再試行回数",
"maxRetriesHelp": "失敗したリクエストの最大再試行回数。デフォルト: 0 (再試行なし)。範囲: 0-10。",
"retryDelay": "再試行遅延 (ms)",
"retryDelayHelp": "再試行の間の初期遅延（ミリ秒）。指数バックオフを使用します。デフォルト: 1000 (1秒)。範囲: 100-10000。",
"enableLogging": "リクエストログを有効化",
"enableLoggingHelp": "Ollama APIリクエスト、応答、エラーの詳細ログを有効化します。ログには時間情報と接続詳細が含まれます。"
```

### Korean (ko)

```json
"test": "테스트",
"testing": "테스트 중...",
"refreshModels": "모델 새로고침",
"refreshing": "새로고침 중...",
"connectionSettings": "연결 설정",
"toolsSupport": "도구 지원",
"noToolsSupport": "도구 지원 없음",
"models": "모델",
"noToolsSupportHelp": "이 모델들은 네이티브 도구 호출을 지원하지 않으며 Roo Code와 함께 사용할 수 없습니다. 참고용으로만 표시됩니다.",
"table": {
  "modelName": "모델 이름",
  "context": "컨텍스트",
  "size": "크기",
  "quantization": "양자화",
  "family": "패밀리",
  "images": "이미지",
  "yes": "예",
  "no": "아니오",
  "sizeFormatting": { "gb": "GB", "mb": "MB" }
},
"streaming": "스트리밍",
"streamingHelp": "Ollama API 요청에 대해 스트리밍이 항상 활성화되어 있습니다. 응답은 생성되는 대로 실시간으로 스트리밍됩니다.",
"requestTimeout": "요청 시간 제한 (ms)",
"requestTimeoutHelp": "LLM API 요청(채팅 완성, 사고 작업)의 시간 제한(밀리초). 기본값: 3600000 (60분). 범위: 1000-7200000 (120분).",
"modelDiscoveryTimeout": "모델 검색 시간 제한 (ms)",
"modelDiscoveryTimeoutHelp": "모델 검색 요청(모델 목록 및 세부 정보 가져오기)의 시간 제한(밀리초). 기본값: 10000 (10초). 범위: 1000-600000 (10분).",
"maxRetries": "최대 재시도",
"maxRetriesHelp": "실패한 요청에 대한 최대 재시도 시도 횟수. 기본값: 0 (재시도 없음). 범위: 0-10.",
"retryDelay": "재시도 지연 (ms)",
"retryDelayHelp": "재시도 시도 간 초기 지연(밀리초). 지수 백오프를 사용합니다. 기본값: 1000 (1초). 범위: 100-10000.",
"enableLogging": "요청 로깅 활성화",
"enableLoggingHelp": "Ollama API 요청, 응답 및 오류의 상세 로깅을 활성화합니다. 로그에는 시간 정보 및 연결 세부 정보가 포함됩니다."
```

### Dutch (nl)

```json
"test": "Testen",
"testing": "Testen...",
"refreshModels": "Modellen vernieuwen",
"refreshing": "Vernieuwen...",
"connectionSettings": "Verbindingsinstellingen",
"toolsSupport": "Toolondersteuning",
"noToolsSupport": "Geen toolondersteuning",
"models": "modellen",
"noToolsSupportHelp": "Deze modellen ondersteunen geen native tool-aanroepen en kunnen niet worden gebruikt met Roo Code. Ze worden alleen ter referentie weergegeven.",
"table": {
  "modelName": "Modelnaam",
  "context": "Context",
  "size": "Grootte",
  "quantization": "Kwantisatie",
  "family": "Familie",
  "images": "Afbeeldingen",
  "yes": "Ja",
  "no": "Nee",
  "sizeFormatting": { "gb": "GB", "mb": "MB" }
},
"streaming": "Streaming",
"streamingHelp": "Streaming is altijd ingeschakeld voor Ollama API-verzoeken. Reacties worden in real-time gestreamd terwijl ze worden gegenereerd.",
"requestTimeout": "Verzoek time-out (ms)",
"requestTimeoutHelp": "Time-out in milliseconden voor LLM API-verzoeken (chatvoltooiingen, denkwerk). Standaard: 3600000 (60 minuten). Bereik: 1000-7200000 (120 minuten).",
"modelDiscoveryTimeout": "Modeldetectie time-out (ms)",
"modelDiscoveryTimeoutHelp": "Time-out in milliseconden voor modeldetectie-verzoeken (lijst en ophalen van modeldetails). Standaard: 10000 (10 seconden). Bereik: 1000-600000 (10 minuten).",
"maxRetries": "Max. nieuwe pogingen",
"maxRetriesHelp": "Maximum aantal nieuwe pogingen voor mislukte verzoeken. Standaard: 0 (geen nieuwe pogingen). Bereik: 0-10.",
"retryDelay": "Vertraging nieuwe poging (ms)",
"retryDelayHelp": "Initiële vertraging tussen nieuwe pogingen in milliseconden. Gebruikt exponentiële backoff. Standaard: 1000 (1 seconde). Bereik: 100-10000.",
"enableLogging": "Verzoeklogboek inschakelen",
"enableLoggingHelp": "Gedetailleerde logboekregistratie van Ollama API-verzoeken, reacties en fouten inschakelen. Logboeken bevatten tijdinformatie en verbindingsdetails."
```

### Polish (pl)

```json
"test": "Testuj",
"testing": "Testowanie...",
"refreshModels": "Odśwież modele",
"refreshing": "Odświeżanie...",
"connectionSettings": "Ustawienia połączenia",
"toolsSupport": "Obsługa narzędzi",
"noToolsSupport": "Brak obsługi narzędzi",
"models": "modele",
"noToolsSupportHelp": "Te modele nie obsługują natywnych wywołań narzędzi i nie mogą być używane z Roo Code. Są wyświetlane tylko w celach informacyjnych.",
"table": {
  "modelName": "Nazwa modelu",
  "context": "Kontekst",
  "size": "Rozmiar",
  "quantization": "Kwantyzacja",
  "family": "Rodzina",
  "images": "Obrazy",
  "yes": "Tak",
  "no": "Nie",
  "sizeFormatting": { "gb": "GB", "mb": "MB" }
},
"streaming": "Przesyłanie strumieniowe",
"streamingHelp": "Przesyłanie strumieniowe jest zawsze włączone dla żądań API Ollama. Odpowiedzi są przesyłane strumieniowo w czasie rzeczywistym podczas generowania.",
"requestTimeout": "Limit czasu żądania (ms)",
"requestTimeoutHelp": "Limit czasu w milisekundach dla żądań API LLM (uzupełnienia czatu, praca myślowa). Domyślnie: 3600000 (60 minut). Zakres: 1000-7200000 (120 minut).",
"modelDiscoveryTimeout": "Limit czasu wykrywania modelu (ms)",
"modelDiscoveryTimeoutHelp": "Limit czasu w milisekundach dla żądań wykrywania modelu (lista i pobieranie szczegółów modelu). Domyślnie: 10000 (10 sekund). Zakres: 1000-600000 (10 minut).",
"maxRetries": "Maks. ponownych prób",
"maxRetriesHelp": "Maksymalna liczba prób ponowienia dla nieudanych żądań. Domyślnie: 0 (brak ponownych prób). Zakres: 0-10.",
"retryDelay": "Opóźnienie ponownej próby (ms)",
"retryDelayHelp": "Początkowe opóźnienie między próbami ponowienia w milisekundach. Używa wykładniczego wycofania. Domyślnie: 1000 (1 sekunda). Zakres: 100-10000.",
"enableLogging": "Włącz rejestrowanie żądań",
"enableLoggingHelp": "Włącz szczegółowe rejestrowanie żądań, odpowiedzi i błędów API Ollama. Dzienniki zawierają informacje o czasie i szczegóły połączenia."
```

### Portuguese (Brazil) (pt-BR)

```json
"test": "Testar",
"testing": "Testando...",
"refreshModels": "Atualizar modelos",
"refreshing": "Atualizando...",
"connectionSettings": "Configurações de conexão",
"toolsSupport": "Suporte a ferramentas",
"noToolsSupport": "Sem suporte a ferramentas",
"models": "modelos",
"noToolsSupportHelp": "Esses modelos não suportam chamadas de ferramentas nativas e não podem ser usados com Roo Code. Eles são mostrados apenas como referência.",
"table": {
  "modelName": "Nome do modelo",
  "context": "Contexto",
  "size": "Tamanho",
  "quantization": "Quantização",
  "family": "Família",
  "images": "Imagens",
  "yes": "Sim",
  "no": "Não",
  "sizeFormatting": { "gb": "GB", "mb": "MB" }
},
"streaming": "Transmissão",
"streamingHelp": "A transmissão está sempre habilitada para solicitações da API Ollama. As respostas são transmitidas em tempo real conforme são geradas.",
"requestTimeout": "Tempo limite da solicitação (ms)",
"requestTimeoutHelp": "Tempo limite em milissegundos para solicitações da API LLM (conclusões de chat, trabalho de pensamento). Padrão: 3600000 (60 minutos). Intervalo: 1000-7200000 (120 minutos).",
"modelDiscoveryTimeout": "Tempo limite de descoberta de modelos (ms)",
"modelDiscoveryTimeoutHelp": "Tempo limite em milissegundos para solicitações de descoberta de modelos (listagem e obtenção de detalhes do modelo). Padrão: 10000 (10 segundos). Intervalo: 1000-600000 (10 minutos).",
"maxRetries": "Máximo de tentativas",
"maxRetriesHelp": "Número máximo de tentativas de repetição para solicitações falhadas. Padrão: 0 (sem tentativas). Intervalo: 0-10.",
"retryDelay": "Atraso de tentativa (ms)",
"retryDelayHelp": "Atraso inicial entre tentativas de repetição em milissegundos. Usa backoff exponencial. Padrão: 1000 (1 segundo). Intervalo: 100-10000.",
"enableLogging": "Habilitar registro de solicitações",
"enableLoggingHelp": "Habilitar registro detalhado de solicitações, respostas e erros da API Ollama. Os registros incluem informações de tempo e detalhes de conexão."
```

### Russian (ru)

```json
"test": "Тест",
"testing": "Тестирование...",
"refreshModels": "Обновить модели",
"refreshing": "Обновляем...",
"connectionSettings": "Настройки подключения",
"toolsSupport": "Поддержка инструментов",
"noToolsSupport": "Без поддержки инструментов",
"models": "модели",
"noToolsSupportHelp": "Эти модели не поддерживают нативные вызовы инструментов и не могут использоваться с Roo Code. Они отображаются только для справки.",
"table": {
  "modelName": "Название модели",
  "context": "Контекст",
  "size": "Размер",
  "quantization": "Квантование",
  "family": "Семейство",
  "images": "Изображения",
  "yes": "Да",
  "no": "Нет",
  "sizeFormatting": { "gb": "ГБ", "mb": "МБ" }
},
"streaming": "Потоковая передача",
"streamingHelp": "Потоковая передача всегда включена для запросов API Ollama. Ответы передаются в реальном времени по мере их генерации.",
"requestTimeout": "Таймаут запроса (мс)",
"requestTimeoutHelp": "Таймаут в миллисекундах для запросов API LLM (завершения чата, мыслительная работа). По умолчанию: 3600000 (60 минут). Диапазон: 1000-7200000 (120 минут).",
"modelDiscoveryTimeout": "Таймаут обнаружения моделей (мс)",
"modelDiscoveryTimeoutHelp": "Таймаут в миллисекундах для запросов обнаружения моделей (список и получение деталей модели). По умолчанию: 10000 (10 секунд). Диапазон: 1000-600000 (10 минут).",
"maxRetries": "Макс. повторов",
"maxRetriesHelp": "Максимальное количество попыток повтора для неудачных запросов. По умолчанию: 0 (без повторов). Диапазон: 0-10.",
"retryDelay": "Задержка повтора (мс)",
"retryDelayHelp": "Начальная задержка между попытками повтора в миллисекундах. Использует экспоненциальный откат. По умолчанию: 1000 (1 секунда). Диапазон: 100-10000.",
"enableLogging": "Включить логирование запросов",
"enableLoggingHelp": "Включить подробное логирование запросов, ответов и ошибок API Ollama. Логи включают информацию о времени и детали подключения."
```

### Turkish (tr)

```json
"test": "Test",
"testing": "Test ediliyor...",
"refreshModels": "Modelleri yenile",
"refreshing": "Yenileniyor...",
"connectionSettings": "Bağlantı ayarları",
"toolsSupport": "Araç desteği",
"noToolsSupport": "Araç desteği yok",
"models": "modeller",
"noToolsSupportHelp": "Bu modeller yerel araç çağrılarını desteklemez ve Roo Code ile kullanılamaz. Yalnızca referans olarak gösterilirler.",
"table": {
  "modelName": "Model adı",
  "context": "Bağlam",
  "size": "Boyut",
  "quantization": "Kuantizasyon",
  "family": "Aile",
  "images": "Görüntüler",
  "yes": "Evet",
  "no": "Hayır",
  "sizeFormatting": { "gb": "GB", "mb": "MB" }
},
"streaming": "Akış",
"streamingHelp": "Ollama API istekleri için akış her zaman etkindir. Yanıtlar oluşturuldukça gerçek zamanlı olarak akışa alınır.",
"requestTimeout": "İstek zaman aşımı (ms)",
"requestTimeoutHelp": "LLM API istekleri (sohbet tamamlama, düşünme işi) için milisaniye cinsinden zaman aşımı. Varsayılan: 3600000 (60 dakika). Aralık: 1000-7200000 (120 dakika).",
"modelDiscoveryTimeout": "Model keşfi zaman aşımı (ms)",
"modelDiscoveryTimeoutHelp": "Model keşfi istekleri (liste ve model ayrıntılarını alma) için milisaniye cinsinden zaman aşımı. Varsayılan: 10000 (10 saniye). Aralık: 1000-600000 (10 dakika).",
"maxRetries": "Maks. yeniden deneme",
"maxRetriesHelp": "Başarısız istekler için maksimum yeniden deneme denemesi sayısı. Varsayılan: 0 (yeniden deneme yok). Aralık: 0-10.",
"retryDelay": "Yeniden deneme gecikmesi (ms)",
"retryDelayHelp": "Milisaniye cinsinden yeniden deneme denemeleri arasındaki başlangıç gecikmesi. Üstel geri çekilme kullanır. Varsayılan: 1000 (1 saniye). Aralık: 100-10000.",
"enableLogging": "İstek günlüğünü etkinleştir",
"enableLoggingHelp": "Ollama API istekleri, yanıtları ve hatalarının ayrıntılı günlüğünü etkinleştir. Günlükler zaman bilgisi ve bağlantı ayrıntılarını içerir."
```

### Vietnamese (vi)

```json
"test": "Kiểm tra",
"testing": "Đang kiểm tra...",
"refreshModels": "Làm mới mô hình",
"refreshing": "Đang làm mới...",
"connectionSettings": "Cài đặt kết nối",
"toolsSupport": "Hỗ trợ công cụ",
"noToolsSupport": "Không hỗ trợ công cụ",
"models": "mô hình",
"noToolsSupportHelp": "Các mô hình này không hỗ trợ gọi công cụ gốc và không thể sử dụng với Roo Code. Chúng chỉ được hiển thị để tham khảo.",
"table": {
  "modelName": "Tên mô hình",
  "context": "Ngữ cảnh",
  "size": "Kích thước",
  "quantization": "Lượng tử hóa",
  "family": "Họ",
  "images": "Hình ảnh",
  "yes": "Có",
  "no": "Không",
  "sizeFormatting": { "gb": "GB", "mb": "MB" }
},
"streaming": "Phát trực tuyến",
"streamingHelp": "Phát trực tuyến luôn được bật cho các yêu cầu API Ollama. Phản hồi được phát trực tuyến theo thời gian thực khi chúng được tạo.",
"requestTimeout": "Hết thời gian yêu cầu (ms)",
"requestTimeoutHelp": "Hết thời gian tính bằng mili giây cho các yêu cầu API LLM (hoàn thành trò chuyện, công việc suy nghĩ). Mặc định: 3600000 (60 phút). Phạm vi: 1000-7200000 (120 phút).",
"modelDiscoveryTimeout": "Hết thời gian khám phá mô hình (ms)",
"modelDiscoveryTimeoutHelp": "Hết thời gian tính bằng mili giây cho các yêu cầu khám phá mô hình (danh sách và lấy chi tiết mô hình). Mặc định: 10000 (10 giây). Phạm vi: 1000-600000 (10 phút).",
"maxRetries": "Số lần thử lại tối đa",
"maxRetriesHelp": "Số lần thử lại tối đa cho các yêu cầu thất bại. Mặc định: 0 (không thử lại). Phạm vi: 0-10.",
"retryDelay": "Độ trễ thử lại (ms)",
"retryDelayHelp": "Độ trễ ban đầu giữa các lần thử lại tính bằng mili giây. Sử dụng backoff theo cấp số nhân. Mặc định: 1000 (1 giây). Phạm vi: 100-10000.",
"enableLogging": "Bật ghi nhật ký yêu cầu",
"enableLoggingHelp": "Bật ghi nhật ký chi tiết về yêu cầu, phản hồi và lỗi API Ollama. Nhật ký bao gồm thông tin thời gian và chi tiết kết nối."
```

### Chinese Simplified (zh-CN)

```json
"test": "测试",
"testing": "测试中...",
"refreshModels": "刷新模型",
"refreshing": "刷新中...",
"connectionSettings": "连接设置",
"toolsSupport": "工具支持",
"noToolsSupport": "无工具支持",
"models": "模型",
"noToolsSupportHelp": "这些模型不支持原生工具调用，无法与 Roo Code 一起使用。它们仅作为参考显示。",
"table": {
  "modelName": "模型名称",
  "context": "上下文",
  "size": "大小",
  "quantization": "量化",
  "family": "系列",
  "images": "图像",
  "yes": "是",
  "no": "否",
  "sizeFormatting": { "gb": "GB", "mb": "MB" }
},
"streaming": "流式传输",
"streamingHelp": "Ollama API 请求始终启用流式传输。响应在生成时实时流式传输。",
"requestTimeout": "请求超时 (ms)",
"requestTimeoutHelp": "LLM API 请求（聊天完成、思考工作）的超时时间（毫秒）。默认值: 3600000 (60 分钟)。范围: 1000-7200000 (120 分钟)。",
"modelDiscoveryTimeout": "模型发现超时 (ms)",
"modelDiscoveryTimeoutHelp": "模型发现请求（列出和获取模型详细信息）的超时时间（毫秒）。默认值: 10000 (10 秒)。范围: 1000-600000 (10 分钟)。",
"maxRetries": "最大重试次数",
"maxRetriesHelp": "失败请求的最大重试尝试次数。默认值: 0 (不重试)。范围: 0-10。",
"retryDelay": "重试延迟 (ms)",
"retryDelayHelp": "重试尝试之间的初始延迟（毫秒）。使用指数退避。默认值: 1000 (1 秒)。范围: 100-10000。",
"enableLogging": "启用请求日志",
"enableLoggingHelp": "启用 Ollama API 请求、响应和错误的详细日志记录。日志包括时间信息和连接详细信息。"
```

### Chinese Traditional (zh-TW)

```json
"test": "測試",
"testing": "測試中...",
"refreshModels": "重新整理模型",
"refreshing": "重新整理中...",
"connectionSettings": "連線設定",
"toolsSupport": "工具支援",
"noToolsSupport": "無工具支援",
"models": "模型",
"noToolsSupportHelp": "這些模型不支援原生工具呼叫，無法與 Roo Code 一起使用。它們僅作為參考顯示。",
"table": {
  "modelName": "模型名稱",
  "context": "上下文",
  "size": "大小",
  "quantization": "量化",
  "family": "系列",
  "images": "影像",
  "yes": "是",
  "no": "否",
  "sizeFormatting": { "gb": "GB", "mb": "MB" }
},
"streaming": "串流",
"streamingHelp": "Ollama API 請求始終啟用串流。回應在生成時即時串流。",
"requestTimeout": "請求逾時 (ms)",
"requestTimeoutHelp": "LLM API 請求（聊天完成、思考工作）的逾時時間（毫秒）。預設值: 3600000 (60 分鐘)。範圍: 1000-7200000 (120 分鐘)。",
"modelDiscoveryTimeout": "模型探索逾時 (ms)",
"modelDiscoveryTimeoutHelp": "模型探索請求（列出和取得模型詳細資訊）的逾時時間（毫秒）。預設值: 10000 (10 秒)。範圍: 1000-600000 (10 分鐘)。",
"maxRetries": "最大重試次數",
"maxRetriesHelp": "失敗請求的最大重試嘗試次數。預設值: 0 (不重試)。範圍: 0-10。",
"retryDelay": "重試延遲 (ms)",
"retryDelayHelp": "重試嘗試之間的初始延遲（毫秒）。使用指數退避。預設值: 1000 (1 秒)。範圍: 100-10000。",
"enableLogging": "啟用請求記錄",
"enableLoggingHelp": "啟用 Ollama API 請求、回應和錯誤的詳細記錄。記錄包括時間資訊和連線詳細資訊。"
```

## Implementation Best Practices

### 1. i18n-ally Integration

The Roo Code environment uses the **i18n-ally** extension for VS Code. Before committing:

- Use the `i18n-ally.sortKeys` command to sort keys in `settings.json` files
- This ensures consistency with the existing repo style
- Keys should be sorted alphabetically within their sections

### 2. React Component Usage

- **Simple strings**: Use `t("settings:providers.ollama.keyName")` for plain text
- **Text with links/HTML**: Use `<Trans>` component (already implemented for `description` field)
- **Current implementation verification**:
    - ✅ `description` field: Already uses `<Trans>` component with `<quickstartLink>` - **No changes needed**
    - ✅ `noToolsSupportHelp`: Uses `t()` which is correct (plain text, no links/HTML)
    - ✅ All other new keys: Use `t()` which is correct (all are plain text)
- **No React component changes required**: The existing implementation is correct

### 3. JSON Formatting Compliance

- **Indentation**: Use 2 spaces (not tabs)
- **Trailing commas**: Do NOT use trailing commas in JSON (except where repo standard explicitly allows)
- **Quotes**: Use double quotes for all keys and string values
- **CI/CD**: VS Code extension CI pipelines are strict about JSON formatting - validate before committing

### 4. Validation Checklist

Before committing translation files:

- [ ] All JSON files are valid (no syntax errors)
- [ ] No trailing commas (except where allowed by repo standard)
- [ ] 2-space indentation throughout
- [ ] Keys sorted using i18n-ally.sortKeys
- [ ] All 29 keys present in all 17 non-English locale files
- [ ] No hardcoded strings in React component (all use translation keys)

## Technical Considerations

### 1. JSON Structure

- Maintain consistent indentation (2 spaces)
- Ensure proper comma placement (no trailing commas)
- Validate JSON syntax before committing

### 2. Special Characters

- Handle quotes properly (escape if needed)
- Support Unicode characters for all languages
- Preserve HTML-like tags in descriptions (e.g., `<quickstartLink>`)

### 3. Variable Interpolation

- The `description` key uses React i18n's `<Trans>` component with `<quickstartLink>`
- Ensure translators understand not to modify these tags
- Document any interpolation patterns

### 4. Pluralization

- The `models` key is used in plural contexts
- Some languages have complex plural rules (e.g., Polish, Russian)
- Consider if pluralization support is needed (currently using simple "models" string)

### 5. Text Length

- Some languages produce longer text than English
- Ensure UI components can accommodate longer translations
- Test table column widths with longer translations
- Consider responsive design for mobile/tablet views

## Testing Plan

### Automated Tests

An automated test file has been created: `webview-ui/src/i18n/__tests__/ollama-coverage.spec.ts`

**Test Coverage**:

1. **Key Existence Test**

    - Verify all English keys exist in all 17 non-English locale files
    - Fail build if any key is missing
    - Provides clear error messages indicating which key is missing in which language

2. **JSON Validity Test**

    - Validate JSON syntax for all language files
    - Catch syntax errors early
    - Verify proper structure (providers.ollama exists)

3. **Plural Object Validation**

    - Ensures "models" key uses plural object format (not simple string)
    - Validates "one" and "other" keys exist for all languages
    - Validates "few" and "many" keys exist for Russian and Polish
    - Ensures all plural form values are non-empty strings

4. **Key Count Consistency**

    - Verifies all locales have the same number of keys as English
    - Catches accidental deletions or additions

5. **Key Reference Test** (Manual)
    - Verify all keys used in `Ollama.tsx` exist in English
    - Prevent runtime errors from missing keys

### Integration Tests

1. **UI Rendering Test**

    - Render Ollama settings component in each language
    - Verify no missing translation warnings in console
    - Check that all text is visible and readable

2. **Text Overflow Test**
    - Test with longest translations (typically German)
    - Verify table columns don't break layout
    - Check button text fits properly

### Manual Testing Checklist

- [ ] Test connection button works in all languages
- [ ] Refresh models button works in all languages
- [ ] Table displays correctly with translated headers
- [ ] Connection settings section expands/collapses properly
- [ ] All help text is readable and properly formatted
- [ ] Error messages display correctly
- [ ] Success messages display correctly

## Implementation Checklist

### Phase 1: Initial Setup

- [ ] Create translation template document
- [ ] **Important**: Install/use i18n-ally VS Code extension
- [ ] Add all 29 keys to Catalan (ca) with translations
- [ ] Run `i18n-ally.sortKeys` on Catalan file to verify sorting
- [ ] Add all 29 keys to German (de) with English placeholders
- [ ] Add all 29 keys to Spanish (es) with English placeholders
- [ ] Add all 29 keys to French (fr) with English placeholders
- [ ] Add all 29 keys to Hindi (hi) with English placeholders
- [ ] Add all 29 keys to Indonesian (id) with English placeholders
- [ ] Add all 29 keys to Italian (it) with English placeholders
- [ ] Add all 29 keys to Japanese (ja) with English placeholders
- [ ] Add all 29 keys to Korean (ko) with English placeholders
- [ ] Add all 29 keys to Dutch (nl) with English placeholders
- [ ] Add all 29 keys to Polish (pl) with English placeholders
- [ ] Add all 29 keys to Portuguese (pt-BR) with English placeholders
- [ ] Add all 29 keys to Russian (ru) with English placeholders
- [ ] Add all 29 keys to Turkish (tr) with English placeholders
- [ ] Add all 29 keys to Vietnamese (vi) with English placeholders
- [ ] Add all 29 keys to Chinese Simplified (zh-CN) with English placeholders
- [ ] Add all 29 keys to Chinese Traditional (zh-TW) with translations
- [ ] **Run `i18n-ally.sortKeys` on all language files** to ensure consistent sorting
- [ ] Verify JSON syntax for all files (no trailing commas, 2-space indentation)
- [ ] Test UI renders without errors

### Phase 2: Translation

- [ ] Translate Catalan (ca)
- [ ] Translate German (de)
- [ ] Translate Spanish (es)
- [ ] Translate French (fr)
- [ ] Translate Hindi (hi)
- [ ] Translate Indonesian (id)
- [ ] Translate Italian (it)
- [ ] Translate Japanese (ja)
- [ ] Translate Korean (ko)
- [ ] Translate Dutch (nl)
- [ ] Translate Polish (pl)
- [ ] Translate Portuguese (pt-BR)
- [ ] Translate Russian (ru)
- [ ] Translate Turkish (tr)
- [ ] Translate Vietnamese (vi)
- [ ] Translate Chinese Simplified (zh-CN)
- [ ] Translate Chinese Traditional (zh-TW)

### Phase 3: Quality Assurance

- [ ] Run automated tests (key existence, JSON validity)
- [ ] Manual testing in each language
- [ ] Review translations for technical accuracy
- [ ] Check for consistency with other provider settings
- [ ] Verify text fits in UI components
- [ ] Test error handling with missing keys (should fallback to English)

## Future Considerations

### 1. Translation Management

- Consider using a translation management platform (e.g., Crowdin, Lokalise)
- Automate translation workflow
- Track translation completion status

### 2. Context for Translators

- Create a translation guide document
- Provide screenshots of the UI
- Document technical terms that should remain in English

### 3. Continuous Improvement

- Collect feedback from users in different languages
- Update translations based on user feedback
- Maintain translation quality over time

### 4. Pluralization Support

- Evaluate if i18n library supports proper pluralization
- Consider implementing plural rules for languages that need them
- Update "models" key usage if needed

## Risk Assessment

### Low Risk

- Adding keys with English placeholders (UI will work, just show English)
- JSON syntax errors (caught by validation)

### Medium Risk

- Text overflow in UI components (mitigated by testing)
- Inconsistent terminology (mitigated by review process)
- Missing context for translators (mitigated by documentation)

### High Risk

- None identified - this is a straightforward i18n addition

## Success Criteria

1. ✅ All 29 translation keys exist in all 18 locale files (17 non-English + English)
2. ✅ UI renders correctly in all languages without errors
3. ✅ All text is properly translated (not showing English placeholders)
4. ✅ No text overflow or layout issues
5. ✅ Translations are technically accurate
6. ✅ Terminology is consistent with the rest of the application
7. ✅ Automated tests pass
8. ✅ Manual testing confirms functionality in all languages

## Timeline Estimate

- **Phase 1 (Add Keys)**: 2-4 hours

    - Adding keys to 17 non-English locale files
    - JSON validation
    - Initial testing

- **Phase 2 (Translation)**: 8-16 hours (depending on translation method)

    - Professional translation: 8-12 hours
    - Community translation: 12-16 hours (with review)

- **Phase 3 (QA)**: 4-6 hours
    - Automated testing
    - Manual testing
    - Review and fixes

**Total Estimated Time**: 14-26 hours

## Conclusion

This design document provides a comprehensive plan for implementing full internationalization support for the Ollama settings page. By following this plan, we ensure:

1. Complete translation coverage across all supported languages
2. Consistent terminology and user experience
3. Maintainable and scalable translation structure
4. Quality assurance through testing and review

The implementation is straightforward and low-risk, primarily involving adding translation keys and ensuring proper translation quality.

## Translation Quality Notes

### Strengths

- **German**: Uses professional "Verbindung wird geprüft..." instead of literal "Testen..."
- **Russian**: Uses natural verb form "Обновляем..." for active processes
- **Spanish**: Includes article "la" for polished phrasing
- **French**: Correctly uses "Go" and "Mo" matching French technical standards

### UI Considerations

- **German button width**: "Modelle aktualisieren" may need UI testing to ensure it fits within button width (German compounds can be long). The translation is correct; this is a UI layout consideration.
- **Russian pluralization**: The "models" key is used with counts (e.g., `{9} {t("models")}`). Russian requires three plural forms:

    - 1 model = "модель" (nominative singular)
    - 2-4 models = "модели" (genitive singular)
    - 5+ models = "моделей" (genitive plural)

    **Current approach**: Using simple string "модели" is acceptable for this use case since:

    - The count is displayed separately
    - The word appears after a number (typically uses genitive plural form)
    - Many UIs use this pattern even in languages with complex plural rules

    **Alternative approach** (if proper pluralization desired): Use i18next pluralization with `{{count}}` interpolation:

    ```json
    "models": "{{count}} модель",
    "models_one": "{{count}} модель",
    "models_few": "{{count}} модели",
    "models_many": "{{count}} моделей"
    ```

    This would require changing the React code to: `{t("settings:providers.ollama.models", { count: modelsWithTools.length })}`

    **Recommendation**: Keep the current simple approach for consistency with existing codebase patterns.

### Technical Accuracy

- **Spanish**: "Cuantización" is the correct technical term for model quantization
- **French**: "Diffusion en continu" is technically accurate and standard for streaming

## Quick Reference for Implementation

### Pre-Implementation Checklist

- [ ] Install i18n-ally VS Code extension
- [ ] Review all translations in this document
- [ ] Verify React component already uses correct translation methods (✅ confirmed)

### During Implementation

1. Add translations to each language file in this order:

    - Add all 29 keys to each `settings.json` file
    - Place keys after existing `warning` key
    - Maintain alphabetical order within logical groups

2. After adding to each file:

    - Run `i18n-ally.sortKeys` command
    - Verify JSON syntax (no trailing commas, 2-space indentation)
    - Check that all 29 keys are present

3. Final validation:
    - All 17 non-English locale files updated
    - All JSON files valid
    - Keys sorted consistently
    - No trailing commas
    - Test UI renders in each language

### Key Implementation Points

- ✅ **No React component changes needed** - existing code is correct
- ✅ **Use i18n-ally.sortKeys** - ensures repo consistency
- ✅ **No trailing commas** - CI/CD requirement
- ✅ **2-space indentation** - repo standard
- ✅ **All translations provided** - ready to copy-paste

---

**Document Version**: 1.1
**Last Updated**: 2024
**Author**: AI Assistant
**Status**: Ready for Implementation
**Implementation Notes**: Includes i18n-ally integration guidance and CI/CD compliance requirements
