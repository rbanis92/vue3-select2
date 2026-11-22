<template>
  <div>
    <select
      class="form-control"
      :id="id || null"
      :name="name || null"
      :disabled="disabled"
      :required="required"
      :multiple="multiple"
    ></select>
  </div>
</template>

<script>
import 'select2/dist/css/select2.min.css'

const NS = '.vue3Select2';

// jQuery + select2 are loaded once and shared by every instance.
let loader = null;
function loadSelect2() {
  if (!loader) {
    loader = Promise.all([import('jquery'), import('select2/dist/js/select2.full')])
      .then(([jqueryModule, select2Module]) => {
        const jquery = jqueryModule.default || jqueryModule;
        if (!jquery.fn.select2) {
          const install = select2Module.default || select2Module;
          if (typeof install === 'function') install(window, jquery);
        }
        return jquery;
      })
      .catch(error => { loader = null; throw error; });
  }
  return loader;
}

const has = (obj, key) => Object.prototype.hasOwnProperty.call(obj, key);
const toList = values => {
  if (Array.isArray(values)) return values;
  if (values && typeof values === 'object') return Object.values(values);
  return [];
};
const toIds = val => (val == null ? [] : (Array.isArray(val) ? val : [val])).map(String);

export default {
  name: 'Select2',
  data() {
    return {
      select2: null,
      jquery: null,
      disposed: false,
      lastSignature: null
    };
  },
  emits: ['update:modelValue', 'change', 'select', 'unselect', 'clear', 'open', 'close', 'error'],
  props: {
    modelValue: { type: [String, Number, Array], default: null },
    id: { type: String, default: '' },
    name: { type: String, default: '' },
    placeholder: { type: String, default: '' },
    // Array of {id, text} / primitives, or an object map whose values are option items.
    options: { type: [Array, Object], default: () => [] },
    disabled: { type: Boolean, default: false },
    required: { type: Boolean, default: false },
    multiple: { type: Boolean, default: false },
    // Any Select2 option: https://select2.org/configuration/options-api
    settings: { type: Object, default: () => ({}) },
    id_field: { type: String, default: 'id' },          // key used as the option id
    text_field: { type: [String, Array], default: 'text' }, // key(s) used as the option text
    text_sep: { type: String, default: ' - ' },         // joins multiple text_field values
    // Drop the `selected` flag from option items, so only v-model decides the selection.
    isolate: { type: Boolean, default: false },
    // Labels for values that are not in `options` (needed with `searchUrl`, where the list
    // lives on the server). Array of items or a single {id, text} item.
    preselected: { type: [Array, Object], default: () => [] },
    // Live (AJAX) search. The endpoint receives ?q=<term>&page=<n> plus `searchParams`
    // and answers {results: [...], pagination: {more: bool}} or a plain array.
    searchUrl: { type: String, default: '' },
    searchParams: { type: Object, default: () => ({}) }, // read on every request
    searchDelay: { type: Number, default: 250 },
    // Optional mapper for non-standard responses: (data, params) => ({results, pagination})
    searchResults: { type: Function, default: null },
    // Selector string, HTMLElement or jQuery object (e.g. a modal) to attach the dropdown to.
    dropdownParent: { default: null },
    // Deprecated alias of `dropdownParent`.
    dParent: { type: String, default: '' }
  },
  watch: {
    options: {
      deep: true,
      handler(val) {
        // With live search the server owns the list: rebuilding would wipe the selection.
        if (this.searchUrl) return;
        const signature = this.optionSignature(val);
        if (signature === this.lastSignature) return;
        this.reinit();
      }
    },
    modelValue: { deep: true, handler(val) { this.setValue(val); } },
    preselected: { deep: true, handler() { this.setValue(this.modelValue, true); } },
    settings: { deep: true, handler() { this.reinit(); } },
    text_field: { deep: true, handler() { this.reinit(); } },
    placeholder() { this.reinit(); },
    id_field() { this.reinit(); },
    text_sep() { this.reinit(); },
    isolate() { this.reinit(); },
    multiple() { this.$nextTick(() => this.reinit()); },
    searchUrl() { this.reinit(); },
    searchDelay() { this.reinit(); },
    dropdownParent() { this.reinit(); },
    dParent() { this.reinit(); }
  },
  methods: {
    normalizeItem(item) {
      if (item == null) return null;
      if (typeof item !== 'object') return { id: item, text: String(item) };
      const { id_field, text_field, text_sep } = this;
      const opt = { ...item };
      if (this.isolate) delete opt.selected;
      if (has(opt, id_field)) opt.id = opt[id_field];
      if (typeof text_field === 'string') {
        if (has(opt, text_field)) opt.text = opt[text_field];
      } else if (Array.isArray(text_field)) {
        opt.text = text_field.filter(f => has(opt, f) && opt[f] != null && opt[f] !== '')
          .map(f => opt[f]).join(text_sep);
      }
      if (Array.isArray(opt.children)) {
        opt.children = this.getOptionDatas(opt.children);
      }
      return opt;
    },
    getOptionDatas(values = []) {
      return toList(values).map(this.normalizeItem).filter(Boolean);
    },
    optionSignature(values) {
      return JSON.stringify(this.getOptionDatas(values).map(o => [o.id, o.text, o.disabled, o.children && o.children.length]));
    },
    resolveParent(parent) {
      if (!parent) return null;
      const el = this.jquery(parent);
      return el.length ? el : null;
    },
    getSelect2Settings() {
      const settings = { ...(this.settings || {}) };
      const parentOption = settings.dropdownParent !== undefined
        ? settings.dropdownParent
        : (this.dropdownParent || this.dParent);
      delete settings.dropdownParent;

      const config = {
        placeholder: this.placeholder,
        width: '100%',
        minimumResultsForSearch: this.searchUrl ? 0 : 10,
        ...settings
      };

      if (this.searchUrl && !config.ajax) {
        config.ajax = {
          url: this.searchUrl,
          dataType: 'json',
          delay: this.searchDelay,
          cache: true,
          // Read searchParams at request time, so a parent field can narrow the list
          // without rebuilding the widget.
          data: params => ({ ...(this.searchParams || {}), q: params.term || '', page: params.page || 1 }),
          processResults: (data, params) => {
            params.page = params.page || 1;
            const out = this.searchResults
              ? this.searchResults(data, params)
              : {
                  results: Array.isArray(data) ? data : (data && data.results) || [],
                  pagination: { more: !!(data && data.pagination && data.pagination.more) }
                };
            return { ...out, results: this.getOptionDatas(out.results) };
          }
        };
      } else if (!config.ajax) {
        config.data = this.getOptionDatas(this.options);
      }

      const parent = this.resolveParent(parentOption);
      if (parent) config.dropdownParent = parent;
      return config;
    },
    // Add <option> tags for selected values that the list does not contain (live search /
    // preselected labels), so select2 can display them.
    ensureOptions(val) {
      const select = this.select2;
      const known = new Set(select.find('option').map((i, o) => o.value).get());
      const labels = this.getOptionDatas(Array.isArray(this.preselected) ? this.preselected : [this.preselected]);
      toIds(val).forEach(v => {
        if (known.has(v)) {
          const item = labels.find(x => String(x.id) === v);
          const option = select.find('option').filter((i, o) => o.value === v);
          if (item && option.text() === v) option.text(item.text); // upgrade a bare-id label
          return;
        }
        const item = labels.find(x => String(x.id) === v);
        // An empty value only gets an option when a label is supplied for it, so the
        // placeholder still shows on blank forms.
        if (v === '' && !item) return;
        if (!this.searchUrl && !item) return;
        select.append(new Option(item ? item.text : v, v, false, false));
      });
    },
    bindEvents() {
      const select = this.select2;
      select.on('change' + NS, () => {
        const value = select.val();
        this.$emit('update:modelValue', value);
        this.$emit('change', value);
      });
      select.on('select2:select' + NS, e => this.$emit('select', e.params.data));
      select.on('select2:unselect' + NS, e => this.$emit('unselect', e.params.data));
      select.on('select2:clear' + NS, () => this.$emit('clear'));
      select.on('select2:open' + NS, () => this.$emit('open'));
      select.on('select2:close' + NS, () => this.$emit('close'));
    },
    destroySelect2() {
      const select = this.select2;
      if (!select) return;
      const instance = select.data('select2');
      // Close first so `close` listeners (scroll locks etc.) run before teardown.
      if (instance && instance.isOpen()) select.select2('close');
      select.off(NS);
      if (instance) {
        // select2 can leave its scroll-pinning handlers on ancestors behind.
        if (instance.id) select.parents().off('scroll.select2.' + instance.id);
        select.select2('destroy');
      }
    },
    init() {
      const select = this.select2;
      if (!this.searchUrl) select.empty();
      this.lastSignature = this.optionSignature(this.options);
      select.select2(this.getSelect2Settings());
      this.bindEvents();
      this.setValue(this.modelValue, true);
    },
    reinit() {
      if (!this.select2 || this.disposed) return;
      this.destroySelect2();
      this.init();
    },
    // Back-compat name used by 0.2.x.
    setOption() {
      this.reinit();
    },
    setValue(val, force = false) {
      if (!this.select2) return;
      const ids = toIds(val);
      this.ensureOptions(ids);
      const current = toIds(this.select2.val());
      if (!force && current.length === ids.length && current.every((v, i) => v === ids[i])) return;
      this.select2.val(this.multiple || Array.isArray(val) ? ids : (ids.length ? ids[0] : null));
      // Namespaced trigger: refreshes select2 without firing our own change listener.
      this.select2.trigger('change.select2');
    },
    // Public helpers for template refs.
    getInstance() { return this.select2; },
    open() { if (this.select2) this.select2.select2('open'); },
    close() { if (this.select2) this.select2.select2('close'); },
    focus() {
      const instance = this.select2 && this.select2.data('select2');
      if (!instance) return;
      const target = instance.$container.find('.select2-search__field')[0] || instance.$selection[0];
      if (target) target.focus();
    }
  },
  async mounted() {
    try {
      const jquery = await loadSelect2();
      if (this.disposed) return;
      this.jquery = jquery;
      this.select2 = jquery(this.$el).find('select');
      this.init();
    } catch (error) {
      if (!this.disposed) this.$emit('error', error);
    }
  },
  beforeUnmount() {
    this.disposed = true;
    this.destroySelect2();
    this.select2 = null;
  }
};
</script>
