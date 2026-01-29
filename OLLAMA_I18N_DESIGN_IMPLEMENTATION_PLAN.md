# Ollama i18n Implementation Plan

## Document Reference

This implementation plan is based on the comprehensive design document: **`OLLAMA_I18N_DESIGN.md`**

Please refer to `OLLAMA_I18N_DESIGN.md` for:

- Complete translation keys and values for all 17 non-English locales
- Translation quality notes and refinements
- Technical considerations and best practices
- Implementation guidelines

## Overview

This plan provides step-by-step instructions to add 29 missing translation keys to 17 non-English locale files for the Ollama provider settings page.

**Total files to modify**: 18 locale files (17 non-English + English for plural object update)
**New test file**: 1 (`ollama-coverage.spec.ts` - automated regression prevention)
**Total keys to add per file**: 29 keys (28 new + 1 updated to plural object)
**Total keys to add**: 493 keys (29 × 17 non-English locales)
**Estimated time**: 2-4 hours
**Key change**: `"models"` key converted to plural object format for future-proofing
**Automated test**: Prevents future regressions and catches missing keys automatically

## Implementation Strategy Summary

### Risk Mitigation Approach

1. **Template First**: Test with English (en) and German (de) files first

    - German has longest strings - catches UI layout issues early
    - Validate UI rendering before proceeding to all languages

2. **Pluralization Future-Proofing**: Convert `"models"` to plural object

    - Most languages: `{ "one": "...", "other": "..." }`
    - Russian: `{ "one": "...", "few": "...", "many": "...", "other": "..." }`
    - Polish: `{ "one": "...", "few": "...", "many": "...", "other": "..." }`

3. **i18n-ally Validation**: Use extension's Progress/Review tab
    - Visual completion tracking
    - Automatic missing key detection
    - Percentage completion per language

## Additional Translation Requirements

### ✅ APPROVED: Missing Translations for Common Terms

**Status**: ✅ **APPROVED** - Ready for implementation

During testing with Indonesian language, it was discovered that several common terms are still in English across multiple languages. These terms appear in the Ollama provider settings and other provider configurations:

1. **"Provider"** - Used in `settings:sections.providers` (navigation sidebar)
2. **"Base"** - Used in `settings:providers.ollama.baseUrl` (as part of "Base URL")
3. **"Model"** - Used in `settings:providers.ollama.modelId` (as part of "Model ID")

**Impact**:

- These terms appear in the main navigation and provider settings pages
- Currently visible as English words in Indonesian, Hindi (partial), and Dutch (partial)
- Affects user experience and consistency across the application

**Languages Requiring Updates**:

- **Indonesian (id)**: All 3 terms need translation
- **Hindi (hi)**: "Base" needs translation (currently "बेस" - English transliteration)
- **Dutch (nl)**: "Provider" needs translation (currently "Providers" - English)

### Proposed Translations Table

The following table provides proposed translations for these three terms across all 17 non-English locales. These translations should be reviewed before implementation.

**Translation Keys:**

- `settings:sections.providers` - Navigation sidebar (should be plural where appropriate)
- `settings:providers.ollama.baseUrl` - Label for Base URL field
- `settings:providers.ollama.modelId` - Label for Model ID field

| Language                  | Code  | Current "Provider" | Proposed "Provider"  | Current "Base URL"        | Proposed "Base"                | Current "Model ID" | Proposed "Model"   | Status              |
| ------------------------- | ----- | ------------------ | -------------------- | ------------------------- | ------------------------------ | ------------------ | ------------------ | ------------------- |
| **Catalan**               | ca    | Proveïdors         | ✅ Proveïdors        | URL base (opcional)       | ✅ Base                        | ID del model       | ✅ Model           | ✅ Complete         |
| **German**                | de    | Anbieter           | ✅ Anbieter          | Basis-URL (optional)      | ✅ Basis                       | Modell-ID          | ✅ Modell          | ✅ Complete         |
| **Spanish**               | es    | Proveedores        | ✅ Proveedores       | URL base (opcional)       | ✅ Base                        | ID del modelo      | ✅ Modelo          | ✅ Complete         |
| **French**                | fr    | Fournisseurs       | ✅ Fournisseurs      | URL de base (optionnel)   | ✅ Base                        | ID du modèle       | ✅ Modèle          | ✅ Complete         |
| **Hindi**                 | hi    | प्रदाता            | ✅ प्रदाता           | बेस URL (वैकल्पिक)        | ⚠️ **आधार URL (वैकल्पिक)** ✅  | मॉडल ID            | ✅ मॉडल            | ⚠️ Needs "Base"     |
| **Indonesian**            | id    | **Provider**       | ⚠️ **Penyedia** ✅   | **Base URL (opsional)**   | ⚠️ **URL Dasar (opsional)** ✅ | **Model ID**       | ⚠️ **ID Model** ✅ | ⚠️ Needs all 3      |
| **Italian**               | it    | Fornitori          | ✅ Fornitori         | URL base (opzionale)      | ✅ Base                        | ID modello         | ✅ Modello         | ✅ Complete         |
| **Japanese**              | ja    | プロバイダー       | ✅ プロバイダー      | ベースURL（オプション）   | ✅ ベース                      | モデルID           | ✅ モデル          | ✅ Complete         |
| **Korean**                | ko    | 공급자             | ✅ 공급자            | 기본 URL (선택사항)       | ✅ 기본                        | 모델 ID            | ✅ 모델            | ✅ Complete         |
| **Dutch**                 | nl    | **Providers**      | ⚠️ **Aanbieders** ✅ | Basis-URL (optioneel)     | ✅ Basis                       | Model-ID           | ✅ Model           | ⚠️ Needs "Provider" |
| **Polish**                | pl    | Dostawcy           | ✅ Dostawcy          | URL bazowy (opcjonalnie)  | ✅ Bazowy                      | ID modelu          | ✅ Model           | ✅ Complete         |
| **Portuguese (BR)**       | pt-BR | Provedores         | ✅ Provedores        | URL Base (opcional)       | ✅ Base                        | ID do Modelo       | ✅ Modelo          | ✅ Complete         |
| **Russian**               | ru    | Провайдеры         | ✅ Провайдеры        | Базовый URL (опционально) | ✅ Базовый                     | ID модели          | ✅ Модель          | ✅ Complete         |
| **Turkish**               | tr    | Sağlayıcılar       | ✅ Sağlayıcılar      | Temel URL (İsteğe bağlı)  | ✅ Temel                       | Model Kimliği      | ✅ Model           | ✅ Complete         |
| **Vietnamese**            | vi    | Nhà cung cấp       | ✅ Nhà cung cấp      | URL cơ sở (tùy chọn)      | ✅ Cơ sở                       | ID mô hình         | ✅ Mô hình         | ✅ Complete         |
| **Chinese (Simplified)**  | zh-CN | 提供商             | ✅ 提供商            | 基础 URL（可选）          | ✅ 基础                        | 模型 ID            | ✅ 模型            | ✅ Complete         |
| **Chinese (Traditional)** | zh-TW | 供應商             | ✅ 供應商            | 基礎 URL（選用）          | ✅ 基礎                        | 模型 ID            | ✅ 模型            | ✅ Complete         |

**Legend:**

- ✅ = Already correctly translated
- ⚠️ = Needs translation update

### Implementation Notes

**Keys to Update:**

1. `settings:sections.providers` - Navigation sidebar label (line ~29 in each locale file)

    - Used in: Settings navigation sidebar
    - Should be plural where grammatically appropriate

2. `settings:providers.ollama.baseUrl` - Base URL field label (line ~430-440 in each locale file)

    - Used in: Ollama provider settings page
    - Current format: "Base URL (optional)" or "Base URL (opsional)"
    - Should use translated word for "Base"

3. `settings:providers.ollama.modelId` - Model ID field label (line ~430-440 in each locale file)
    - Used in: Ollama provider settings page
    - Current format: "Model ID"
    - Should use translated word for "Model"

**Note**: These same keys may also be used in other provider configurations (LM Studio, etc.), so updating them will improve consistency across all providers.

### Implementation Checklist for Common Terms Translation

**Pre-Implementation:**

- [ ] Audit the 14 "complete" languages to verify they follow standardized patterns:
    - [ ] Check `settings:providers.ollama.baseUrl` uses "URL [Base]" format
    - [ ] Check `settings:providers.ollama.modelId` uses "ID [Model]" format
    - [ ] Verify consistency across all provider configurations

**Implementation:**

- [ ] Update Indonesian (id):

    - [ ] `settings:sections.providers`: "Provider" → "Penyedia"
    - [ ] `settings:providers.ollama.baseUrl`: "Base URL (opsional)" → "URL Dasar (opsional)"
    - [ ] `settings:providers.ollama.modelId`: "Model ID" → "ID Model"

- [ ] Update Hindi (hi):

    - [ ] `settings:providers.ollama.baseUrl`: "बेस URL (वैकल्पिक)" → "आधार URL (वैकल्पिक)"

- [ ] Update Dutch (nl):
    - [ ] `settings:sections.providers`: "Providers" → "Aanbieders"

**Post-Implementation:**

- [ ] Update automated test (`ollama-coverage.spec.ts`) to verify these keys:

    - [ ] Add check for `settings:sections.providers` non-empty
    - [ ] Add check for `settings:providers.ollama.baseUrl` non-empty
    - [ ] Add check for `settings:providers.ollama.modelId` non-empty
    - [ ] Verify no English words in these keys (for non-English locales)

- [ ] UI Testing:
    - [ ] Test sidebar width with "Aanbieders" (Dutch) - verify no overflow
    - [ ] Test sidebar width with "Penyedia" (Indonesian) - verify no overflow
    - [ ] Verify all three languages display correctly in UI

**Current State Analysis:**

- **Fully Translated**: ca, de, es, fr, it, ja, ko, pl, pt-BR, ru, tr, vi, zh-CN, zh-TW (14 languages) ✅
- **Needs Updates (✅ APPROVED Translations)**:
    - **Hindi (hi)**: Has "प्रदाता" (Provider) ✅, but uses "बेस" (English transliteration) → **Approved: "आधार URL (वैकल्पिक)"**
    - **Indonesian (id)**: All 3 terms in English → **Approved: "Penyedia", "URL Dasar (opsional)", "ID Model"**
    - **Dutch (nl)**: Has "Basis" (Base) ✅ and "Model" ✅, but sidebar shows "Providers" (English) → **Approved: "Aanbieders"**

**Translation Context (✅ APPROVED):**

- **"Provider"** in sidebar context (`settings:sections.providers`): Should be plural where grammatically appropriate
    - ✅ Indonesian: "Penyedia" (approved - can be used as plural in Indonesian)
    - ✅ Dutch: "Aanbieders" (approved - plural form, idiomatic for Dutch software interfaces)
- **"Base URL"** (`settings:providers.ollama.baseUrl`): Standardized to **"URL [Base] (optional)"** format
    - ✅ Indonesian: "URL Dasar (opsional)" (approved - noun before modifier pattern)
    - ✅ Hindi: "आधार URL (वैकल्पिक)" (approved - replace "बेस" with "आधार" for professional UI)
    - ✅ Standardization: All languages should use "URL [Base]" format for consistency
- **"Model ID"** (`settings:providers.ollama.modelId`): Standardized to **"ID [Model]"** format
    - ✅ Indonesian: "ID Model" (approved - consistent with Spanish/French patterns)
    - ✅ Standardization: All languages should use "ID [Model]" format (e.g., "ID del modelo", "ID du modèle")

### Review Required - ✅ APPROVED

**Review Status**: ✅ **APPROVED** - Ready for implementation

**Approved Translations and Decisions:**

1. **Indonesian (id)** - All 3 terms approved:

    - ✅ Provider: "Penyedia" (singular/plural) - **APPROVED**
    - ✅ Base URL: "URL Dasar (opsional)" - **APPROVED** (noun before modifier pattern)
    - ✅ Model ID: "ID Model" - **APPROVED** (consistent with Spanish/French patterns)

2. **Hindi (hi)** - "Base" translation approved:

    - ✅ Current: "बेस URL (वैकल्पिक)" (uses English transliteration "बेस")
    - ✅ Proposed: "आधार URL (वैकल्पिक)" - **APPROVED** (more professional than transliteration)

3. **Dutch (nl)** - "Provider" translation approved:

    - ✅ Current: "Providers" (English)
    - ✅ Proposed: "Aanbieders" (plural) - **APPROVED** (idiomatic for Dutch software interfaces)

4. **Word Order Standardization** - ✅ **APPROVED**:

    - ✅ Standardize on **"URL [Base]"** format (e.g., "URL Dasar", "URL base")
    - ✅ Standardize on **"ID [Model]"** format (e.g., "ID Model", "ID del modelo")
    - **Rationale**: Consistent with majority of languages and reduces cognitive load

5. **Verification Required**:
    - ⚠️ Audit the 14 "complete" languages to ensure they follow standardized patterns:
        - Verify `settings:providers.ollama.baseUrl` uses "URL [Base]" format
        - Verify `settings:providers.ollama.modelId` uses "ID [Model]" format
    - ⚠️ Check sidebar width compatibility for "Aanbieders" (Dutch) and "Penyedia" (Indonesian)
    - ⚠️ Update automated test to verify these common terms are translated (non-empty, no English words)

---

## Additional Missing Translations - Post-Implementation Review

### ✅ APPROVED: Additional English Words Found in UI

**Status**: ✅ **APPROVED** - All decisions made, ready for implementation

During UI testing with Indonesian language, additional English words were discovered that were not addressed in the initial common terms translation effort. This section documents what was missed, why it was missed, and how to address each issue.

---

### Issue Analysis

#### Issue 1: "Provider API" Label Still in English

**Location**: `settings:providers.apiProvider`
**Current Indonesian Value**: "Provider API" (English)
**Used In**: Main provider selection dropdown label in `ApiOptions.tsx` (line 501)
**Impact**: High - This is a prominent label visible at the top of the provider settings page

**Why It Was Missed**:

- The initial common terms review focused on `settings:sections.providers` (sidebar navigation)
- This key (`settings:providers.apiProvider`) is used in a different component (`ApiOptions.tsx`) and was not part of the Ollama-specific settings
- The key is shared across all providers, not just Ollama

**Current State Across Languages**:

- ✅ **Translated**: de ("API-Anbieter"), es ("Proveedor de API"), fr ("Fournisseur d'API"), it ("Fornitore API"), ja ("APIプロバイダー"), ko ("API 제공자"), pl ("Dostawca API"), pt-BR ("Provedor de API"), ru ("Провайдер API"), tr ("API Sağlayıcı"), vi ("Nhà cung cấp API"), zh-CN ("API提供商"), zh-TW ("API 供應商"), ca ("Proveïdor d'API"), hi ("API प्रदाता")
- ⚠️ **Still English**: id ("Provider API"), nl ("API-provider" - partially translated)

**Proposed Translation for Indonesian**:

- Option A: "Penyedia API" (Provider API) - follows pattern of other languages
- Option B: "API Penyedia" (API Provider) - alternative word order
- **✅ APPROVED**: "Penyedia API"
- **Rationale**: Maintains consistency with "Penyedia" used in sidebar. Matches patterns of other Romance/Germanic languages (Noun + "API"), making the interface feel professional and standardized.

---

#### Issue 2: "ID" in "ID Model" Still in English

**Location**: `settings:providers.ollama.modelId`
**Current Indonesian Value**: "ID Model"
**Used In**: Ollama provider settings - Model ID input field label
**Impact**: Medium - Technical acronym, but should be consistent with other languages

**Why It Was Missed**:

- "ID" is a technical acronym (Identifier) commonly used in software interfaces
- Many languages keep "ID" as-is (e.g., Spanish "ID del modelo", French "ID du modèle")
- The focus was on translating "Model", not the acronym "ID"
- However, some languages do translate it (e.g., Turkish "Model Kimliği" = "Model Identity")

**Current State Across Languages**:

- **Keeps "ID" as acronym**: es ("ID del modelo"), fr ("ID du modèle"), it ("ID modello"), ja ("モデルID"), ko ("모델 ID"), pl ("ID modelu"), pt-BR ("ID do Modelo"), ru ("ID модели"), zh-CN ("模型 ID"), zh-TW ("模型 ID"), ca ("ID del model"), de ("Modell-ID"), nl ("Model-ID")
- **Translates "ID"**: tr ("Model Kimliği" = Model Identity), vi ("ID mô hình" - but "ID" is still there), hi ("मॉडल ID" - "ID" is still there)
- **Question**: Should Indonesian translate "ID" to "Identitas" or keep it as "ID"?

**Proposed Translation for Indonesian**:

- Option A: Keep "ID Model" (consistent with majority of languages, "ID" is a widely recognized acronym)
- Option B: "Identitas Model" (fully translated, matches Turkish pattern)
- **✅ APPROVED**: Keep "ID Model"
- **Rationale**: "ID" is a universally understood technical acronym in Indonesia. Using "Identitas" can make the UI feel "over-translated" and less intuitive for developers who are the primary audience for Roo Code.
- **Technical Note**: Ensure casing is correct - use "ID Model" (uppercase ID) for UI labels, while keys use camelCase (modelId)

---

#### Issue 3: "opsional" - Is This Really Indonesian?

**Location**: Multiple keys using "(opsional)" suffix
**Current Indonesian Value**: "opsional"
**Used In**: Various labels like "URL Dasar (opsional)", "Base URL (opsional)"
**Impact**: Medium - Word is commonly used but is a loanword from English

**Why It Was Missed**:

- "Opsional" is a commonly used loanword in Indonesian (from English "optional")
- It's widely accepted in Indonesian software interfaces
- However, there are more native Indonesian alternatives

**Current State**:

- Indonesian uses "opsional" throughout the codebase
- Other languages use native translations: es ("opcional"), fr ("optionnel"), de ("optional"), etc.

**Native Indonesian Alternatives**:

- "pilihan" (choice/optional) - more native
- "tidak wajib" (not mandatory) - more descriptive
- "boleh tidak" (may or may not) - informal
- "opsional" - loanword, widely accepted in tech context

**Proposed Translation for Indonesian**:

- Option A: Keep "opsional" (widely accepted loanword in Indonesian tech interfaces)
- Option B: Use "pilihan" (more native, but less common in software)
- Option C: Use "tidak wajib" (more descriptive, but longer)
- **✅ APPROVED**: Keep "opsional"
- **Rationale**: In software UI, "opsional" is the standard loanword. While "tidak wajib" is more native, it is significantly longer and could cause layout issues (similar to concerns in the German/French "Template First" strategy).

---

#### Issue 4: "Rate limit" Still in English

**Location**: `settings:providers.rateLimitSeconds.label`
**Current Indonesian Value**: "Rate limit" (English)
**Used In**: Advanced settings section - Rate limiting control (`RateLimitSecondsControl.tsx`, line 15)
**Impact**: Medium - Advanced setting, but should be translated for completeness

**Why It Was Missed**:

- This key is in the advanced settings section, not the main Ollama provider settings
- The initial review focused on the main Ollama settings page
- The key is shared across all providers, not Ollama-specific

**Current State Across Languages**:

- ✅ **Translated**: de ("Ratenbegrenzung"), es ("Límite de tasa"), fr ("Limitation du débit"), it ("Limitazione della frequenza"), ja ("レート制限"), ko ("속도 제한"), pl ("Ograniczenie szybkości"), pt-BR ("Limitação de taxa"), ru ("Ограничение частоты"), tr ("Hız sınırlaması"), vi ("Giới hạn tốc độ"), zh-CN ("请求频率限制"), zh-TW ("速率限制"), ca ("Limitació de taxa"), hi ("दर सीमा")
- ⚠️ **Still English**: id ("Rate limit")

**Proposed Translation for Indonesian**:

- "Pembatasan laju" (Rate limitation) - technical term
- "Batas kecepatan" (Speed limit) - more common term
- "Pembatasan rate" (Rate limitation, using loanword) - matches chat.json usage
- **✅ APPROVED**: "Pembatasan laju"
- **Rationale**: Aligns with technical Indonesian standards and existing chat.json usage (line 155). "Batas kecepatan" is more commonly associated with driving/physical speed rather than network frequency.
- **Layout Note**: "Pembatasan laju" is longer than "Rate limit" - verify Advanced Settings section layout during QA

---

#### Issue 5: "model" Word Used as English Loanword Throughout Indonesian Text

**Location**: Multiple keys containing "model" in descriptive text
**Examples**:

- `settings:providers.ollama.description`: "Ollama memungkinkan kamu menjalankan **model** secara lokal..."
- `settings:providers.ollama.warning`: "...bekerja terbaik dengan **model** Claude. **Model** yang kurang mampu..."
- `settings:providers.ollama.models`: `{ "one": "model", "other": "model" }`
- Many other keys throughout the settings

**Current Indonesian Usage**: "model" (English loanword, used as noun)
**Impact**: High - The word "model" appears frequently throughout the UI and documentation text

**Why It Was Missed**:

- "Model" in the context of LLMs is a relatively new technical term (post-2020)
- It's commonly used as a loanword in Indonesian tech contexts
- The initial review focused on UI labels, not descriptive text
- Assumed it was acceptable as a technical term (similar to "API", "URL")

**Current State Across Languages**:

| Language            | Translation Pattern                                     | Example from `ollama.description`                         |
| ------------------- | ------------------------------------------------------- | --------------------------------------------------------- |
| **German (de)**     | ✅ Translated: "Modell" (singular), "Modelle" (plural)  | "Modelle lokal auf deinem Computer auszuführen"           |
| **Spanish (es)**    | ✅ Translated: "modelo" (singular), "modelos" (plural)  | "ejecutar modelos localmente"                             |
| **French (fr)**     | ✅ Translated: "modèle" (singular), "modèles" (plural)  | "exécuter des modèles localement"                         |
| **Italian (it)**    | ✅ Translated: "modello" (singular), "modelli" (plural) | "eseguire modelli localmente"                             |
| **Russian (ru)**    | ✅ Translated: "модель" (singular), "модели" (plural)   | "запускать модели локально"                               |
| **Japanese (ja)**   | ✅ Translated: "モデル" (katakana)                      | "モデルを実行できます"                                    |
| **Korean (ko)**     | ✅ Translated: "모델"                                   | "모델을 실행할 수 있습니다"                               |
| **Chinese (zh-CN)** | ✅ Translated: "模型"                                   | "运行模型"                                                |
| **Chinese (zh-TW)** | ✅ Translated: "模型"                                   | "運行模型"                                                |
| **Polish (pl)**     | ⚠️ **Kept as "model"** (English loanword)               | "uruchamianie modeli" (but uses "model" in plural object) |
| **Dutch (nl)**      | ⚠️ **Kept as "model"** (English loanword)               | "modellen lokaal op je computer draaien"                  |
| **Indonesian (id)** | ⚠️ **Kept as "model"** (English loanword)               | "menjalankan model secara lokal"                          |

**Analysis**:

- **Most languages translate "model"**: 12 out of 17 non-English locales translate it
- **Only 3 languages keep it as loanword**: Polish, Dutch, Indonesian
- **Pattern**: Romance languages (Spanish, French, Italian) and Germanic languages (German) translate it
- **Asian languages**: All translate it (Japanese, Korean, Chinese)
- **Slavic languages**: Mixed - Russian translates it, Polish keeps it

**Proposed Translation for Indonesian**:

- **Option A**: Keep "model" (current approach)

    - **Pros**: Widely recognized in Indonesian tech community, matches Polish/Dutch pattern, "model" in LLM context is a very new term (post-2020)
    - **Cons**: Inconsistent with majority of languages (12 out of 17 translate it)

- **Option B**: Translate to native Indonesian term (if one exists)
    - **Research needed**: Is there a native Indonesian term for "AI model" or "language model"?
    - Possible terms: "pola" (pattern), "acuan" (reference), but these don't capture the LLM meaning
    - **Challenge**: "Model" in the LLM context is a very new concept, and Indonesian may not have an established native term yet

**✅ APPROVED**: **Keep "model" as a loanword**

**Rationale**:

1. **Consistency**: Already following the pattern set by Dutch and Polish
2. **Accuracy**: "Model" has become the de facto term in the Indonesian AI/Tech community (e.g., "Model Bahasa Besar" for LLM)
3. **Risk Mitigation**: Translating "model" into a more native term like "acuan" would confuse users, as it loses the specific LLM context
4. "Model" in the LLM context is a very new term (post-2020)
5. Indonesian tech community commonly uses English loanwords for new technical concepts
6. Similar to how "API", "URL", "ID" are kept as acronyms
7. The word "model" is already widely used in Indonesian tech documentation

**Policy Decision**: ✅ **APPROVED** - Keep "model" as loanword. This is now documented as the standard approach for Indonesian translations of new technical terms in the LLM/AI context.

**Action Required**:

- ✅ Decision made - no changes needed
- ✅ Already consistent across all keys (all use "model")
- ✅ Policy documented in this plan

---

### Summary of Missing Translations

| Issue            | Key                                         | Current (id)               | Approved (id)        | Priority | Status      |
| ---------------- | ------------------------------------------- | -------------------------- | -------------------- | -------- | ----------- |
| 1. Provider API  | `settings:providers.apiProvider`            | "Provider API"             | ✅ "Penyedia API"    | High     | ✅ Approved |
| 2. ID Acronym    | `settings:providers.ollama.modelId`         | "ID Model"                 | ✅ Keep "ID Model"   | Medium   | ✅ Approved |
| 3. Optional Word | Multiple keys with "(opsional)"             | "opsional"                 | ✅ Keep "opsional"   | Medium   | ✅ Approved |
| 4. Rate Limit    | `settings:providers.rateLimitSeconds.label` | "Rate limit"               | ✅ "Pembatasan laju" | Medium   | ✅ Approved |
| 5. Model Word    | Multiple keys with "model"                  | "model" (English loanword) | ✅ Keep "model"      | High     | ✅ Approved |

---

### Root Cause Analysis

**Why These Were Missed**:

1. **Scope Limitation**: Initial review focused on Ollama-specific settings (`settings:providers.ollama.*`), but missed shared provider keys (`settings:providers.apiProvider`, `settings:providers.rateLimitSeconds.*`)

2. **Component Separation**: Some keys are used in different components:

    - `apiProvider` is in `ApiOptions.tsx` (shared across all providers)
    - `rateLimitSeconds` is in `RateLimitSecondsControl.tsx` (advanced settings)
    - Initial review focused on `Ollama.tsx` component

3. **Acronym Assumption**: Assumed technical acronyms like "ID" and "API" would remain in English (common practice), but didn't verify consistency across languages

4. **Loanword Acceptance**: "opsional" is a widely accepted loanword in Indonesian tech interfaces, so it wasn't flagged as needing translation

5. **Incomplete UI Testing**: Initial testing may not have covered all sections (advanced settings, provider selection dropdown)

6. **New Technical Term Assumption**: "model" in the LLM context is a very new term (post-2020), and was assumed to be acceptable as a loanword (similar to "API", "URL"). However, most languages (12 out of 17) do translate it, making Indonesian inconsistent.

---

### Recommendations for Complete Translation

**Immediate Actions Required**:

1. **Audit All Provider-Related Keys**:

    - Review all keys under `settings:providers.*` (not just `ollama.*`)
    - Check shared components: `ApiOptions.tsx`, `RateLimitSecondsControl.tsx`, etc.
    - Verify consistency across all 17 non-English locales

2. **Technical Acronym Policy**:

    - **Decision Needed**: Should "ID" and "API" be translated or kept as acronyms?
    - **Recommendation**: Keep "ID" and "API" as acronyms (universally recognized in tech)
    - **Exception**: If a language has a strong preference for full translation (like Turkish "Model Kimliği"), follow that pattern

3. **Loanword Policy**:

    - **Decision Needed**: Should loanwords like "opsional" be replaced with native terms?
    - **Recommendation**: Keep widely accepted tech loanwords for consistency with user expectations
    - **Exception**: If native term is more commonly used in software interfaces, use native term

4. **Comprehensive UI Testing**:

    - Test all sections: main settings, advanced settings, provider selection
    - Use i18n-ally extension to identify all untranslated keys
    - Verify no English words appear in non-English locales (except technical acronyms)

5. **Update Automated Tests**:
    - Enhance `ollama-coverage.spec.ts` to check for English words in non-English locales
    - Add exceptions for technical acronyms ("ID", "API", "URL") if policy is to keep them
    - Add check for `settings:providers.apiProvider` and `settings:providers.rateLimitSeconds.label`

---

### Implementation Checklist (Pending Review)

**After Review Approval**:

- [ ] Update `settings:providers.apiProvider` for Indonesian: "Provider API" → "Penyedia API"
- [ ] Update `settings:providers.apiProvider` for Dutch: "API-provider" → "API-aanbieder" (if needed)
- [ ] **Decision**: Keep "ID Model" or change to "Identitas Model" for Indonesian
- [ ] **Decision**: Keep "opsional" or change to "pilihan" for Indonesian
- [ ] Update `settings:providers.rateLimitSeconds.label` for Indonesian: "Rate limit" → "Pembatasan laju" or "Batas kecepatan"
- [ ] Verify consistency with `chat.json` usage (line 155: "Pembatasan rate")
- [ ] **Research & Decision**: Should "model" be translated or kept as loanword in Indonesian?
    - [ ] Research Indonesian AI/LLM documentation for standard terminology
    - [ ] Consult with native Indonesian speaker familiar with tech/AI terminology
    - [ ] Document final decision as translation policy
    - [ ] If keeping "model", ensure consistency across all keys
    - [ ] If translating, identify appropriate Indonesian term and update all occurrences
- [ ] Update automated test to catch these issues in the future
- [ ] Re-test UI with Indonesian language to verify all English words are gone (except approved loanwords/acronyms)

---

## Prerequisites

1. **Install i18n-ally VS Code extension** (if not already installed)

    - Extension ID: `Lokalise.i18n-ally`
    - Used for sorting keys and maintaining consistency

2. **Verify current state**:
    - English file (`en/settings.json`) has all 29 keys ✅
    - React component (`Ollama.tsx`) already uses correct translation methods ✅
    - No React component changes needed ✅
    - **NEW**: Review proposed translations for "Provider", "Base", and "Model" terms above

## File Structure

All translation files are located at:

```
webview-ui/src/i18n/locales/{locale}/settings.json
```

The Ollama provider settings are nested under:

```json
{
	"providers": {
		"ollama": {
			// Existing keys (8 keys)
			// NEW KEYS TO ADD (29 keys)
		}
	}
}
```

## Risk Reduction: Automated Testing

### Automated Test for i18n Coverage

A test file has been created to prevent future regressions: `webview-ui/src/i18n/__tests__/ollama-coverage.spec.ts`

**Test Coverage**:

- ✅ Verifies all English keys exist in each locale (17 non-English locales)
- ✅ Validates JSON syntax and structure
- ✅ Ensures "models" key uses plural object format
- ✅ Validates Russian/Polish have full plural forms (one, few, many, other)
- ✅ Checks key count consistency across all locales
- ⚠️ **Important**: Test checks ALL locales - will fail until ALL are complete (intentional for CI correctness)

**Benefits**:

- Prevents "the one that broke the French build" scenarios
- Catches missing keys automatically in CI/CD
- Provides safety net for future contributors
- Idiomatic for VS Code extensions

**Running the Test**:

```bash
cd webview-ui
npm test ollama-coverage
# Or run all tests
npm test
```

## Implementation Strategy Refinements

### 1. Template First Approach (Risk Mitigation)

Before adding translations to all 17 non-English files, we'll do a "Dry Run" with English and German files first:

- **English (en)**: Update to plural object format, serves as template
- **German (de)**: Test case since German often has the longest strings
- **Purpose**: Immediately identify if new buttons or table headers break UI layout
- **Validation**: Test UI rendering with German before proceeding to other languages
- **Benefit**: Catch UI issues early before investing time in all languages
- **Note**: The automated test will still fail during dry run (it checks all locales). Use i18n-ally Progress tab to track completion instead.

### 2. Pluralization Future-Proofing

The `"models"` key will be changed from a simple string to a plural object:

- **Current**: `"models": "models"` (simple string)
- **New Format (Most Languages)**:
    ```json
    "models": {
      "one": "model",
      "other": "models"
    }
    ```
- **New Format (Russian)**:
    ```json
    "models": {
      "one": "модель",
      "few": "модели",
      "many": "моделей",
      "other": "моделей"
    }
    ```
- **New Format (Polish)**:
    ```json
    "models": {
      "one": "model",
      "few": "modele",
      "many": "modeli",
      "other": "modeli"
    }
    ```
- **Reason**: Russian and Polish require different forms (1, 2-4, 5+)
- **Benefit**: Makes the repo future-proof even if we only use "other" form now
- **Note**: React component **must** be updated to use pluralization with `{count}` parameter (see Step 2.6 - REQUIRED)
- **i18next Support**: i18next automatically handles plural forms when count is provided

### 3. i18n-ally Validation

Use i18n-ally extension's "Review" or "Progress" tab:

- Shows completion percentage for Ollama namespace across all languages
- Identifies missing keys automatically
- Provides visual progress tracking

## Detailed Implementation Steps

### Step 1: Prepare Workspace

**Action**: Open VS Code and ensure i18n-ally extension is active

**Files**: N/A

**Commands**:

```bash
# Navigate to project root
cd /home/das/Downloads/Roo-Code

# Verify i18n-ally extension is installed
# Check VS Code Extensions panel for "i18n Ally"
```

---

### Step 2: Backup Current State (Optional but Recommended)

**Action**: Create a backup or ensure you're on a feature branch

**Files**: N/A

**Commands**:

```bash
# Create a new branch for i18n work
git checkout -b feat/ollama-i18n-translations

# Or ensure you're on your working branch
git status
```

---

### Step 2.5: Update English File with Plural Object (Template)

**File**: `webview-ui/src/i18n/locales/en/settings.json`

**Location**: Find `"models": "models"` key (around line 455)

**Action**:

1. Change from simple string to plural object format
2. This serves as the template for all other languages

**Current format** (line ~455):

```json
"models": "models",
```

**New format**:

```json
"models": {
  "one": "model",
  "other": "models"
},
```

**Validation**:

- [ ] JSON syntax valid
- [ ] No trailing comma after object
- [ ] 2-space indentation

**Note**: This change makes the structure future-proof for languages with complex plural rules.

---

### Step 2.6: Update React Component for Pluralization (REQUIRED)

**File**: `webview-ui/src/components/settings/providers/Ollama.tsx`

**Location**: Lines 251-252 and 335-336

**Why Required**: Once "models" is converted to a plural object, calling `t("...models")` without `{count}` will fail. i18next requires the `count` parameter to select the correct plural form (one/other/few/many).

**Current code** (Line 251-252):

```tsx
{t("settings:providers.ollama.toolsSupport")} ({modelsWithTools.length}{" "}
{t("settings:providers.ollama.models")})
```

**Updated code** (Line 251-252):

```tsx
{t("settings:providers.ollama.toolsSupport")} ({modelsWithTools.length}{" "}
{t("settings:providers.ollama.models", { count: modelsWithTools.length })})
```

**Current code** (Line 335-336):

```tsx
{t("settings:providers.ollama.noToolsSupport")} ({modelsWithoutTools.length}{" "}
{t("settings:providers.ollama.models")})
```

**Updated code** (Line 335-336):

```tsx
{t("settings:providers.ollama.noToolsSupport")} ({modelsWithoutTools.length}{" "}
{t("settings:providers.ollama.models", { count: modelsWithoutTools.length })})
```

**Action**:

1. Update both usage sites to include `{ count: ... }` parameter
2. Build and verify UI displays correctly with English
3. Test with different counts (0, 1, 2, 5+) to verify plural forms work

**Validation**:

- [ ] Both usage sites updated (lines 251-252 and 335-336)
- [ ] UI displays "9 models" correctly (plural)
- [ ] UI displays "1 model" correctly (singular, if count is 1)
- [ ] No console errors
- [ ] Translation key resolves properly (not showing key name or [object Object])

---

### Step 3: Template First - Add Translations to German (de) - DRY RUN

**File**: `webview-ui/src/i18n/locales/de/settings.json`

**Location**: Find the `"ollama"` object (around line 430-439)

**Action**:

1. Open the file
2. Locate the closing brace `}` after `"warning"` key
3. Add a comma after `"warning"` value
4. Insert all 29 new keys (see translation block below)
5. **IMPORTANT**: Use plural object format for `"models"` key
6. Run `i18n-ally.sortKeys` command in VS Code
7. Verify JSON syntax (no trailing commas, 2-space indentation)
8. **Test UI**: Build extension and test with German language to check for layout issues

**Line numbers**: Approximately after line 438

**Translation block to add** (note plural object for "models"):

```json
"test": "Testen",
"testing": "Verbindung wird geprüft...",
"refreshModels": "Modelle aktualisieren",
"refreshing": "Aktualisiere...",
"connectionSettings": "Verbindungseinstellungen",
"toolsSupport": "Tools-Unterstützung",
"noToolsSupport": "Keine Tools-Unterstützung",
"models": {
  "one": "Modell",
  "other": "Modelle"
},
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
  "sizeFormatting": {
    "gb": "GB",
    "mb": "MB"
  }
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

**Validation**:

- [ ] JSON syntax valid (no errors)
- [ ] No trailing commas
- [ ] 2-space indentation
- [ ] All 29 keys present
- [ ] Keys sorted (after running i18n-ally.sortKeys)
- [ ] **UI TEST**: Build extension, switch to German, verify:
    - [ ] Button text fits (especially "Modelle aktualisieren")
    - [ ] Table headers display correctly
    - [ ] No text overflow or layout breaks
    - [ ] All text is visible and readable

**If UI issues found**: Adjust translations or UI component before proceeding to other languages.

---

### Step 3.5: Validate with i18n-ally Progress Tab

**Action**: Use i18n-ally extension to check progress

**Steps**:

1. Open i18n-ally extension panel in VS Code
2. Navigate to "Progress" or "Review" tab
3. Filter by namespace: `settings`
4. Filter by key pattern: `providers.ollama.*`
5. Check completion percentage for German (de)
6. Verify all 29 keys show as "translated" for German

**Expected Result**:

- German (de): ~100% complete for Ollama namespace (after Step 3)
- Other languages: Will show missing keys (expected)
- **Note**: Use i18n-ally Progress tab to track completion during development. The automated test will only pass once ALL locales are complete.

**Validation**:

- [ ] i18n-ally shows German as complete
- [ ] All 29 keys visible in progress view
- [ ] No missing key warnings for German

---

### Step 4: Add Translations to Catalan (ca)

**File**: `webview-ui/src/i18n/locales/ca/settings.json`

**Location**: Find the `"ollama"` object (should be around line 430-439 based on other language files)

**Action**:

1. Open the file
2. Locate the closing brace `}` after `"warning"` key
3. Add a comma after `"warning"` value
4. Insert all 29 new keys (see translation block below)
5. **IMPORTANT**: Use plural object format for `"models"` key (see Step 2.5)
6. Run `i18n-ally.sortKeys` command in VS Code
7. Verify JSON syntax (no trailing commas, 2-space indentation)

**Line numbers**: Approximately after line 438 (after `"warning"` key)

**Translation block to add**:

```json
"test": "Provar",
"testing": "Provant...",
"refreshModels": "Actualitzar models",
"refreshing": "Actualitzant...",
"connectionSettings": "Configuració de connexió",
"toolsSupport": "Suport d'eines",
"noToolsSupport": "Sense suport d'eines",
"models": {
  "one": "model",
  "other": "models"
},
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
  "sizeFormatting": {
    "gb": "GB",
    "mb": "MB"
  }
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

**Validation**:

- [ ] JSON syntax valid (no errors)
- [ ] No trailing commas
- [ ] 2-space indentation
- [ ] All 29 keys present
- [ ] Keys sorted (after running i18n-ally.sortKeys)

---

### Step 4: Add Translations to German (de)

**File**: `webview-ui/src/i18n/locales/de/settings.json`

**Location**: Find the `"ollama"` object (around line 430-439)

**Action**: Same as Step 3, but use German translations

**Line numbers**: Approximately after line 438

**Translation block to add**:

```json
"test": "Testen",
"testing": "Verbindung wird geprüft...",
"refreshModels": "Modelle aktualisieren",
"refreshing": "Aktualisiere...",
"connectionSettings": "Verbindungseinstellungen",
"toolsSupport": "Tools-Unterstützung",
"noToolsSupport": "Keine Tools-Unterstützung",
"models": {
  "one": "Modell",
  "other": "Modelle"
},
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
  "sizeFormatting": {
    "gb": "GB",
    "mb": "MB"
  }
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

**Note**: Uses "Verbindung wird geprüft..." for professional IDE feel (refined translation)

**Validation**: Same checklist as Step 3

---

### Step 5: Add Translations to Spanish (es)

**File**: `webview-ui/src/i18n/locales/es/settings.json`

**Location**: Find the `"ollama"` object (around line 430-439)

**Action**: Same as Step 3, but use Spanish translations

**Line numbers**: Approximately after line 438

**Translation block to add**:

```json
"test": "Probar",
"testing": "Probando...",
"refreshModels": "Actualizar modelos",
"refreshing": "Actualizando...",
"connectionSettings": "Configuración de conexión",
"toolsSupport": "Soporte de herramientas",
"noToolsSupport": "Sin soporte de herramientas",
"models": {
  "one": "modelo",
  "other": "modelos"
},
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
  "sizeFormatting": {
    "gb": "GB",
    "mb": "MB"
  }
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

**Note**: Uses "Tiempo de espera de la solicitud" with article "la" (refined translation)

**Validation**: Same checklist as Step 3

---

### Step 6: Add Translations to French (fr)

**File**: `webview-ui/src/i18n/locales/fr/settings.json`

**Location**: Find the `"ollama"` object (around line 430-439)

**Action**: Same as Step 3, but use French translations

**Line numbers**: Approximately after line 438

**Translation block to add**:

```json
"test": "Tester",
"testing": "Test en cours...",
"refreshModels": "Actualiser les modèles",
"refreshing": "Actualisation...",
"connectionSettings": "Paramètres de connexion",
"toolsSupport": "Support des outils",
"noToolsSupport": "Pas de support des outils",
"models": {
  "one": "modèle",
  "other": "modèles"
},
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
  "sizeFormatting": {
    "gb": "Go",
    "mb": "Mo"
  }
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

**Note**: Uses "Go" and "Mo" (French technical standards - confirmed correct)

**Validation**: Same checklist as Step 3

---

### Step 7: Add Translations to Hindi (hi)

**File**: `webview-ui/src/i18n/locales/hi/settings.json`

**Location**: Find the `"ollama"` object (around line 430-439)

**Action**: Same as Step 3, but use Hindi translations

**Line numbers**: Approximately after line 438

**Translation block to add**:

```json
"test": "परीक्षण",
"testing": "परीक्षण कर रहे हैं...",
"refreshModels": "मॉडल ताज़ा करें",
"refreshing": "ताज़ा कर रहे हैं...",
"connectionSettings": "कनेक्शन सेटिंग्स",
"toolsSupport": "टूल्स सपोर्ट",
"noToolsSupport": "कोई टूल्स सपोर्ट नहीं",
"models": {
  "one": "मॉडल",
  "other": "मॉडल"
},
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
  "sizeFormatting": {
    "gb": "GB",
    "mb": "MB"
  }
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

**Validation**: Same checklist as Step 3

---

### Step 8: Add Translations to Indonesian (id)

**File**: `webview-ui/src/i18n/locales/id/settings.json`

**Location**: Find the `"ollama"` object (around line 430-439)

**Action**: Same as Step 3, but use Indonesian translations

**Line numbers**: Approximately after line 438

**Translation block to add**:

```json
"test": "Uji",
"testing": "Menguji...",
"refreshModels": "Muat ulang model",
"refreshing": "Memuat ulang...",
"connectionSettings": "Pengaturan koneksi",
"toolsSupport": "Dukungan alat",
"noToolsSupport": "Tidak ada dukungan alat",
"models": {
  "one": "model",
  "other": "model"
},
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
  "sizeFormatting": {
    "gb": "GB",
    "mb": "MB"
  }
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

**Validation**: Same checklist as Step 3

---

### Step 9: Add Translations to Italian (it)

**File**: `webview-ui/src/i18n/locales/it/settings.json`

**Location**: Find the `"ollama"` object (around line 430-439)

**Action**: Same as Step 3, but use Italian translations

**Line numbers**: Approximately after line 438

**Translation block to add**:

```json
"test": "Testa",
"testing": "Test in corso...",
"refreshModels": "Aggiorna modelli",
"refreshing": "Aggiornamento...",
"connectionSettings": "Impostazioni di connessione",
"toolsSupport": "Supporto strumenti",
"noToolsSupport": "Nessun supporto strumenti",
"models": {
  "one": "modello",
  "other": "modelli"
},
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
  "sizeFormatting": {
    "gb": "GB",
    "mb": "MB"
  }
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

**Validation**: Same checklist as Step 3

---

### Step 10: Add Translations to Japanese (ja)

**File**: `webview-ui/src/i18n/locales/ja/settings.json`

**Location**: Find the `"ollama"` object (around line 430-439)

**Action**: Same as Step 3, but use Japanese translations

**Line numbers**: Approximately after line 438

**Translation block to add**:

```json
"test": "テスト",
"testing": "テスト中...",
"refreshModels": "モデルを更新",
"refreshing": "更新中...",
"connectionSettings": "接続設定",
"toolsSupport": "ツールサポート",
"noToolsSupport": "ツールサポートなし",
"models": {
  "one": "モデル",
  "other": "モデル"
},
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
  "sizeFormatting": {
    "gb": "GB",
    "mb": "MB"
  }
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

**Validation**: Same checklist as Step 3

---

### Step 11: Add Translations to Korean (ko)

**File**: `webview-ui/src/i18n/locales/ko/settings.json`

**Location**: Find the `"ollama"` object (around line 430-439)

**Action**: Same as Step 3, but use Korean translations

**Line numbers**: Approximately after line 438

**Translation block to add**:

```json
"test": "테스트",
"testing": "테스트 중...",
"refreshModels": "모델 새로고침",
"refreshing": "새로고침 중...",
"connectionSettings": "연결 설정",
"toolsSupport": "도구 지원",
"noToolsSupport": "도구 지원 없음",
"models": {
  "one": "모델",
  "other": "모델"
},
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
  "sizeFormatting": {
    "gb": "GB",
    "mb": "MB"
  }
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

**Validation**: Same checklist as Step 3

---

### Step 12: Add Translations to Dutch (nl)

**File**: `webview-ui/src/i18n/locales/nl/settings.json`

**Location**: Find the `"ollama"` object (around line 430-439)

**Action**: Same as Step 3, but use Dutch translations

**Line numbers**: Approximately after line 438

**Translation block to add**:

```json
"test": "Testen",
"testing": "Testen...",
"refreshModels": "Modellen vernieuwen",
"refreshing": "Vernieuwen...",
"connectionSettings": "Verbindingsinstellingen",
"toolsSupport": "Toolondersteuning",
"noToolsSupport": "Geen toolondersteuning",
"models": {
  "one": "model",
  "other": "modellen"
},
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
  "sizeFormatting": {
    "gb": "GB",
    "mb": "MB"
  }
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

**Validation**: Same checklist as Step 3

---

### Step 13: Add Translations to Polish (pl)

**File**: `webview-ui/src/i18n/locales/pl/settings.json`

**Location**: Find the `"ollama"` object (around line 430-439)

**Action**: Same as Step 3, but use Polish translations

**Line numbers**: Approximately after line 438

**Translation block to add**:

```json
"test": "Testuj",
"testing": "Testowanie...",
"refreshModels": "Odśwież modele",
"refreshing": "Odświeżanie...",
"connectionSettings": "Ustawienia połączenia",
"toolsSupport": "Obsługa narzędzi",
"noToolsSupport": "Brak obsługi narzędzi",
"models": {
  "one": "model",
  "few": "modele",
  "many": "modeli",
  "other": "modeli"
},
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
  "sizeFormatting": {
    "gb": "GB",
    "mb": "MB"
  }
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

**Validation**: Same checklist as Step 3

---

### Step 14: Add Translations to Portuguese (Brazil) (pt-BR)

**File**: `webview-ui/src/i18n/locales/pt-BR/settings.json`

**Location**: Find the `"ollama"` object (around line 430-439)

**Action**: Same as Step 3, but use Portuguese (Brazil) translations

**Line numbers**: Approximately after line 438

**Translation block to add**:

```json
"test": "Testar",
"testing": "Testando...",
"refreshModels": "Atualizar modelos",
"refreshing": "Atualizando...",
"connectionSettings": "Configurações de conexão",
"toolsSupport": "Suporte a ferramentas",
"noToolsSupport": "Sem suporte a ferramentas",
"models": {
  "one": "modelo",
  "other": "modelos"
},
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
  "sizeFormatting": {
    "gb": "GB",
    "mb": "MB"
  }
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

**Validation**: Same checklist as Step 3

---

### Step 15: Add Translations to Russian (ru)

**File**: `webview-ui/src/i18n/locales/ru/settings.json`

**Location**: Find the `"ollama"` object (around line 430-439)

**Action**: Same as Step 3, but use Russian translations

**Line numbers**: Approximately after line 438

**Translation block to add**:

```json
"test": "Тест",
"testing": "Тестирование...",
"refreshModels": "Обновить модели",
"refreshing": "Обновляем...",
"connectionSettings": "Настройки подключения",
"toolsSupport": "Поддержка инструментов",
"noToolsSupport": "Без поддержки инструментов",
"models": {
  "one": "модель",
  "few": "модели",
  "many": "моделей",
  "other": "моделей"
},
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
  "sizeFormatting": {
    "gb": "ГБ",
    "mb": "МБ"
  }
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

**Note**: Uses "Обновляем..." (verb form for active processes - refined translation)

**Validation**: Same checklist as Step 3

---

### Step 16: Add Translations to Turkish (tr)

**File**: `webview-ui/src/i18n/locales/tr/settings.json`

**Location**: Find the `"ollama"` object (around line 430-439)

**Action**: Same as Step 3, but use Turkish translations

**Line numbers**: Approximately after line 438

**Translation block to add**:

```json
"test": "Test",
"testing": "Test ediliyor...",
"refreshModels": "Modelleri yenile",
"refreshing": "Yenileniyor...",
"connectionSettings": "Bağlantı ayarları",
"toolsSupport": "Araç desteği",
"noToolsSupport": "Araç desteği yok",
"models": {
  "one": "model",
  "other": "modeller"
},
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
  "sizeFormatting": {
    "gb": "GB",
    "mb": "MB"
  }
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

**Validation**: Same checklist as Step 3

---

### Step 17: Add Translations to Vietnamese (vi)

**File**: `webview-ui/src/i18n/locales/vi/settings.json`

**Location**: Find the `"ollama"` object (around line 430-439)

**Action**: Same as Step 3, but use Vietnamese translations

**Line numbers**: Approximately after line 438

**Translation block to add**:

```json
"test": "Kiểm tra",
"testing": "Đang kiểm tra...",
"refreshModels": "Làm mới mô hình",
"refreshing": "Đang làm mới...",
"connectionSettings": "Cài đặt kết nối",
"toolsSupport": "Hỗ trợ công cụ",
"noToolsSupport": "Không hỗ trợ công cụ",
"models": {
  "one": "mô hình",
  "other": "mô hình"
},
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
  "sizeFormatting": {
    "gb": "GB",
    "mb": "MB"
  }
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

**Validation**: Same checklist as Step 3

---

### Step 18: Add Translations to Chinese Simplified (zh-CN)

**File**: `webview-ui/src/i18n/locales/zh-CN/settings.json`

**Location**: Find the `"ollama"` object (around line 430-439)

**Action**: Same as Step 3, but use Chinese Simplified translations

**Line numbers**: Approximately after line 438

**Translation block to add**:

```json
"test": "测试",
"testing": "测试中...",
"refreshModels": "刷新模型",
"refreshing": "刷新中...",
"connectionSettings": "连接设置",
"toolsSupport": "工具支持",
"noToolsSupport": "无工具支持",
"models": {
  "one": "模型",
  "other": "模型"
},
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
  "sizeFormatting": {
    "gb": "GB",
    "mb": "MB"
  }
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

**Validation**: Same checklist as Step 3

---

### Step 19: Add Translations to Chinese Traditional (zh-TW)

**File**: `webview-ui/src/i18n/locales/zh-TW/settings.json`

**Location**: Find the `"ollama"` object (around line 430-439)

**Action**: Same as Step 3, but use Chinese Traditional translations

**Line numbers**: Approximately after line 438

**Translation block to add**:

```json
"test": "測試",
"testing": "測試中...",
"refreshModels": "重新整理模型",
"refreshing": "重新整理中...",
"connectionSettings": "連線設定",
"toolsSupport": "工具支援",
"noToolsSupport": "無工具支援",
"models": {
  "one": "模型",
  "other": "模型"
},
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
  "sizeFormatting": {
    "gb": "GB",
    "mb": "MB"
  }
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

**Validation**: Same checklist as Step 3

---

## Final Validation Steps

### Step 20: Global Validation with i18n-ally

**Action**: Use i18n-ally to validate all translations

**Files**: All 17 non-English locale files

**Steps**:

1. Open i18n-ally extension panel
2. Navigate to "Progress" or "Review" tab
3. Filter by namespace: `settings`
4. Filter by key pattern: `providers.ollama.*`
5. Review completion status for all languages

**Expected Results**:

- All languages should show 100% completion for Ollama namespace
- All 29 keys should show as "translated" for each language
- No missing key warnings

**Commands**:

```bash
# Verify JSON syntax for all files
find webview-ui/src/i18n/locales -name "settings.json" -exec echo "Checking {}" \; -exec python3 -m json.tool {} > /dev/null \;

# Count keys in each file (should be 37 total: 8 existing + 29 new)
# This is a manual check - open each file and verify
```

**Validation Checklist**:

- [ ] All 17 non-English locale files updated
- [ ] All JSON files have valid syntax (no errors)
- [ ] No trailing commas in any file
- [ ] 2-space indentation throughout
- [ ] All 29 keys present in each file
- [ ] Keys sorted using i18n-ally.sortKeys
- [ ] **i18n-ally Progress tab shows 100% for all languages**
- [ ] **All keys show as "translated" in i18n-ally**
- [ ] No hardcoded strings in React component (already verified ✅)

### Step 21: Test UI Rendering

**Action**: Test the UI in different languages

**Files**:

- `webview-ui/src/components/settings/providers/Ollama.tsx` (no changes needed)

**Steps**:

1. Build the extension
2. Change VS Code language to each supported language
3. Navigate to Settings > Providers > Ollama
4. Verify all text displays correctly
5. Check for any missing translation warnings in console

**Test Checklist**:

- [ ] UI renders without errors in all languages
- [ ] All buttons display translated text
- [ ] Table headers are translated
- [ ] Help text is translated
- [ ] Connection settings section is translated
- [ ] No console errors or warnings

### Step 21.5: Add Automated Test for i18n Coverage

**File**: `webview-ui/src/i18n/__tests__/ollama-coverage.spec.ts` (NEW FILE)

**Action**: Create automated test to prevent future regressions

**Purpose**:

- Prevents "the one that broke the French build" scenarios
- Catches missing keys automatically
- Validates plural object structure
- Ensures consistent key counts across all languages
- Provides safety net for future contributors

**File Status**: ✅ Already created at `webview-ui/src/i18n/__tests__/ollama-coverage.spec.ts`

**When to Run**:

- **Before implementation**: Test will fail (expected - keys are missing in all locales)
- **During implementation**: Test will continue to fail until ALL 17 non-English locales are updated
    - The test checks ALL locales against English keys
    - Partial completion (e.g., only German updated) will still cause test failures for other locales
- **After all locales complete**: Test should pass for all languages
- **Before commit**: Must pass for all languages

**Note**: The test is designed to verify complete coverage across all locales. It will not pass with partial updates. This is intentional - it ensures CI correctness and prevents incomplete translations from being merged.

**Test Coverage**:

1. **Key Existence Test**: Verifies all English keys exist in each locale
2. **JSON Structure Test**: Validates JSON syntax and structure
3. **Plural Object Test**: Ensures "models" key uses plural object format
4. **Key Count Consistency**: Verifies all locales have same number of keys

**Validation**:

- [x] Test file created at correct location ✅
- [ ] **Before implementation**: Run test to see it fail (confirms test works - all locales missing keys)
- [ ] **During implementation**: Test will continue to fail until ALL 17 non-English locales are updated
    - This is expected behavior - the test checks all locales
    - Use i18n-ally Progress tab to track completion per locale instead
- [ ] **After all locales complete**: Run test - should pass for all languages
- [ ] Test catches missing keys (try temporarily removing a key to verify)
- [ ] Test validates plural object structure correctly

**Commands**:

```bash
# Run the specific test
cd webview-ui
npm test ollama-coverage

# Or run all tests
npm test
```

**Expected Result**:

- **During development**: Tests will fail (expected - not all locales updated yet)
- **After all 17 non-English locales are complete**: All tests pass
- **CI/CD**: Test ensures no incomplete translations are merged

---

### Step 22: Commit Changes

**Action**: Commit all translation files

**Files**: All 17 modified non-English locale files

**Commands**:

```bash
# Stage all translation files and test
git add webview-ui/src/i18n/locales/*/settings.json
git add webview-ui/src/i18n/__tests__/ollama-coverage.spec.ts

# Verify what will be committed
git status

# Run tests to ensure everything passes
cd webview-ui && npm test ollama-coverage && cd ..

# Commit with descriptive message
git commit -m "feat(i18n): add Ollama provider translations for all 17 non-English locales

- Add 29 missing translation keys to all 17 non-English locale files
- Convert 'models' key to plural object format for future-proofing (Russian/Polish support)
- Includes button labels, table headers, help text, and connection settings
- Translations reviewed and refined for professional IDE feel
- Template-first approach: tested with German (longest strings) before full rollout
- All keys sorted using i18n-ally.sortKeys
- JSON formatting compliant (2-space indentation, no trailing commas)
- Validated with i18n-ally Progress tab (100% completion per language)
- Add automated test (ollama-coverage.spec.ts) to prevent future regressions

Refs: OLLAMA_I18N_DESIGN.md, OLLAMA_I18N_DESIGN_IMPLEMENTATION_PLAN.md"
```

---

## Implementation Summary

### Files to Modify

1. `webview-ui/src/i18n/locales/ca/settings.json`
2. `webview-ui/src/i18n/locales/de/settings.json`
3. `webview-ui/src/i18n/locales/es/settings.json`
4. `webview-ui/src/i18n/locales/fr/settings.json`
5. `webview-ui/src/i18n/locales/hi/settings.json`
6. `webview-ui/src/i18n/locales/id/settings.json`
7. `webview-ui/src/i18n/locales/it/settings.json`
8. `webview-ui/src/i18n/locales/ja/settings.json`
9. `webview-ui/src/i18n/locales/ko/settings.json`
10. `webview-ui/src/i18n/locales/nl/settings.json`
11. `webview-ui/src/i18n/locales/pl/settings.json`
12. `webview-ui/src/i18n/locales/pt-BR/settings.json`
13. `webview-ui/src/i18n/locales/ru/settings.json`
14. `webview-ui/src/i18n/locales/tr/settings.json`
15. `webview-ui/src/i18n/locales/vi/settings.json`
16. `webview-ui/src/i18n/locales/zh-CN/settings.json`
17. `webview-ui/src/i18n/locales/zh-TW/settings.json`

### Files to Modify (Updated)

- `webview-ui/src/i18n/locales/en/settings.json` - Update "models" to plural object format
- `webview-ui/src/components/settings/providers/Ollama.tsx` - **REQUIRED**: Update for pluralization (see Step 2.6)
- `webview-ui/src/i18n/__tests__/ollama-coverage.spec.ts` - NEW: Automated test for i18n coverage

### Files NOT to Modify

- No files are excluded - all need updates

### Key Statistics

- **Total files to modify**: 18 (17 non-English + English for plural object update)
- **New test file**: 1 (`ollama-coverage.spec.ts`)
- **Keys to add per file**: 29 (28 new + 1 updated to plural object)
- **Total keys to add**: 493 (29 × 17 non-English locales)
- **Estimated time**: 2-4 hours
- **React component changes**: 1 (REQUIRED - pluralization support - see Step 2.6)
- **Pluralization**: All languages use plural object format for "models" key
- **Test coverage**: Automated regression prevention for all 17 non-English locales

---

## Troubleshooting

### Common Issues

1. **JSON Syntax Error**

    - **Symptom**: File won't parse, VS Code shows red squiggles
    - **Solution**: Check for missing commas, extra commas, unmatched braces
    - **Tool**: Use `python3 -m json.tool filename.json` to validate

2. **Trailing Comma Error**

    - **Symptom**: CI/CD fails with JSON parsing error
    - **Solution**: Remove trailing comma after last key in object
    - **Prevention**: Always run i18n-ally.sortKeys which handles this

3. **Key Not Found in UI**

    - **Symptom**: UI shows translation key instead of text
    - **Solution**: Verify key name matches exactly (case-sensitive)
    - **Check**: Compare with English file key names

4. **Keys Not Sorted**

    - **Symptom**: Keys appear in wrong order
    - **Solution**: Run `i18n-ally.sortKeys` command in VS Code
    - **Location**: Right-click in JSON file → "Sort Keys" or use command palette

5. **Pluralization Not Working**

    - **Symptom**: UI shows "models" key or object structure instead of text
    - **Solution**: Verify React component passes `count` parameter: `t("settings:providers.ollama.models", { count: modelsWithTools.length })`
    - **Check**: Ensure plural object format is correct in translation file
    - **Note**: i18next automatically selects correct plural form based on count

6. **i18n-ally Not Showing Progress**
    - **Symptom**: Progress tab shows 0% or doesn't detect keys
    - **Solution**:
        - Reload VS Code window
        - Check i18n-ally configuration in `.vscode/settings.json`
        - Verify file paths match i18n-ally's expected structure
        - Check that namespace is correctly set to `settings`

---

## Success Criteria

- [x] All 18 locale files updated (17 non-English + English plural object)
- [ ] All JSON files valid (no syntax errors)
- [ ] No trailing commas
- [ ] 2-space indentation throughout
- [ ] All 29 keys present in each file
- [ ] `"models"` key uses plural object format in all files
- [ ] Russian and Polish have full plural forms (one, few, many, other)
- [ ] Keys sorted consistently (i18n-ally.sortKeys)
- [ ] **i18n-ally Progress tab shows 100% for all languages**
- [ ] **Template-first validation: German UI tested and verified**
- [ ] UI renders correctly in all languages
- [ ] No console errors or warnings
- [ ] Pluralization works correctly (React component updated with count parameter)
- [ ] **Automated test (ollama-coverage.spec.ts) passes**
- [ ] **Test catches missing keys (verified by temporarily removing a key)**
- [ ] **Phase 3 QA - Visual Layout Verification**:
    - [ ] Verify "Pembatasan laju" fits in Advanced Settings section without layout issues (Indonesian)
    - [ ] Verify "Penyedia API" fits in provider selection dropdown (Indonesian)
    - [ ] Test all provider settings pages in Indonesian mode to ensure consistency
    - [ ] Verify sidebar width with "Aanbieders" (Dutch) - no overflow
    - [ ] Verify sidebar width with "Penyedia" (Indonesian) - no overflow
    - [ ] Check Advanced Settings section for all languages with longer terms
- [ ] CI/CD pipeline passes

---

**Document Version**: 1.0
**Last Updated**: 2024
**Based on**: OLLAMA_I18N_DESIGN.md
**Status**: Ready for Review and Implementation
