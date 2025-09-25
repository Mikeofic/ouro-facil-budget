<script setup lang="ts">
import { defineProps, defineEmits } from 'vue'

const props = defineProps<{
  cliente: string
  pesoEmGramas: number
  precoPorGrama: number
  custoMercadoriaPorGrama: number
  custosExtras: number
  taxaMaquinaPercent: number
  parcelas: number
  validadeEmDias: number
  margemPercent: number
  margemMinimaPercent: number
  errors: string[]
}>()

const emit = defineEmits<{
  (e: 'update:cliente', v: string): void
  (e: 'update:pesoEmGramas', v: number): void
  (e: 'update:precoPorGrama', v: number): void
  (e: 'update:custoMercadoriaPorGrama', v: number): void
  (e: 'update:custosExtras', v: number): void
  (e: 'update:taxaMaquinaPercent', v: number): void
  (e: 'update:parcelas', v: number): void
  (e: 'update:validadeEmDias', v: number): void
  (e: 'saveSettings'): void
  (e: 'copyMessage'): void
  (e: 'openWhatsApp'): void
}>()

function asNumber(e: Event): number {
  const v = (e.target as HTMLInputElement).value
  const n = Number(v)
  return isNaN(n) ? 0 : n
}
</script>

<template>
  <section class="space-y-4">
    <h2 class="text-base font-medium text-violet-300">Dados do orçamento</h2>
    <div class="space-y-3">
      <label class="block">
        <span class="text-sm text-zinc-400">Nome do cliente (opcional)</span>
        <input :value="props.cliente" @input="emit('update:cliente', ($event.target as HTMLInputElement).value)" type="text" class="mt-1 h-10 w-full rounded-md bg-zinc-900 border border-zinc-800 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-violet-600" placeholder="Ex.: Maria Silva" />
      </label>
      <div class="grid grid-cols-2 gap-3">
        <label class="block">
          <span class="text-sm text-zinc-400">Peso (g)</span>
          <input :value="props.pesoEmGramas" @input="emit('update:pesoEmGramas', asNumber($event))" type="number" min="0" step="0.01" class="mt-1 h-10 w-full rounded-md bg-zinc-900 border border-zinc-800 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-violet-600" />
        </label>
        <label class="block">
          <span class="text-sm text-zinc-400">Preço por grama (R$)</span>
          <input :value="props.precoPorGrama" @input="emit('update:precoPorGrama', asNumber($event))" type="number" min="0" step="0.01" class="mt-1 h-10 w-full rounded-md bg-zinc-900 border border-zinc-800 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-violet-600" />
        </label>
      </div>
      <div class="grid grid-cols-2 gap-3">
        <label class="block">
          <span class="text-sm text-zinc-400">Custo por grama (mercadoria)</span>
          <input :value="props.custoMercadoriaPorGrama" @input="emit('update:custoMercadoriaPorGrama', asNumber($event))" type="number" min="0" step="0.01" class="mt-1 h-10 w-full rounded-md bg-zinc-900 border border-zinc-800 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-violet-600" placeholder="Ex.: 650,00" />
        </label>
        <label class="block">
          <span class="text-sm text-zinc-400">Custos extras</span>
          <input :value="props.custosExtras" @input="emit('update:custosExtras', asNumber($event))" type="number" min="0" step="0.01" class="mt-1 h-10 w-full rounded-md bg-zinc-900 border border-zinc-800 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-violet-600" placeholder="Ex.: 50,00" />
        </label>
      </div>
      <div class="grid grid-cols-2 gap-3 flex items-end">
        <label class="block">
          <span class="text-sm text-zinc-400">Taxa da máquina (%)</span>
          <input :value="props.taxaMaquinaPercent" @input="emit('update:taxaMaquinaPercent', asNumber($event))" type="number" min="0" max="99.99" step="0.01" class="mt-1 h-10 w-full rounded-md bg-zinc-900 border border-zinc-800 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-violet-600" />
        </label>
        <label class="block">
          <span class="text-sm text-zinc-400">Parcelas</span>
          <select :value="props.parcelas" @change="emit('update:parcelas', asNumber($event))" class="mt-1 h-10 w-full rounded-md bg-zinc-900 border border-zinc-800 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-violet-600">
            <option v-for="n in 12" :key="n" :value="n">{{ n }}</option>
          </select>
        </label>
      </div>
      <label class="block">
        <span class="text-sm text-zinc-400">Validade (dias)</span>
        <input :value="props.validadeEmDias" @input="emit('update:validadeEmDias', asNumber($event))" type="number" min="1" step="1" class="mt-1 h-10 w-full rounded-md bg-zinc-900 border border-zinc-800 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-violet-600" />
      </label>

      <div class="flex flex-wrap gap-2">
        <button class="px-3 py-2 rounded-md bg-violet-600 hover:bg-violet-500 shadow-sm" @click="emit('saveSettings')">Salvar preferências</button>
        <button class="px-3 py-2 rounded-md bg-zinc-800 hover:bg-zinc-700 disabled:opacity-50" :disabled="props.margemPercent < props.margemMinimaPercent" @click="emit('copyMessage')">Copiar texto</button>
        <button class="px-3 py-2 rounded-md bg-emerald-600 hover:bg-emerald-500 disabled:opacity-50" :disabled="props.margemPercent < props.margemMinimaPercent" @click="emit('openWhatsApp')">WhatsApp</button>
      </div>

      <div v-if="props.errors.length" class="rounded-md border border-rose-700 bg-rose-900/30 p-3 text-sm">
        <p class="font-medium mb-1">Corrija os campos:</p>
        <ul class="list-disc pl-5 space-y-1">
          <li v-for="e in props.errors" :key="e">{{ e }}</li>
        </ul>
      </div>
    </div>
  </section>
</template>