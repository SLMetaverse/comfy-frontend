<template>
  <ButtonGroup
    class="p-buttongroup-vertical absolute bottom-[10px] right-[10px] z-[1000]"
  >
    <Button
      severity="secondary"
      icon="pi pi-plus"
      v-tooltip.left="t('graphCanvasMenu.zoomIn')"
      :aria-label="$t('graphCanvasMenu.zoomIn')"
      @mousedown="repeat('Comfy.Canvas.ZoomIn')"
      @mouseup="stopRepeat"
    />
    <Button
      severity="secondary"
      icon="pi pi-minus"
      v-tooltip.left="t('graphCanvasMenu.zoomOut')"
      :aria-label="$t('graphCanvasMenu.zoomOut')"
      @mousedown="repeat('Comfy.Canvas.ZoomOut')"
      @mouseup="stopRepeat"
    />
    <Button
      severity="secondary"
      icon="pi pi-expand"
      v-tooltip.left="t('graphCanvasMenu.fitView')"
      :aria-label="$t('graphCanvasMenu.fitView')"
      @click="() => commandStore.execute('Comfy.Canvas.FitView')"
    />
    <Button
      severity="secondary"
      v-tooltip.left="
        t(
          'graphCanvasMenu.' +
            (canvasStore.canvas?.read_only ? 'panMode' : 'selectMode')
        ) + ' (Space)'
      "
      :aria-label="
        t(
          'graphCanvasMenu.' +
            (canvasStore.canvas?.read_only ? 'panMode' : 'selectMode')
        )
      "
      @click="() => commandStore.execute('Comfy.Canvas.ToggleLock')"
    >
      <template #icon>
        <i-material-symbols:pan-tool-outline
          v-if="canvasStore.canvas?.read_only"
        />
        <i-simple-line-icons:cursor v-else />
      </template>
    </Button>
    <Button
      severity="secondary"
      :icon="linkHidden ? 'pi pi-eye-slash' : 'pi pi-eye'"
      v-tooltip.left="t('graphCanvasMenu.toggleLinkVisibility')"
      :aria-label="$t('graphCanvasMenu.toggleLinkVisibility')"
      @click="() => commandStore.execute('Comfy.Canvas.ToggleLinkVisibility')"
      data-testid="toggle-link-visibility-button"
    />
    <Button
      severity="secondary"
      :icon="'pi pi-window-maximize'"
      v-tooltip.left="'Toggle Fullscreen'"
      :aria-label="'Toggle Fullscreen'"
      @click="toggleFullscreen"
    />
  </ButtonGroup>
</template>

<script setup lang="ts">
import { LiteGraph } from '@comfyorg/litegraph'
import Button from 'primevue/button'
import ButtonGroup from 'primevue/buttongroup'
import { computed } from 'vue'
import { useI18n } from 'vue-i18n'

import { useCommandStore } from '@/stores/commandStore'
import { useCanvasStore } from '@/stores/graphStore'
import { useSettingStore } from '@/stores/settingStore'

const { t } = useI18n()
const commandStore = useCommandStore()
const canvasStore = useCanvasStore()
const settingStore = useSettingStore()

const linkHidden = computed(
  () => settingStore.get('Comfy.LinkRenderMode') === LiteGraph.HIDDEN_LINK
)

let interval: number | null = null
const repeat = (command: string) => {
  if (interval) return
  const cmd = () => commandStore.execute(command)
  cmd()
  interval = window.setInterval(cmd, 100)
}
const stopRepeat = () => {
  if (interval) {
    clearInterval(interval)
    interval = null
  }
}

const toggleFullscreen = () => {
  if (document.fullscreenElement) {
    document.exitFullscreen()
  } else {
    document.documentElement.requestFullscreen()
  }
}
</script>

<style scoped>
.p-buttongroup-vertical {
  display: flex;
  flex-direction: column;
  border-radius: var(--p-button-border-radius);
  overflow: hidden;
  border: 1px solid var(--p-panel-border-color);
}

.p-buttongroup-vertical .p-button {
  margin: 0;
  border-radius: 0;
}
</style>
