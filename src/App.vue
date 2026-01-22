<script setup lang="js">
import { ref, provide, watch, onMounted } from 'vue'
import ModuleFooter from './components/ModuleFooter.vue'
import ModuleTodo from './components/ModuleTodo.vue'
import { BApp } from 'bootstrap-vue-next'
import './assets/main.css'

// begin dark mode
const isDark = ref(false)

const toggleDark = () => {
  isDark.value = !isDark.value
}

provide('isDark', isDark)
provide('toggleDark', toggleDark)

watch(isDark, (val) => {
  document.documentElement.setAttribute('data-bs-theme', val ? 'dark' : 'light')
})

onMounted(() => {
  if (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches) {
    isDark.value = true
  }
})
// end dark mode
</script>

<template>
  <h1 class="header">shopper</h1>
  <BApp>
    <main>
      <ModuleHome />

      <ModuleTodo />
      <ModuleFooter />
    </main>
    <router-view />
  </BApp>
</template>

<style></style>
