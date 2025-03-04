<template>
  <div
    :class="
      twMerge('flex flex-col gap-1 p-2 text-sm sm:gap-2', horizontal && 'flex-row flex-wrap pb-0')
    "
    v-if="configs"
  >
    <div
      :class="twMerge('flex flex-col-reverse gap-2', horizontal && 'flex-row')"
      v-if="proxyProviederList.length"
    >
      <div
        role="tablist"
        class="tabs-boxed tabs tabs-sm"
        v-if="proxyProviederList.length"
      >
        <a
          role="tab"
          class="tab"
          v-for="type in PROXY_TAB_TYPE"
          :key="type"
          :class="{ 'tab-active': proxiesTabShow === type }"
          @click="proxiesTabShow = type"
        >
          {{ $t(type) }}</a
        >
      </div>
      <div
        class="flex flex-col gap-2"
        v-if="proxiesTabShow === PROXY_TAB_TYPE.PROVIDER"
      >
        <button
          :class="twMerge('btn btn-sm', isUpgrading ? 'animate-pulse' : '')"
          @click="handlerClickUpdateAllProviders"
        >
          {{ $t('updateAllProviders') }}
        </button>
      </div>
    </div>
    <div class="w-full sm:hidden"></div>
    <div :class="twMerge('flex w-full items-center gap-2', horizontal && 'w-24')">
      <select
        class="select select-bordered select-sm w-1/2 flex-1"
        :value="configs.mode"
        @change="handlerModeChange"
      >
        <option
          v-for="mode in modeList"
          :key="mode"
          :value="mode"
        >
          {{ $t(mode.toLowerCase()) || mode }}
        </option>
      </select>
    </div>
    <div :class="twMerge('flex w-full items-center gap-2', horizontal && 'w-72 max-sm:flex-1')">
      <select
        class="select select-bordered select-sm w-1/2 flex-1"
        v-model="proxySortType"
      >
        <option
          v-for="type in Object.values(PROXY_SORT_TYPE)"
          :key="type"
          :value="type"
        >
          {{ $t(type) }}
        </option>
      </select>
      <span class="shrink-0"> {{ $t('hideUnavailable') }}: </span>
      <input
        class="toggle"
        type="checkbox"
        v-model="hideUnavailableProxies"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { updateProxyProviderAPI } from '@/api'
import { useProxies } from '@/composables/proxies'
import { PROXY_SORT_TYPE, PROXY_TAB_TYPE } from '@/config'
import { configs, updateConfigs } from '@/store/config'
import { fetchProxies, proxyProviederList } from '@/store/proxies'
import { hideUnavailableProxies, proxySortType } from '@/store/settings'
import { twMerge } from 'tailwind-merge'
import { computed, ref } from 'vue'

defineProps<{
  horizontal?: boolean
}>()

const { proxiesTabShow } = useProxies()
const isUpgrading = ref(false)
const handlerClickUpdateAllProviders = async () => {
  if (isUpgrading.value) return
  isUpgrading.value = true
  try {
    await Promise.all(
      proxyProviederList.value.map((provider) => updateProxyProviderAPI(provider.name)),
    )
    await fetchProxies()
    isUpgrading.value = false
  } catch {
    await fetchProxies()
    isUpgrading.value = false
  }
}

const modeList = computed(() => {
  return configs.value?.['mode-list'] || configs.value?.['modes'] || ['direct', 'rule', 'global']
})

const handlerModeChange = (e: Event) => {
  const mode = (e.target as HTMLSelectElement).value
  updateConfigs({ mode })
}
</script>
