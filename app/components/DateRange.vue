<script setup lang="ts">
const giris = defineModel<string>('giris', { default: '' })
const cikis = defineModel<string>('cikis', { default: '' })
defineProps<{ disabled?: boolean }>()
const open = ref(false)
const root = ref<HTMLElement>()
const tr = (d: string) => new Date(d).toLocaleDateString('tr-TR')
const text = computed(() => giris.value ? `${tr(giris.value)} – ${cikis.value ? tr(cikis.value) : '…'}` : '')
function out(e: FocusEvent) {
  if (!root.value?.contains(e.relatedTarget as Node)) open.value = false
}
</script>

<template>
  <div ref="root" class="drange" @focusout="out">
    <input class="inp" readonly placeholder="Giriş – Çıkış tarihi" :value="text" :disabled="disabled" @focus="open = true" @click="open = true">
    <div v-if="open && !disabled" class="drange-pop">
      <label>Giriş<input v-model="giris" class="inp" type="date" :max="cikis || undefined"></label>
      <label>Çıkış<input v-model="cikis" class="inp" type="date" :min="giris || undefined"></label>
    </div>
  </div>
</template>
