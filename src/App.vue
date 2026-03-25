<script setup lang="js">
import { ref, provide, watch, onMounted, onBeforeUnmount, nextTick } from 'vue'
import ModuleFooter from './components/ModuleFooter.vue'
import ModuleTodo from './components/ModuleTodo.vue'
import { BApp, BModal } from 'bootstrap-vue-next'
import './assets/main.css'

// begin dark mode
const isDark = ref(false)
const dialogVisible = ref(false)
const dialogMode = ref('alert')
const dialogMessage = ref('')
const activeDialog = ref(null)
const queuedDialogs = []
let nativeAlert = null
let nativeConfirm = null

const toggleDark = () => {
  isDark.value = !isDark.value
}

provide('isDark', isDark)
provide('toggleDark', toggleDark)

watch(isDark, (val) => {
  document.documentElement.setAttribute('data-bs-theme', val ? 'dark' : 'light')
})

function queueDialog(mode, message) {
  return new Promise((resolve) => {
    queuedDialogs.push({
      mode,
      message: String(message ?? ''),
      resolve,
    })
    processDialogQueue()
  })
}

function processDialogQueue() {
  if (activeDialog.value || queuedDialogs.length === 0) {
    return
  }

  activeDialog.value = queuedDialogs.shift()
  dialogMode.value = activeDialog.value.mode
  dialogMessage.value = activeDialog.value.message
  dialogVisible.value = true
}

function resolveActiveDialog(result) {
  if (!activeDialog.value) {
    return
  }
  activeDialog.value.resolve(result)
  activeDialog.value = null
}

function onDialogHide(event) {
  const wasConfirm = dialogMode.value === 'confirm'
  resolveActiveDialog(wasConfirm ? event?.trigger === 'ok' : true)
}

async function onDialogHidden() {
  await nextTick()
  processDialogQueue()
}

onMounted(() => {
  if (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches) {
    isDark.value = true
  }

  nativeAlert = window.alert.bind(window)
  nativeConfirm = window.confirm.bind(window)
  window.alert = (message) => queueDialog('alert', message)
  window.confirm = (message) => queueDialog('confirm', message)
})

onBeforeUnmount(() => {
  if (nativeAlert) {
    window.alert = nativeAlert
  }
  if (nativeConfirm) {
    window.confirm = nativeConfirm
  }
})
// end dark mode
</script>

<template>
  <h1 class="header">shopper</h1>
  <BApp>
    <main>
      <ModuleTodo />
      <ModuleFooter />
    </main>
    <BModal
      v-model="dialogVisible"
      :title="dialogMode === 'confirm' ? 'Confirm' : 'Alert'"
      :ok-only="dialogMode === 'alert'"
      ok-title="OK"
      cancel-title="Cancel"
      @hide="onDialogHide"
      @hidden="onDialogHidden"
    >
      <p class="mb-0">{{ dialogMessage }}</p>
    </BModal>
    <router-view />
  </BApp>
</template>

<style></style>
