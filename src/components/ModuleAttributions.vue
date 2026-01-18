<script setup lang="js">
import { BTable } from 'bootstrap-vue-next'

const attributionsItems = [
  {
    img: 'noimg.jpg',
    caption: '[sample attribution]',
    url: '[sample link]',
  },
]

const fields = [
  {
    key: 'img',
    label: 'Work',
    sortable: false,
  },
  {
    key: 'caption',
    label: 'Attribution',
    sortable: false,
  },
  {
    key: 'source',
    label: 'Source',
    sortable: false,
  },
]
</script>

<template>
  <h1>Module:Attributions</h1>
  <BTable striped bordered :items="attributionsItems" :fields="fields">
    <template #cell(img)="{ item }">
      <img v-if="item.img" :src="item.img" style="max-width: 250px; max-height: 250px" />
    </template>
    <template #cell(caption)="{ item }">
      <span>{{ item.caption }}</span>
    </template>
    <template #cell(source)="{ item }">
      <span>{{ item.url }}</span>
    </template>
  </BTable>
</template>
