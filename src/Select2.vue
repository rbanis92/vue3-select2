<template>
  <div>
    <select class="form-control" :id="id" :name="name" :disabled="disabled" :required="required"></select>
  </div>
</template>

<script>


import 'select2/dist/css/select2.min.css'
export default {
  name: 'Select2',
  data() {
    return {
      select2: null, disposed: false
    };
  },
  emits: ['update:modelValue', 'change', 'select', 'error'],
  props: {
    modelValue: [String, Number, Array], // previously was `value: String`
    id: {
      type: String,
      default: ''
    },
    name: {
      type: String,
      default: ''
    },
    placeholder: {
      type: String,
      default: ''
    },
    options: {
      type: Array,
      default: () => []
    },
    disabled: {
      type: Boolean,
      default: false
    },
    required: {
      type: Boolean,
      default: false
    },
    settings: {
      type: Object,
      default: () => ({})
    },
    id_field: {                 //if option item has other identity key
      type: String,
      default: 'id'
    },
    text_field: {             //if option item has other text colum. May be one column or multiple column (will be join with seperator)
      type: [String,Array],
      default: 'text'
    },
    text_sep: {               // seperator for multiple text column
      type: String,
      default: ' - '
    },
  },
  watch: {
    settings: { deep: true, handler() { this.setOption(this.options); } },
    placeholder() { this.setOption(this.options); },
    id_field() { this.setOption(this.options); },
    text_field: { deep: true, handler() { this.setOption(this.options); } },
    text_sep() { this.setOption(this.options); },
    options: {
      handler(val) {
        this.setOption(val);
      },
      deep: true
    },
    modelValue: {
      handler(val) {
        this.setValue(val);
      },
      deep: true
    },
  },
  methods: {
    getOptionDatas(values = []) {
  const { id_field, text_field, text_sep } = this; // Destructuring for efficiency

  return (Array.isArray(values) ? values : []).filter(opt => opt != null).map(item => {
    const opt = typeof item === 'object' ? { ...item } : item;
    if (typeof opt !== 'object') return opt; // Early return for non-objects

    opt.id = Object.prototype.hasOwnProperty.call(opt, id_field) ? opt[id_field] : opt.id; // Conditional assignment for id

    if (typeof text_field !== 'string') { // Handle single or multiple text fields
      const texts = [];
      for (const m of Object.keys(text_field)) { // Use Object.keys for iteration efficiency
        if (Object.prototype.hasOwnProperty.call(opt, text_field[m])) {
          texts.push(opt[text_field[m]]);
        }
      }
      opt.text = texts.join(text_sep || ''); // Use default separator if not defined
    } else {
      opt.text = Object.prototype.hasOwnProperty.call(opt, text_field) ? opt[text_field] : opt.text; // Conditional assignment for text
    }

    return opt;
  });
},
    setOption(val = []) {
      if (!this.select2) return;
      this.select2.select2('destroy');
      this.select2.empty();
      this.select2.select2({
        placeholder: this.placeholder,
        minimumResultsForSearch: 10,
        ...this.settings,
        data: this.getOptionDatas(val)
      });
      this.setValue(this.modelValue);
    },
    setValue(val) {
      if (!this.select2) return;
      if (val instanceof Array) {
        this.select2.val([...val]);
      } else {
        this.select2.val(val == null ? null : String(val));
      }
      this.select2.trigger('change.select2');
    }
  },
  async mounted() {
    try {
      const [jqueryModule, select2Module] = await Promise.all([
        import('jquery'), import('select2/dist/js/select2.full')
      ]);
      if (this.disposed) return;
      const jquery = jqueryModule.default || jqueryModule;
      if (!jquery.fn.select2) {
        const install = select2Module.default || select2Module;
        install(window, jquery);
      }
      this.select2 = jquery(this.$el).find('select');
      this.select2.select2({
        placeholder: this.placeholder,
        minimumResultsForSearch: 10,
        ...this.settings,
        data: this.getOptionDatas(this.options)
      });
      this.select2.on('change.vue3Select2', () => {
        const value = this.select2.val();
        this.$emit('update:modelValue', value);
        this.$emit('change', value);
      });
      this.select2.on('select2:select.vue3Select2 select2:unselect.vue3Select2', event => {
        this.$emit('select', event.params.data);
      });
      this.setValue(this.modelValue);
    } catch (error) {
      if (!this.disposed) this.$emit('error', error);
    }
  },
  beforeUnmount() {
    this.disposed = true;
    if (!this.select2) return;
    this.select2.off('.vue3Select2');
    if (this.select2.data('select2')) this.select2.select2('destroy');
    this.select2 = null;
  }
};
</script>
