# vue-phone-input-base

Instructions for AI agents working with the repository.

---

## 0) Quick facts about this repository

* Purpose: a **Vue 3** phone input component with slots and built-in phone-number logic powered by **libphonenumber-js**. This repository is a fork of `base-vue-phone-input` (see README).
* Upstream:
    * Original project: `SidVerson/base-vue-phone-input`
    * This fork repackages and stabilizes the component as `vue-phone-input-base`.
* Stack: `Vue 3`, `TypeScript`, `Vite` (library build), `libphonenumber-js`.
* Core entities:
    * `PhoneInput` — main exported component (default export)
    * `src/components/PhoneInput.vue` — orchestration (state, emits, slots, UI)
    * `src/helpers/use-libphonenumber.ts` — adapter layer (parse, validation, AsYouType, Results)
    * `src/helpers/use-phone-input.ts` — sanitize, countries/locales, IP lookup, list helpers
    * `src/helpers/types.ts` — public types (`Results`, `Country`, `PhoneNumberDisplayFormat`, ...)
* Fork-specific behavior:
    * `phoneNumberDisplayFormat: 'national' | 'international'` controls the display format for **valid** numbers.
* Runtime I/O:
    * Optional IP-based country detection via `fetch('https://ipwho.is')`
    * Lazy loading of libphonenumber examples (`examples.mobile.json`) for placeholders

> Agents: before making changes, read `README.md`, `package.json`, and the generated `dist/*.d.ts` (props/emits/types are part of the public contract).

---

## 1) Agent protocol

1. **Understand the task:**

    * Create a concise plan (3–7 steps) and follow it
2. **Minimal diff:**

    * Modify only the files and sections required for the task.  
      ❌ No mass refactoring or formatting changes across unrelated code.
3. **Maintain public API stability:**

    * Preserve `PhoneInput` props, emits, slots, and exported types unless a breaking change is explicitly required and approved
4. **Phone semantics are the source of truth:**

    * Phone parsing/validation/formatting is defined by **libphonenumber-js** and the wrapper’s documented policies. Avoid ad-hoc “manual” phone logic.
5. **Tests and build:**

    * Run library build and type-check before committing. Add regression tests when behavior changes.
6. **Documentation:**

    * When functionality changes, update README and examples

---

## 2) Architecture and code placement

### 2.1. Responsibilities

This package is a component + adapter around libphonenumber-js.

**libphonenumber-js owns:**

* Parsing (`parsePhoneNumberFromString`)
* Validation (`isValid`, `isPossible`)
* As-you-type formatting (`AsYouType`)
* Country metadata and formatting rules

**This repository owns:**

* Input sanitization and UX consistency
* Country selection, default-country policy, locale/country lists
* Stable events and a stable `Results` payload
* Slots and slot-bindings as customization API

Requirements:

* Keep phone-number logic centralized in `src/helpers/use-libphonenumber.ts`
* Keep sanitize/country/locale logic in `src/helpers/use-phone-input.ts`
* Keep orchestration and emits in `src/components/PhoneInput.vue`

### 2.2. Main integration points

* **`PhoneInput.vue` is the main integration point.** Changes affecting runtime behavior usually belong here.
* Formatting/validation/Results must be implemented through `use-libphonenumber.ts`.
* Any changes to sanitize/country resolution must be implemented through `use-phone-input.ts`.

### 2.3. Async behavior and fallbacks

The component may perform optional network/file operations:

* `fetchCountry` — IP-based country detection (`ipwho.is`)
* Placeholder examples — lazy import of `libphonenumber-js/examples.mobile.json`

Requirements:

* External calls must be **optional** and **failure-tolerant**
* Fallback order for default country must remain predictable:
    1. `props.countryCode`
    2. IP detection (if `fetchCountry`)
    3. Browser locale (unless `noUseBrowserLocale`)
    4. Otherwise keep `undefined` and remain usable

---

## 3) Public API contract

Public API stability is critical for downstream wrappers (e.g. `vue-phone-input-kz`).

### 3.1. Exports

* Default export: `PhoneInput`
* Re-exported public types from `src/helpers/types.ts`

### 3.2. Props

Key props (names may evolve, but semantics must stay consistent):

* `modelValue`, `countryCode`
* `preferredCountries`, `ignoredCountries`, `onlyCountries`
* `fetchCountry`, `noUseBrowserLocale`, `customCountriesList`, `countryLocale`
* `autoFormat`, `noFormattingAsYouType`
* `phoneNumberDisplayFormat: 'national' | 'international'`

When adding new props:

* Prefer backward-compatible defaults
* Document the prop
* Add usage examples

### 3.3. Emits

The current contract uses **kebab-case** event names. Treat this as an **as-is** public contract.

Common emits include:

* `update:model-value`
* `country-code`
* `update:country-code`
* `update`
* `data`

Rules:

* Always keep payload shapes consistent
* Always emit `Results` via `update` and `data`

### 3.4. Model value semantics

`modelValue` must remain stable and predictable:

* If the number is **valid** and `e164` exists, emit **E.164** into `update:model-value`
* If the number is **invalid**, emit the current display string (formatted-as-you-type or raw, depending on flags)

---

## 4) Results model & exported types

The type layer is part of the public contract.

### 4.1. `Results`

`Results` includes fields such as:

* `isValid`, `isPossible`
* `countryCode`, `countryCallingCode`
* `nationalNumber`, `type`
* `formatInternational`, `formatNational`
* `uri`, `e164`, `rfc3966`
* `phoneNumber` (display value)

Rules:

* Never silently treat invalid data as valid
* Do not remove or rename `Results` fields without a breaking-change process

### 4.2. Other public types

* `Country`
* `PhoneNumberDisplayFormat`
* `Translations`

---

## 5) Sanitization & formatting

### 5.1. Sanitization

Sanitization must remove unexpected characters and keep input safe and predictable.

Allowed characters:

* Digits (`0-9`)
* Space
* `(`, `)`
* `+`, `-`

All other characters must be removed.

### 5.2. Formatting modes

Formatting behavior is controlled by props:

* `autoFormat` — apply formatting to the visible display when possible
* `noFormattingAsYouType` — preserve raw/as-entered behavior in the display
* `phoneNumberDisplayFormat` — for valid numbers, choose `formatNational` vs `formatInternational` for the display

Rules:

* Formatting must not change correctness of emitted metadata
* Validity and E.164 normalization must come from libphonenumber-js Results

---

## 6) Country resolution and changes

### 6.1. Default country policy

Default country should be resolved in a predictable priority order:

1. `props.countryCode`
2. IP-based detection (if enabled)
3. Browser locale (if not disabled)
4. Otherwise keep undefined

### 6.2. Country changes during input

If the parsed Results indicate a different country than the current selected country, the component may update the country via the existing change hooks.

Rules:

* Do not lose the selected country unexpectedly
* Keep `countryCode` observable/controllable via props/emits

---

## 7) Slots and UI customization

Slots are a first-class API.

* Preserve slot names and bindings
* Keep slot bindings stable and documented

When changing slot bindings:

* Treat as a breaking change unless backward compatible
* Update docs and examples

Caution:

* Slot-driven inputs may pass raw strings instead of DOM events. Ensure `updateInputValue` and related handlers accept and correctly process the documented input types.

---

## 8) Tests, validation, and regressions

This repository must keep behavior stable across releases.

When changing logic related to parsing/formatting/emits/slots:

* Add regression tests (recommended: Vitest)
* Cover at least:
    * Sanitization (`abc!!!` -> cleaned, `isValid=false`)
    * Model semantics (valid -> E.164; invalid -> display)
    * `Results` payload shape stability
    * Slot-driven custom input flows
    * Async fallback behavior when `fetchCountry` fails

At minimum, before committing:

* Run build
* Run `vue-tsc` type-check

---

## 9) Documentation & examples

Docs are part of the public contract.

When behavior changes:

* Update `README.md`
* Update examples/playground if present

Recommended minimal example:

```vue
<script setup lang="ts">
import { ref } from 'vue';
import PhoneInput from 'vue-phone-input-base';

const value = ref('');
const meta = ref();
</script>

<template>
  <PhoneInput v-model="value" @update="meta = $event" />
</template>
```

Slot example:

```vue
<PhoneInput>
  <template #input="{ inputValue, updateInputValue, placeholder }">
    <input
      :value="inputValue"
      :placeholder="placeholder"
      @input="updateInputValue"
    />
  </template>
</PhoneInput>
```

---

## 10) Prohibitions & caution

* ❌ Don’t re-implement phone parsing/validation rules manually — use `libphonenumber-js`
* ❌ Don’t change `Results` shape, field names, or semantics without a breaking-change process
* ❌ Don’t rename kebab-case emits to camelCase without a migration plan (this is a public contract)
* ❌ Don’t break `modelValue` semantics (valid -> E.164; invalid -> display)
* ❌ Don’t make network dependencies mandatory for correctness (IP detection, external resources)
* ❌ Don’t break slot names or slot bindings
* ❌ Don’t introduce heavy dependencies without justification

---

## 11) Pre-commit checklist

* [ ] Task is completed according to the plan
* [ ] Public API has not been broken unintentionally (props/emits/slots/types)
* [ ] Model semantics are preserved (valid -> E.164; invalid -> display)
* [ ] Regression tests added/updated when behavior changed
* [ ] README/examples updated if behavior changed
* [ ] Build and type-check pass without errors
