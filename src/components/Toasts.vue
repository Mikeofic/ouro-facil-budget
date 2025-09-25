<script setup lang="ts">
import { defineProps } from 'vue'

const props = defineProps<{
  toasts: { id: number, text: string, type?: 'success'|'error' }[]
}>()
</script>

<template>
  <div class="fixed bottom-4 right-4 space-y-2">
    <div v-for="t in props.toasts" :key="t.id" :class="['rounded-md px-3 py-2 text-sm shadow-lg', t.type === 'error' ? 'bg-rose-600/90' : 'bg-emerald-600/90']">
      {{ t.text }}
    </div>
  </div>
</template>