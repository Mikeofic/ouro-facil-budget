<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import BudgetForm from './components/BudgetForm.vue'
import ResultCards from './components/ResultCards.vue'
import SavedBudgets from './components/SavedBudgets.vue'
import Toasts from './components/Toasts.vue'

// Defaults (editáveis)
const cliente = ref('')
const pesoEmGramas = ref<number>(1)
const precoPorGrama = ref<number>(1200)
const taxaMaquinaPercent = ref<number>(15)
const parcelas = ref<number>(12)
const validadeEmDias = ref<number>(7)
// NOVOS CAMPOS DE CUSTO E MARGEM
const custoMercadoriaPorGrama = ref<number>(650)
const custosExtras = ref<number>(0)
const margemMinimaPercent = ref<number>(30)

// Persistência
const SETTINGS_KEY = 'of:settings:v1'
function loadSettings() {
  try {
    const raw = localStorage.getItem(SETTINGS_KEY)
    if (!raw) return
    const s = JSON.parse(raw)
    if (typeof s?.precoPorGrama === 'number') precoPorGrama.value = s.precoPorGrama
    if (typeof s?.taxaMaquinaPercent === 'number') taxaMaquinaPercent.value = s.taxaMaquinaPercent
    if (typeof s?.parcelas === 'number') parcelas.value = s.parcelas
    if (typeof s?.validadeEmDias === 'number') validadeEmDias.value = s.validadeEmDias
    // Persistência de custos
    if (typeof s?.custoMercadoriaPorGrama === 'number') custoMercadoriaPorGrama.value = s.custoMercadoriaPorGrama
    if (typeof s?.custosExtras === 'number') custosExtras.value = s.custosExtras
    if (typeof s?.margemMinimaPercent === 'number') margemMinimaPercent.value = s.margemMinimaPercent
  } catch (_) {}
}
function saveSettings() {
  const s = {
    precoPorGrama: precoPorGrama.value,
    taxaMaquinaPercent: taxaMaquinaPercent.value,
    parcelas: parcelas.value,
    validadeEmDias: validadeEmDias.value,
    // novos campos
    custoMercadoriaPorGrama: custoMercadoriaPorGrama.value,
    custosExtras: custosExtras.value,
    margemMinimaPercent: margemMinimaPercent.value,
  }
  localStorage.setItem(SETTINGS_KEY, JSON.stringify(s))
}
loadSettings()

// Validações simples
const errors = ref<string[]>([])
function validate() {
  const e: string[] = []
  if (pesoEmGramas.value <= 0) e.push('Peso deve ser maior que 0g')
  if (precoPorGrama.value <= 0) e.push('Preço por grama deve ser maior que 0')
  if (taxaMaquinaPercent.value < 0 || taxaMaquinaPercent.value >= 100) e.push('Taxa da máquina deve estar entre 0 e 99,99')
  if (parcelas.value <= 0) e.push('Parcelas deve ser maior que 0')
  if (validadeEmDias.value <= 0) e.push('Validade deve ser maior que 0')
  errors.value = e
  return e.length === 0
}

watch([pesoEmGramas, precoPorGrama, taxaMaquinaPercent, parcelas, validadeEmDias], validate, { immediate: true })

// Cálculo
const precoBase = computed(() => round2(pesoEmGramas.value * precoPorGrama.value))
const precoAVista = computed(() => round2(precoBase.value))
const precoAPrazo = computed(() => {
  const taxa = taxaMaquinaPercent.value / 100
  const val = precoAVista.value / (1 - taxa)
  return round2(val)
})
const valorParcela = computed(() => round2(precoAPrazo.value / parcelas.value))
// CUSTOS, LUCRO E MARGEM
const custoTotal = computed(() => round2(pesoEmGramas.value * custoMercadoriaPorGrama.value + custosExtras.value))
const lucroBrutoAVista = computed(() => round2(precoAVista.value - custoTotal.value))
const margemPercent = computed(() => {
  return precoAVista.value > 0 ? round2((lucroBrutoAVista.value / precoAVista.value) * 100) : 0
})
const isLucroPositivo = computed(() => lucroBrutoAVista.value >= 0)

function round2(n: number) { return Math.round(n * 100) / 100 }
function brl(n: number) { return new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(n) }

// Mensagem
const mensagem = computed(() => {
  const avista = `À Vista sai por: ${brl(precoAVista.value)}`
  const parcelaFmt = `*${parcelas.value}x sem juros de ${brl(valorParcela.value)}*`
  return `Orçamento válido por ${validadeEmDias.value} dias a partir da data de hoje!\n\n*${avista}* 🥇\n\nValor a prazo: ${brl(precoAPrazo.value)} ou ${parcelaFmt}`
})

// Orçamentos salvos
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
const BUDGETS_KEY = 'of:budgets:v1'
const budgets = ref<Budget[]>(loadBudgets())
function loadBudgets(): Budget[] {
  try { return JSON.parse(localStorage.getItem(BUDGETS_KEY) || '[]') } catch { return [] }
}
function saveBudgets() { localStorage.setItem(BUDGETS_KEY, JSON.stringify(budgets.value)) }
// Função robusta para gerar IDs sem depender de crypto.randomUUID
function generateId(): string {
  return `${Date.now()}-${Math.random().toString(36).slice(2, 8)}`
}
function addBudget() {
  if (!validate()) return toast('Preencha os campos corretamente', 'error')
  const b: Budget = {
    id: generateId(),
    cliente: cliente.value.trim(),
    criadoEmISO: new Date().toISOString(),
    pesoEmGramas: pesoEmGramas.value,
    precoPorGrama: precoPorGrama.value,
    taxaMaquinaPercent: taxaMaquinaPercent.value,
    parcelas: parcelas.value,
    validadeEmDias: validadeEmDias.value,
    precoAVista: precoAVista.value,
    precoAPrazo: precoAPrazo.value,
    valorParcela: valorParcela.value,
    mensagem: mensagem.value,
  }
  budgets.value.unshift(b)
  saveBudgets()
  toast('Orçamento salvo!')
}

// Busca & paginação
const search = ref('')
const page = ref(1)
const perPage = ref(10)
const startDate = ref<string>('')
const endDate = ref<string>('')
const filtered = computed(() => {
  const q = search.value.trim().toLowerCase()
  const start = startDate.value ? new Date(startDate.value + 'T00:00:00') : null
  const end = endDate.value ? new Date(endDate.value + 'T23:59:59') : null
  return budgets.value.filter(b => {
    const nameOk = !q || (b.cliente || '').toLowerCase().includes(q)
    const d = new Date(b.criadoEmISO)
    const afterOk = !start || d >= start
    const beforeOk = !end || d <= end
    return nameOk && afterOk && beforeOk
  })
})
const totalPages = computed(() => Math.max(1, Math.ceil(filtered.value.length / perPage.value)))
const paged = computed(() => {
  const start = (page.value - 1) * perPage.value
  return filtered.value.slice(start, start + perPage.value)
})
// function goTo(p: number) { page.value = Math.min(Math.max(1, p), totalPages.value) }

// Clipboard & WhatsApp
async function copyMessage() {
  if (!validate()) return toast('Preencha os campos corretamente', 'error')
  // bloqueio por margem
  if (margemPercent.value < margemMinimaPercent.value) {
    return toast(`Margem menor que ${margemMinimaPercent.value}% — ajuste o preço por grama ou os custos`, 'error')
  }
  addBudget()
  try { await navigator.clipboard.writeText(mensagem.value); toast('Mensagem copiada!') } catch { toast('Falha ao copiar', 'error') }
}
function openWhatsApp() {
  if (!validate()) return toast('Preencha os campos corretamente', 'error')
  // bloqueio por margem
  if (margemPercent.value < margemMinimaPercent.value) {
    return toast(`Margem menor que ${margemMinimaPercent.value}% — ajuste o preço por grama ou os custos`, 'error')
  }
  addBudget()
  const url = `https://wa.me/?text=${encodeURIComponent(mensagem.value)}`
  window.open(url, '_blank')
}
// Copiar mensagem de orçamento salvo com feedback
async function copySavedMessage(msg: string) {
  try { await navigator.clipboard.writeText(msg); toast('Mensagem copiada!') } catch { toast('Falha ao copiar', 'error') }
}

// Toast simples
const toasts = ref<{ id: number, text: string, type?: 'success'|'error' }[]>([])
function toast(text: string, type: 'success'|'error' = 'success') {
  const id = Date.now()
  toasts.value.push({ id, text, type })
  setTimeout(() => { toasts.value = toasts.value.filter(t => t.id !== id) }, 2000)
}
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-indigo-950 via-zinc-950 to-black text-zinc-100">
    <!-- Topbar -->
    <header class="sticky top-0 z-20 border-b border-zinc-800/70 bg-zinc-900/60 backdrop-blur">
      <div class="mx-auto max-w-6xl px-3 sm:px-4 py-6 flex items-center justify-between">
        <h3 class="text-sm md:text-base font-semibold tracking-tight text-zinc-100">Orçamento - Ouro fácil</h3>
      </div>
    </header>

    <main class="mx-auto max-w-6xl px-3 sm:px-4 lg:px-16 py-6 grid grid-cols-1 md:grid-cols-2 gap-4 sm:gap-6">
      <BudgetForm
        :cliente="cliente"
        :pesoEmGramas="pesoEmGramas"
        :precoPorGrama="precoPorGrama"
        :custoMercadoriaPorGrama="custoMercadoriaPorGrama"
        :custosExtras="custosExtras"
        :taxaMaquinaPercent="taxaMaquinaPercent"
        :parcelas="parcelas"
        :validadeEmDias="validadeEmDias"
        :margemPercent="margemPercent"
        :margemMinimaPercent="margemMinimaPercent"
        :errors="errors"
        @update:cliente="v => (cliente = v)"
        @update:pesoEmGramas="v => (pesoEmGramas = v)"
        @update:precoPorGrama="v => (precoPorGrama = v)"
        @update:custoMercadoriaPorGrama="v => (custoMercadoriaPorGrama = v)"
        @update:custosExtras="v => (custosExtras = v)"
        @update:taxaMaquinaPercent="v => (taxaMaquinaPercent = v)"
        @update:parcelas="v => (parcelas = v)"
        @update:validadeEmDias="v => (validadeEmDias = v)"
        @saveSettings="saveSettings"
        @copyMessage="copyMessage"
        @openWhatsApp="openWhatsApp"
      />

      <ResultCards
        :custoTotal="custoTotal"
        :lucroBrutoAVista="lucroBrutoAVista"
        :margemPercent="margemPercent"
        :isLucroPositivo="isLucroPositivo"
        :precoAVista="precoAVista"
        :precoAPrazo="precoAPrazo"
        :valorParcela="valorParcela"
        :parcelas="parcelas"
        :brl="brl"
        :mensagem="mensagem"
        :margemMinimaPercent="margemMinimaPercent"
        @copyMessage="copyMessage"
        @openWhatsApp="openWhatsApp"
      />

      <SavedBudgets
        :paged="paged"
        :search="search"
        :startDate="startDate"
        :endDate="endDate"
        :page="page"
        :totalPages="totalPages"
        :brl="brl"
        :copySavedMessage="copySavedMessage"
        @update:search="v => (search = v)"
        @update:startDate="v => (startDate = v)"
        @update:endDate="v => (endDate = v)"
        @update:page="v => (page = v)"
      />
    </main>

    <Toasts :toasts="toasts" />
  </div>
</template>
