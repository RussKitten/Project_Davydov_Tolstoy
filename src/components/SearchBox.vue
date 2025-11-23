<template>
  <div>
    <input
      type="search"
      :placeholder="placeholder"
      v-model="term"
      @keydown.esc="clear"
    />
    <p class="small" v-if="hint">Подсказка: нажмите <kbd>Esc</kbd> чтобы очистить</p>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  modelValue: { type: String, default: '' },
  placeholder: { type: String, default: 'Поиск…' },
  hint: { type: Boolean, default: true }
})
const emit = defineEmits(['update:modelValue'])

const term = ref(props.modelValue)
watch(() => props.modelValue, v => { if (v !== term.value) term.value = v })
watch(term, v => emit('update:modelValue', v))

function clear() { term.value = '' }
</script>