<script setup lang="ts">
import { defineProps, defineEmits } from 'vue'

const props = defineProps<{
  custoTotal: number
  lucroBrutoAVista: number
  margemPercent: number
  isLucroPositivo: boolean
  precoAVista: number
  precoAPrazo: number
  valorParcela: number
  parcelas: number
  brl: (n: number) => string
  mensagem: string
  margemMinimaPercent: number
}>()

const emit = defineEmits<{
  (e: 'copyMessage'): void
  (e: 'openWhatsApp'): void
}>()
</script>

<template>
  <section class="space-y-4">
    <h2 class="text-base font-medium text-violet-300">Resultados</h2>
     <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
       <!-- 1º: Lucro -->
       <div class="of-panel p-4 sm:p-5">
        <h3 class="text-sm font-medium text-zinc-200">Lucro na venda</h3>
        <p class="text-xs text-neutral-400 mt-1">Custo total: {{ props.brl(props.custoTotal) }}</p>
        <p class="text-lg font-semibold mt-2" :class="props.margemPercent >= props.margemMinimaPercent ? 'text-emerald-400' : 'text-rose-400'">{{ props.brl(props.lucroBrutoAVista) }}</p>
        <p class="text-xs mt-1" :class="props.margemPercent >= props.margemMinimaPercent ? 'text-emerald-300' : 'text-rose-300'">Margem: {{ props.margemPercent.toFixed(0) }}%</p>
      </div>
      <!-- 2º: À vista -->
       <div class="of-panel p-4 sm:p-5">
        <h3 class="text-sm font-medium text-zinc-200">À vista</h3>
        <p class="text-xs text-neutral-400 mt-1">Total</p>
        <p class="text-2xl font-semibold mt-2">{{ props.brl(props.precoAVista) }}</p>
      </div>
      <!-- 3º: A prazo -->
       <div class="of-panel p-4 sm:p-5">
        <h3 class="text-sm font-medium text-zinc-200">A prazo</h3>
        <p class="text-xs text-neutral-400 mt-1">Total</p>
        <p class="text-2xl font-semibold mt-2">{{ props.brl(props.precoAPrazo) }}</p>
      </div>
      <!-- 4º: Parcela -->
       <div class="of-panel p-4 sm:p-5">
        <h3 class="text-sm font-medium text-zinc-200">Parcela</h3>
        <p class="text-xs text-neutral-400 mt-1">{{ props.parcelas }}x sem juros</p>
        <p class="text-2xl font-semibold mt-2">{{ props.brl(props.valorParcela) }}</p>
      </div>
    </div>

     <div class="of-panel p-4 sm:p-5 space-y-3">
      <h3 class="text-base font-semibold text-zinc-200">Mensagem gerada</h3>
      <pre class="whitespace-pre-wrap text-sm">{{ props.mensagem }}</pre>
       <div class="flex justify-center gap-2">
         <button class="of-btn of-btn--green" :disabled="props.margemPercent < props.margemMinimaPercent" @click="emit('copyMessage')">Copiar</button>
         <button class="of-btn of-btn--green" :disabled="props.margemPercent < props.margemMinimaPercent" @click="emit('openWhatsApp')">WhatsApp</button>
       </div>
    </div>
  </section>
</template>