<script setup>
import { ref, inject } from 'vue'
import { version } from '../../package.json'
import ModuleAttributions from './ModuleAttributions.vue'
import { BModal } from 'bootstrap-vue-next'

const showAttributions = ref(false)

const isDark = inject('isDark')
const toggleDark = inject('toggleDark')
</script>

<template>
  <hr />
  <div class="footer-wrap">
    <span class="left-wrap">
      <a class="link-btn footer-btn" target="_blank" href="https://willowyx.dev/projects/shopper">about shopper</a>
      <a class="link-btn footer-btn" target="_blank" href="https://github.com/willowyx/shopper">github</a>
      <a class="link-btn footer-btn" target="_blank" href="https://github.com/willowyx/shopper/commits/main/">update
        history</a>
      <a class="link-btn footer-btn" href="#" @click.prevent="showAttributions = true">attributions</a>
      <a class="link-btn footer-btn" href="#" @click.prevent="toggleDark">{{ isDark ? 'light' : 'dark' }} mode</a>
      <BModal v-model="showAttributions" title="Attributions" size="lg" hide-footer ok-only>
        <ModuleAttributions />
      </BModal>
    </span>
    <span class="right-wrap">
      <img class="favicon" alt="shopper" src="../../public/favicon.ico">
      willowyx/shopper:v{{ version }} (pre)
    </span>
  </div>
</template>
