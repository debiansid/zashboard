<template>
  <!-- backend -->
  <div class="card card-compact">
    <div class="card-title px-4 pt-4">
      {{ $t('backend') }}
    </div>
    <div class="card-body gap-4">
      <BackendSwitch />
      <BackendVersion />

      <template v-if="!isSingBox">
        <div class="divider"></div>
        <div class="grid max-w-screen-md grid-cols-2 gap-2 lg:grid-cols-3">
          <div
            class="flex items-center gap-2"
            v-if="configs?.tun"
          >
            {{ $t('tunMode') }}:
            <input
              class="toggle"
              type="checkbox"
              v-model="configs.tun.enable"
              @change="hanlderTunModeChange"
            />
          </div>
          <div
            class="flex items-center gap-2"
            v-for="portConfig in portList"
            :key="portConfig.key"
          >
            <span class="shrink-0"> {{ $t(portConfig.label) }}: </span>
            <input
              class="input input-sm input-bordered w-20"
              type="number"
              v-model="configs[portConfig.key as keyof Config]"
              @change="
                updateConfigs({ [portConfig.key]: Number(configs[portConfig.key as keyof Config]) })
              "
            />
          </div>
        </div>
      </template>

      <div class="grid max-w-screen-md grid-cols-2 gap-2 sm:grid-cols-4">
        <template v-if="!isSingBox">
          <div class="indicator w-full">
            <span
              v-if="isCoreUpdateAvailable"
              class="indicator-item flex"
            >
              <span class="badge badge-xs absolute animate-ping bg-secondary"></span>
              <span class="badge badge-xs bg-secondary"></span>
            </span>
            <button
              :class="
                twMerge('btn btn-primary btn-sm flex-1', isCoreUpgrading ? 'animate-pulse' : '')
              "
              @click="handlerClickUpgradeCore"
            >
              {{ $t('upgradeCore') }}
            </button>
          </div>
          <button
            :class="twMerge('btn btn-sm', isCoreRestarting ? 'animate-pulse' : '')"
            @click="handlerClickRestartCore"
          >
            {{ $t('restartCore') }}
          </button>
          <button
            :class="twMerge('btn btn-sm', isConfigReloading ? 'animate-pulse' : '')"
            @click="handlerClickReloadConfigs"
          >
            {{ $t('reloadConfigs') }}
          </button>
        </template>
        <button
          class="btn btn-sm"
          @click="flushFakeIPAPI"
        >
          {{ $t('flushFakeIP') }}
        </button>
      </div>
      <div class="divider"></div>
      <DnsQuery />
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  flushFakeIPAPI,
  isCoreUpdateAvailable,
  isSingBox,
  reloadConfigsAPI,
  restartCoreAPI,
  upgradeCoreAPI,
} from '@/api'
import BackendVersion from '@/components/common/BackendVersion.vue'
import BackendSwitch from '@/components/settings/BackendSwitch.vue'
import DnsQuery from '@/components/settings/DnsQuery.vue'
import { configs, fetchConfigs, updateConfigs } from '@/store/config'
import { fetchProxies } from '@/store/proxies'
import { fetchRules } from '@/store/rules'
import type { Config } from '@/types'
import { twMerge } from 'tailwind-merge'
import { ref } from 'vue'

const portList = [
  {
    label: 'mixedPort',
    key: 'mixed-port',
  },
  {
    label: 'httpPort',
    key: 'port',
  },
  {
    label: 'socksPort',
    key: 'socks-port',
  },
  {
    label: 'redirPort',
    key: 'redir-port',
  },
  {
    label: 'tproxyPort',
    key: 'tproxy-port',
  },
]

const reloadAll = () => {
  fetchConfigs()
  fetchRules()
  fetchProxies()
}

const isCoreRestarting = ref(false)
const handlerClickRestartCore = async () => {
  if (isCoreRestarting.value) return
  isCoreRestarting.value = true
  try {
    await restartCoreAPI()
    setTimeout(() => {
      reloadAll()
    }, 500)
    isCoreRestarting.value = false
  } catch {
    isCoreRestarting.value = false
  }
}

const isCoreUpgrading = ref(false)
const handlerClickUpgradeCore = async () => {
  if (isCoreUpgrading.value) return
  isCoreUpgrading.value = true
  try {
    await upgradeCoreAPI()
    reloadAll()
    isCoreUpgrading.value = false
  } catch {
    isCoreUpgrading.value = false
  }
}

const isConfigReloading = ref(false)
const handlerClickReloadConfigs = async () => {
  if (isConfigReloading.value) return
  isConfigReloading.value = true
  try {
    await reloadConfigsAPI()
    reloadAll()
    isConfigReloading.value = false
  } catch {
    isConfigReloading.value = false
  }
}

const hanlderTunModeChange = async () => {
  await updateConfigs({ tun: { enable: configs.value?.tun.enable } })
}
</script>
