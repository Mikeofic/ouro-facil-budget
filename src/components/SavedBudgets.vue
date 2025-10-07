<script setup lang="ts">
import { defineProps, defineEmits } from 'vue'

type Budget = {
  id: string
  cliente: string
  criadoEmISO: string
  pesoEmGramas: number
  precoPorGrama: number
  taxaMaquinaPercent: number
  parcelas: number
  validadeEmDias: number
  precoAVista: number
  precoAPrazo: number
  valorParcela: number
  mensagem: string
}

const props = defineProps<{
  paged: Budget[]
  search: string
  startDate: string
  endDate: string
  page: number
  totalPages: number
  brl: (n: number) => string
  copySavedMessage: (msg: string) => void | Promise<void>
  deleteBudget: (id: string) => void
}>()

const emit = defineEmits<{
  (e: 'update:search', v: string): void
  (e: 'update:startDate', v: string): void
  (e: 'update:endDate', v: string): void
  (e: 'update:page', v: number): void
}>()
</script>

<template>
  <section class="space-y-4 md:col-span-2">
    <div class="flex flex-col gap-3 sm:flex-row sm:items-end sm:justify-between">
      <h2 class="text-base font-medium text-violet-300">Orçamentos salvos</h2>
      <div class="flex flex-wrap gap-2">
        <input :value="props.search" @input="emit('update:search', ($event.target as HTMLInputElement).value)" type="text" placeholder="Buscar por cliente" class="h-10 w-full sm:w-48 rounded-md bg-zinc-900 border border-zinc-800 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-violet-600" />
        <input :value="props.startDate" @input="emit('update:startDate', ($event.target as HTMLInputElement).value)" type="date" class="h-10 w-full sm:w-40 rounded-md bg-zinc-900 border border-zinc-800 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-violet-600" />
        <input :value="props.endDate" @input="emit('update:endDate', ($event.target as HTMLInputElement).value)" type="date" class="h-10 w-full sm:w-40 rounded-md bg-zinc-900 border border-zinc-800 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-violet-600" />
      </div>
    </div>

    <!-- Tabela (desktop) -->
    <div class="hidden md:block of-panel overflow-hidden">
      <table class="min-w-full text-sm">
        <thead class="bg-zinc-900/70">
          <tr>
            <th class="text-left px-4 py-2 uppercase text-xs text-neutral-400">Cliente</th>
            <th class="text-left px-4 py-2 uppercase text-xs text-neutral-400">Criado em</th>
            <th class="text-left px-4 py-2 uppercase text-xs text-neutral-400">Peso (g)</th>
            <th class="text-left px-4 py-2 uppercase text-xs text-neutral-400">À vista</th>
            <th class="text-left px-4 py-2 uppercase text-xs text-neutral-400">A prazo</th>
            <th class="text-left px-4 py-2 uppercase text-xs text-neutral-400">Ações</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="b in props.paged" :key="b.id" class="hover:bg-zinc-900/40">
            <td class="text-left px-4 py-2 align-top">{{ b.cliente || '--' }}</td>
            <td class="text-left px-4 py-2 align-top">{{ new Date(b.criadoEmISO).toLocaleDateString() }}</td>
            <td class="text-left px-4 py-2 align-top">{{ b.pesoEmGramas }}</td>
            <td class="text-left px-4 py-2 align-top">{{ props.brl(b.precoAVista) }}</td>
            <td class="text-left px-4 py-2 align-top">{{ props.brl(b.precoAPrazo) }}</td>
            <td class="text-left px-4 py-2 align-top">
              <button class="of-btn of-btn--green of-btn--sm" @click="props.copySavedMessage(b.mensagem)">Copiar</button>
              <button class="of-btn of-btn--green of-btn--sm" style="margin-left: 8px;" @click="props.deleteBudget(b.id)">Excluir</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <!-- Cards (mobile) -->
    <div class="block md:hidden space-y-3">
      <div v-for="b in props.paged" :key="b.id" class="of-panel p-4 flex flex-col gap-2">
        <div class="flex justify-between items-center">
          <span class="font-semibold text-violet-300">{{ b.cliente || '--' }}</span>
          <span class="text-xs text-neutral-400">{{ new Date(b.criadoEmISO).toLocaleDateString() }}</span>
        </div>
        <div class="flex flex-wrap gap-4 text-sm">
          <span>Peso: <span class="font-semibold">{{ b.pesoEmGramas }}g</span></span>
          <span>À vista: <span class="font-semibold">{{ props.brl(b.precoAVista) }}</span></span>
          <span>A prazo: <span class="font-semibold">{{ props.brl(b.precoAPrazo) }}</span></span>
        </div>
        <div class="flex gap-2 mt-2">
          <button class="of-btn of-btn--green of-btn--sm" @click="props.copySavedMessage(b.mensagem)">Copiar</button>
          <button class="of-btn of-btn--green of-btn--sm" @click="props.deleteBudget(b.id)">Excluir</button>
        </div>
      </div>
    </div>

    <!-- Paginação -->
    <div class="flex items-center justify-between">
      <p class="text-xs text-neutral-400">Página {{ props.page }} de {{ props.totalPages }}</p>
      <div class="flex gap-2">
        <button class="of-btn of-btn--green of-btn--sm" :disabled="props.page <= 1" @click="emit('update:page', props.page - 1)">Anterior</button>
        <button class="of-btn of-btn--green of-btn--sm" :disabled="props.page >= props.totalPages" @click="emit('update:page', props.page + 1)">Próxima</button>
      </div>
    </div>
  </section>
</template>