<script setup lang="ts">
const model = defineModel<number>({ default: 0 })
defineProps<{ disabled?: boolean; placeholder?: string }>()
const fmt = (n: number) => (n ? n.toLocaleString('tr-TR', { minimumFractionDigits: 2, maximumFractionDigits: 2 }) : '')
const text = computed(() => fmt(model.value))
// Rakamlar sağdan dolar: 1 → 0,01 · 125075 → 1.250,75
function onInput(e: Event) {
  const el = e.target as HTMLInputElement
  const digits = el.value.replace(/\D/g, '').slice(0, 13)
  model.value = digits ? Number(digits) / 100 : 0
  el.value = fmt(model.value)
}
</script>

<template>
  <input type="text" inputmode="numeric" :value="text" :placeholder="placeholder ?? '0,00'" :disabled="disabled" @input="onInput">
</template>
