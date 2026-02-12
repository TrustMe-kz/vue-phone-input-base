# vue-phone-input-base

**vue-phone-input-base** is a customizable Vue 3 phone input component with country selector, formatting, localization, and slot-based UI override support.

This repository is a fork of: [base-vue-phone-input on npm](https://www.npmjs.com/package/base-vue-phone-input)

## The fork

`vue-phone-input-base` keeps the core behavior of the original library and adapts it for our internal style, packaging, and further development needs.

## Features

- Country filtering via `preferred-countries`, `ignored-countries`, or `only-countries`
- Multiple default country detection strategies:
  - Browser locale (default, can be disabled with `no-use-browser-locale`)
  - Network lookup via `fetch-country` (`https://ipwho.is`)
- Phone formatting while typing
- Country search in selector dropdown
- Keyboard navigation support (`ArrowDown`, `ArrowUp`, `Escape`)
- Country-specific placeholder examples
- Full UI customization through named slots

## Installation

```bash
npm i vue-phone-input-base
```

## Example

```vue
<script setup lang="ts">
import { ref } from 'vue'
import PhoneInput from './index'

const tel = ref('')
const res = ref()
</script>

<template>
  <PhoneInput
    v-model="tel"
    @update="res = $event"
  />
  {{ res }}
  {{ tel }}
</template>
```

## Props API

| Prop name | Description | Type | Values | Default |
| --- | --- | --- | --- | --- |
| `modelValue` | `v-model` value: country calling code + phone number (international format) | `string` | - | `undefined` |
| `countryCode` | `v-model` country code. Example: `"FR"` | `CountryCode` | - | `undefined` |
| `placeholder` | Input placeholder | `string` | - | `undefined` |
| `label` | Input label | `string` | - | `undefined` |
| `preferredCountries` | Countries shown first in the selector. Example: `['FR', 'BE', 'GE']` | `Array` | - | `undefined` |
| `ignoredCountries` | Countries removed from the selector. Example: `['FR', 'BE', 'GE']` | `Array` | - | `undefined` |
| `onlyCountries` | Restricts selector to provided countries only. Example: `['FR', 'BE', 'GE']` | `Array` | - | `undefined` |
| `translations` | Component locale strings | `Partial` | - | `undefined` |
| `noUseBrowserLocale` | Disables browser-locale based default country detection | `boolean` | - | `false` |
| `fetchCountry` | Enables network lookup (`https://ipwho.is`) for default country detection | `boolean` | - | `false` |
| `customCountriesList` | Overrides country names in selector | `Record` | - | `undefined` |
| `autoFormat` | Enables final valid-number auto-formatting | `boolean` | - | `true` |
| `noFormattingAsYouType` | Disables as-you-type formatting | `boolean` | - | `false` |
| `phoneNumberDisplayFormat` | Display mode for valid numbers when auto-format is enabled: `national` or `international` | `string` | `national \| international` | `national` |
| `countryLocale` | Locale for country list. Example: `"fr-FR"` | `string` | - | browser locale |
| `excludeSelectors` | Selectors to ignore when handling dropdown close behavior (useful for custom flag UI) | `Array` | - | `undefined` |

## Events API

| Event     | Return |
|-----------| --- |
| `@input`  | [AsYouType value](https://github.com/catamphetamine/libphonenumber-js#as-you-type-formatter). Emitted on phone input changes and country changes |
| `@update` | Full result object (`Results` type) |

## Phone Format

Use `phone-number-display-format="international"` to keep country calling code visible in the final displayed value.

```vue
<PhoneInput
  v-model="tel"
  country-code="RU"
  phone-number-display-format="international"
/>
```

## Keyboard Accessibility

| Key | Action |
| --- | --- |
| `ArrowDown` | Move down in countries list |
| `ArrowUp` | Move up in countries list |
| `Escape` | Close countries list |

## Named Slots

| Name        | Description | Bindings |
|-------------| --- | --- |
| `#selector` | Replace country selector UI | `input-value: string` (current country code), `updateInputValue` (country change handler), `countries` (full countries array) |
| `#input`    | Replace input UI | `input-value: string` (current phone value), `updateInputValue` (phone change handler), `placeholder` |

## Translations

### Labels and placeholders

```vue
<PhoneInput
  :translations="{
    phoneInput: {
      placeholder: 'Phone number',
      example: 'Example:',
    },
  }"
/>
```

### Country list

Two supported approaches:

1. Set locale for built-in country names:

```vue
<PhoneInput country-locale="fr-FR" />
```

2. Provide custom country labels:

```vue
<PhoneInput
  :custom-countries-list="{
    FR: 'France',
    BE: 'Belgique',
    DE: 'Allemagne',
    US: 'Etats-Unis',
  }"
/>
```

## Types

Results emitted by `@update` (or `@data`, if used in your integration):

```ts
export type Results = {
  isValid: boolean
  isPossible?: boolean
  countryCode?: CountryCode
  countryCallingCode?: CountryCallingCode
  nationalNumber?: NationalNumber
  type?: NumberType
  formatInternational?: string
  formatNational?: string
  uri?: string
  e164?: string
  rfc3966?: string
};
```

Country object shape:

```ts
export interface Country {
  iso2: CountryCode
  dialCode: CountryCallingCode
  name: string
}
```

---
**vue-phone-input-base** by Kenny Romanov  
**base-vue-phone-input** by Vladislav Sidoryk (original)
