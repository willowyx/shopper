<script setup>
import { ref } from 'vue'
import { version } from '../../package.json'
import ModuleAttributions from './ModuleAttributions.vue'
import { BModal } from 'bootstrap-vue-next'

const showAttributions = ref(false)
</script>

<template>
  <hr />
  <div class="footer-wrap">
    <span class="left-wrap"> willowyx/shopper:v{{ version }} (pre) </span>
    <span class="right-wrap">
      <a class="link-btn" target="_blank" href="https://willowyx.dev/projects/shopper">website</a>
      <a class="link-btn" target="_blank" href="https://github.com/willowyx/shopper">github</a>
      <a class="link-btn" target="_blank" href="https://github.com/willowyx/shopper/commits/main/"
        >update history</a
      >
      <a class="link-btn" href="#" @click.prevent="showAttributions = true">attributions</a>
      <BModal v-model="showAttributions" title="Attributions" size="lg" hide-footer ok-only>
        <ModuleAttributions />
      </BModal>
    </span>
  </div>
</template>
