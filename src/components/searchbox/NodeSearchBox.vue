<template>
  <div
    class="comfy-vue-node-search-container flex justify-center items-center w-full min-w-96"
  >
    <div
      class="comfy-vue-node-preview-container absolute left-[-350px] top-[50px]"
      v-if="enableNodePreview"
    >
      <NodePreview
        :nodeDef="hoveredSuggestion"
        :key="hoveredSuggestion?.name || ''"
        v-if="hoveredSuggestion"
      />
    </div>

    <Button
      icon="pi pi-filter"
      severity="secondary"
      class="filter-button z-10"
      @click="nodeSearchFilterVisible = true"
    />
    <Dialog
      v-model:visible="nodeSearchFilterVisible"
      class="min-w-96"
      dismissable-mask
      modal
      @hide="reFocusInput"
    >
      <template #header>
        <h3>Add node filter condition</h3>
      </template>
      <div class="_dialog-body">
        <NodeSearchFilter @addFilter="onAddFilter"></NodeSearchFilter>
      </div>
    </Dialog>

    <AutoCompletePlus
      :model-value="filters"
      class="comfy-vue-node-search-box z-10 flex-grow"
      scrollHeight="40vh"
      :placeholder="placeholder"
      :input-id="inputId"
      append-to="self"
      :suggestions="suggestions"
      :min-length="0"
      :delay="100"
      :loading="!nodeFrequencyStore.isLoaded"
      @complete="search($event.query)"
      @option-select="emit('addNode', $event.value)"
      @focused-option-changed="setHoverSuggestion($event)"
      complete-on-focus
      auto-option-focus
      force-selection
      multiple
      :optionLabel="'display_name'"
    >
      <template v-slot:option="{ option }">
        <NodeSearchItem :nodeDef="option" :currentQuery="currentQuery" />
      </template>
      <!-- FilterAndValue -->
      <template v-slot:chip="{ value }">
        <SearchFilterChip
          v-if="Array.isArray(value) && value.length === 2"
          :key="`${value[0].id}-${value[1]}`"
          @remove="onRemoveFilter($event, value as FilterAndValue)"
          :text="value[1]"
          :badge="value[0].invokeSequence.toUpperCase()"
          :badge-class="value[0].invokeSequence + '-badge'"
        />
      </template>
    </AutoCompletePlus>
  </div>
</template>

<script setup lang="ts">
import Button from 'primevue/button'
import Dialog from 'primevue/dialog'
import { computed, nextTick, onMounted, ref } from 'vue'
import { useI18n } from 'vue-i18n'

import NodePreview from '@/components/node/NodePreview.vue'
import AutoCompletePlus from '@/components/primevueOverride/AutoCompletePlus.vue'
import NodeSearchFilter from '@/components/searchbox/NodeSearchFilter.vue'
import NodeSearchItem from '@/components/searchbox/NodeSearchItem.vue'
import { type FilterAndValue } from '@/services/nodeSearchService'
import {
  ComfyNodeDefImpl,
  useNodeDefStore,
  useNodeFrequencyStore
} from '@/stores/nodeDefStore'
import { useSettingStore } from '@/stores/settingStore'

import SearchFilterChip from '../common/SearchFilterChip.vue'

const settingStore = useSettingStore()
const { t } = useI18n()

const enableNodePreview = computed(() =>
  settingStore.get('Comfy.NodeSearchBoxImpl.NodePreview')
)

const { filters, searchLimit = 64 } = defineProps<{
  filters: FilterAndValue[]
  searchLimit?: number
}>()

const nodeSearchFilterVisible = ref(false)
const inputId = `comfy-vue-node-search-box-input-${Math.random()}`
const suggestions = ref<ComfyNodeDefImpl[]>([])
const hoveredSuggestion = ref<ComfyNodeDefImpl | null>(null)
const currentQuery = ref('')
const placeholder = computed(() => {
  return filters.length === 0 ? t('g.searchNodes') + '...' : ''
})

const nodeDefStore = useNodeDefStore()
const nodeFrequencyStore = useNodeFrequencyStore()
const search = (query: string) => {
  const queryIsEmpty = query === '' && filters.length === 0
  currentQuery.value = query
  suggestions.value = queryIsEmpty
    ? nodeFrequencyStore.topNodeDefs
    : [
        ...nodeDefStore.nodeSearchService.searchNode(query, filters, {
          limit: searchLimit
        })
      ]
}

const emit = defineEmits(['addFilter', 'removeFilter', 'addNode'])

let inputElement: HTMLInputElement | null = null
const reFocusInput = () => {
  inputElement ??= document.getElementById(inputId) as HTMLInputElement
  if (inputElement) {
    inputElement.blur()
    nextTick(() => inputElement?.focus())
  }
}

onMounted(reFocusInput)
const onAddFilter = (filterAndValue: FilterAndValue) => {
  nodeSearchFilterVisible.value = false
  emit('addFilter', filterAndValue)
}
const onRemoveFilter = (event: Event, filterAndValue: FilterAndValue) => {
  event.stopPropagation()
  event.preventDefault()
  emit('removeFilter', filterAndValue)
  reFocusInput()
}
const setHoverSuggestion = (index: number) => {
  if (index === -1) {
    hoveredSuggestion.value = null
    return
  }
  const value = suggestions.value[index]
  hoveredSuggestion.value = value
}
</script>
