<template>
  <div
    ref="parentRef"
    class="flex h-full w-full overflow-y-auto p-2"
  >
    <div
      :style="{
        height: `${totalSize}px`,
      }"
      class="relative w-full max-sm:min-h-[calc(100%+1px)]"
    >
      <div
        class="absolute left-0 top-0 w-full"
        :style="{
          transform: `translateY(${virtualRows[0]?.start ?? 0}px)`,
        }"
      >
        <div
          v-for="row in virtualRows"
          :key="row.key.toString()"
          :data-index="row.index"
          :ref="measureElement"
        >
          <slot
            :item="data[row.index]"
            :index="row.index"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useVirtualizer } from '@tanstack/vue-virtual'
import { computed, ref } from 'vue'

const parentRef = ref<HTMLElement | null>(null)
const props = defineProps<{
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  data: any[]
}>()

const virutalOptions = computed(() => {
  return {
    count: props.data.length,
    getScrollElement: () => parentRef.value,
    estimateSize: () => 55,
    overscan: 24,
  }
})

const rowVirtualizer = useVirtualizer(virutalOptions)

const virtualRows = computed(() => rowVirtualizer.value.getVirtualItems())
const totalSize = computed(() => rowVirtualizer.value.getTotalSize())

const measureElement = (el: Element) => {
  if (!el) {
    return
  }

  rowVirtualizer.value.measureElement(el)

  return undefined
}
</script>
