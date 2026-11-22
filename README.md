# vue3-select2

[Select2](https://select2.org/) as a Vue 3 component: `v-model`, single and multiple select, AJAX live
search, custom id/text fields, option groups and modal-friendly dropdowns.

Built on [Vue 3](https://vuejs.org/), [jQuery](https://jquery.com/) and [Select2 4](https://select2.org/).
jQuery and Select2 are installed as dependencies and loaded by the component.

## Install

```bash
npm install vue3-select2
```

## Usage

Register it globally:

```js
import { createApp } from 'vue'
import Select2 from 'vue3-select2'

const app = createApp(App)
app.component('Select2', Select2)
```

or locally:

```vue
<template>
  <Select2
    v-model="myValue"
    :options="myOptions"
    placeholder="Choose one"
    :settings="{ allowClear: true }"
    @change="onChange"
    @select="onSelect"
  />
  <p>Value: {{ myValue }}</p>
</template>

<script>
import Select2 from 'vue3-select2'

export default {
  components: { Select2 },
  data() {
    return {
      myValue: '',
      myOptions: ['op1', 'op2', 'op3'] // or [{ id: 1, text: 'One' }, ...]
    }
  },
  methods: {
    onChange(value) { console.log(value) },
    onSelect({ id, text }) { console.log({ id, text }) }
  }
}
</script>
```

### Multiple

```vue
<Select2 v-model="tags" :options="tagOptions" multiple />
```

### Custom fields

```vue
<!-- items look like { customer_id: 7, name: 'Ann', phone: '555-1234' } -->
<Select2 v-model="customer" :options="customers"
         id_field="customer_id" :text_field="['name', 'phone']" text_sep=" / " />
```

### Live (AJAX) search

```vue
<Select2
  v-model="productId"
  search-url="/api/products/search"
  :search-params="{ category: categoryId }"
  :preselected="[{ id: savedProduct.id, text: savedProduct.name }]"
/>
```

The endpoint receives `?q=<term>&page=<n>` plus `searchParams`, and should return:

```json
{ "results": [{ "id": 1, "text": "Apple" }], "pagination": { "more": true } }
```

A plain array of items also works. For any other response shape, pass
`:search-results="(data, params) => ({ results: data.items, pagination: { more: data.hasMore } })"`.
Results go through `id_field` / `text_field` as well. With live search, the selected item isn't
in the page, so pass its label with `preselected`.

### Inside a modal

```vue
<Select2 v-model="value" :options="opts" dropdown-parent="#myModal" />
```

## Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `v-model` / `modelValue` | `String \| Number \| Array` | `null` | Selected id, or array of ids when `multiple` |
| `options` | `Array \| Object` | `[]` | Items: `{id, text}`, primitives, or groups `{text, children: [...]}`. Object maps use their values |
| `multiple` | `Boolean` | `false` | Allow selecting more than one value |
| `placeholder` | `String` | `''` | Placeholder text |
| `disabled` | `Boolean` | `false` | Disable the select |
| `required` | `Boolean` | `false` | `required` attribute |
| `id` / `name` | `String` | `''` | Attributes for the underlying `<select>` |
| `id_field` | `String` | `'id'` | Item key to use as the option id |
| `text_field` | `String \| Array` | `'text'` | Item key, or keys joined by `text_sep`, to use as the text |
| `text_sep` | `String` | `' - '` | Separator for multiple `text_field` keys |
| `settings` | `Object` | `{}` | Any [Select2 option](https://select2.org/configuration/options-api); overrides the defaults below |
| `isolate` | `Boolean` | `false` | Ignore `selected` flags on items; only `v-model` decides the selection |
| `preselected` | `Array \| Object` | `[]` | Labels for selected values that are not in `options` |
| `searchUrl` | `String` | `''` | Enables AJAX live search against this URL |
| `searchParams` | `Object` | `{}` | Extra query params, read on every request |
| `searchDelay` | `Number` | `250` | Debounce in ms |
| `searchResults` | `Function` | `null` | `(data, params) => ({results, pagination})` response mapper |
| `dropdownParent` | `String \| Element` | `null` | Where to attach the dropdown (e.g. a modal). `dParent` is a deprecated alias |

Defaults passed to Select2 (each can be overridden through `settings`): `width: '100%'`,
`minimumResultsForSearch: 10` (`0` with `searchUrl`).

## Events

| Event | Payload | When |
|---|---|---|
| `update:modelValue` / `change` | value | The user changed the selection |
| `select` | `{id, text, ...}` | An item was selected |
| `unselect` | `{id, text, ...}` | An item was removed (multiple / allowClear) |
| `clear` | – | Cleared with `allowClear` |
| `open` / `close` | – | Dropdown opened / closed |
| `error` | `Error` | jQuery / Select2 failed to load |

## Methods (via template ref)

`open()`, `close()`, `focus()`, `getInstance()` (the jQuery-wrapped `<select>`).

## Changelog

See [CHANGELOG.md](./CHANGELOG.md).
