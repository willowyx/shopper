<script setup lang="js">
// import ModuleHome from './components/ModuleHome.vue'
import ModuleFooter from './components/ModuleFooter.vue'
import ModuleTodo from './components/ModuleTodo.vue'
import { BApp } from 'bootstrap-vue-next'
import './assets/main.css'
</script>

<template>
  <h1 class="header">shopper</h1>
  <BApp>
    <main>
      <ModuleTodo />
      <ModuleFooter />
    </main>
    <router-view />
  </BApp>
</template>

<style>
header {
  line-height: 1.5rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
